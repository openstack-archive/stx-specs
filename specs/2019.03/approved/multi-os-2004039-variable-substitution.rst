..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

===============================================
StarlingX: Variable Subsitution of System Paths
===============================================

Storyboard:
https://storyboard.openstack.org/#!/story/2004039

The Multi-OS effort requires that we can modify the locations that some
files might be found during installation and later for configuration. While
the Linux Standard Base (LSB) includes the Filesystem Hierarchy Standard
(FHS) not all Linux distributions follow it exactly, therefore there can be
differences.

Problem description
===================

Many scripts and configuration files hardcode certain paths and filenames,
depending on the Linux distribution these may not be accurate from one
distribution to the next. By providing a mechanism for variable substitution
during installation and update, this problem can be addressed.

Use Cases
=========

This is for the Developer and system level engineer working on implementing
changes to support OS independence in the StarlingX service (Flock) and
system level configuration files contained within StarlingX stx repos.

Proposed change
===============

Using the existing naming conventions, change the existing hardcode paths
to the common naming convention and then do substitution via sed or other
tool as appropriate. This renaming could occur in any source or configuration
file, not limited to Makefiles, .ini, .config, or scripts.

Examples:

RPM Variable      Make Variable  Substitution Variable
%{_sysconfdir}     SYSCONFDIR       @SYSCONFDIR@
%{_buildroot}      DESTDIR          @DESTDIR@
%{pythonroot}      PYTHONROOT       @PYTHONROOT@

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

No additional configuration options are being added, but existing ones may
change based on how the variables are substituted.

Developer impact
=================

Developers will need to be aware of the variables and how they get substitued
and added. This can be addressed in the Developer Guide Wiki.

Upgrade impact
===============

None


Implementation
==============

Assignee(s)
===========

Primary assignee:
  Shuicheng Lin (shuicheng)

Other contributors:
  Mingyuan Qi
  Zhipeng Liu
  Saul Wold (sgw-starlingx)


Repos Impacted
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

Initial work:
 * https://storyboard.openstack.org/#!/story/2004039
   - Task: 27043 stx-nfv: contains hardcoded path to /etc and
     /usr/lib64/python2.7

Ongoing Discovery
 * Create additional tasks in the 2004039 Story as we discover additional
   hardcoded paths that need substitution.
 * Ensure that the StarlingX services are using the correct LSB/FHS directory
   structure as implemented by the upstream distribution (this can still vary
   slightly).

Dependencies
============

None

Testing
=======

Verify configuration is correct before and after the change and test that the
resulting system boots correctly and has the correct configuration.

Documentation Impact
====================

Developer Guide Wiki will need to be updated to list the canonical variables and
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
   * - 2019.03
     - Introduced
