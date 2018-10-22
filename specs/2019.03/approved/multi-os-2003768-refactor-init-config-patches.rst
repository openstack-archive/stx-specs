..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode
  http://creativecommons.org/licenses/by/3.0/legalcode

============================
Refactor init/config patches
============================

Storyboard: https://storyboard.openstack.org/#!/story/2003768

For RPM patches related to adding or changing StarlingX specific systemd
service files, configuration files or other types,
1)if they are not overwritting files already delivered by the upstream RPM,
  refactor and have an specific config rpm that delivers the new file instead
  of original patch.
2)if they need to change files delivered by the upstream RPM, use puppet to
  change it instead of original patch.

Problem description
===================

There are many patches related to adding or changing init script, config file,
systemd service adding and changing. We'd better to do some refactor on these
patches and remove them, so that we can decrease total patch number and use RPM
instead of SRPM for some package.

Use Cases
=========

There are totally 8 cases for all init/config patches.
1)      Add configuration files to system folder.
2)      Add scripts to system folder.
3)      Add service files to /etc/systemd/system/.
4)      Change configuration files with noreplace attribute in system folder
5)      Change files including configuration file, script and service without
        noreplace attribute in system folder.
6)      Change service files in usr/lib/systemd/system/ that will be started
        by SM after power on.
7)      Change service files in usr/lib/systemd/system/ that will be started
        by systemd after power on.
8)      Already uses puppet instead of earlier patch, or paired patches exist
        that we can rework and decrease unnecessary meta patch.

Proposed change
===============

Proposed solutions for all these cases except for case 5/6
1)      Use configuration package
        a) Add a configuration package folder under the folder where the
           original package sit.
           Name package like nfs-utils-config.
        b) Use this configuration package to package configuration files to
           system folder.
        c) Add "requires: original package" in spec file like
           “Requires: nfs-utils” to make sure original package is installed
           before configuration package.
        d) If we need to add files, we can package customized files to system
           folder
        e) If we need to change files with “noreplace”, we can add customized
           files to /usr/share/starlingx/stx.***.conf first, then in %post
           section of the spec, copy it to destination folder as ***.conf
           during initiation installation.
        f) For case 7, services are enabled by systemctl command in spec file.
           We can package customized service file to /etc/systemd/system/,
           like we did for memcached.
        please refer to  https://review.openstack.org/#/c/600868/
                         https://review.openstack.org/#/c/610459/
2)      Use Puppet
        This is the preferred method for updating configuration in existing
        files.
        a) We can do some modifications on existed pp files in
           stx/stx-config/puppet-manifests/src/modules/platform/manifests/
        b) For some basic configuration file change with “noreplace” attribute,
           we can also add a pp file in above folder.
        To use puppet, there are two things need to be care:
        a) Puppet will execute in each boot up, so the configuration file will
           be overwritten. We need make sure the file will not be changed in
           runtime. If the file will be changed in runtime, then we cannot use
           puppet for it.
        b) Puppet will be executed based on node type. Also RPM will be
           installed based on node type. Need to make sure both method align to
           be executed in the same node type.

3)      Rework current patches
        Some META patches includes not only configuration related patch, but
        also src patch or spec change like build flag.
        After removing configuration change part out, we need rework Meta
        patch.
        Propose to use "spec-include-TiS-changes.patch" to include all spec
        change or src related patch adding if there is no corresponding META
        patch.

Proposed solutions for case 5 and 6
1)      Change files including configuration file, script and service without
        "%config(noreplace)” attribute in package spec file.
        Such as systemd, we need to change rule file, tmp.conf, tmp.mount and
        so on.
        If we change this kind of file to customized one without patch, after
        in-service patching on this package, the file will be overwritten to
        original one. How to handle this patching scenario? Any proposal?
        From my point, we can utilize existed in-service patching-script
        mechanism to call in-service script to copy customized file to
        destination folder after patching. Any comment?

2)      Change service files in usr/lib/systemd/system/ that will be started by
        SM after power on.
        Such as net-snmp, we have 3 meta patches related to snmpd.service
        change.
        We cannot do it like we do for case 7.  Can we use patching-script as
        well for this in-service patching?
        If we use puppet, we can only resolve boot-required patching.

Alternatives
============

NA

Data model impact
=================

NA

REST API impact
===============

NA

Security impact
===============

Current solution is just used for refactoring patches and use config package or
puppet to package or change init/config files instead of existed patches.
No obvious security impact.

Other end user impact
=====================

NA

Performance Impact
==================

NA

Other deployer impact
=====================

NA

Developer impact
=================

The target of this feature is separating configuration part apart from source
patch and try the best to decrease the number of patch. We will also get
benefit when we consider multi-OS support on StarlingX.
For new joining developers, when we do some changes that refer to configuration
file, please keep this idea in your mind.

Upgrade impact
===============

NA

Implementation
==============

We have splitted the work to some tasks by SRPM package and planned to get it
done package by package.
General steps is below.
1) Rework existed meta patch and remove the part for configuration that
   we have analyzed.
2) Remove the patch that will not be used anymore.
3) Add configuration RPM package for corresponding package that we are working
   on, or add puppet file or modify existed puppet file to implement the logic
   that we did with patches before.
4) Rebuild and do deployment and related test to see if the change can work and
   meet our expectation.
5) Submit patch and get it reviewed before code merge.

Assignee(s)
===========

Zhipeng Liu will leading the writing of the code.
Shuicheng lin will join the task as well.
Welcome other contributors join.

Primary assignee:
zhipengs

Other contributors:
Shuicheng

Repos Impacted
==============

openstack/stx-integ

Work Items
===========

There are more than 20 tasks created under story 2003768.

Dependencies
============

NA

Testing
=======

Basically, we will do deployment test and related configuration file check on
different node after power on and first reboot to see that if the configuration
file is expected in specific folders.
For configuration file change scenario, we need to do additional patching test
to see that if the configuration file is expected after patching.
For service file, we need to check service status after power on, reboot
or patching.

Documentation Impact
====================

NA

References
==========

NA

History
=======

NA
