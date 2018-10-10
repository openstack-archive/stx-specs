..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

==================================
StarlingX: Docker Image Generation
==================================

https://storyboard.openstack.org/#!/story/2003907

This story will update the StarlingX build system to include generation of
Docker images for infrastructure services that will run in containers. These
services will initially include all OpenStack services supported by StarlingX,
nova-api-proxy and fault-management and this story will extend the build system
to generate docker images for each.

We will make use of OpenStack LOCI to help build the StarlingX service images.
In support of this, we will also build python wheels for required modules to
align with the appropriate upper constraints.

Problem description
===================

We need a mechanism to build images for the StarlingX services to support
containerization.

Use Cases
=========

Developers need to be able to build images for the StarlingX services.

Automated build tools need to be able to build images.

Proposed change
===============

For building python wheels, we will make use of the python2/3 pip and wheels
packages. These allow us to generate the wheels during the regular loadbuild,
for python modules we currently build. Additionally, we will provide a
mechanism for building wheels for specific versions of a base set of python
modules from trusted sources, such as pypi.org. This combined set of wheels
will be provided as input to LOCI when building the images.

For building the StarlingX images, we will create a set of image build
configuration files and a utility to process these files, using the directives
within to provide the appropriate configuration to LOCI to build the necessary
images.

Alternatives
============

We could create our own docker files to generate the StarlingX images,
installing the necessary components, but this would be reimplementing the
functionality provided by LOCI. As well, LOCI is already being used by
OpenStack Helm for building images for the OpenStack services, so it already
has support within the community.

Data model impact
=================

None

REST API impact
===============

None

Security impact
===============

None

Other end user impact
=====================

None

Performance Impact
==================

None

Other deployer impact
=====================

None

Developer impact
=================

Introduction of this feature will add additional steps to a developer build,
but these should be optional.

Upgrade impact
===============

None

Implementation
==============

Assignee(s)
===========

Primary assignee:
  Don Penney <dpenney>

Other contributors:
  Al Bailey <albailey>

Repos Impacted
==============

 * stx-config
 * stx-fault
 * stx-gui
 * stx-ha
 * stx-integ
 * stx-nfv
 * stx-root
 * stx-update
 * stx-upstream

Work Items
===========

* Update StarlingX python modules to generate wheels
* Create mechanism and config for building base python wheels
* Create mechanism and config for building StarlingX images

Dependencies
============

None

Testing
=======

StarlingX images will be verified to ensure images can be launched and contain
all necessary software.

Documentation Impact
====================

Documentation for the build and developer workflow will need to be updated.

References
==========

https://github.com/openstack/loci

History
=======

.. list-table:: Revisions
      :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced

