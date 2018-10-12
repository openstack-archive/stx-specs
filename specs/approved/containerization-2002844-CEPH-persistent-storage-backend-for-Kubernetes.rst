==============================================
CEPH persistent storage backend for Kubernetes
==============================================

Storyboard: https://storyboard.openstack.org/#!/story/2002844

Kubernetes applications (e,g, openstack) will require access to persistent
storage. StarlingX has standardized on CEPH as that backend which requires
that CEPH be supported on 1 node and 2 nodes configurations.

This story implements:

* Helm chart to configure kubernetes RBD provisioner and related sysinv
  configuration for it
* CEPH support for 1 node configuration (One node system)
* CEPH support for 2 node configuration (Two node system)

Problem description
===================

In order to enable Openstack's helm charts on StarlingX we need a distributed
persistent storage for Kubernetes that leverages our existing storage
configurations. For this stage we will enable CEPH's RBD to work with
Kubernetes RBD provisioner through a new Helm chart.

Since RBD will be the persistence storage solution, CEPH support will need to
be extended to the 1 and 2 node configurations.

Use Cases
=========

Kubernetes PODs needs access to persistent, distributed, storage.

Proposed changes
================

This story will have 3 main activities:

1. Helm chart to configure kubernetes RBD provisioner
-----------------------------------------------------
What will be implemented:

* A new helm chart that will leverage the rbd-provisioner incubator project
  (we will be using the upstream image).
* A new service, 'rbd-provisioner', will be added to CEPH storage backend.
  This will set necessary CEPH configuration for the helm chart (e.g. by
  issuing system storage backend-add ceph -s cinder,glance,rbd-provisioner).
* High availability implemented through kubernetes using replication=1 and
  autorestart for the POD.
* Helm chart will install only if proper CEPH configuration is in place
  (ceph cluster is up, credentials are validated etc.).
* CEPH tiers: a single rbd-provisioner chart can be instantiate at a time.
  This instance will support multiple storage classes. One storage class will
  be added for each tier with rbd-provisioner service configured.
* Removing a provisioner from one tier will be supported – old pool will not
  be deleted. Credentials (secrets) of the old provisioner will be removed
  from kubernetes. CEPH credentials will remain in place.
* CEPH pool will be managed by sysinv. Pool name format will be 'kube-' plus
  the tier name (except for primary tier which will be named kube-rbd). Quota
  modifications will be supported (both cli & dashboard). Alarms on pool
  threshold usage. CEPH pool PG Num will autoadjust.
* As this is intended to be generally available (vs. openstack specific), user
  will be able to configure the namespace and the name of the storage class.
  This naming should comply to the kubernetes naming scheme (i.e. kubernetes
  has a regex for validating these names).

What will not be implemented:

* Containerizing CEPH itself
* Support for multiple storage provisioners
* Autoscaling
* Custom health monitoring, we will conform to what upstream provides

2. CEPH support for 1 node configuration (One node system)
----------------------------------------------------------
* Allow CEPH to be configured on a 1 node configuration with a single monitor.
* Allow CEPH OSDs to be assigned to controller disks (only dedicated disks
  allowed)
* No enforcement on the number of OSD's.
* Enable OSD level redundancy versus node redundancy that is present on the
  standard system cluster,
* Adding/Modifying CEPH backend configuration require node to be locked.
* After adding the backend & before unlock: Monitor will be immediately
  available; Backend will be in “provisioned” state; Secondary tiers can
  be added at this stage.
* Configuration will be applied on unlock. CEPH OSDs will start after reboot.
* Adding OSDs require node to be locked, they will be operational after
  unlock.
* Unlocked is allowed only if number of configured OSDs per tier matches
  replication number configured for that tier.
* Enable Horizon CEPH Dashboards for AIO SX controller.
* Services supported in this configuration: cinder, glance, rbd-provisioner,
  swift, nova (with remote ephemeral storage).
* Support for replication 1, 2 and 3 per tier, default to 1. Replication will
  be changeable on the fly for this configuration.
* Update StarlingX HA for storage process groups - we no longer have
  2 controllers.

3. CEPH support for 2 node configuration (Two node system)
----------------------------------------------------------
* Allow CEPH to be configured on a 1 node configuration with a single monitor.
* For two nodes, we'll have one floating monitor, managed by SM.
* The CEPH monitor filesystem will be DRBD replicated.
* CEPH crushmap will be similar to the one for multinode deployments.
  Difference is that both controllers will be in the same group.
* Redundancy will be nodal.
* Only replication 2 is supported.


Alternatives
============

