..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode


==================================================
 Containerization: Helm Chart Override Generation
==================================================

Storyboard: `#2003909`_

This spec is part of the larger effort to containerize the StarlingX
infrastructure. Please see the background information regarding this effort on
the StarlingX `wiki`_.

For managing the life-cycle of containerized services, StarlingX will be using
two open source technologies: `Helm`_, the Kubernetes package manager and
`Armada`_, a tool for managing multiple Helm charts with dependencies.

This specification will outline the use of `Helm`_ in a StarlingX containerized
platform and the need to generate system compatible `Helm chart`_ overrides.

`Armada`_ integration and use will be discussed in an additional `Armada
integration`_ story.

.. _#2003909: https://storyboard.openstack.org/#!/story/2003909
.. _wiki: https://wiki.openstack.org/wiki/Containerizing_StarlingX_Infrastructure
.. _Helm: https://helm.sh
.. _Armada: https://airship-armada.readthedocs.io/en/latest/
.. _Armada integration: https://storyboard.openstack.org/#!/story/2003908
.. _Helm chart: https://docs.helm.sh/developing_charts


Problem description
===================

Each StarlingX native service that will be containerized shall be described by
a `Helm chart`_.

The `Helm chart`_ for a given service will be built from a chart provided by
one of the following open source projects: `openstack-helm`_ (for openstack
services), `openstack-helm-infra`_ (for openstack support services),
`external-storage`_ (for volume provisioners), and StarlingX (nova-api proxy,
fault manager, etc)

The current set of native services that will be containerized are: AODH,
Ceilometer, Cinder, Fault Manager, Glance, Gnocchi, Heat, Horizon, Ironic,
Keystone, Libvirt, Magnum, Murano, Neutron, Nova, Nova-API Proxy, Panko, and
Rabbitmq.

To support these services, the following charts and corresponding services are
also required: Ingress, MariaDB, Memcached, and an RBD volume provisioner.

A chart comes with a default set of values that will describe the behavior of
the service installed within the Kubernetes cluster. `Helm`_ provides a
mechanism to override these default values so the service can be updated based
on the desired service configuration.

In the current StarlingX code-base, native services are configured via puppet
modules with supplied hiera data derived from the contents of the system
inventory database. The values in the database reflect the current physical
components of installation along with the configuration choices of the
Deployer.

In order to migrate the native services to containerized services, the
following problems need to be addressed:

* Need a sysinv based framework to examine the system inventory database and
  generate chart specific system overrides for each native service.

* The framework should be easily extensible for adding chart support.

* Need an API for the Deployer to easily override or complement the system
  generated overrides for a specific chart

* Need an API to generate the final overrides for a specific chart or set of
  charts. The final generated set of overrides should be overrides with the
  following superseding priority: default values (base) <- system generated <-
  Deployer provided.

It should be noted that this does not preclude any `Helm chart`_ from being
installed on the system and used with standard `Helm`_ mechanisms for managing
the life-cycle of a chart release.

.. _openstack-helm: https://github.com/openstack/openstack-helm
.. _openstack-helm-infra: https://github.com/openstack/openstack-helm-infra
.. _external-storage: https://github.com/kubernetes-incubator/external-storage


Use Cases
=========

The Deployer needs to be able manage and generate the helm overrides for a
given `Helm chart`_. The following API actions need to be enabled

* List system supported helm charts.
* Show overrides for a chart.
* Delete overrides for a chart.
* Update helm chart user overrides.


.. _proposed_change:

Proposed change
===============

In order to properly configure a `Helm chart`_ for a containerized service the
following is proposed:

* Provide a plugin framework as part of sysinv to allow adding support for new
  charts as required.
* Provide a basic plugin for each chart in the `openstack-helm`_ project.
* Provide specific plugin logic to generate chart overrides for each of the
  currently supported StarlingX native services.

With the plugin framework and chart plugins deployed as part of sysinv, a
mechanism is required to generate the overrides for a specific chart. For this
the following is proposed:

* Implement sysinv API/CLI for managing system and Deployer provided
  overrides:

  * helm-override-list   - List system helm charts.
  * helm-override-show   - Show overrides for a chart.
  * helm-override-delete - Delete overrides for a chart.
  * helm-override-update - Update helm chart user overrides.

* Additional options will be provided to the helm-overrides-show CLI to
  extract the combined overrides in a yaml form to feed into `Helm`_ or
  `Armada`_.

  * --format [ helm | armada ]

    * Specifying armada will add the `Armada`_ compliant header to the yaml
      output.

  * --file=<filename>

    * An optional parameter, provided with the use of the --format option
      will generate a yaml file containing the overrides. If this option is
      not specified, then the resulting yaml file will be delivered to
      stdout.

Alternatives
============

An alternative approach would be to leave the override configuration for each
charts as an exercise up to the Deployer using standard `Helm`_ and/or
`Armada`_ mechanisms. This approach would increase initial system provisioning
times and would lead to a hit-or-miss integration strategy to find the correct
overrides for a give configuration.


