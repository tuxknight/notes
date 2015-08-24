.. _add_del_server:

For changes to be processed, a minimum quorum of servers (N/2)+1 must be available. That means if there are 3 server nodes, at least 2 must be available.

In general, if you are ever adding and removing nodes simultaneously, it is better to first add the new nodes and then remove the old nodes.

Add Servers
============

Adding new servers is generally straightforward. Simply start the new agent with the -server flag. At this point, the server will not be a member of any cluster.

we can now add this node to the existing cluster using ``join`` . From the new server, we can join any member of the existing cluster::

  consul join <node address>

It is important to note that any node, including a non-server may be specified for join. The gossip protocol is used to properly discover all the nodes in the cluster.

Check out the ``last_log_index`` on leader server by running ``consul info`` which is the latest log on leader. Then run the same ``consul info`` on the new node to how far behind it is.

.. note::

  It is best to add servers one at a time, allowing them to catch up. This avoids the possibility of data loss in case the existing servers fail while bringing the new servers up-to-date.

Remove Servers
===============

For a cluster of N servers, at least (N/2)+1 must be available for the cluster to function. 

To avoid this, it may be necessary to first add new servers to the cluster, increasing the failure tolerance of the cluster, and then to remove old servers.

Once you have verified the existing servers are healthy, and that the cluster can handle a node leaving, the actual process is simple. You simply issue a ``leave`` command to the server.

Forced Removal
================

If the server can be recovered, it is best to bring it back online and then gracefully leave the cluster. However, if this is not a possibility, then the ``force-leave`` command can be used to force removal of a server.