* As an alternative we can enable CephFS and cephfs-provisioner. There is
  currently no usecase to support a shared PVC. We can do this in another
  story.
* For 1 & 2 node systems we don't have an alternative distributed storage to
  CEPH.

Data model impact
=================

None - sysinv storage backends DB tables supports a dictionary type
'capabilities' field which can hold rbd-provisioner custom, configurable,
options.


REST API impact
===============

No impact to the calls themselves. One new service will appear in the service
list (i.e. rbd-provisioner) with two new fields in the 'capabilities'
dictionary: 'rbd_storageclass_name', 'rbd_provisioner_namespaces'.


Security impact
===============

None

Other end user impact
=====================

None

Performance Impact
==================

1 & 2 node configurations will now run CEPH. This leads mainly to an increase
in RAM usage: 1GB max for ceph-mon and 1GB max or each configured OSD.


Other deployer impact
=====================

* Adding CEPH as a new configuration option to 1 & 2 node kubernetes systems.
* Adding 'rbd-provisioner' optional CEPH storage backend service.
* The controller LVM storage will be replaced by CEPH.

Developer impact
=================

Work on Kubernetes Platform Support
(https://storyboard.openstack.org/#!/story/2002843) should be synchronized
with work on this story.

Upgrade impact
===============

None

Implementation
==============

Assignee(s)
===========

Primary assignee:
  Ovidiu Poncea (ovidiu.poncea)

Other contributors:
  Irina Mihai (irina.mihai)

Repos Impacted
==============

stx-config, stx-gui, stx-ceph, external-storage

NOTE: A small change is required in stx-ceph due to a StarlingX script
that is currently packaged in stx-ceph. Plans are in place to move StarlingX
scripts out of stx-ceph and this change will move into stx-config when those
scripts are moved into stx-config.'


Work Items
===========

Helm chart to configure kubernetes RBD provisioner and related sysinv
configuration:

* sysinv: Add rbd-provisioner as a service to ceph backend
* sysinv: Add rbd-provisioner overrides generation to sysinv
* sysinv: Generate CEPH pool keys and k8s secrets
* chart: Create Base Helm chart for rbd provisioner
* chart: Add chart hooks for checking CEPH prerequisites
* kube CEPH pool: Add cli quota support
* kube CEPH pool: Add dashboard quota support
* kube CEPH pool: Usage threshold Alarms and pg_num autoadjust
* sysinv: Support removing rbd-provisioner from tier
* sysinv: Add support for multiple namespaces

CEPH support for 1 node configuration (One node system):

* Enable CEPH to work with a single monitor on 1 node system
* Enable OSD configuration on controller
* Update CRUSH map
* Make replication  1->3 and back configurable on the fly
* Semantic check updates
* Make sure cinder, glance, rbd-provisioner and swift work in this config
* Update StarlingX SM processes group
* Make sure CEPH processes are not stopped when node is locked
* Enable ceph horizon dashboard for controllers when kubernetes is enabled

CEPH support for 2 node configuration (Two node system):

* Enable a floating CEPH monitor
* Enable OSD configuration on 2nd controller
* Enable the DRBD replication of the CEPH monitor filesystem
* Update CRUSH map
* Semantic check updates
* Make sure cinder, glance, nova, rbd-provisioner and swift work in this
  config

Common to all:

* Test & bug fix
* Code review, rebase, retest


Dependencies
============

Story: [Feature] Kubernetes Platform Support at
https://storyboard.openstack.org/#!/story/2002843

This requires existing functionality from some projects that are not
currently used by StarlingX:

* docker
* kubernetes
* helm
* external-storage


Testing
=======

Rbd-provisioner can be tested separately from work on enabling CEPH.

For rbd-provisioner:

* Sysinv chart options should be correctly generated for single tier,
  multi-tier, removal of provisioner. Check that CEPH keys and kubernetes
  secrets are correct.
* RBD-provisioner is correctly started in any of the sysinv configurations.

For CEPH:

* Test both early and late deployments, with and without services
  (including swift and nova).
* Test adjusting replication number, early and late.
* Extensive semantic checks (see Proposed Changes).

Regression of a new build with all changes, without kubernetes.

* Full install of 1 and 2 node systems.
* Full install of 2 controller, 2 compute system.
* Full install of 2 controller, 2 storage and 2 compute system (early and
  late).
* Host maintenance operations (e.g. lock, unlock, swact, power cycle).
* Start instances


Documentation Impact
====================

None - this does not impact existing deployments


References
==========

Kubernetes Platform Support:
https://storyboard.openstack.org/#!/story/2002843

Kubernetes RBD Provisioner:
https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/rbd


History
=======

First iteration

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced

