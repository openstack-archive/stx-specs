..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

==============================================================
StarlingX: Repo creation for ubuntu based Flock Services rules
==============================================================

Storyboard: https://storyboard.openstack.org/#!/story/2004792


StarlingX currently combines source code and build meta-data in the same
directory structure, as we move to support multiple OSes, this structure needs
to be separated and further refined for the benefit of developers in the
community to keep history and source code separated. Simplifying things to a
set of common features also has the dual benefit of making the code structure
easier to understand and contribute to.


Problem description
===================

By definition an Spec file is a file used for building .RPM (Red Hat Package
Manager) packages, The spec file, defines all the actions the rpmbuild command
should take to build your application, as well as all the actions necessary for
the rpm command to install and remove the application. The same applies for the
Rules files for Ubuntu distributions. Rules file provides the instructions to
compile the code and turns it into a binary package (.DEB).

One of the advantages of using these package managers is that each package
includes metadata that describes the package’s components, version, release,
size, project URL, installation instructions, and so on. that could be different
on each distribution.

At the same time, the packages allow  to take pristine software sources and
package them into source and binary packages for users. In source packages, is
provided the pristine sources along with any patches that were used, plus
complete build instructions. This design eases the maintenance of the packages
as new versions of your software are released.

One of the recommendation the RPM Packaging Guide[3] provides is describe the
Source of the software project in the Source0 Filed on the Spec file. The
Source0 field is where the upstream software’s source code should be able to be
downloaded from. This URL should link directly to the specific version of the
source code release that this RPM Package is packaging.

However in StarlinX we don't follow this recommendation for the flock services,
and we deliver the source code of the flock services as well as the Spec file
to build the source code on CentOS using STX build tooling.

As we add additional OS support (for example Ubuntu), additional meta-data is
needed and should be reorganized into a new structure. We realized we needed to
standardize the structure of our open source projects in order to reduce the
amount of time we spent figuring out how things work. Simplifying things to a
set of common features also had the dual benefit of making it easier for others
to understand and contribute to our projects. There are some Directory
Structures that could open source use as standard [1] [2], right now we are not
using any of that standard directory structure.

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

Most distribution takes the source code of the upstream software projects and
manage it according to their needs. They provide the meta-data that meet the
requirements of their specific distribution, such as paths, configurations and
possibly additional patches. In some cases, upstream software provides some
default packaging meta-data, but the downstream distributions ignore that info
as it does not meet the requirements. Since StarlingX is both a distribution
and Upstream, we are carrying packaging meta-data in stx-integ and stx-upstream
with specfiles that override upstream packages such as mariadb, kubernetes and
tpm. As the StarlingX distribution, we need a better separation of the distro
meta-data from the upstream components, such that the developers of the distro
consume upstream source using more standard tarballs, and the developers of the
flock services and other tools provide the changes directly to the upstream.

Reorganize the existing source code to separate the build meta-data from the
base source code, this would leave the existing stx-<flock items> git repos
with source code and a new stx-package git (1 git) for the packaging meta-data
for all the flock sub-components.

Example of the new stx-package directory structure::

 stx-package/
 ├── fault
 │   ├── fm-api
 │   │   └── ubuntu
 │   │       └── fm-api.dsc
 │   ├── fm-common
 │   │   └── ubuntu
 │   │       └── fm-common.dsc
 │   ├── fm-mgr
 │   │   └── ubuntu
 │   │       └── fm-mgr.dsc
 │   └── fm-restapi
 │       └── ubuntu
 │           └── fm-restapi.dsc
 └── nfv
     ├── guest-agent
     │   └── ubuntu
     │       └── guest-agent.dsc
     ├── guest-client
     │   └── ubuntu
     │       └── guest-client.dsc
     └── guest-comm
         └── ubuntu
             └── guest-comm.dsc


With this structure is easy for developers to keep a clean software project in
the current stx-<flock items> repositories and the capability to change CFLAGS
or installation paths in stx-package repository

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
- Make a source code and build system split of directories inside each flock
  service directory. This will require to update the path in build_srpm.data
  files but it won't require any change in the build system scripts, i.e.::

   stx-fault/
   ├─── os-packaging/
   │    ├── fm-api
   │    │   └── centos
   │    ├── fm-common
   │    │   └── centos
   │    ├── fm-mgr
   │    │   └── centos
   │    └── fm-restapi
   │        └── centos
   ├──── fm-api
   │     └── src
   ├──── fm-common
   │     └── src


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

- Create repository stx-package (1 day)
- Copy necessary meta data from stx-<flock items> to stx-package repository
  This in order to do not break the current build system (2 days)

Assignee(s)
===========

Primary assignee:
    - Victor Rodriguez

Other contributors:

Repos Impacted
==============

None

Work Items
===========

The following items propose an estimated timeline, numbers are not exact:

- Create repository stx-package (1 day)
- Copy necessary meta data from stx-<flock items> to stx-package repository
  This in order to do not break the current build system (2 days)
- If a new build system for multiOS is created this should be using the
  stx-package repository metadata to build the flock services ( 5 days )
- Adjust the current build system to use the new stx-package repository, doing
  the development in a devel branch until tested ( 3 days )
- Test build management repositories in the package build system, if
  functionality is tested, merge into master
- When new MultiOs build system is complete, migrate to just use the MultiOS
  build system to avoid duplication of work on build systems

Dependencies
============

None

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
[3] https://rpm-guide.readthedocs.io/en/latest/rpm-guide.html

History
=======

.. list-table:: Revisions
      :header-rows: 1

   * - Release Name
     - Description
   * - 2019.05
     - Introduced

