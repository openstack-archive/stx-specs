..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

===================================================
StarlingX: Meta-Specification for MultiOS StarlingX
===================================================

Storyboard: https://storyboard.openstack.org/#!/story/2004415

Starling X is growing as an edge cloud solution, cloud operators today have
their favorite operating systems as the basis of their platforms. A recent
analysis of  the most used operating systems on the Compute Cloud (EC2) [1]
shows Ubuntu as the most used OS, in the same list we can find CentOS and Red
Hat Enterprise Linux (RHEL). Starling X is currently based on the CentOS
operating system. To accelerate the adoption of Starling X, multiple OSs must
be supported particularly Ubuntu.

The current build system under STX is focused on CentOS. In order to give full
support for another operating system, we are proposing to develop a new build
model and generate a separation of source code vs build metadata for the STX
flock services. This will initially target the addition of Ubuntu and allow
users to enhance the build with their required distribution in the future.

Problem description
===================

The diversity on the operating systems used at the cloud is not only for the
guest OS, but also for the host OS. The desired host OS may not match the
existing OS base currently used by StarlingX. Among the most used OS,
CentOS/RHEL represents approximately 55% of OpenStack and Ubuntu Server
represents 35% of deployments

Linux distributions have many different approaches to building their Linux
platform. Adding MultiOS support for the STX project will allow STX
developers/users in the community to select the base platform which most
benefits their use case:

- CentOS uses a more conservative approach and uses older versions of packages
  and keep security and stability by backport many fixes and features to their
  current version.

- Ubuntu is more aggressive and uses more recent versions

- Clear Linux OS is an open source, rolling release Linux distribution
  optimized for performance and security for the x86 platforms since a high
  percentage of the data center servers are based on x86 architectures, enable
  Starling X in an x86 optimized OS could benefit the STX users

Currently the StarlingX build system only generates RPMs specifically for
CentOS. The tooling understands how to parse the metadata (srpm_path,
build_srpm) and then patch and re-builds the SRPM, if needed, then the OS and
StarlingX RPM spec-files are parsed and built. There is a dependency generator
that can be used after the first complete build to reduce the build time.
Additionally, there are checks to determine if a package needs to be rebuilt.

The existing build system cannot modify or build other Linux/GNU based
Operating Systems as they use different packaging systems.  Furthermore,
differing versions of packages between OSs may need different patch sets.
Therefore we need a new build system that is able to handle multiple Linux
OSes and packaging systems. The current build system is smart enough to detect
missing dependencies and what packages to rebuild if one package has a change.
This feature should remain on any multi-OS strategy.

STX developers need to have a solid solution for multiple OSes where they want
to build STX solution on either RPMs based or DEBs based OSes. The current STX
build system only supports the build of a distro based on CentOS and RPMs,
this specification is written to create a methodology that covers not only the
CentOS RPMs but also a Ubuntu distro based on deb files.

Use Cases
=========

a) A developer shall be able to apply a bug fix, feature, or security fix for
the same package on multiple operating systems as well as building with
unstaged changes for all the supported OS using the same toolset in their
workstation.

b) The build system shall be able to generate images and installer files for
all supported OS.

c) Operators that want alternate Operating System other than CentOS in both
the host OS and container guest OS.

Proposed change
===============

If this change is completed the end-to-end big picture for the developer/user
will be:

- The user can select the Linux OS used for the host OS

- The user can select the Linux OS used for the containers used in a StarlingX
  deployment

- The selection of the host OS and the container OS is independent

- Proper documentation to the developer documentation as STX Developer’s Guide

In order to achieve these goals, this specification proposes a step-wise
approach with a number of an additional specification that will break down the
steps outlined below:

- Reorganize the STX Flock source


Create a new specification “Flock Source reorganization” to define a split
directory structure that separates the flock source code from the packaging
meta-data. By doing this reorganization the flock source code becomes an
upstream source for the build system. A new stx-flock repo will contain the
packaging meta-data that the build system will consume, just as it does with
the stx-integ and stx-upstream repos.

If we keep the current code structure inside the flock directories, by
increasing the number of supported operating system the number of different
kind of build/installation scripts (Ubuntu rules file  or Centos spec files)
will make hard to keep clean the isolation of packaging scripts from actual
Flock services source code. Many OS distributions separate their meta-data
from the upstream source code.

Also, as the build system handles the versioning for some components, this
spec will consider a proposal to handle this in a different way.

- Define a Multi OS directory layout for the StarlingX upstream packages

A specification “MultiOS Directory Layout” to organize the build management
code for multiple operating systems. This specification would explain how the
OS specific packaging meta-data (spec, rules, …) and patches (if there is a
patch) could be reorganized inside the stx-integ, stx-upstream and the new
stx-flock repos for upstream packages.

The intention is not to split the stx-integ (or similar repos) into many small
repos, but only to refactor if and as necessary to support additional OSes.
The existing structure may change (to be defined in the specification), but be
comparable to the existing repo.

- Dependency Generator for multiple OS

One of the features of the current build system is that if one of the packages
is modified, during the build of that package the build system will identify
what other packages get affected by the rebuild of the first package. This
feature could be labeled as Package Dependency Resolver. The proposed change
is that same Dependency resolver tool be created for multiple OS as a scalable
tool. This specification will spell out how the build time dependencies could
be generated in a package independent fashion.

- Build tools for Multiple OSes

A new set of build tools (for host OS and containers) to support MultiOS would
need to be created in order to handle the different dependency and package
build tools. The new build tools will also have to manage patching upstream
source package as defined by the OS specific tools. The existing build tools
would stay while the development of a new build tool is ongoing. All current
features in the build system shall be preserved in this proposal.

In the case of containers, if we decide to use the same homogeneous OS image
across our container solution, the build tools should be able to generate a
containers images based on the OS supported by STX.

A specification “MultiOS Build Tool” will clarify the details and requirements
to achieve building images and/or container for multiple OSes. The decision of
what tools, scripting language and other implementation details will be
clarified this specification.

- Installation, Configuration and Updating for Multiple OS

The installer and configuration management would need to be addressed as well
as the StarlingX image update tools and process (stx-update), these would need
specifications as appropriate and will be later in the process.

Alternatives
============

A possible alternative is to use Bitbake, a python based tool that parses
recipes files creates a dependency tree and schedules tasks to build an OS. We
could create recipes for the Flock, modified kernel package and modified
userspace packages. By using a sub-set of recipes and the Bitbake fetcher to
get the upstream rpm, SRPM, deb or .tar.gz (as appropriate), one can then
build the packages using the native compiler and tools. Since Bitbake already
contains a dependency generator, task scheduler, and a fetcher it can be used
to generate the binary packages. It can also be used to generate images.

Another option would be to re-use the existing scripts and refactor the tools
to understand additional meta-data for Debian packaging format.

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

In the end, the End user will have:

stx-centos.iso
Stx-ubuntu.iso
stx-clearlinux.iso


Along with containerized images as appropriate.

Performance Impact
==================

None

Other Deployer impact
=====================

We expect there to be changes for the installer based on the different
installation methods used by the different OSes, those differences will be
defined in the Installation specification.

Developer impact
================

Developers would need to understand that the tools and metadata now support
multiple operating systems and the effect that a change they need to make
would mean on those different OSes. They may also have to modify the existing
source to refactor OS-specific requirements such as path, libraries, ...

Upgrade impact
===============

We should ensure that the system is correctly upgradable when installing with
the new installation and upgrading tools such that there will be no impact.

Implementation
==============

Implementation will be the generation of the following additional
specifications:

Flock Source Reorganization
MultiOS Directory Layout
Build Dependency Generator
Build Tool for MultiOS
ISO Generation for MultiOS
Installer for MultiOS
Configuration management
Update management

Assignee(s)
===========


Primary assignee:
   - Victor Rodriguez

Other contributors:
   - Jesus Ornelas
   - Mario Carrillo

Repos Impacted
==============

https://git.starlingx.io/cgit/stx-integ/

Work Items
===========

- Create Specifications listed at Implementation section

Dependencies
============


Testing
=======

Create unit tests for build system

Generate a CI/CD that builds daily an image of each Linux flavor :

- Ubuntu
- CentOS
- Clear Linux

And then run a basic test that proves:

- Boot
- The launch of VMs with OpenStack
- Minimal STX application

Documentation Impact
====================

New documentation will be generated for this multi-OS case

References
==========

[1] https://thecloudmarket.com/stats#/by_platform_definition


History
=======

.. list-table:: Revisions
      :header-rows: 1

   * - Release Name
     - Description
   * - 2019.05
     - Introduced
