===========================
Kubernetes Platform Support
===========================

Storyboard: https://storyboard.openstack.org/#!/story/2002843

This story will implement the initial items to integrate kubernetes on
StarlingX. The StarlingX kubernetes cluster will initially support:

* Docker runtime
* Calico CNI plugin
* Ceph as persistent storage backend
* Helm as the package manager
* Local Docker Image Registry
* 2 node HA solution
* Node labeling integration with inventory subsystem

Problem description
===================

In order to support containerized applications on the StarlingX platform,
we need to package and deploy kubernetes and associated components, such
as docker, calico and helm.

This is a first step in a larger effort. Please see the following for
details of the end goal:
https://wiki.openstack.org/wiki/Containerizing_StarlingX_Infrastructure

Use Cases
=========

Developers need the ability to install and configure a StarlingX system
with kubernetes master and worker nodes, in order to allow further
development efforts to support containerized services.

Developers/testers/users need the ability to install and configure a
StarlingX system without any kubernetes functionality, in order to avoid
any impacts to non-kubernetes related development and testing activities.

Proposed change
===============

The packages required to support a basic kubernetes deployment will be added
to the load. This includes packages related to:

* kubernetes
* etcd
* docker and docker registry
* helm client
 
A --kubernetes option will be added to config_controller. When this option is
not used, the system will be configured without any kubernetes functionality.

When the --kubernetes option is used, kubernetes will be configured. This
includes the following:

* The etcd will be started and will be managed by stx-ha Service Management 
  (SM) in an active/standby configuration.
* The bare metal OpenStack services will not be started and will not appear
  in SM, with the exception of keystone, which will continue to run on
  bare metal to provide authentication for platform services (e.g. System
  Inventory (sysinv), patching, stx-nfv Virtual Infrastructure Manager (VIM)).
* The kubeadm tool will be used to automatically configure both controller
  hosts as kubernetes master nodes. Kubernetes will initially use the existing
  management network and floating management IP for the kube-apiserver.
* Kubeadm will download any required images from the usual external
  image registries. An iptables NAT rule will be added on controller hosts to
  allow compute hosts to access the external image registries through the
  controller's OAM interface.
* The calico CNI plugin will be installed to provide pod networking.
* The k8s-keystone-auth will be configured as the authorization backend for
  the kube-apiserver.
* Helm will be configured to manage kubernetes applications.
* When a compute host is installed, the kubeadm tool will be used to
  automatically configure it as a kubernetes worker node.
* The VIM will provide activity management for kubernetes hosts. When hosts
  are removed from service (by locking them or because they have failed),
  the VIM will prevent dynamic pods from running on these hosts by applying
  a NoExecute taint to the assocatied kubernetes node.
* New inventory CLIs (system host-label-assign/list/remove) will be provided
  to allow labels to be added/removed to kubernetes nodes.
* When a host acting as a master/worker node is deleted, use the kubernetes
  API to delete all resources associated with the node.

Alternatives
============

There are many alternatives to kubeadm that can be used to install a
kubernetes cluster (e.g. bootkube, kubespray, puppetlabs-kubernetes,
promenade). We chose kubeadm because:

* It has the most community support and is the most up-to-date with
  kubernetes features.
* It should be easy to integrate with our existing installation/commissioning
  architecture. Will get us up and running and allow us to understand any
  limitations better.
* Promenade/bootkube/kubespray are too complex and do not fit well with our
  existing architecture.

Data model impact
=================

To support applying kubernetes labels to hosts, a new label table will be
added to the sysinv DB, with the following columns:

* created_at, updated_at, deleted_at: standard columns
* id: primary key
* uuid: unique identifier
* host_id: reference to i_host table
* label: actual label

As usual, a new data migration will be added to create this table in the
sysinv DB.

REST API impact
===============

This impacts the sysinv REST API:

* The new resource /labels is added and the POST method will accept a
  key/value pair as a label to be assigned to a host.

  * URLS:

    * /v1/labels/{host_uuid}

  * Request Methods:

    * POST

  * JSON response example::

     {"uuid": "cf415922-4242-4c58-94c7-785d69dc8971",
      "host_uuid": "01f1d819-830d-4a84-a533-88fc5e7deba8",
      "label": "prefix/name=value"}

* The GET method would return all the labels assigned to a specified host.

  * URLS:

    * /v1/ihosts/{host_uuid}/labels

  * Request Methods:

    * GET

  * JSON response example::
      
     {"labels": [{"uuid": "692773cf-a2be-4b32-bc09-89a22521503d",
                  "host_uuid": "01f1d819-830d-4a84-a533-88fc5e7deba8",
                  "label": "prefix/name=value"}]}

  * URLS:

    * /v1/labels/{uuid}

  * Request Methods:

    * DELETE

Security impact
===============

This change does not expose any new interfaces to the external world. The
kubernetes API will only be visible on the internal management network,
which is considered to be a secure network with no external access.

The kubeadm tool sets up the following security measures for the kubernetes
components:

* Certificates and keys are generated for the kube-apiserver. These
  certificates are also used to provide secure access between other
  kubernetes components (e.g. kubelet) and the kube-apiserver.
* Keystone tokens are used to authenticate users of the kube-apiserver.
* Bootstrap tokens are generated for each new worker node and are used to
  ensure that only authorized worker nodes are allowed to join the cluster.

Other end user impact
=====================

None - this does not affect existing deployments

Performance Impact
==================

None - this does not affect existing deployments

Other deployer impact
=====================

A new --kubernetes option will be added to config_controller, which will
only be used for internal development purposes.

Developer impact
================

This change allows developers to install and configure a StarlingX system
with kubernetes master and worker nodes, in order to allow further
development efforts to support containerized services.

Upgrade impact
==============

None - this is the first StarlingX release so there are no previous releases
that we need to support upgrades from.

Implementation
==============

Assignee(s)
===========

Primary assignee:

* Bart Wensley (bartwensley)
  
Other contributors:

* Smith (kevin.smith.wrs)
* Chris Friesen (cbf123)
* Shoaib Nasir (snasir)
* Teresa Ho (teresaho)

Repos Impacted
==============

* stx-integ
* stx-config
* stx-nfv

Work Items
==========

* Add packages to the load that are required to support a basic kubernetes
  deployment, including kubernetes, etcd, docker, docker registry and helm
  client.
* Add --kubernetes option to config_controller command.
* Integrate etcd as a new active/standby SM service, configured with puppet.
* Update SM puppet configuration to prevent configuration of the bare metal
  OpenStack services, with the exception of keystone and horizon.
* Create new puppet manifest to use kubeadm tool to automatically configure
  both controller hosts as kubernetes master nodes and install the calico
  CNI plugin. The manifest will also use the kubeadm tool to automatically
  configure compute hosts as kubernetes worker nodes.
* Allow the default NoSchedule taint to remain on controllers and use
  tolerations on all pods that must run on controllers.
* Automatically add firewall rule to allow compute hosts to have external
  network access through controller hosts.
* Integrate k8s-keystone-auth as authorization backend for the kube-apiserver.
* Create new puppet manifest to configure Helm.
* Update the VIM to provide activity management for kubernetes hosts.
* Add new inventory CLIs (system host-label-assign/list/remove) to allow
  labels to be added/removed to kubernetes nodes.
* Support host-delete for master/worker nodes by using the kubernetes API to
  delete all resources associated with the node.

Dependencies
============

This requires existing functionality from some projects that are not
currently used by StarlingX:

* docker
* etcd
* kubernetes
* helm

Testing
=======

One main thrust of the testing is to ensure the kubernetes functionality is
dormant when the --kubernetes option is not used. Existing unit tests will
cover some of this, but it is important to test full system installs (this
can be done in a virtual environment). Some of the key testcases are:

* Full install of 2 controller, 2 compute system
* Full install of 2 controller, 2 storage and 2 compute system
* Host maintenance operations (e.g. lock, unlock, swact, power cycle)
* Basic nova instance operations (e.g. stop, start, migrate, delete)

Testing for kubernetes enabled systems will also be done (can be done in a
virtual environment). Some of the key testcases are:

* Installation of 2 master, 2 worker system
* Host maintenance operations (e.g. lock, unlock, swact, power cycle)
* Deploying pods on all hosts and verifying pod networking

Documentation Impact
====================

None - this does not impact existing deployments

References
==========

None

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced
