
Documentation re-org and user documentation framework
=====================================================

This specification proposes changes to the structure of the StarlingX
documentation to make it easier for new contributors and users to find
the information they need.  We are also proposing a number of new
documents to cover gaps in the current documentation set.  This spec
defines the proposed new structure of the stx.docs repo and includes
links to the Storyboard entries for the new documents proposed.  The
new documents needed have been prioritized by the Docs team to allow for
the implementation to occur over time.

Story board entries for the numerous SB entries proposed here will
be created once the document is close to approval.

Problem description
===================

The StarlingX documentation structure can be improved to provide more
information for new users and contributors, as well as providing additional
reference information for more experienced users.  It can also be
improved to make it easier for users and contributors to navigate the
document hierarchy so they can find what they need.

Use Cases
=========

1. As a new user, I would like to learn the basics about StarlingX
so I can plan and complete an evaluation of the software.
2. As a user, I would like to learn how to deploy, install and run
StarlingX, so I can successfully deploy Starlingx for my Edge Cloud
applications.
3. As a user administering a StarlingX edge cloud deployment, I would like
to learn how to use all of the features of StarlingX and have all of
the API and CLI interfaces documented, so I can manage my
StarlingX Edge Cloud.
4. As a new contributor, I would like to learn about how to contribute
to the StarlingX project, so that my contributions are accepted.
5. As a new contributor, I would like to learn about the process
for accessing the StarlingX source code and building it from source, so
I can make changes and test them.
6. As a contributor, I would like to learn about the architecture of
StarlingX and the project’s development process, so I can contribute to
the project more effectively.

Proposed Change
===============

This section describes the changes to be made to the documentation.  The
changes are broken down into multiple stories each of which has a
priority as proposed by the Docs team.

New landing page
----------------

We propose to change the StarlingX documentation hierarchy on
docs.starlingx.io from this:

- Installation Guide
- Developer Guide
- Project Specifications
- REST API Reference
- Release Notes
- Contribute

To this:

- Introduction to StarlingX
- Deployment Guides
- Installation Guides
- Operation Guides
- Contributor Guides
- Releases and Release Notes
- Internals

Each of the new sections in the document hierarchy is described below.
Note that we are separating Deployment from Installation.  We define
Deployment as all the steps needed starting from empty racks in the
data center to a running StarlingX instance.  Installation is defined
as installing the software and is a subset of Deployment.  Both sets
of documents will leverage content from the existing StarlingX documentation.

SB link for the restructuring: TBD
Priority: VH - release gating

Introduction to StarlingX
-------------------------

This new page in the document tree is a landing page is intented
to be a Quick Start for people who are new to StarlingX.  It contains
links to the following documents:

- StarlingX Project Introduction
- StarlingX for Kubernetes Users
- StarlingX for OpenStack Users
- StarlingX Background Information
- StarlingX in a Box
- StarlingX Evaluation Guide
- Roadmap to StarlingX Documentation

SB entry: TBD
Priority: VH - Release gating

The following documents are included in this section of the documentation.

**StarlingX Project Introduction (NEW)**
This document should start with a project overview wrt objectives,
capabilities, basic approaches.  Introduce the concept of base
Kubernetes Cluster + optional OpenStack Cloud Application.  Introduce
the various use cases for StarlingX and the available deployment
options.  The document should include a link to the Release page,
noting that pre-built and tested images are available.
SB entry: TBD
Priority: VH - release gating

**StarlingX for Kubernetes Users (NEW)**
This document should describe the Kubernetes solution installed
and supported by StarlingX; e.g. what Kubernetes services are
included, what networking options, what container runtime
options, what storage solutions and the local registry. It should
highlight key similarities and differences between StarlingX
and standard Kubernetes.
SB entry: TBD
Priority: H

**StarlingX for OpenStack Users (NEW)**
This document shall describe the OpenStack solution installed and
supported by StarlingX; e.g. what OpenStack Services are supported,
what Neutron backends are supported and what storage backends are
supported. This document shall highlight key similarities
and differences between StarlingX and standard OpenStack.
SB entry: TBD
Priority: H

**StarlingX Background Information (NEW)**
This document shall contain a sanitized and updated version of
Abraham’s internal New Hire spreadsheet.  It contains links to
many other docs, web pages and blogs that provide context
for StarlingX.
SB entry: TBD
Priority: M

**StarlingX in a Box (NEW)**
This doc will describe how to boot and run
“StarlingX in a Box” (SIAB).  Once we actually create
it (SB 2004811).  The document should take users through
the process of bringing SIAB up and show them some basic commands
such as create VM, load image into VM, start/stop VM and other
similar commands.   The
challenge will be to not over document things documented elsewhere
or familiar to experienced OpenStack or K8S users, while
guiding less experienced users to performing real world tasks
in StarlingX.
SB entry: TBD
Priority: VH - release gating

**StarlingX Evaluation Guide (NEW)**
This is a new document describing how we expect end users who are
new to StarlingX to evaluate the software.  It should describe
basic evaluation use cases and how to implement them in
StarlingX.  It should call out the open source functional and
performance test suites (once they exist) and how to run them.
The document’s main goal is to show users who are "kicking the tires"
of StarlingX how to use the key
differentiating features of StarlingX and guide them to a
very positive evaluation.
SB entry: TBD
Priority: M

**Roadmap to the StarlingX Documentation (New)**
This new document provides the reader with a brief overview of
the entire documentation set.  It could be based on use cases
listed above  e.g. “if you are a dev looking to contribute, you
should read X, Y and Z.  If you are an operator planning a
deployment read A & B.".  The contents of this spec itself
may be a good starting place for this document.
SB entry: TBD
Priority: H

Deployment Guides
------------------

This is a new landing page in the document hierarchy.  It contains
links to the following documents:

- StarlingX Deployment Planning
- StarlingX Deployment Options
- AIO-Simplex Deployment Guide
- AIO-Duplex Deployment Guide
- AIO-Duplex with Computes Deployment Guide
- Small Standard Deployment Guide
- Standard Deployment Guide
- Standard with Ironic Deployment Guide
- Multi-Region Deployment Guide
- Distributed Cloud Deployment Guide

SB entry: TBD
Priority: VH - release gating

The following documents are included in this section of the documentation.

**StarlingX Deployment Planning (New)**
This is a new document for how to plan a deployment of StarlingX.
Needs to include references to the Deployment Options (or maybe
just include it).  Discuss why, how and when the various deployment
options should be used.  More focused on how to define what
hardware to buy and how to cable it up.  THe existing HW
requirements documents would go here.
SB entry: TBD
Priority: VH - release gating

**StarlingX Deployment Options (New)**
This is a new document that describes at a high level the different
ways that StarlingX can be deployed.  It describes each option at
a high level.
SB entry: TBD
Priority: VH - release gating

**AIO-Simplex Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the All-in-one Simplex configuration.
SB entry: TBD
Priority: VH - release gating

**AIO-Duplex Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the All-in-one Duplex configuration.
SB entry: TBD
Priority: VH - release gating

**AIO-Duplex with Computes Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the Duplex with Compute nodes configuration.  Optionally, at the
discretion of the author of the AID-Duplex Deployment Guide, this
could be just an additional section of that document.
SB entry: TBD
Priority: VH - release gating

**Small Standard Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the Small Standard (no storage) configuration.
SB entry: TBD
Priority: VH - release gating

**Standard Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the Standard (with storage nodes) configuration.
SB entry: TBD
Priority: VH - release gating

**Standard with Ironic Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the Standard configuration with OpenStack Ironic to allow use of
bare metal Compute nodes.  This is basically just the existing
how-to doc on Ironic, updated to deploy it in a Container.
SB entry: TBD
Priority: M

**Multi-Region Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the Multi-Region configuration.
SB entry: TBD
Priority: VH - release gating

**Distributed Cloud Deployment Guide (New)**
This is a new document that describes how to deploy StarlingX in
the Distributed Cloud configuration.
SB entry: TBD
Priority: VH - release gating

Installation Guides
-------------------

This is a new landing page in the document hierarchy.  It contains
links to the following documents:

- AIO-Simplex Installation Guide
- AIO-Duplex Installation Guide
- AIO-Duplex with Computes Installation Guide
- Small Standard Installation Guide
- Standard Installation Guide
- Multi-Region Installation Guide
- Distributed Cloud Installation Guide
- Additional OpenStack Services Installation Guide

SB entry: TBD
Priority: VH - release gating

The following documents are included in this section of the documentation.

**AIO-Simplex Installation Guide (New)**
This is a new document that describes how to install StarlingX in
the All-in-one Simplex configuration.
SB entry: TBD
Priority: VH - release gating

**AIO-Duplex Installation Guide (New)**
This is a new document that describes how to install StarlingX in
the All-in-one Duplex configuration.
SB entry: TBD
Priority: VH - release gating

**AIO-Duplex with Computes Installation Guide (New)**
This is a new document that describes how to install StarlingX
in the Duplex with Compute nodes configuration.
SB entry: TBD
Priority: VH - release gating

**Small Standard Installation Guide (New)**
This is a new document that describes how to install StarlingX
in the Small Standard (no storage) configuration.
SB entry: TBD
Priority: VH - release gating

**Standard Installation Guide (New)**
This is a new document that describes how to install StarlingX
in the Standard (with storage nodes) configuration.
SB entry: TBD
Priority: VH - release gating

**Multi-Region Installation Guide (New)**
This is a new document that describes how to install StarlingX
in the Multi-Region configuration.
SB entry: TBD
Priority: VH - release gating

**Distributed Cloud Installation Guide (New)**
This is a new document that describes how to install StarlingX
in the Distributed Cloud configuration.
SB entry: TBD
Priority: VH - release gating

**Additional OpenStack Services Installation Guide (New)**
This is a new document that describes how to install and configure
additional OpenStack services (beyond those supported by StarlingX)
in a StarlingX deployment.  Example services include Octavia,
Trove and Sahara, all of which have been mentioned in the
community as of interest.
SB entry: TBD
Priority: L

Operation Guides
----------------

This is a new landing page in the document hierarchy.  It is intended to
serve as the home page for “how to” documents and user/operator focused
documentation.  The page should contain links to the following documents:

- StarlingX API Reference
- StarlingX CLI Reference
- StarlingX Provider Network Configuration
- StarlingX CEPH Storage Configuration
- StarlingX SDN Networking
- StarlingX Kubernetes Cluster Guide
- StarlingX SWIFT Configuration and Management
- StarlingX Fault Management
- StarlingX Patching Guide
- StarlingX Upgrade Guide

SB entry: TBD
Priority: VH - release gating

**StarlingX API Reference**
This is the existing API Reference documentation.

**StarlingX CLI Reference (New)**
This is a new document the defines all of the CLI commands
accepted by the StarlingX unique services (the Flock).
SB entry: TBD
Priority: M

**StarlingX Provider Network Configuration (New)**
This is a new document for how to configure the provider network.
SB entry: TBD
Priority: M

**StarlingX CEPH Storage Configuration (New)**
This is a new document for how to configure CEPH
SB entry: TBD
Priority: M

**StarlingX SDN Networking (New)**
This is a new document for how to configure SDN networking.
SB entry: TBD
Priority: L

**StarlingX Kubernetes Cluster Guide (New)**
This is a new document for how to operate the Kubernetes
within StarlingX.
SB entry: TBD
Priority: M

**StarlingX SWIFT Configuration and Management (New)**
This is a new document describing how to configure and use
SWIFT within StarlingX.
SB Entry: TBD
Priority: M

**StarlingX Fault Management (New)**
This is a new document describing the fault management
capabilities of StarlingX and how to use them, how to find and
read logs, etc…
SB entry: TBD
Priority: M (H?)

**StarlingX Patching Guide (New)**
This is a new document describing the software patching
capabilities of StarlingX and how to use them.
SB entry: TBD
Priority: L

