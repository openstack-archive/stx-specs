
..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

==============================================
StarlingX: Define directory layout for MultiOS
==============================================

Storyboard: https://storyboard.openstack.org/#!/story/2004891

This specification will extend the current directory layout used by the
current StarlingX build system to allow for future build system(s). There are
currently other directories that contain patches or configuration files, these
files are typically version specific and will need to be considered in any
solution.


Problem description
===================

The current dirtectory structure is based around the build working with CentOS
OS and the current build tooling. The current diectory structure also contains
additional directories and files that provide patches and/or configuration
data, as these may be OS or version specific, they will need to move.

In order to support building packages for new, different OSes additional
meta-data, patches and/or configuration data will be needed and require
additional directories.

Use Cases
=========

a) Devepolers need to support multiple OSes beyond the current CentOS


Proposed change
===============

Define an extended directory layout that will support other Operating Systems
beyond the existing CentOS layout. The existing layout will remain in place as
the new build system is developed in order to not distrub the current
workflow.

The terms "patched" and "unpatched" are used to distigusish the different
file structure of RPM packages. The patched example is the file structure for
RPMs that we patch. While the unpatched example is the file structure for new
RPMs that we are creating from a specfile. These terms will not be in the
actual directory structure.

The following is the proposed layout:

::

<package-name>
├── centos (patched)
│   ├── build_srpm.data
│   ├── files
│   │   ├── 0001-First-centos-source.patch
│   │   └── centos.conf
│   ├── meta_patches
│   │   ├── 0001-First-centos-spec-file.patch
│   │   └── PATCH_ORDER
│   └── srpm_path
├── centos (unpatched)
│   ├── build_srpm.data
│   ├── files
│   │   ├── 0001-First-centos-source.patch
│   │   └── centos.conf
│   └── package.spec
├── clear
│   ├── auto-spec-files
│   └── stx-specific.patch
└── ubuntu
    ├── package_version.dsc
    └── package_version_starlingx.diff.gz

Both Clear Linux [0] and Ubuntu [1] have suggested file organization that work
with their respective build tools. In the Clear Linux case, autospec generates
an rpm specfile based on additional control files, those control files need
be in their own directlry. The Ubuntu example above follow the Debian packaging
policy, the ".dsc" file defines what the dependencies are and where to fetch
the source from, the ".diff.gz" file contains patches to both source and other
control files (ie rules file). In the case of a "unpatched" package as defined
above, the diff.gz file contains the needed control file to build the package.

Alternatives
============

Create a new set of repos to represent each new operating system.

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

Developers will need to ensure any changes are correctly rebased and tested
against each upstream Operating system being supported by the current build.

Upgrade impact
===============

None

Implementation
==============

Implementation will be in parallel to the current build system, the existing
file structure will not change until the changes are made to the build
system and/or meta-data (srpm*) files. Any new directories and/or files can
be created as needed based on the implementation of the new build system for
the Operating system that is being added.

Assignee(s)
===========


Primary assignee:
   - Victor Rodriguez

Other contributors:
   - Erich Cordoba Malibran


Repos Impacted
==============

https://git.starlingx.io/cgit/stx-root/ - as needed when we move files/patches
 directories into centos
https://git.starlingx.io/cgit/stx-integ/
https://git.starlingx.io/cgit/stx-upstream/
All Flock related repos currently containing cento meta-data or the new
https://git.starlingx.io/cgit/stx-flock/ if created.


Work Items
===========

- Create directory tree and files as new Operating Systems are added.

Dependencies
============


Testing
=======

Ensure that the current build continues to work as the directory layout is
extended.

Documentation Impact
====================

New documentation will be generated to define the contents of the extended
directory layout.

References
==========

[0] https://clearlinux.org/documentation/clear-linux/concepts/autospec-about
[1] https://www.debian.org/doc/debian-policy/index.html

History
=======

.. list-table:: Revisions
      :header-rows: 1

   * - Release Name
     - Description
   * - 2019.10
     - Introduced
