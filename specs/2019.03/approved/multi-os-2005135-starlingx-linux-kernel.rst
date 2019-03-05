..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

======================
StarlingX Linux Kernel
======================

Storyboard: https://storyboard.openstack.org/#!/story/2005135

`StarlingX`_ relies on the Linux kernel to manage the hardware, run user
programs, and maintain the security and integrity of the whole system. As a
fully featured cloud project for the distributed edge, there is a commitment
to support emerging requirements, compute intensive use cases and workloads,
which will make become a reality the imperatives of quality of service,
performance, security and low latency.

Problem description
===================

The more devices, the more data; the more data, the more compute power, and
any nano second, counts. All this flood of information requires new ways of
integration to solve the existing challenges and limitations. The lack of
latest software features from the different Open Source communities including
the Linux kernel, and the absence of a hardware platform strategy including
specialized hardware such as GPU/VPU/FPGA, will limit StarlingX to become
the preferred end to end hybrid acceleration solution. Security fixes,
stability improvements, new and updated features and performance improvements
are not part of old software versions.

Use Cases
---------

As pointed out by the whitepaper "Cloud Edge Computing: Beyond the Data
Center" [#1]_, there is an exhaustive list of use cases that benefit from
a distributed architecture, a category that StarlingX belongs to:

- Data collection and analytics
- Security
- Compliance requirements
- Network function virtualization
- Real time
- Inmmersive
- Network Efficiency
- Self-contained and autonomous site operations
- Privacy

The following list contains examples of projects related to specialized
hardware and the Linux kernel version they were merged in:

:v3.17: Intel(R) QAT Driver Framework
:v4.4: FPGA Manager Core
:v5.0: Intel Stratix10 SoC FPGA Manager Driver

Proposed change
===============

Emerging requirements, use cases and workloads would benefit from an updated
Linux kernel. This initiative is divided in the following integration phases:

1. StarlingX Ubuntu based proof of concept [#2]_ is enabled with STD kernel.
2. StarlingX Ubuntu based proof of concept [#2]_ is enabled with RT kernel.

Initial scope does not target a kernel that constantly rolls with the mainline
kernel, integration targets a specific kernel version that is supported by a
community for a certain period of time. StarlingX kernel candidates are one
of the following stable kernels:

1. Supported by the Distribution
2. Latest Stable Release
3. Latest LTS Release
4. Older LTS Release

Alternatives
------------

Tbd

Data model impact
-----------------

Tbd

REST API impact
---------------

Tbd

Security impact
---------------

Tbd

Other end user impact
---------------------

Tbd

Performance Impact
------------------

Tbd

Other deployer impact
---------------------

Tbd

Developer impact
----------------

Tbd

Upgrade impact
--------------

Tbd

Implementation
==============

Assignee(s)
-----------

Tbd

Repos Impacted
--------------

Tbd

Work Items
----------

Tbd

Dependencies
============

Tbd

Testing
=======

Tbd

Documentation Impact
====================

Tbd

References
==========

.. [#1] https://www.openstack.org/edge-computing/cloud-edge-computing-beyond-the-data-center
.. [#2] https://storyboard.openstack.org/#!/story/2004792

History
=======

- First draft, request for comments.

.. _StarlingX: https://www.starlingx.io/

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Stein
     - Introduced
