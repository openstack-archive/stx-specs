..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Standardize makefiles for multi-os support
==========================================

Storyboard: https://storyboard.openstack.org/#!/story/2004043

The building and installation of some components in the flock are handled
directly from the specfiles. At the beginning, this implementation was
sufficient as only one OS was supoported, however in multi-os context, these
components needs to be decoupled to let different building methods. In this
proposal the intention is to decouple all the components that uses makefiles
and let standard targets[1] like "install" to be used from specfiles, deb files
or other build procedure.


Problem description
===================

Some components in the flock don't have a proper build system, in the current
implementation the building procedure is handled directly by the specfile and
running an isolated building is not possible, this could be problematic
considering the multi-os support. Thinking in a scenario where multiple OS can
be package using these components, the usage of different build methods (rpm or
deb) should be available.

There are different examples of non-standard building, consider this code:

stx-nfv/guest-comm/centos/host-guest-comm.spec#n75

The build target in this specfile is done through a makefile, however the
%install section is managed directly from the specfile using "install"
commands.

Also, in the code below, this component doesn't have a makefile at all and the
whole procedure is done in the specfile:

stx-metal/mtce-storage/centos/cgts-mtce-storage.spec#n32

In another cases, although there is a target to perform the installation is not
an standard way:

stx-fault/fm-common/sources/Makefile#n28


Use Cases
=========

A developer wants to build a StarlingX component for CentOS and Ubuntu:

Having a standard way to build the component, the developer will be able to
create, easily, a rpm or deb package. Not having this implementation will cause
that the rpm and deb will use different instructions to build the component,
resulting in maintenance problems.


Proposed change
===============

Create or modify makefiles to support only standard targets, decoupling the
building and installation from the current CentOS specific specfiles.

Modify the affected specfiles to meet the change in the build and installation
procedure.


Alternatives
============

None

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

Performance impact
==================

None

Other deployer impact
=====================

None

Developer impact
================

Consider a developer working in multi-os support, more specific in enabling the
building of StarlingX packages into different distributions, this work will
reduce the applied effort. Having a decoupled building will ensure that the
process to be similar as any other package maintainer.


Upgrade impact
==============

None


Implementation
==============

Two types of components are considered in this change:

  - Components that have a makefile but it is incomplete or non-standard.
    Typically these components are C or C++ projects.
  - Components that don't have a makefile but the files are installed by the
    specfile. The most common case of these are components that include only
    scripts or configuration files.

The proposal is:

For the components in the types mentioned above, create or modify a makefile
having the following characteristics:

  - The building and installation shall be managed by the makefile.
  - Support of environment variables for installation paths.
  - Only use standard targets. "all", "clean", "install". In some cases change
    "install_non_bb" to "install".

A typical implementation workflow would be that for each item listed in the
"Repos Impacted" section:

  - Identify the building and installation procedure.
  - Create or modify the makefile to use only standard targets. Consider
    variables to configure installation directories, version or other required
    settings.
  - Verify that the specfile uses the makefile without additional steps.
  - Build the package with the new specfile.
  - Send one review per component changed.


The support for "dist" target is out of this proposal. That change will require
it's own spec.


Assignee(s)
===========

Erich Cordoba <ericho>


Repos Impacted
==============

The repositories and components affected are:

  - stx-ha/service-mgmt/sm-1.0.0
  - stx-ha/service-mgmt/sm-db-1.0.0
  - stx-ha/service-mgmt/sm-common-1.0.0
  - stx-fault/fm-common
  - stx-fault/fm-mgr
  - stx-fault/snmp-ext
  - stx-fault/snmp-audittrail
  - stx-config/storageconfig
  - stx-config/computeconfig
  - stx-config/puppet-manifests
  - stx-config/compute-huge
  - stx-config/config-gate
  - stx-config/puppet-modules-wrs
  - stx-clients/remote-clients
  - stx-clients/install-log-server
  - stx-metal/mtce-storage
  - stx-metal/mtce-compute
  - stx-metal/mtce
  - stx-metal/cgts-mtce-control
  - stx-metal/mtce-common
  - stx-nfv/guest-client
  - stx-nfv/guest-agent
  - stx-nfv/mtce-guest
  - stx-nfv/guest-comm


Work items
==========

  - https://storyboard.openstack.org/#!/story/2004011
  - https://storyboard.openstack.org/#!/story/2004012
  - https://storyboard.openstack.org/#!/story/2004013


Dependencies
============

None


Testing
=======

This change should be transparent for the common developer, so no changes
should be noticed after this implementation. The proposed testing is:

  - To have a full build: The build procedure shall be transparent for the
    developer.
  - A full sanity test cycle should pass without issues.


Documentation Impact
====================

None


References
==========

  - [1] https://www.gnu.org/prep/standards/html_node/Standard-Targets.html
