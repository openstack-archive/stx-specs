..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

=============================================================
StarlingX: Reorganize Flock Services Source Code repositories
=============================================================

Storyboard: https://storyboard.openstack.org/#!/story/2004792

StarlingX currently combines source code and build meta-data in the same
directory structure, as we move to support multiple OSes, this structure
needs to be seperated and further defined for the benefit of developers in the
community to keep history and source code separated. Simplifying things to a
set of common features also had the dual benefit of making is easier for others
to understand and contribute to our project.

Problem description
===================

The current dirtectory structure is based around the build working with CentOS
OS and the current build tooling, as we add additional OS support, additional
meta-data is needed and should be added into a new structure. We realized we
needed to standardize the structure of our open source projects in order to
reduce the amount of time we spent figuring out how things work. Simplifying
things to a set of common features also had the dual benefit of making is
easier for others to understand and contribute to our projects. There are some
Directory Structures that could open source use as standard [1] [2], right
now we are not using many of that standard directory structure.

Use Cases
=========

1. Developers want to apply a change in the CentOS spec file but not to the
Ubuntu rules file.

2. Developers want to apply a performance change into CentOS but is not
necesary in Ubuntu.

3. Developers want to change the installation path at the CentOS spec file but
not at the Ubuntu build scripts . Developer will not change anything at the src
code of the service, no need to touch the git history of the source code
project.

Proposed change
===============

Reorganize the existing source code to seperate the build meta-data from the
base source code and create indvidial git repos for each STX Flock
sub-component.

One example of a package with multi OS support could be:

::

    <flock_package_name>-pkg/
    ├── ubuntu
    │   ├── patches
    │   │   └── cve_fix.patch
    │   ├── rules
    │   └── <package_name>.dsc
    ├── centos
    │   ├── cve_fix.patch
    │   ├── improve_perf.patch
    │   ├── <package_name>.spec
    ├── clear
    │   ├── autospec files
    │   └── cve_fix.patch
    └── kubernetes

An output of this solution of this could be:

https://git.starlingx.io/cgit/stx-integ/stx-nfv

and

https://git.starlingx.io/cgit/stx-integ/stx-nfv-pkg

where stx-nfv will hold just the source code of the service and stx-nfv-build
will host the build scripting for multiple operating systems and even
containers/kubernetes


Alternatives
============

Keep the existing directory structure and add additional sub-directories for
the new Operating Systems, which will clutter the current repositories.

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

Improve developer experience to isolate each package increasing the modularity
of the development, having delimitated the boundaries of each package and how
they are built.

Upgrade impact
===============

None

Implementation
==============

Is possible to create separate branch for now and merge until is proved that
does not break the build or the sanity of the system

Assignee(s)
===========

Primary assignee:
    - Victor Rodriguez

Other contributors:

Repos Impacted
==============

- https://git.starlingx.io/cgit/stx-integ/stx-clients
- https://git.starlingx.io/cgit/stx-integ/stx-config
- https://git.starlingx.io/cgit/stx-integ/stx-distcloud
- https://git.starlingx.io/cgit/stx-integ/stx-distcloud-client
- https://git.starlingx.io/cgit/stx-integ/stx-fault
- https://git.starlingx.io/cgit/stx-integ/stx-governance
- https://git.starlingx.io/cgit/stx-integ/stx-gplv2
- https://git.starlingx.io/cgit/stx-integ/stx-gplv3
- https://git.starlingx.io/cgit/stx-integ/stx-gui
- https://git.starlingx.io/cgit/stx-integ/stx-ha
- https://git.starlingx.io/cgit/stx-integ/stx-nfv
- https://git.starlingx.io/cgit/stx-integ/stx-update
- https://git.starlingx.io/cgit/stx-integ/stx-upstream
- https://git.starlingx.io/cgit/stx-integ/stx-utils


Work Items
===========
- Create development branch on current repositories
- Create build managment repositories for each service
- Move necesary build scripts to build managment repositories
- Test build managment repositories in package build system

Dependencies
============


Testing
=======

After building a proper image with the re org of the repositories we can:

- Test build managment repositories can generate current RPMs
- Build an STX image
- Run sanity tests for generated image

Documentation Impact
====================

Create section for developer guide, that guide themhow to do a propper
development contribution for the project , a good example for this could be:

https://devguide.python.org/

References
==========

[1] https://www.gun.io/blog/maintaining-an-open-source-project

[2] https://github.com/kriasoft/Folder-Structure-Conventions

History
=======

.. list-table:: Revisions
      :header-rows: 1

   * - Release Name
     - Description
   * - 2019.03
     - Introduced
