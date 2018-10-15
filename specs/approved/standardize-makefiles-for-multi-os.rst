..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Standardize makefiles for multi-os support
==========================================

Storyboard: https://storyboard.openstack.org/#!/story/2004043

The building and installation of some components in the flock are handled
directly from the specfiles. At the beginning, this implementation was sufficient
as only one OS was supoported, however in multi-os context, these components
needs to be decoupled to let different building methods. In this proposal the
intention is to decouple all the components that uses makefiles and let standard
targets[1] like "install" to be used from specfiles, deb files or other build
procedure.


Problem description
===================

Some components in the flock doesn't has a proper build system, in the current
implementation the building procedure is handled directly by the specfile and
running an isolated building is not possible, this could be problematic
considering the multi-os support. Thinking in a scenario where multiple OS can
be build using this components, the usage of different build methods (rpm or deb)
should be available.

There are different examples of non-standard building, consider this code:

http://git.starlingx.io/cgit/stx-nfv/tree/guest-comm/centos/host-guest-comm.spec#n75

The build target in this specfile is done through a makefile, however the
%install section is managed directly from the specfile using "install" commands.

Also, in the code below, this component doesn't have a makefile at all and the
whole procedure is done in the specfile:

http://git.starlingx.io/cgit/stx-metal/tree/mtce-storage/centos/cgts-mtce-storage.spec#n32

In another cases, although there is a target to perform the installation is not
an standard way:

http://git.starlingx.io/cgit/stx-fault/tree/fm-common/sources/Makefile#n28


Use Cases
=========

A developer wants to build a StarlingX component for CentOS and Ubuntu:

Having a standard way to build the component, the developer will be able to
create, easily, a rpm or deb package. Not having this implementation will cause
that the rpm and deb will use different instructions to build the component,
resulting in maintenance problems.


Proposed change
===============

Two types of components are considered in this change:

  - Components that has a makefile but it is incomplete or non-standard.
    Typically these components are C or C++ projects.
  - Components that doesn't has a makefile but the files are installed by the
    specfile. The most common case of these are components that includes only
    scripts or configuration files.

The proposal is:

For the components in the types mentioned above, create or modify a makefile
having the following characteristics:

  - The building and installation shall be managed by the makefile.
  - Support of environment variables for installation paths.
  - Only use standard targets. "all", "clean", "install". In some cases change
    "install_non_bb" to "install".

The support for "dist" target is out of this proposal. That change will require
it's own spec.


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

<To be filled later>


Assignee(s)
===========

Erich Cordoba <ericho>


Repos Impacted
==============

 - stx-fault
 - stx-ha
 - stx-nfv
 - stx-metal


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

This change should be transparent for the common developer, so no changes should
be noticed after this implementation. Having a full complete build should be
enough.


Documentation Impact
====================

None


References
==========

 - [1] https://www.gnu.org/prep/standards/html_node/Standard-Targets.html
