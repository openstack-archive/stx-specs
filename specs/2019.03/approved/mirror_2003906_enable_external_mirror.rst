..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

=============================================================
StarlingX: Enable External Mirror for the StarlingX Community
=============================================================

https://storyboard.openstack.org/#!/story/2003906

An external mirror is a great enabler for the open source development
community of StarlingX. The goal of the mirror is to quickly enable new
developers and support continuous development / continuous integration
environments for the community.

This story will implement setup of an external mirror provided by CENGN.

Problem Description
===================

Preparing to build or use StarlingX for the first time can be difficult
and time consuming as many components must be downloaded prior to building.
A pre-populated mirror that shares the required components would simplify
the developer experience.

For the initial release, this mirror would pull source and binaries from:
* CentOS
* OpenStack
* CEPH
* DPDK.org
* and others.

Use Cases
=========

Developers need to be able to build current and branched builds of StarlingX
without having to download all the mirrored files from multiple locations.

Developers need to be able to request alternative versions of RPMs be added to
the public mirror.

Automated build tools need to be able to build from the new mirror.

Mirror hosts recent builds of StarlingX.  This will enable the installation
and usage of the packaged software quickly without the complexity of creating
their own StarlingX development environment.

Build Evolution:
* Modify distro packages built and added to another versioned external mirror
* Only build these packages when there is a change

Proposed Change
===============

Insulate StarlingX developers from public mirrors that drop packages, or
are not reliably accessible, by setting up a mirror on a public server.
Mirrored artifacts will include all Centos 7 and EPEL 7 content, as well
as any other downloadable artifacts (packages, tarballs, installers)
required for a successful build.

Modify download_mirror.sh and other supporting scripts to preferentially
pull content from our new mirror.  The ability to fall back to original sources
shall be retained to protect against failure of the mirror.

Alternatives
============

The existing build and mirror procedure is the alternative.  This is simply
meant to replace and improve upon what already exists.

Data Model Impact
=================

None

REST API Impact
===============

None

Security Impact
===============

The master branch mirror will pull the latest versions of packages from the
source upstream mirrors periodically.

This will include picking up new package versions containing CVE fixes that
have been addressed in the upstream mirrors.

There will be a process put in place to purge older package versions from the
mirror.

Since one of the objectives of the mirror is to ensure package versions do not
disappear randomly on developers, this purge will be done in a controlled
manner.


Other End User Impact
=====================

RPMs used by StarlingX will be more reliably available.

Performance Impact
==================

Build times should improve.

Other Deployer Impact
=====================

None

Developer Impact
=================

Introduction of this feature will modify the steps run by a developer build.

Upgrade Impact
===============

None

Implementation
==============


Assignee(s)
===========

Primary assignee:
  Scott Little <slittle1>

Other contributors:
  Don Penney <dpenney>

  Al Bailey <albailey>

Repos Impacted
==============

 * stx-tools

Work Items
===========

* Setup Bare Metal (CENGN) Server
* Setup basic mirror for the community
* Build tool changes for using mirror
* Implement daily mirror updates
* Support for release versioning in mirror and tools

Dependencies
============

None

Testing
=======

StarlingX will be verified to ensure developers can build.
Verify that a new RPM can be added.
Verify that a version of an existing mirror RPM can be changed.

Documentation Impact
====================

Documentation for the build and developer workflow will need to be updated.

References
==========

https://www.cengn.ca/

History
=======

.. list-table:: Revisions
      :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced

