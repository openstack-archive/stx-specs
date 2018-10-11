===========================================================
System Deployment of Containerized OpenStack Infrastructure
===========================================================

Storyboard: https://storyboard.openstack.org/#!/story/2003910

This story will update StarlingX to deploy its OpenStack infrastructure in a
containerized configuration. Changes will be identified and implemented to
enable end-to-end configuration of all the containers that will host
OpenStack infrastructure services.

Problem description
===================

Please see the following for background on the effort to containerize the
StartlingX OpenStack infrastructure:
https://wiki.openstack.org/wiki/Containerizing_StarlingX_Infrastructure

This story implements the end-to-end configuration of the StarlingX
platform with the OpenStack infrastructure services running in containers.
It builds on several other stories which are listed in the Dependencies
section below.

Use Cases
=========

Developers/testers/users need the ability to install and configure a StarlingX
system with kubernetes master and worker nodes, in order to support a
variety of containerized applications.

Developers/testers/users need the ability to install and configure the
OpenStack application on a running StarlingX system.

Proposed change
===============

The initial install of a StarlingX system will no longer include the OpenStack
software (with the exception of Keystone and Horizon which are required by
bare metal platform services). The OpenStack application will have its own
instance of Keystone and Horizon.

To install the OpenStack application, the user will first label the hosts to
be used for OpenStack controller functions (label: openstack-controller-node)
and OpenStack compute functions (label: openstack-compute-node) using the
following CLI:

  system host-label-assign <hostname> <label>

Then the user will use the following CLI to import the OpenStack application:

  system application-upload <app-name> <app-tarfile>

Finally, the user will install the OpenStack application:

  system application-install <app-name>

The above CLIs are created as part of the stories listed in the Dependencies
section below (see the Armada Integration story for details). This story
provides the "glue" that performs the necessary configuration of platform
services (e.g. System Inventory (sysinv), Virtual Infrastructure Manager
(VIM)) to support the OpenStack application.

Alternatives
============

This is the only reasonable approach to integrate the containerized OpenStack
application with the StartlingX platform.

Data model impact
=================

None - any data model impacts are described in the stories listed in the
Dependencies section below.

REST API impact
===============

None - any REST API impacts are described in the stories listed in the
Dependencies section below.

Security impact
===============

This story does not expose any new interfaces to the external world. Any
security impacts are described in stories listed in the Dependencies section
below.

Other end user impact
=====================

Prior to this story, all StarlingX systems would come up with the OpenStack
services installed by default (on bare metal) after initial system
configuration.

After this story, all StarlingX systems will come up without the OpenStack
services. If a user wants to install the OpenStack application
(containerized), they will perform the actions described in the Proposed
Change section above.

Performance Impact
==================

The move from a bare metal OpenStack deployment to a containerized deployment
will impact the performance of the system. These impacts will be assessed
as the development is done and negative impacts will be mitigated.

Other deployer impact
=====================

In order to install the StarlingX system, external network access will be
required over the OAM interface on the controller hosts. This is necessary
to download the required docker images, which will not be included in the
StarlingX iso. This is both due to size considerations (the images required by
he OpenStack application will total several GB in size) and to decouple the
StarlingX software release from OpenStack docker image releases.

Patching of a StarlingX system will be different as the existing RPM based
patching mechanism will not work for software running in containers (i.e. all
the OpenStack components). The new patching mechanism for containers is TBD.

Developer impact
================

Developers working on StarlingX will now need to build docker images in order
to test changes to OpenStack components (e.g. keystone, nova, neutron) that
were previously running on bare metal. They will also need to generate the
OpenStack application tarfile, which includes manifests and Helm charts.

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

* Al Bailey (albailey)
* Chris Friesen (cbf123)
* Don Penney (dpenney)
* Jerry Sun (jerry-sun-u)
* Kevin Smith (kevin.smith.wrs)
* Lachlan Plant (lachlan.plant)
* Robert Church (rchurch)
* Shoaib Nasir (snasir)
* Tee Ngo (teewrs)

Repos Impacted
==============

* stx-config
* stx-integ
* stx-nfv

Work Items
==========

* Sysinv:

  * Update VIM puppet plugin to disable all OpenStack plugins in the VIM when
    the OpenStack application is NOT installed and to enable them when the
    OpenStack application is installed.
  * Support new keystone configuration (e.g. auth URL, username, password) for
    containerized OpenStack services.
  * Use the pod based keystone when accessing OpenStack services (if it is
    configured).
  * Update sysinv/puppet to supply kubernetes configuration to sysinv/VIM when
    OpenStack application has been installed.
  * Update sysinv/puppet to generate platform openrc file in
    /etc/platform/openrc.
  * Detect when OpenStack application installation is complete and:
    * Reconfigure sysinv with pod based keystone configuration.
    * Reconfigure VIM with pod based keystone configuration.
  * Perform different semantic checks for worker nodes with OpenStack
    installed vs. worker nodes without OpenStack.
  * Add semantic checks to prevent label and override modifications for
    unlocked hosts.
  * Send notifications to VIM whenever labels are modified.
  * Avoid modifications to host aggregates for pure kubernetes worker nodes.

* VIM:

  * Support new keystone configuration (e.g. auth URL, username, password) for
    containerized OpenStack services.
  * Use the pod based keystone when accessing OpenStack services (if it is
    configured).
  * Disable all interactions with nova, neutron and guest services for pure
    kubernetes worker nodes (based on absence of openstack-compute-node label).
  * host-add:

    * When sysinv notifies the VIM that a host has been added, the VIM will
      no longer create the nova/neutron/guest services, as the host will
      start out as a pure kubernetes worker node.

  * label-add:

    * When openstack-compute-node label is added to compute, create the
      nova/neutron/guest services (previously done on host-add).

  * host-delete:

    * Only delete OpenStack services if openstack-compute-node label applied
      to host.

* Remove --kubernetes option from config_controller, along with the
  now-obsolete bare metal OpenStack related code (e.g. python code, puppet,
  service scripts, OCF scripts, SM config, pmon config).

Dependencies
============

This requires new functionality being developed under the following stories:

* Kubernetes Platform Support:
  https://storyboard.openstack.org/#!/story/2002843
* CEPH persistent storage backend for Kubernetes:
  https://storyboard.openstack.org/#!/story/2002844
* Local Docker Registry:
  https://storyboard.openstack.org/#!/story/2002840
* Docker Image Generation:
  https://storyboard.openstack.org/#!/story/2003907
* Infrastructure HELM Chart Override Generation:
  https://storyboard.openstack.org/#!/story/2003909
* Create HELM chart for nova-api-proxy:
  https://storyboard.openstack.org/#!/story/2004007
* Create HELM chart for Fault project:
  https://storyboard.openstack.org/#!/story/2004008
* Armada Integration:
  https://storyboard.openstack.org/#!/story/2003908

Testing
=======

This story affects the configuration and deployment of all OpenStack services
on StarlingX. In addition to the usual unit testing in the impacted code
areas, this will require a full system regression of all StarlingX
functionality. It will also require extensive performance testing in order
to identify and address any performance impacts.

In addition, this story changes the way a StarlingX system is installed and
configured, which will require changes in existing automated installation and
testing tools.

Documentation Impact
====================

This story affects the StarlingX installation and configuration documentation.
Specific details of the documentation changes will be addressed once the
implementation is complete.

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
