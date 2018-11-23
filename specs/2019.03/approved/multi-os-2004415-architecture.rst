..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

================================================
StarlingX: Build system architecture for multiOS
================================================

Storyboard: https://storyboard.openstack.org/#!/story/2004415

Starling X is based on CentOS operating system, however, we would like to
support other operating systems such as Ubuntu or Clear. The current build
system under STX will not support the transition to other OS based on rpm or
deb files. In order to give a full support for other operating systems it is
necessary to refactor the current build model and generate an separation of
source code vs build metadata for the STX flock services, patches, and build
tools.


Problem description
===================

Currently the StarlingX build system only generates RPMs specifically for
CentOS7. The tooling understands how to parse the metadata (srpm_path,
build_srpm) and then patch and re-builds the SRPM, if needed, then the OS and
StarlingX RPM spec-files are parsed and built. There is a dependency generator
that can be used after the first complete build to reduce the build time.
Additionally there are checks to determine if a package needs to be rebuilt.

The existing build system can not modify or build other Linux/GNU based
Operating Systems as they use different packing systems or versions of packages
that are getting patches applied (that may not apply to different version).

STX build tools currently generate currently only RPMs for CentOS operating
system due to two main reasons:

- STX RPMs cannot be installed on Ubuntu/Debian

In Ubuntu Linux, installation of software can be done on using  Ubuntu Software
Center, Synaptic package manager or apt-get command line mode.  Current STX
solution with YUM or rpm install does not work. Ubuntu documentation recommends
to use Alien ( described in Alternatives section) tool to transfer RPMs to DEB
files, however, it can lead to dependency issues or runtime crashes

- STX RPMs has hardcoded runtime and build requirements for CentOS

CentOS RPM's can have dependencies on package names and versions of software
that may not match what is contained in other distributions such as Ubuntu or
Clear. Additionally RPM spec-files may contain commands (pre/post scripts) that
are not available in other distributions.

The current build system is smart enough to detect missing dependencies and
what packages to rebuild if one package has a change. This feature should
remain on any multi OS strategy.

STX developers need to have a solid solution for multiple OSes where they want
to deploy STX solution on either RPMs based or DEBs based OSes. The current STX
build system only supports the build of a distro based on CentOS and RPMs, this
specification is written to create a methodology that cover not only CentOS
RPMs but also Ubuntu base distros based on deb files.

Use Cases
=========

a) Developers need to generate an image based on alternate OSes for some End
User client or Deployer.

b) If developers need to apply a bug fix, feature, or security fix for the
same package on multiple operating systems.

c) Operators how want alternate Operating System other than CentOS in both the
host OS and container guest OS.


Proposed change
===============

This specification proposes a step-wise approach with a number of additional
specification that will break down the steps outlined below.

Reorganize the STX Flock source, a specification will be created to detail the
implementation. The source and build specific metadata should be separated to
allow for better workflow, this would include creating gits for each flock
service sub-component, adding appropriate infrastructure tooling to these
sub-components, such as autotools. Autotools provides a mechanism to generate
OS specific makefiles, setup.py based on templates and ensures the correct
buildtime dependencies are in place. The "Source Reorg" specification will
detail the proposed directory layout and tools and build targets. Initially,
the StarlingX flock could be built manually and installed based on this new
layout.

The next specification would the "Dependency Generator" specification, which
would spell out how the dependencies could be generated for multiple packaging
formats or in a package independent fashion.

The existing build tools would also need to be modified to support the new
directory layout, dependency generation and have different packaging support.
This will also require a specification.

The installer and configuration would need to be addressed as well as the
updater process, these would need specification as appropriate and will be
later in the process.


Alternatives
============

A possible alternative is to use Bitbake and create recipes for the Flock,
modified kernel package and modified userspace packages. By using a sub-set of
recipes and the Bitbake fetcher to get the upstream rpm, SRPM, deb or .tar.gz
(as appropriate), one can then build the packages using the native compiler
and tools. Since Bitbake already contains a dependency generator, task
scheduler and a fetcher it can be used to generate the binary packages. It can
also be used to generate ISOs.

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
stx-ubuntu.iso
stx-clearlinux.iso


Performance Impact
==================

None

Other Deployer impact
=====================

None

Developer impact
=================

Developers would need to understand that the tools and metadata now support
multiple operating systems and the effect that a change they need to make would
mean on those different OSes.

Upgrade impact
===============

None

Implementation
==============

Implementation will be the generation of the following additional
specifications:

Source Reorg
Dependency Generator
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

- Create Specifications!

Dependencies
============


Testing
=======

Generate a CI/CD  that builds daily an image of each Linux flavor :

- Ubuntu
- CentOS
- Clear Linux

And then run a basic test that proves:

- Boot
- Launch of VMs with Open Stack
- Minimal STX application

Documentation Impact
====================

New documentation will be generated for this multi-OS case

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
