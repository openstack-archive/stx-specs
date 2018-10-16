..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

=====================
Local Docker Registry
=====================

Storyboard: https://storyboard.openstack.org/#!/story/2002840

A local Docker registry should be included in a StarlingX containerized
deployment to allow users to store Docker images that they might not want
to share publicly through Docker Hub.

Problem Description
===================

Without the local Docker registry, the only way to obtain Docker images is
through public means, such as Docker Hub. Users of StarlingX might have images
that they do not want to publicly share. Docker Hub might also not meet the
users' performance requirements. Deploying a local Docker registry allows the
user to deploy custom images without uploading it to Docker Hub first.

Use Cases
=========

End users and StarlingX administrators can push and deploy images from the
local registry without publicly sharing them on Docker Hub.

Proposed change
===============

An instance of the Docker Distribution project would be integrated on
StarlingX:
https://github.com/docker/distribution

Docker Distribution (Docker Registry) shall be deployed in StarlingX, managed
by STX-HA's SM in active standby mode. The registry process shall be running
as a host process as opposed to a container. Docker Distribution shall use a
resizable DRBD synced file system as its root directory.

The registry will be deployed with token authentication. A token server shall
be deployed as well, and managed by SM, running in active standby mode. The
token server will use Openstack Keystone as a back end, and allow users to log
in using their Openstack credentials to work with Docker images in the
registry. The token server shall provide minimal authorization support where
users shall only be allowed to modify resources belonging to their own repo.
For example, user1 shall not be allowed to push user2/testimage:v1.0.

Alternatives
============

Docker registry shall be deployed as a host process as opposed to a container.
This allows the registry to hold kube-system container images for starting up
Kubernetes itself in the future if needed.

Data model impact
=================

None

REST API impact
===============

None

Security impact
===============

Docker registry shall run on port 9001 while the registry token server shall
run on port 9002. Authentication and authorization are done through Keystone,
so no credentials need to be stored on the Docker registry or token server
side. Communications between the registry, Docker client, and registry token
server will be through HTTPS by default, using the system self-signed
certificate by default. All communication is done over the management network.

Other end user impact
=====================

None

Performance Impact
==================

None

Other deployer impact
=====================

None

Developer impact
=================

None

Upgrade impact
===============

None

Implementation
==============

Assignee(s)
===========

Primary assignee:
  jerry-sun-u

Repos Impacted
==============

stx-integ, stx-config, stx-ha

Work Items
===========

Build, configure, and manage through STX-HA's SM, the Docker registry process.
Build, configure, and manage through SM, a token server that can communicate
with Openstack Keystone and can act as an authentication and authorization
service for the Docker registry.

Dependencies
============

Docker Distribution and its dependencies
SM from stx-ha

Testing
=======

Docker Distribution comes with its own unit tests. The token server can be
tested by doing a deployment on StarlingX. Specifically, ensure that one
user cannot delete images from another user.

For now, Docker registry and token server processes should be run only if
kubernetes is configured during config controller.

Documentation Impact
====================

None

References
==========

None

History
=======

First Iteration