Data model impact
=================

To support managing chart overrides, a new table will be added to the sysinv
DB, with the following columns:

* created_at, updated_at, deleted_at: standard columns
* id: primary key
* name: chart name
* namespace: chart namespace
* user_overrides: store deltas as compared to system overrides
* unique constraint is a combination of name and namespace


As usual, a new data migration will be added to create this table in the sysinv
DB.


REST API impact
===============

This impacts the sysinv REST API:

* The new resource /helm_charts is added and the GET method would return all
  the charts that provide system overrides along with their namespaces.

  * URLS:

    * /v1/helm_charts/

  * Request Methods:

    * GET

  * JSON response example::

      {"charts": [
        {"name": "ingress", "namespaces": ["kube-system", "openstack"]},
        {"name": "rbd-provisioner", "namespaces": ["kube-system"]},
        {"name": "openvswitch", "namespaces": ["openstack"]},
        {"name": "rabbitmq", "namespaces": ["openstack"]},
        {"name": "libvirt", "namespaces": ["openstack"]},
        {"name": "heat", "namespaces": ["openstack"]},
        {"name": "keystone", "namespaces": ["openstack"]},
        {"name": "nova", "namespaces": ["openstack"]},
        {"name": "horizon", "namespaces": ["openstack"]},
        {"name": "cinder", "namespaces": ["openstack"]},
        {"name": "glance", "namespaces": ["openstack"]},
        {"name": "mariadb", "namespaces": ["openstack"]},
        {"name": "memcached", "namespaces": ["openstack"]},
        {"name": "neutron", "namespaces": ["openstack"]}]}

* By providing the GET method a specific chart name along with key/value pair
  specifying a namespace, the values of a chart can be retrieved

  * URLS:

    * /v1/helm_charts/{name}?namespace={namespace}

  * Request Methods:

    * GET

  * Example:

    * Request: GET /v1/helm_charts/rabbitmq?namespace=openstack
    * JSON response example::

        {"namespace": "openstack",
         "name": "rabbitmq",
         "system_overrides": "pod:\n  replicas: {server: 1}\n",
         "user_overrides": "",
         "combined_overrides": "pod:\n  replicas:\n    server: 1\n"}

