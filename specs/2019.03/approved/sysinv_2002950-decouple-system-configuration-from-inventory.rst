..  This work is licensed under a Creative Commons Attribution 3.0 Unported License.  http://creativecommons.org/licenses/by/3.0/legalcode

===================================================
Decouple System Configuration from System Inventory
===================================================

Storyboard: https://storyboard.openstack.org/#!/story/2002950

This story decouples System Configuration from System Host
Management and Inventory.


Problem description
===================

SysInv currently consists of Inventory and Configuration components.

The Inventory component performs Host management and physical inventory
discovery.  The Inventory component interacts with stx-metal and stx-nfv
components for host management.

The Configuration component performs System, Storage configuration and
management, and generation of configuration data, such as for puppet
hierdata or helm charts.

Currently, the inventory resources are coupled to configuration resources
via its relational data model and does not allow external configuration
services.


Use Cases
=========

The decoupled system provides the same functional capabilities as the
coupled SysInv.  The physical and logical resource management are
decoupled for a focused metal management and a more flexible logical
resource management framework.

Allow Inventory Host Management component to interact with the plugin
Configuration to create the required configuration. The plugin
allows the bind in of abstract configuration methods required for the
Inventory host management operations.  This increases flexibility to
provide alternative logical configuration drivers.

Configuration obtains Inventory data via GET request on the
Inventory REST API resources.  This allows Configuration to develop
with a published API.

Upversion to oslo standard libraries to facilitate Developer integration
of components.

Allow move of Inventory component to stx-metal from stx-config for
architectural alignment of bare metal management.
inventory depends on interfaces with stx-metal; systemconfig does
not depend upon stx-metal.
stx-metal is thus focused on physical inventory and bare metal management.


Proposed change
===============

Create separate Inventory database, and Inventory services for REST api,
conductor and agent.  The configuration driver is developed as a plugin
to Inventory for configuration.

Update Inventory python client to support sessions. This facilitates
keystone token management, for consumers such as SystemConfiguration
and Distributed Cloud.

Create separate Configuration database, and Configuration services for api,
conductor, and agent.
The high availability aspect of the api and conductor services are managed
by stx-ha; and agent by stx-metal pmon.

Update datamodels to decouple Inventory from Configuration.
Relationships between Inventory and Configuration database tables are removed
and referenced externally via each resource's globally unique identifier.
Information required between services are obtained via the service's REST API.

Update infrastructure to reference the oslo standard libraries.


Alternatives
============

Alternatively, maintain SysInv as combined Inventory and Config with its
current built-in puppet and helm configuration drivers.  Update to allow plugin
of alternative Configuration drivers based upon its Sysinv service
configuration.

Advantages:
    * Facilitates access to data required for configuration without
      requiring an external REST API, since internal database reference.
    * Maintains referential integrity when inventory updates without requiring
      external notification or audit.
    * Upgrades: database is not split across different services.
    * Lower development cost as a separate API, service is not created and
      data model split could still have some interdependencies.

Disadvantages:
    * Interdependencies between inventory and configuration hinder
      introduction of alternative configuration drivers, and raise risk of
      introducing additional dependencies between the physical inventory and
      baremetal management and logical configuration management.
    * Referential integrity only applies to internal configuration implementation
      with access to data model.  Formal contract for API is still required when
      introducing flexibility of an external configuration driver.
    * Coupling physical and logical configuration resources lessens flexibility
      of each component to evolve independently.
    * The Inventory and Host bare-metal management component remains
      within stx-config.
    * Configuration Resources which are populated could result in changes to
      Inventory Resource data; thus the datamodel split would still be required.


Data model impact
=================

The current datamodel relations which reference the split Inventory into
Configuration component needs to be removed from the service.
Configuration will obtain the required reference based upon the resource uuid
which it GETs from the Inventory API.

The SysInv datamodels are split according to:

Inventory:
    * Host
    * System - (Inventory reference) - uuid
    * Ports
    * Lldp
    * Pci_devices
    * Disks
    * Cpus
    * Memory
    * Sensors,
    * SensorGroups

Configuration:
    * System
    * Host, for configuration host info such as config_status
    * Interfaces
    * Networks
    * Interface_Networks
    * Addresses
    * AddressPools
    * Routes
    * Helm_overrides
    * Certificates
    * Community
    * ControllerFS
    * DNS
    * DRBDConfig
    * NTP
    * PTP
    * RemoteLogging
    * ServiceParameter
    * Storage: lvgs, pvs, clusters, peers, partition, ceph_mon, journal
    * Storage: storage_backend, _ceph, _external, _lvm, _file, _tiers,
    * TpmConfig, Tpmdevices
    * User


REST API impact
===============

Existing APIs from SysInv are migrated to Inventory or Configuration.
    * New Configuration APIs are introduced and migrated APIs from Inventory are deprecated.
    * Remove the ‘i’ prefix in URL resource, where applicable.

There are no policy changes. admin was required to access the SysInv API,
and is required for the Inventory and Configuration APIs.

New Configuration APIs to support the generation  of configuration for the host:
    PUT v1/hosts/<host_uuid>/<action>
        The following actions are required:
            * configure
            * configure_check
                * check whether config is sufficient (e.g. for host-unlock)
            * update_operational
                * For storage host, config performs update_add_ceph_state disable_check
            * Determines whether host config may be disabled (i.e. pre host-lock)


Security impact
===============

Security of the new Configuration service is equivalent to Sysinv:
    * systemconfig-api REST API service with keystone authentication and
      haproxy for https configured oam interface.
    * api service requires admin keystone policy and runs under
      systemconfig user privilege.
    * database, amqp/rabbit access are protected by username and password.


