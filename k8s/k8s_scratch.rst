Kubernetes from scratch
==========================

.. contents:: Topics

Binaries
---------

`Latest binary_`

.. _Latest binary: https://github.com/GoogleCloudPlatform/kubernetes/releases/latest `

Find *./kubernetes/server/kubernetes-server-linux-amd64.tar.gz* and unzip. In *./kubernetes/server/bin* contains all the necessary binaries.

Selecting Images
--------------------

Docker, kubelet, kube-proxy will run outside of a container.

Etcd, kube-apiserver, kube-controller-manager, kube-scheduler will run as containers. An images might be needed.

* Use images hosted on Google Container Registry
  
  * eg **gcr.io/google_containers/kube-apiserver:$TAG** , where $TAG is the latest release.

* Build your own image

  * you can find tarballs in *./kubernetes/server/bin* , so import these tarballs to docker::
    
          cat kube-apiserver.tar |docker import - kube-apiserver
          cat kube-controller-manager.tar |docker import - kube-controller-manager
          cat kube-scheduler.tar|docker import - kube-scheduler

For ectd

* Use images hosted on GCR, such as **gcr.io/google_containers/etcd:2.0.12**
  
* Use images hosted on hub.docker.com or quay.io

* Use etcd binary included in your OS distro.

* Build your own image ``cd kubernetes/cluster/images/etcd; make``

Configuring and Installing Base Software on Nodes
-------------------------------------------------------

docker
`````````

.. note:: 

  The newest stable release is a good choice. If you previously had docker installed without setting Kubernets-specific options, you may have a Docker-created bridge and iptables rules. You may want to remove these as follows before proceeding to configure Docker for Kubernetes::

    iptables -t nat -F
    ifconfig docker0 down
    brctl delbr docker0

The way you configure docker will depend in whether you have chosen the *routable-vip* or *overlay-network* approaches for your network. Some suggested docker options:

  * create your own bridge for the per-node CIDR ranges, call it ``cbr0`` , and set ``--bridge=cbr0`` option on docker.

  * set ``--iptables=false`` so docker will not manipulate iptables for host-ports so that kube-proxy can manage iptables instead of docker.

  * ``--ip-masq=false``

    * if you have setup PodIPs to be routable, then you want this false, otherwise, docker will rewrite the PodIP source-address to a NodeIP.

    * some environments still need you to masquerade out-bound traffic when it leaves the cloud environment. This is very environment specific.

  * ``--mtu=``

    * may required when using Flannel, because of the extra packet size due to udp encapsulation

  * ``--insecure-registry $CLUSTER_SUBNET``

    * to connect to a private registry, if you set one up, without using SSL.

  * You may want to increase the number of open files for docker

    * ``DOCKER_NOFILE=1000000`` where this config goes depends on your node OS. For example, Debian-based distro uses **/etc/default/docker**


Kubelet
````````

All nodes should run *kubelet*

Arguments to consider:

* if following the HTTPS security approach:

  * ``--api-server=htts://$MASTER_IP``

  * ``--kubeconfig=/var/lib/kubelet/kubeconfig``

* Otherwise, if taking the firewall-based security approach

  * ``--api-server=http://$MASTERIP``

* ``--config=/etc/kubernetes/manifests``

* ``--cluster-dns=`` to the address of the DNS server you will setup

* ``--cluster-domain=`` to the dns domain prefix to use for cluster DNS addresses

* ``--docker-root=``

* ``--root-dir=``

* ``--configure-cbr0=``

* ``--register-node``

kube-proxy
````````````

All nodes should run *kube-proxy* .

Arguments to consider:

* If following the HTTPS security approach:

  * ``--api-servers=https://$MASTER_IP``

  * ``--kubeconfig=/var/lib/kube-proxy/kubeconfig``

* Otherwise, if taking the firewall-based security approach

  * ``--api-servers=http://$MASTER_IP``

Networking
`````````````

Each node needs to be allocated its own CIDR range for pod networking. Call this *NODE_X_POD_CIDR*

A bridge called *cbr0* needs to be created on each node. The bridge itself needs an address from **$NODE_X_POD_CIDR** - by convention the firest IP. Call this *NODE_X_BRIDGE_ADDR*

* Recommended, automatic approach

  #. Set ``--configure-cbr0=true`` option in kubelet init script and restart kubelet service. Kubelet will configure cbr0 automatically. It will wait to do this until the node controller has set Node.Spec.PodCIDR. Since you have not setup apiserver and node controller yet, the bridge will not be setup immediately.

* Alternate, manual approach:

  #. Set ``--configure-cbr0=false`` on kubelet and restart.

  #. Create a bridge ``brctl addbr cbr0``

  #. Set appropriate MTU  ``ip link set dev cbr0 mtu 1460``

  #. Add the clusters network to the bridge, docker will go on other side of bridge. ``ip addr add $NODE_X_BRIDGE_ADDR dev eth0``

  #. Turn it on ``ip link set dev cbr0 up``


Bootstrapping the Cluster
-----------------------------

etcd
```````

* Recommended approach: run one etcd instance, with its log written to a directory backed by durable storage.

* Alternative: run 3 or 5 etcd instances.

  * log can be written to non-durable storage because storage is replicated.

  * run a single apiserver which connects to one of the etc nodes.

To run the etc instance:

#. copy **cluster/saltbase/salt/etcd/etcd.manifest**

#. make any modifications needed

#. start the pod by putting it into the kubelet manifest directory
