Containerization -- Armada Integration
==========================================

Storyboard: https://storyboard.openstack.org/#!/story/2003908

This story will build on the basic Kubernetes/Helm support, making use of the
``airship-armada`` project to provide a higher-level management of the
multiple helm charts which together comprise an application such as OpenStack.

Problem description
===================

Please see the following for background on the effort to containerize the
StartlingX infrastructure:
https://wiki.openstack.org/wiki/Containerizing_StarlingX_Infrastructure

As part of the switch to containerization, during initial installation
StarlingX will no longer make any assumptions about which applications will
ultimately be installed on the cluster.

As such, we will add additional commands within the ``system`` CLI command
to support installing an ``application``, where the installed application
consists of Kubernetes resources.  The ``application`` would be uploaded onto
the system as a tarball, the format of which will be discussed later.

In the case of the OpenStack application, the ``openstack-helm`` project
provides a set of helm charts which can be used to set up a functional
installation, but it would be a significant amount of work for the operator
to manually bring up the charts in the correct order and ensure that the
appropriate charts are serialized while others are brought up in parallel.

Use Cases
=========

As a developer/tester/operator I need the ability to install and configure
various applications (including OpenStack) on a running StarlingX system.

As a developer/tester I need the ability to uninstall various applications
(including OpenStack) on a running StarlingX system.

Proposed change
===============

In order to manage the multiple charts which comprise an application, we
propose to make use of the ``airship-armada`` project.  This project is
developed under the OpenStack umbrella as part of the ``Airship`` suite
and is intended to provide a mechanism to group together multiple helm
charts, as well as specify the dependencies and ordering between them.  It
also provides a way to specify dependencies between charts which is more
rigorous than what Helm itself provides.

Packaging of Armada In the Load
-------------------------------

Upstream Armada [1]_ is built/tested for Ubuntu only, and is built against a
newer set of packages than are present in StarlingX.  It also has dependencies
on packages which are not present in CentOS and are not available as pre-built
RPMs.  This makes it difficult and potentially time-consuming to build Armada
natively for StarlingX on CentOS.  Accordingly, we propose to run Armada from
within a Docker container using the upstream Docker image as described in the
Armada documentation. [2]_  As all use of Armada will be indirect via the
``system`` command implemented as part of the StarlingX ``sysinv`` tool, the
added commandline complexity of running Armada commands within a container
will be hidden from the end user.

Application Distribution
------------------------

Each "application" will be provided as a compressed tarball, consisting of a
"metadata.yaml" file, a mandatory "manifest.yaml" armada manifest, an optional
"charts" directory containing local helm charts, and an optional "images"
directory containing local docker images.

The "metadata.yaml" file will contain information about node labels.  We may
add more to it later as needed.

System Application Upload
-------------------------

Add a new sysinv command that will take as an argument the filename of a
tarball representing an application.  Initially, this will need to be run from
the controller node but eventually we should be able to support remote clients
as well.

The sysinv conductor will then unpack the tarball and load any Docker images
embedded within the tarball to the local Docker registry.  It will also upload
any Helm charts embedded within the tarball to the local Helm repository.
Finally, the armada manifest(s) from the tarball will be moved over to
``/opt/platform/armada/<uuid>``, and the sysinv server will set the application
status to ``uploaded``.

System Application Apply
--------------------------

Add a new sysinv command that will take as an argument one application name.

The sysinv conductor will loop over all manifest files and for each chart in the
manifest will query the system and user overrides for that chart and write the
combined overrides to a file in a format suitable for armada.

The sysinv conductor will run ``armada apply`` with the ``--values`` option to
specify the combined overrides file(s).  If all goes well, this should apply
all the various helm charts in the correct order as well as running self-tests
on them if specified. Outputs will be logged and echoed back to the user where
possible in case anything goes wrong.  On success, the sysinv conductor sets
the application status to ``installed``.

This command can also be used to apply changes to an installed manifest after
the system or user overrides have been modified.

System Application List
-----------------------

Add a new sysinv command that will take no arguments and will list the various
uploaded applications along with their status.

System Application Show
-----------------------

Add a new sysinv command that will take as an argument one application name and
will display information about the application.


System Application Remove
-------------------------

Add a new sysinv command that will take as an argument one application name.

The sysinv conductor will run ``armada delete --manifest`` against the
application manifest.

If anything has been deleted, the sysinv conductor sets the application status
to ``uploaded``.

