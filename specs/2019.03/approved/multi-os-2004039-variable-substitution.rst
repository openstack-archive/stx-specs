..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

===============================
StarlingX: Variable Subsitution
===============================

Storyboard:
https://storyboard.openstack.org/#!/story/list?status=active&project_group_id=86

The Multi-OS effort requires that we can modify the locations that somes
files might be found during installation and later for configuration. While
the Linux Standard Base (LSB) includes the Filesystem Hierarchy Standard
(FHS) not all Linux distrubutions follow it exactly, therefore there can be
difference.

Problem description
===================

Many scripts and configuration files hardcode certain paths and filenames,
depending on the Linux distribution these may not be accurate from one
distribution to the next. By providing a mechanism for variable substitution
during installation and update this problem can be addressed.

Use Cases
=========

This is for the Developer and system level engineer working on implementing
changes to support OS independance in the StarlingX service (Flock) and 
system level configuration files contained within StarlingX stx repos.

Proposed change
===============

Using the existing naming conventions, change the existing hardcode paths
to the common naming convention and then do substitution via sed from either
a Makefile, spec file, or post install script.

Examples:

RPM Variable      Make Variable
%{sysconfdir}     SYSCONFDIR
%{buildroot}      DESTDIR


Posting to get preliminary feedback on the scope of this spec.

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

Performance Impact
==================

None

Other deployer impact
=====================

No additiona configuration options are being added, but exisiting ones may
change based on how the variables are substituted.

Developer impact
=================

Developers will need to be aware of the variables and how they get substitued
and add

Upgrade impact
===============

None


Implementation
==============

Assignee(s)
===========

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  Shuicheng Lin (shuicheng)

Other contributors:
  Mingyuan Qi
  Zhipeng Liu
  Saul Wold (sgw-starlingx)


ReposImpacted
==============

stx-clients
stx-config
stx-fault
stx-gui
stx-ha
stx-integ
stx-metal
stx-nfv
stx-update
stx-upstream


Work Items
===========

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.

<TBD> as found

Dependencies
============

None

Testing
=======

Verify configuration is correct before and after the change and test that the
resulting system boots correctly and has the correct configuration.

Documentation Impact
====================

Developer guide will need to be updated to list the canoncial variables and
their default paths.

References
==========

Linux Standard Base: https://wiki.linuxfoundation.org/lsb/start
Hierarchy Filesystem Standard: https://wiki.linuxfoundation.org/lsb/fhs

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Stein
     - Introduced
..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

..
  Many thanks to the OpenStack Nova team for the Example Spec that formed the
  basis for this document.

