..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

..
  Many thanks to the OpenStack Nova team for the Example Spec that formed the
  basis for this document.

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

Provision a server at CENGN such that it is able to build the StarlingX
project.  Allow for this build to produce artifacts based on either
the master branch, or release branches (e.g. 2018.10).  Provide a mechanism
for the build to publish its artifacts (RPMs, ISO, installer) to the
existing mirror hosted by CENGN.

Periodic building of the master branch, and publishing of the resulting
artifacts via the StarlingX mirror, will be performed.  Release branches
(e.g. 2018.10) will also be built and published, but only as required.

The publication path for each build shall be unique and will include branch
and UTC time stamp.  An alias for the latest build of the branch shall be
provided.

Rate of builds is TBD based on demand and performance.

The exact retention period of builds is TBD.  However the retention time for
release branch builds will be much greater then master branch builds.

Proposed directory structure:

http://mirror.starlingx.cengn.ca/mirror/starlingx/<path>

<path> = <release>/<distro>/<build-date>/outputs/<output-artifact-type>
         <release>/<distro>/<build-date>/<release>/inputs/<input-artifact-type>

<distro> = 'centos'

<release> = 'master'
            <release-tag>

<release-tag> = 2018.10.0
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
                         CONTEXT
                         <container-artifacts>
                         <build-result>
                         logs/

<build-phase> = std
                rt
                installer
                container
                ...

<container-artifacts> = TBD

<build-result> = 'success' | 'fail'


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
developers from the existent StarlingX Build team.  The server
should be kept up to date with security related OS updates.  The
address and access mechanism of the server are not to be published
to those outside the StarlingX Build team or StarlingX Security
team.

Build initiation:
There shall be a mechanism for triggering builds.  This mechanism
shall support self-initiated builds (periodic, or based on
observed external git commits, etc).  The mechanism shall also
support manually-initiated builds.  The mechanism for manually
initially builds shall initially be through the administration
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