System Application Delete
-------------------------
Add a new sysinv command that will take as an argument one application name.

The sysinv API will exit with an error message if the application status is
``installed``, otherwise the sysinv API will delete the
``/opt/platform/armada/<application>`` subdirectory.  For now it will not
delete the helm charts or docker images as they could be in use by other
applications.  As a future work item, we could conceivably add a check to
see if the charts/images are used by any uploaded application.

Alternatives
============

The main alternative to using airship-armada is to create a tool that will
handle tracking overrides (both system and user) for each helm chart and
apply them along with handling any inter-chart dependencies that helm doesn't
handle properly.

Data model impact
=================

Knowledge of the OpenStack application helm charts and their customization
will no longer be implicitly embedded in puppet manifests and sysinv code
and database.  Instead, it will be explicitly encoded in the Armada manifest,
helm charts, and helm chart overrides.

Many changes to the application configuration (including the OpenStack
application) will no longer require code changes to implement.  Instead, they
can be implemented as helm chart user overrides.

REST API impact
===============

The sysinv API will be extended to support the following operations:

* **POST /v1/apps**

  * The new resource /apps is added and the POST method will accept a
    a dictionary in the request body which specifies the application name
    and the location of the application tarfile as the input to register
    (i.e. upload) the application with the system.
  * Request body example::

      {'name':  'stx-openstack',
       'tarfile': '/home/wrsroot/stx-openstack-app.tgz'}

  * Response body example::

      {'status': 'uploading',
       'name': 'stx-openstack',
       'created_at': '2018-10-03T06:12:12.719093+00:00',
       'update_at': None,
       'manifest_name': 'armada-manifest',
       'manifest_file': 'armada-osh.yaml'}

* **PATCH /v1/apps/{app_name}?directive={directive}**

  * The PATCH method will apply the specified directive to an existing
    application. Acceptable directives are `apply` or `remove`.
    Initially the request body can be empty but they will contain values
    as the software evolves.

  * Request body example::

     {'values': {}}

  * Response body::

      Same content as for POST

* **GET /v1/apps**

  * The GET method would return all kubenetes applications known to the system.
  * Response body::

      Same content as for POST

* **GET /v1/apps/{app_name}**

  * The GET method would return the details of the specified application.
  * Response body::

      {'apps': [{<app1 data>}, {app2 data}]}

* **DELETE /v1/apps/{app_name}**

  * The DELETE method would purge a removed/uploaded application from the
    system.

  * Response body::

     There is no body content for the response to a successful DELETE request.


Security impact
===============

Armada makes use of Helm.  As such, it does not introduce any additional
security impact.

Other end user impact
=====================

The end user is expected to interact with the feature via the ``system``
client for sysinv.

Performance Impact
==================

The impact is unknown at this time, but any impact would primarily be to the
application install phase which is not expected to be a high-runner operation.

Other deployer impact
=====================

None

Developer impact
=================

Developers adding new charts to an application will need to update the
Armada manifest for the application.

Upgrade impact
===============

None, as this is the initial release of StarlingX.


Implementation
==============

Assignee(s)
===========

Primary assignee:
  Chris Friesen (cfriesen)

Other contributors:
  Tee Ngo (teewrs)

Repos Impacted
==============

* stx-config

Work Items
===========

* Create initial OpenStack manifest based on the one in openstack-helm.
* Tweak the OpenStack Armada manifest for StarlingX.
* Modify sysinv to emit helm chart system overrides formatted for Armada
  rather than bare Helm.
* Add application upload/apply/remove/delete commands to sysinv.

Dependencies
============

* airship-armada
* Kubernetes Platform Support:
  https://storyboard.openstack.org/#!/story/2002843
* Infrastructure HELM Chart Override Generation:
  https://storyboard.openstack.org/#!/story/2003909

In addition, the work for ``System Deployment of Containerized Infrastructure``
(https://storyboard.openstack.org/#!/story/2003910) will need to be done in
conjunction with the Armada manifest to ensure they are suitably aligned.


Testing
=======

This story affects the configuration and deployment of all OpenStack services
on StarlingX. In addition to the usual unit testing in the impacted code
areas, this will require a full system regression of all StarlingX
functionality. It will also require performance testing in order to identify
and address any performance impacts.

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

.. [1] https://github.com/openstack/airship-armada

.. [2] https://airshipit.readthedocs.io/projects/armada/en/latest/operations/guide-use-armada.html

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced

