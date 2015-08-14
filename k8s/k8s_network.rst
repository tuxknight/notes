.. _k8s_network:

K8S - Network
==============

#. Highly-coupled container-to-container communications.

#. Pop-to-Pod communications.

#. Pod-to-Service communications.

#. External-to-Service communications.

Kubernetes model
------------------

Kubernetes imposes some fundamental requirements on any networking implementation::

  * All containers can communicate with all other containers without NAT.

  * All nodes can communicate with all containers without NAT.

  * The IP that a container sees itself as is the same IP that others see it as.

What this means in practice is that you can not just take two computers running Docker and expect Kubernetes to work.

You must ensure that the fundamental requirements are met.

Kubernetes applies IP addresses at the ``pod`` scope, containers within a ``pod`` share their network namespace, including their IP address. This means that containers within a ``pod`` can all reach each other's ports on ``localhost`` . This does imply that containers within a ``pod`` must coordinate port usage, but this is no different that processes in a VM. This is what we call **IP-per-pod** model.


.. note::
  
    This is implemented in Docker as a "pod container" which holds the network namespace open while "app container" join that namespace with Docker's ``--net=container:<id>`` function.

As with Docker, it is possible to request host ports, but this is reduced to a very niche operation. In this case a port will be allocated on the host *Node* and traffic will be forwarded to the *Pod* . The *Pod* itself is blind to the existence or non-existence of host ports.

How to achieve this
----------------------

#. Google Compute Engine (GCE)

#. L2 networks and linux bridging

   `Four ways to connect a docker container to a local network <http://blog.oddbit.com/2014/08/11/four-ways-to-connect-a-docker/>`_


#. Flannel

   `Flannel <https://github.com/coreos/flannel#flannel>`_ is a very simple overlay network that satisfies the Kubernetes requirements. 


#. OpenVSwitch

   `OpenVSwitch <http://kubernetes.io/v1.0/docs/admin/ovs-networking.html>`_ is a somewhat more mature but also complicated way to build an overlay network. :ref:`OpenVSwitch networking`


#. Weave

   `Weave <https://github.com/zettio/weave>`_ is yet another way to build an overlay network, primarily aiming at Docker integration.


#. Calico

   `Calico <https://github.com/Metaswitch/calico>`_ uses BGP to enable real container IPs.


.. _OpenVSwitch networking:

Kubernetes OpenVSwitch GRE/VxLAN networking
----------------------------------------------

This document describes how OpenVSwitch is used to setup networking between pods across nodes. The tunnel type could be GRE or VxLAN. VxLAN is preferable when large scale isolation needs to be performed within the network.

.. figure:: ovs-networking.png

The vagrant setup in Kubernetes does the following:

The docker bridge is replaced with a brctl generated linux bridge(kbr0) with a 256 address space subnet. Basically, a node gets 10.244.x.0/24 subnet and docker is configured to use that bridge instead of the default docker0 bridge.

Also, an OVS bridge is created(obr0) and added as a port to the kbr0 bridge. All OVS bridges across all nodes are linked with GRE tunnels. So, each node has an outgoing GRE tunnel to all other nodes. It does not need to be a complete mesh really, just meshier the better. STP(spanning tree) mode is enabled in the bridges to prevent loops.

Routing rules enable any 10.244.0.0/16 target to become reachable via the OVS bridge connected with the tunnels.

