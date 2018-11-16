..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

..

===================================================
StarlingX: Host Build Artifacts on StarlingX Mirror
===================================================

https://storyboard.openstack.org/#!/story/2003906

An external mirror hosting build artifacts allows for quicker ramp-up
of community members, and provides for reference artifacts.

This story will implement the building of artifacts on a server and
the population of those artifacts to the existing community-accessible
mirror.


Problem description
===================

A new community member who wishes to evaluate StarlingX currently needs
to download the source code, set up a build environment, and build all
the StarlingX components.  They will then generate their own ISO.  This
process can take a significant amount of time, and lead to a poor initial
impression of StarlingX.

The initial implementation of hosted builds would have a server:
* Build all RPMs used for the build
* Build an installer image based on the produced RPMs
* Build a reference ISO based on the produced RPMs
* Publish RPMs, installer, and reference ISO to the existing mirror

The initial implementation would provide artifacts for the r/2018.10
and the master branch of StarlingX.  Future work would build and host
artifacts for evolving branches at an interval to be determined.


Use Cases
=========

Evaluators of the StarlingX project want to use a prebuilt ISO image to
install and test StarlingX.

Developers want to compare their local changes to a package with a
reference image containing an unmodified version of the package

Producers of an ISO want to use the prebuilt installer image rather than
one they produce themselves.


Proposed change
===============

Provision a server such that it is able to build the StarlingX
project.  Allow for this build to produce artifacts based on either
the master branch, release branches (e.g. r/2018.10), or feature branches
(e.g. f/somefeature).  Building and hosting of other builds (branches or
tags) should be accomodated.  Provide a mechanism for the build to publish its
artifacts (RPMs, ISO, installer) to the existing mirror.  Initial
implementation will use a build server at CENGN.

Periodic building of the master branch, and publishing of the resulting
artifacts via the StarlingX mirror, will be performed.  Other branches
(e.g. r/2018.10) will also be built and published, but only as required.

The publication path for each build shall be unique and will include branch
and UTC time stamp.  An alias for the latest build of the branch shall be
provided.

Initial rate of builds is 1 build of the master branch per day.  This
cadence may be adjusted in the future.  Non-master branch builds shall
be performed on an on-demand basis as determined by the build team.
Non-master branch builds include release builds, and milestone builds.
Milestone builds are to be built periodically, at a cadence to be determined.

The retention period of master branch build shall initially be 14 days.
The retention period may be revisited and adjusted at a later time.
The retention time for release branch builds will be much greater then
master branch builds, possibly indefinitely.  Milestone builds shall
be retained for at least one release period past their introduction.

Hosted artifacts proposed directory structure:

Build artifacts hosted by the mirror shall be available in following
directory structure:

http://mirror.starlingx.cengn.ca/mirror/starlingx/<path>

<path> = <release-identifier>/<distro>/<build-date>/outputs/<output-artifact-type>
         <release-identifier>/<distro>/<build-date>/inputs/<input-artifact-type>

<release-identifier> = 'master'
                       'feature/'<feature-branch-name>
                       'release/'<release-branch-name>
                       'milestone/'<milestone-tag>
                       ...
                       'r2018.10' (legacy symlink to release/2018.10)

<distro> = 'centos'

<feature-branch-name> = 'centos76'
                        ...

<release-branch-name> = '2018.10'
                        ...

<build-date> = $(date +%Y%m%dT%H%M%SZ)   # time stamp of start of 'repo sync'
               <latest_build>   # a symlink to the last successful build date

<input-artifact-type> = RPMS/
                        SRPMS/
                        downloads/

<output-artifact-type> = RPMS/<build-phase>/
                         SRPMS/<build-phase>/
                         iso/
                         installer/
                         wheels/
                         helm-charts/
                         CONTEXT
                         <build-result>
                         logs/

<build-phase> = std
                rt
                installer
                container
                ...

<build-result> = 'success' | 'fail'

Non-hosted artifacts:

Other artifacts produced by the build may be hosted outside of the
mirror.  In particular, container images shall be hosted on DockerHub.

The DockerHub container image path shall be

<image-name>=<org>/<image>:<tag>

<org>=starlingx

<image>=stx-<component>

<tag>=<git-tag-or-branch>-<os>-<openstack-release>[-<qualifier>]

<os>=centos | ubuntu | clear-linux

<openstack-release>=pike | queens | rocky ...

<component>=aodh | ceilometer| cinder | glance | gnocchi | heat | horizon | ironic | keystone | libvirt | magnum | murano | neutron | nova-api-proxy | nova | panko ...

<qualifier>=<timestamp> | latest | stable

<git-tag-or-branch>=dev | r-2018.10.0 | m-2019.05b1 | ...
    (This naming convention is for reference only, and has not been finalized)



Alternatives
============

The existing mechanism of requiring all persons to perform their own
builds is an alternative.

Providing a prebuilt version of StarlingX via a container (rather than
as build artifacts) has not been considered, as it would still depend
on an official build of StarlingX artifacts to install in said
container.


Data model impact
=================

None


REST API impact
===============

None


Security impact
===============

Server administration:
The build server is to be administered by a small group of core
developers from the existing StarlingX Build team.  The server
should be kept up to date with security related OS updates.  The
address and access mechanism of the server are not to be published
to those outside the StarlingX Build team or StarlingX Security
team.

Build initiation:
There shall be a mechanism for triggering builds.  This mechanism
shall support self-initiated builds (periodic, or based on
observed external git commits, etc).  The mechanism shall also
support manually-initiated builds.  The mechanism for manually
initiated builds shall initially be through the administration
interface (discussed above).  Providing other mechanisms to
initialize builds (for example, providing a web interface through
Jenkins, or a similar mechanism) is beyond the scope of this
document, however security concerns must be taken into account
if providing an additional interface.


Other end user impact
=====================

End user will only see a change in StarlingX deployment if they
choose to make use of the pre-published artifacts.  The automated
use of pre-published artifacts (example: having build scripts
use the pre-built installer by default) are potential future
improvements outside the scope of this document.


Performance Impact
==================

No impact to running systems.  Time to obtain and evaluation
copy of StarlingX will be drastically reduced.


Other Deployer Impact
=====================

None


Developer impact
=================

Reference RPMs, Installer image, and ISO can be used to compare code
in development.


Upgrade impact
===============

None


Implementation
==============


Assignee(s)
===========

Primary assignee:
  Scott Little <slittle1>

Other contributors:
  Jason McKenna <jmckenna>
  Don Penney <dpenney>
  Al Bailey <albailey>


Repos Impacted
==============

stx-tools - scripts to assist in publication of build outputs


Work Items
===========

* Setup base metal (CENGN) Server to perform builds
* Create mechanism for server to perform periodic build of master branch
* Create mechanism for server to perform build of designated release branch
* Create script or mechanism to publish builds to existing mirror


Dependencies
============

None


Testing
=======

Basic installation and load sanity testing of produced iso.


Documentation Impact
====================

Documentation for the artifacts will be posted on the StarlingX wiki


References
==========

https://www.cengn.ca/
http://mirror.starlingx.cengn.ca/mirror/


History
=======

