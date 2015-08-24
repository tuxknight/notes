.. _consul_bootstrap:

Bootstrap
===========

An agent can run in both client and server mode. Server nodes are responsible for running the consensus protocol and storing the cluster state. The client nodes are mostly stateless and rely heavily on the server nodes.

The recommended way to bootstrap is to use the ``-bootstrap-expect`` configuration option. This option informs Consul of the expected number of server nodes and automatically bootstraps when that many servers are available. 

All servers should either specify the same value for -bootstrap-expect or specify no value at all. Only servers that specify a value will attempt to bootstrap the cluster.

You can bootstrap each node using command ``consul agent -server -data-dir /tmp/default.consul -bootstrap-expect 3 -dc dev`` . These nodes will expect 3 peers but none of them are known to each other now. So, the servers will not elect themselves leader.

Creating a Cluster
====================

To trigger leader election, we must join these machines together and create a cluster. There are two options for joining; you can use `Atlas by HashiCorp <https://atlas.hashicorp.com/>`_ to auto-join or you can access the machine and manually run the join.

To manually create a cluster, access one of the machines and run the following::

  consul join <node address>

Since a join operation is symmetric, it does not matter which node initiates it.As a sanity check, the ``consul info`` command is a useful tool. It can be used to verify ``raft.num_peers`` , and you can view the latest log index under ``raft.last_log_index`` . When running ``consul info`` on the followers, you should see ``raft.last_log_index`` converge to the same value once the leader begins replication. That value represents the last log entry that has been stored on disk.

Now that the servers are all started and replicating to each other, all the remaining clients can be joined. Clients are much easier as they can join against any existing node. All nodes participate in a gossip protocol to perform basic discovery, so once joined to any member of the cluster, new clients will automatically find the servers and register themselves.
