.. _consul_agent:

Consul Agent
==============

The Consul agent is the core process of Consul. The agent maintains *membership information* , *registers services* , *runs checks* , *responds to queries* , and *more* . The agent must run on every node that is part of a Consul cluster.

Any agent may run in one of two modes: client or server.

+ A server node takes on the additional responsibility of being part of the consensus quorum. These nodes take part in Raft and provide strong consistency and availability in the case of failure. 

+ Client nodes make up the majority of the cluster, and they are very lightweight as they interface with the server nodes for most operations and maintain very little state of their own.

Running An Agent
=================

While running ``consul agent`` , you should see output similar to this

::

  Node name: 'sclg124'
  Datacenter: 'local'
  Server: true (bootstrap: false)
  Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
  Cluster Addr: 10.13.182.124 (LAN: 8301, WAN: 8302)
  Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
  Atlas: <disabled>

* Node name:

  This is a unique name for the agent. By default, this is the hostname of the machine. You also could customize it using the ``-node`` flag.

* Datacenter:

  This is the datacenter in which the agent is configured to run. Consul has first-class support for multiple datacenters. The ``-dc`` flag can be used to set the datacenter. For single-DC configurations, the agent will default to "dc1".

* Server:

  This indicate whether the agent is running in server or client mode. Additionally, a server may be in "bootstrap" mode. Multiple servers cannot be in bootstrap mode as that would put the cluster in an inconsistent state.

* Client Addr:

  This is the address used for client interfaces to the agent. This includes the ports for the HTTP, DNS, and RPC interfaces.

* Cluster Addr:

  This is the address and set of ports used for communication between Consul agents in a cluster.

Stopping An Agent
==================

An agent can be stopped in two ways: gracefully or forcefully.

* To gracefully halt an agent, send the process an interrupt signal. When gracefully exiting, the agent first notifies the cluster it intends to leave the cluster. This way, other cluster members notify the cluster that the node has left.

* Alternatively, you can force kill the agent by sending it a kill signal and the agent ends immediately. The rest of the cluster will detect that the node has died and notify the cluster that the node has failed.

It is especially important that a server node be allowed to leave gracefully so that there will be a minimal impact on availability as the server leaves the consensus quorum.

Lifecycle
==========

When an agent is first started, it does not know about any other node in the cluster. To discover its peers, it must **join** the cluster. This is done with the ``join`` command or by providing the proper configuration to auto-join on start. 

Once a node joins, this information is gossiped to the entire cluster, meaning all nodes will eventually be aware of each other. If the agent is a server, existing servers will begin replicating to the new node.

In the case of a network failure, some nodes may be unreachable by other nodes. In this case, unreachable nodes are marked as failed. It is impossible to distinguish between a network failure and an agent crash, so both cases are handled the same. Once a node is marked as failed, this information is updated in the service catalog. 

.. note::

   There is some nuance here since this update is only possible if the servers can still form a quorum. Once the network recovers or a crashed agent restarts the cluster will repair itself and unmark a node as failed. The health check in the catalog will also be updated to reflect this.

When a node *leaves* , it specifies its intent to do so, and the cluster marks that node as having *left* . Unlike the failed case, all of the services provided by a node are immediately deregistered. If the agent was a server, replication to it will stop. To prevent an accumulation of dead nodes, Consul will automatically reap failed nodes out of the catalog as well. This is currently done on a non-configurable interval of 72 hours. Reaping is similar to leaving, causing all associated services to be deregistered.