* Using the PATCH method will allow the Deployer to update override values

  * URLS:

    * /v1/helm_charts/{name}?namespace={namespace}

  * Request Methods:

    * PATCH

  * Example:

    * Request: PATH /v1/helm_charts/rabbitmq?namespace=openstack
    * JSON request example::

        {"flag": "reuse", "values": {"files": [], "set": ["pod.replicas=2"]}

    * JSON response example::

        {"user_overrides": "pod:\n  replicas: 2\n", "namespace": "openstack",
         "name": "rabbitmq"}

* Using the DELETE method will allow the Deployer to delete all Deployer
  provided overrides. Once deleted, the system generated override will still be
  applied for the chart.

  * URLS:

    * /v1/helm_charts/{name}?namespace={namespace}

  * Request Methods:

    * DELETE

  * Example:

    * Request: DELETE /v1/helm_charts/rabbitmq?namespace=openstack


Security impact
===============

Passwords may be provided in the overrides. Considerations need to be made when
displaying/setting these override values. One potential solution is to prevent
overriding these values via the API.

Other end user impact
=====================

As mentioned in the :ref:`proposed_change` section, new CLI commands will be
provided and will behave as follows:

* helm-override-list   - List system helm charts.::

    $ system helm-override-list
    +-----------------+--------------------------------+
    | chart name      | overrides namespaces           |
    +-----------------+--------------------------------+
    | ceilometer      | [u'openstack']                 |
    | cinder          | [u'openstack']                 |
    | glance          | [u'openstack']                 |
    | gnocci          | [u'openstack']                 |
    | heat            | [u'openstack']                 |
    | horizon         | [u'openstack']                 |
    | ingress         | [u'kube-system', u'openstack'] |
    | keystone        | [u'openstack']                 |
    | libvirt         | [u'openstack']                 |
    | mariadb         | [u'openstack']                 |
    | memcached       | [u'openstack']                 |
    | neutron         | [u'openstack']                 |
    | nova            | [u'openstack']                 |
    | openvswitch     | [u'openstack']                 |
    | rabbitmq        | [u'openstack']                 |
    | rbd-provisioner | [u'kube-system']               |
    +-----------------+--------------------------------+

* helm-override-update - Update helm chart user overrides.::

    $ system helm-override-update rabbitmq openstack \
     --reuse-values \
     --set pod.replicas=2
     +----------------+---------------+
     | Property       | Value         |
     +----------------+---------------+
     | name           | rabbitmq      |
     | namespace      | openstack     |
     | user_overrides | pod:          |
     |                |   replicas: 2 |
     |                |               |
     +----------------+---------------+

* helm-override-show   - Show overrides for a chart.::

    $ system helm-override-show rabbitmq openstack
    +--------------------+-------------------------+
    | Property           | Value                   |
    +--------------------+-------------------------+
    | combined_overrides | pod:                    |
    |                    |   replicas: 2           |
    |                    |                         |
    | name               | rabbitmq                |
    | namespace          | openstack               |
    | system_overrides   | pod:                    |
    |                    |   replicas: {server: 1} |
    |                    |                         |
    | user_overrides     | pod:                    |
    |                    |   replicas: 2           |
    |                    |                         |
    +--------------------+-------------------------+

* helm-override-delete - Delete overrides for a chart.::

    $ system helm-override-delete rabbitmq openstack
    Deleted chart overrides for rabbitmq:openstack



Performance Impact
==================

Minimal impact to the system controllers is to be expected. The proposed
changes to generate and manage the chart overrides will access the sysinv
database and execute helm commands to determined the combined overrides.

Other deployer impact
=====================

The API and ability to extract the combined overrides will enable Armada to be
integrated as a mechanism to manage and launch all the current bare metal
native service as a kubetnetes application.

Developer impact
================

Developers working in StarlingX will need to use the API changes provided here
to view, adjust, and generate chart overrides for containerized service
deployment via `Helm`_.

Upgrade impact
==============

None. This is the first StarlingX release with `Helm`_ support. No previous
release supports this feature and no upgrade support is required.

Implementation
==============

As the system inventory database contains all the pertinent information about
the installed system, changes will be required in stx-config/sysinv to generate
the overrides for the `Helm`_ charts and provide the APIs to access them.

The implementation will duplicate the plugin framework and associated
constructs used for extracting and generating the puppet hiera for existing
native services.

Once fully implemented, the puppet manifests for the native services and the
plugins that generated the hiera data for those manifests will be removed. In
their stead, `Helm`_ chart overrides will be generated and provided `Armada`_
for launching the containerized services.

Assignee(s)
===========

Primary assignee:

* Robert Church (rchurch)

Other contributors:

* Chris Friesen (cbf123)
* Gerry Kopec (gerry-kopec)
* Joseph Richard (josephrichard)
* Tyler Smith (tyler.smith)
* Angie Wang (angiewang)
* Irina Mihai (irina.mihai.wrs)
* Ovidiu Poncea (ovidiu.poncea)
* Al Bailey (albailey1974)
* Lachlan Plant (lachlan.plant)


Repos Impacted
==============

stx-config

Work Items
==========

* stx-config/sysinv:

  * Provide a plugin framework as part of sysinv to allow adding support for
    new charts as required.
  * Provide a basic plugin for each chart in the `openstack-helm`_ project.
  * Provide specific plugin logic to generate chart overrides for each of the
    currently supported StarlingX native services.
  * Implement API changes to support listing supported charts that provide
    system overrides.
  * Implement API changes to support showing the system, Deployer, and combined
    overrides.
  * Implement API changes to support adding and modifying Deployer provided
    overrides.
  * Implement API changes to support deleting Deployer provided overrides.

* stx-config/cgts-client:

  * Implement sysinv CLI for managing system and Deployer provided overrides by
    calling the sysinv API:

    * helm-override-list   - List system helm charts.
    * helm-override-show   - Show overrides for a chart.
    * helm-override-delete - Delete overrides for a chart.
    * helm-override-update - Update helm chart user overrides.
    * Add helm-override-show options for generating the final view of overrides

      * Add: --format [ helm | armada ]
      * Add: --file=<filename>


Dependencies
============

This requires new functionality being developed under the following stories:

* Kubernetes Platform Support: `#2002843`_
* CEPH persistent storage backend for Kubernetes: `#2002844`_
* Local Docker Registry: `#2002840`_
* Docker Image Generation: `#2003907`_
* Infrastructure HELM Chart Override Generation: `#2003909`_
* Armada Integration: `#2003908`_

.. _#2002843: https://storyboard.openstack.org/#!/story/2002843
.. _#2002844: https://storyboard.openstack.org/#!/story/2002844
.. _#2002840: https://storyboard.openstack.org/#!/story/2002840
.. _#2003907: https://storyboard.openstack.org/#!/story/2003907
.. _#2003909: https://storyboard.openstack.org/#!/story/2003909
.. _#2003908: https://storyboard.openstack.org/#!/story/2003908


Testing
=======

The following testing will be performed in association with these proposed
changes:

* The sysinv REST API will be exercised using curl to verify/validate it's
  operation. It will be used for documenting API access.
* The cgts-client commands will be exercised along with any supported options.
* The resulting combined overrides yaml files will be used with `Helm`_ and
  `Armada`_ to ensure compatibility.


Documentation Impact
====================

This story affects the following StarlingX documentation:

* Installation and configuration of containerized services.
* sysinv REST API documentation.

Specific details of the documentation changes will be addressed once the
implementation is complete.


References
==========

References are provided throughout this document at the point when terms or
items are introduced. No additional references are needed at this time.


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced
