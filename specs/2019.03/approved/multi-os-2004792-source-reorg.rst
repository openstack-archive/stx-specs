..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

=============================================================
StarlingX: Reorganize Flock Services Source Code repositories
=============================================================

Storyboard: https://storyboard.openstack.org/#!/story/2004792


StarlingX currently combines source code and build meta-data in the same
directory structure, as we move to support multiple OSes, this structure needs
to be separated and further refined for the benefit of developers in the
community to keep history and source code separated. Simplifying things to a
set of common features also has the dual benefit of making the code structure
easier to understand and contribute to.


Problem description
===================

The current directory structure is based around the build working with CentOS
and the current build tooling. As we add additional OS support, additional
meta-data is needed and should be reorganized into a new structure. We realized
we needed to standardize the structure of our open source projects in order to
reduce the amount of time we spent figuring out how things work. Simplifying
things to a set of common features also had the dual benefit of making it
easier for others to understand and contribute to our projects. There are some
Directory Structures that could open source use as standard [1] [2], right now
we are not using any of that standard directory structure.

This is an initial reorganization of just the StarlingX Flock source a
subsequent specification the “MultiOS Directory Layout” will further define the
layout within the sub-component directory structure.

Use Cases
=========

1. Developers want to apply a change in the CentOS spec file but not to the
Ubuntu rules file

2. Developers want to apply a performance change into CentOS but is not
necessary in Ubuntu.

3. Developers want to change the installation path at the CentOS spec file but
not at the Ubuntu build scripts

Proposed change
===============

Reorganize the existing source code to separate the build meta-data from the
base source code, this would leave the existing stx-<flock items> git repos
with source code and a new stx-flock git (1 git) for the packaging meta-data
for all the flock sub-components.

Example of the new stx-flock directory structure::

 stx-flock/
 ├── fault
 │   ├── fm-api
 │   │   └── centos
 │   │       └── fm-api.spec
 │   ├── fm-common
 │   │   └── centos
 │   │       └── fm-common.spec
 │   ├── fm-mgr
 │   │   └── centos
 │   │       └── fm-mgr.spec
 │   └── fm-restapi
 │       └── centos
 │           └── fm-restapi.spec
 └── nfv
     ├── guest-agent
     │   └── centos
     │       └── guest-agent.spec
     ├── guest-client
     │   └── centos
     │       └── guest-client.spec
     └── guest-comm
         └── centos
             └── guest-comm.spec


With this structure is easy for developers to keep a clean software project in
the current stx-<flock items> repositories and the capability to change CFLAGS
or installation paths in stx-flock repository

This structure even gave us the capability to have patches for the flock
services in cases where we want to add an special change of code only for an
specific os, an example of this could be a patch necessary to mitigate compiler
warning/errors generated with  GCC 7 or newer that maybe Ubuntu with
GCC 5 might not have. Instead of detecting the compiler version on the source
code with

::
 #if GCC_VERSION > 30200

We can have a patch that only applies the fixes to the functions in the build
script that we know is using a version of GCC that generates compile errors.

Alternatives
============

- Keep the existing directory structure and add additional sub-directories for
  the new Operating Systems, which will clutter the current repositories.
- Make a split of directories inside each flock service directory, ie::

   stx-fault/
   └── os-packaging/
       ├── fm-api
       │   └── centos
       │   └── ubuntu
       ├── fm-common
       │   └── centos
       │   └── ubuntu
       ├── fm-mgr
       │   └── centos
       │   └── ubuntu
       └── fm-restapi
           └── centos
           └── ubuntu

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

Other Deployer impact
=====================

None

Developer impact
=================

Improve developer experience to isolate each package increasing the modularity
of the development, having delimited the boundaries of each package and how
they are built.

One negative developer impact could be that the changes in packaging source are
no longer atomic, which means in one single commit in the same repo. Proper
dependencies and workflow management will be required to ensure this type of
change goes in at the same time. However, doing a quantitative analysis inside
the flock repositories show us how often a change in metadata for CentOS has
been performed along the history of the development.

- stx-config = 3.800 %
- stx-distcloud = 0 %
- stx-distcloud-client = 0 %
- stx-fault = 10.200 %
- stx-gui = 1.800 %
- stx-ha = 3.300 %
- stx-nfv = 2.300 %
- stx-update = 22.000 %
- stx-metal = 6.100 %

These numbers show us that stx-update might be the only one with more than
20% of changes related to metadata, which means that most of the changes
are for pure Flock source code.

Upgrade impact
===============

None

Implementation
==============

- Create repository stx-flock (1 day)
- Copy necessary meta data from stx-<flock items> to stx-flock repository
  This in order to do not break the current build system (2 days)

Assignee(s)
===========

Primary assignee:
    - Victor Rodriguez

Other contributors:

Repos Impacted
==============

- https://git.starlingx.io/cgit/stx-clients
- https://git.starlingx.io/cgit/stx-config
- https://git.starlingx.io/cgit/stx-distcloud
- https://git.starlingx.io/cgit/stx-distcloud-client
- https://git.starlingx.io/cgit/stx-fault
- https://git.starlingx.io/cgit/stx-gui
- https://git.starlingx.io/cgit/stx-ha
- https://git.starlingx.io/cgit/stx-nfv
- https://git.starlingx.io/cgit/stx-update

Work Items
===========

The following items propose an estimated timeline, numbers are not exact:

- Create repository stx-flock (1 day)
- Copy necessary meta data from stx-<flock items> to stx-flock repository
  This in order to do not break the current build system (2 days)
- If a new build system for multiOS is created this should be using the
  stx-flock repository metadata to build the flock services ( 5 days )
- Adjust the current build system to use the new stx-flock repository, doing
  the development in a devel branch until tested ( 3 days )
- Test build management repositories in the package build system, if
  functionality is tested, merge into master
- When new MultiOs build system is complete, migrate to just use the MultiOS
  build system to avoid duplication of work on build systems

Dependencies
============


Testing
=======

After building a proper image with the reorg of the repositories we can:

- Test build management repositories can generate current RPMs
- Build an STX image
- Run sanity tests for generated image

Documentation Impact
====================

Create a section for developer guide, that guide them how to do a proper
development contribution to the project, a good example of this could be:

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
   * - 2019.05
     - Introduced