**StarlingX Upgrade Guide (New)**
This is a new document describing the software upgrade
capabilities of StarlingX and how to use them.
SB entry: TBD
Priority: L

Contributor Guides
------------------

This is a new landing page in the document hierarchy.  It is intended
to serve as the home page for “how to” documents and user/operator
focused documentation.  The page should contain links to the
following documents:
- StarlingX Contributor Guide
- StarlingX Development Process
- StarlingX Build Guide
- StarlingX API Contributor Guide
- StarlingX Release Notes Contributor Guide
- StarlingX Documentation Contributor Guide

**StarlingX Contributor Guide (New)**
This is a new document providing a high level overview of how
to contribute to StarlingX.  It should describe the
communication channels that are used by the project team, the
way we have divided up the project into sub-projects, our
wiki page, our weekly community and sub-project meetings, and
other similar topics.  It should point to the build and
installation documents and describe our expectations for
pre-commit testing needed before changes can be accepted.  It
should point to the project's formal Governance docuemnts
and describe the roles of the TSC members and Core Reviewers
in reviewing and approving code changes.
SB entry: TBD
Priority: H

**StarlingX Development Process (New)**
This is a new document that can leverage existing content from
the wiki.  The document should cover the basic tools used
(git / gerrit / etc…), the feature development and spec
approval process, the bug resolution process, our release
planning process and other similar topics.
SB entry: TBD
Priority: H

**StarlingX Build Guide**
This is the existing Build documentation, updated as needed to
fit within the new hierarchy and for the Containers changes.
SB entry: TBD
Priority: H

**StarlingX API Contributor Guide**
This is the existing API Contributor Guide

**StarlingX Release Notes Contributor Guide**
This is the existing Release Notes Contributor Guide

**StarlingX Documentation Contributor Guide**
This is the existing Documentation Contributor Guide

Releases and Release Notes
--------------------------

This should be a landing page with links to the Cengn images and Release
notes for all releases.  Releases that are no longer supported should
be included (for historical reasons) but should be marked as “obsolete”
or “unsupported”.
SB entry: TBD
Priority: VH - release gating

StarlingX Internals
-------------------

This is a new landing page within the documentation and will contain
links to the following documents:

- How to Navigate the StarlingX Source Code
- StarlingX Architecture Documents
- StarlingX Specifications

SB entry: TBD
Priority: VH - release gating

**How to Navigate the StarlingX Source Code (New)**
This is a new document describing the structure, layout and high
level architecture of the StarlingX git repos and source code.
SB entry: TBD
Priority: H

**StarlingX Architecture Documents (New)**
This is a landing page for architecture documents, which do not
yet exist.
SB entry: TBD
Priority: L

**StarlingX Specifications**
This is a link to the existing StarlingX Specifications page.

Alternatives
============

There are many ways to organize the StarlingX document repository.  The
proposal here is the result of multiple discussions, drafts and reviews
within the Docs team.

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

End users should find it significantly easier to deploy and manage StarlingX
Edge Clouds.  New contributors should find it significantly easier to
make contributions to the project.

Performance Impact
==================

None

Other deployer impact
=====================

None

Developer impact
================

Developers will have to write, contribute to and maintain additional
documents.  Since these documents will help them do their jobs, and
hopefully help attract new users and contributors to the project, it’s
worth the effort :)

Upgrade impact
==============

None

Implementation
===============

This work will be implemented as a set of related Storyboard entries, as
called out in the Proposed Change.  Each Story has a priority defined
for it so the work can be managed over time.

Assignee(s)
===========

Members of the Docs team will lead.  Contributions from the broader
community will be needed.

Primary assignee:
=================

Several will be needed.

Other contributors:
===================

Many will be needed.

Repos Impacted
==============

Stx.docs and likely the Flock services repos

Work Items
==========

See the SB entries called out in the Proposed Change

Dependencies
============

None significant

Testing
=======

Testing will be needed to ensure that the documents written accurately
describe the software.

Documentation Impact
====================

Lots :)