Other end user impact
=====================

A new python-client, python-systemconfigclient is introduced for the
Configuration component.  The systemconfig cli will retain 'system',
whereas the inventory cli will be under 'inventory'.

The interface will no longer be automatically created.  Previously, in the
case of AE config, interfaces need to be configured to 'none' before being
configured again.  This transition is no longer required in this case.


Performance Impact
==================

With Sysinv Decoupling, additional REST API calls are required between the
independent Inventory and Configuration components.

In particular, the following additional REST API interactions:
* Inventory notifies Configuration via REST API to perform
host configuration action
* Configuration requests up-to-date view on configuration action,
the scope is generally limited to the amount of data to be transferred
for the host inventory resource.
* Periodic audits from system config is required to ensure the Inventory
Hosts view is accurate and up-to-date.


Other deployer impact
=====================

Initial Bootstrap (config_controller) now initializes both Inventory and
Configuration services and populates the services with the required host
and configuration data.  This will be transparent to the users.

The association of interfaces to ethernet_ports was performed by default
to a single interface previously.  With the separation of the ports and
interfaces data model, the admin must now associate the interface to
the port as required (ie. via the system host-if-add).

Support for profiles is removed.  The current implementation requires
reference between inventory and configuration resources.


Developer impact
=================

* A new driver API for Configuration is introduced.


Upgrade impact
===============

Upgrades from N-1 are not supported for this update.


Implementation
==============

Assignee(s)
===========

Primary assignee:
  john.kung@windriver.com
  louie.kwan@windriver.com

Other contributors:


Repos Impacted
==============

stx-config:  systemconfig
             python-systemconfigclient
             stx-metal pmon is configured to manage systemconfig-agent.

stx-ha:      SM management of systemconfig and inventory.

stx-metal:   mtce integration
             wrsroot user update via systemconfig-api rather than
             inventory-api.

stx-nfv:     vim interacts with the inventory and configuration components
             and will need a new client to access the host information
             from both the inventory and configuration components

distributedcloud:
             update to api-proxy, dcorch engine to bind to systemconfig
             optional: simplify framework to utilize keystone sessions
             as per other resources managed by dcorch.

stx-gui:     update to reference inventory and configuration apis.
             datamodels within api/sysinv.py need to be refactored to
             fill in the data from the respective service.

stx-clients: add python-systemconfigclient to remoteclients

stx-integ:   ceph-manager deprecate
             restapi-doc updates


Work Items
==========

Phase 1: Create new configuration services and update required
infrastructure

* systemconfig:
    * data model: decouple from internal inventory resource references
    * create systemconfig-api
        * REST API for new config actions (see REST API section)
        * propagate sysinv-api to systemconfig-api
            * obtain inventory resources as required from inventory api
            * remove 'i' prefix from URL
    * create systemconfig-conductor
    * update framework to use standard libraries rather than the internal
      libraries in sysinv (e.g. openstack.common is migrated to oslo_service, paste,
      keystoneauth1, oslo_db, oslo_messaging, oslo_log)

* sysinv:
    * data model: decouple from configuration resource references
    * update REST API

* ha management:
    * stx-ha service-management of systemconfig-api, systemconfig-conductor
    * stx-metal pmon service management of systemconfig-agent(via configuration
    * implemented in stx-config)

* vim:
    * update VIM to handle sysinv API changes

* update config_controller bootstrap to systemconfig.  Startup services,
  populate initial configuration.

* python-sysinvclient:
    * update CLI and client to include keystone session support
      for token  management.

* python-systemconfigclient:
    * create CLI and client


Phase 2: Decouple Major focus areas

Major decoupling focus areas:
    * interface decoupling
      ethernet_ports in inventory and interfaces in systemconfig

      * remove interface_id from ports table in inventory
      * remove autoprovisioning of interfaces in systemconfig

    * storage decoupling

      * disk in inventory and partition/lvg/stor in config
      * ceph_manager
      * deprecate ceph-manager: move rpc endpoint functionality into
        systemconfig-conductor, ceph-manger-audit to config-audit,
        and alarm monitoring into collectd plugin.

Create plugin model in Inventory for configuration.  The plugin is
implemented as a stevedore driver and selection of driver is driven by
config. The plugin allows the bind in of abstract configuration methods
required for the Inventory host management operations.

Phase 3: Decoupling of remaining resources and Integration

* decouple remaining configuration resources from sysinv

* distributedcloud
    * proxy SystemController systemconfig-api requests into dcorch-engine
    * dcorch-engine to interface systemconfig-api for configuration
    * dcmananger-api to interface with python-systemconfigclient
      (network list, address_pools, routes)

* stx-gui  inventory dashboard updated to reference the inventory and
           configuration REST APIs

* tox unit tests (this could be started earlier, however initial focus
  is verification in lab)


Dependencies
============

2002827 Decouple Service Management REST API from sysinv
https://storyboard.openstack.org/#!/story/2002827

2002828 Decouple Fault Management from stx-config
https://storyboard.openstack.org/#!/story/2002828


Testing
=======

* Bootstrap Initialization and Configuration
* Host Configuration and Management
* Interface Configuration
* Storage Configuration
* Service Parameter Configuration
* HA verification
* Distributed Cloud Verification
* Horizon GUI
* Devstack


Documentation Impact
====================

systemconfig and sysinv REST API documentation
End User Guide: installation and configuration


References
==========


History
=======

.. list-table:: Revisions
         :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced
