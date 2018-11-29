..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

======================================
StarlingX: Multi OS Linux builder tool
======================================

Storyboard: https://storyboard.openstack.org/#!/story/2004471


Starling X is based on CentOS operating system, however, is necessary to give
support of the product in other base OS layers. The current build system under
STX will not support the transition to other OS based on rpm or deb files. In
order to give a full support for other operating systems is necessary to create
a tool that makes linux images based on deb files and in rpms from other Linux
distributions ( like fedora )


Problem description
===================

STX image creator only supports CentOS based images, users that want to
generate a different linux image like ubuntu or fedora , will not be able to
generate the .iso with current tools

Use Cases
=========

For users that want to install STX solution under Ubuntu base OS or diferent
rpm base OS like fedora


Proposed change
===============

Linuxbuilder helps stx developers  to create a Linux image based on multiple OS
flavors . This project is intended for anyone that wants to build a Linux Image
(img/iso/kvm/cloud) based on RPMs and DEBs.

We create this project to avoid the process of develop your own tools/scripts
for the creation of your customized distribution, this project provide the
tools to do that with a simple command line (Make).

POC of the project with README and examples:

https://github.com/VictorRodriguez/linuxbuilder

Linuxbuilder helps to create a Linux image based on multiple OS flavors
(Centos,ClearLinux,Fedora,debian). This project is intented for anyone that
wants to build a Linux Image (img/iso/kvm/cloud) based on RPMs or DEBs.

As a POC for now the tool create an image based on an standard ISO:

$ curl -O http://releases.ubuntu.com/16.04/ubuntu-16.04.5-server-amd64.iso
$ make iso-ubuntu

Or

$ curl -O
http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1804.iso
$ make iso-centos

Developers can put their own packages in the RPMS/ DEBS/ directory, After that
the tool will overwrite the packages with ones provided by the developers. This
functionality could be extended if POC is approved to create an ISO image from
scratch instead of using as template to install/overwrite the necesary packages

The tool was develop and tested in a Clear Linux development machine, for now
the requirements and dependencies match only for Clear Linux , in the future if
POC is approved the tool shoudl be able to run in multiple operating systems
such as CentOS,Fedora and Ubuntu.

Alternatives
============

Linaro provides a suite of tools used for creating images (only for debs). The
tools create Ubuntu and Android images. The most commonly used tools are
linaro-media-create and linaro-android-media-create.

https://github.com/tobetter/linaro-image-tools


This package offers a set of tools for use with Linaro images.

However, the provided linaro-media-create command only allows writing Linaro
images to a SD card, or generating an image file which you can boot in QEMU.

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

None, the implementation will be in parallel to the current build system and
will be available for the community to be evaluated and used if needed. In the
end, the developers will have be able to generate multi OS images:

stx-centos.iso
stx-ubuntu.iso
stx-clearlinux.iso


Performance Impact
==================

None

Other deployer impact
=====================

None

Developer impact
=================

Extend functionality of STX developers to ubuntu base systems and other rpm
base OSs to take advantage of these other OSs

Upgrade impact
===============

None

Implementation
==============

Implementation will be in parallel to the current build system and will be
available for the community to be evaluated and used if needed

Assignee(s)
===========

Primary assignee:

   - Jesus Ornelas
   - Victor Rodriguez

Other contributors:

Repos Impacted
==============

https://git.starlingx.io/cgit/stx-root/

Work Items
===========


Dependencies
============

The RPMs or DEBs need to be pre generated. For the second phace of the tool we
can add a multiOS package builder like, described in:

"How to build a package" section of the readme at:

https://github.com/VictorRodriguez/linuxbuilder#how-to-build-a-package


Testing
=======

Generate a CI/CD  that builds daily an image of each Linux flavor :

- Ubuntu
- Centos
- Clear Linux

And then run a basic test that proves:

- Boot
- Lauch of VMs with Open Stack
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
   * - Stein
     - Introduced
