..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode


============================
Ansible Bootstrap Deployment
============================

Storyboard: https://storyboard.openstack.org/#!/story/2004695.

This spec describes the initial phase of StarlingX deployment improvement
effort.

Problem description
===================

The primary controller is currently configured using the ``config_controller``
Python script which can only be executed on the controller console. The script
requires input for many networking aspects upfront in order to run both
bootstrap operations and host configuration to completion. Over time, the
script logic has grown overly complex to accommodate a plethora of host
configuration scenarios and so has increased the configuration time.

Furthermore, once all required input configuration parameters have been
successfully validated, the script will run all its steps. If the script fails
due to a software issue or a configuration mistake, a re-install will be
required. It is not possible for the user to apply a software patch and/or
rerun the script to apply updated configurations.

Use Cases
=========

* As a developer/tester/operator, I need the ability to configure the
  controller remotely.
* As a developer/tester/operator, I need to the ability to modify and
  reapply configurations during initial host config.
* As a developer/tester/operator, I need the ability to automate the
  initial host deployment and build out my system from there.
* As a developer of StarlingX community, I would like to streamline
  the initial host config using an industry adopted tool to enable
  automation and to promote process/code visibility and customization.

Proposed change
===============

Existing workflow with config_controller (high level)
-----------------------------------------------------
**Config_controller:**

1. Create bootstrap config
2. Apply bootstrap manifest
3. Persist local configuration
4. Populate initial system inventory
5. Create system configuration
6. Apply controller manifest
7. Finalize controller configuration
8. Activate all services

**Host-configuration:**

   Manual or scripted configurations required for unlock.

**Host-unlock:**

1. Apply controller manifest (and worker, storage manifests for AIO)
2. Activate all services

Proposed workflow with Ansible Playbook (high level)
----------------------------------------------------
The bootstrap and configuration of the initial host will be orchestrated
by an Ansible Playbook [1]_.

**Playbook:**

1. Apply bootstrap manifest
2. Populate system configuration (with defaults and user-supplied config)
3. Bring up Kubernetes master node and essential services

**Host-configuration:**

   Manual or scripted configurations required for unlock.

**Host-unlock**

1. Apply controller manifest (and worker, storage manifests for AIO)
2. Activate all services

After phase #2 of the Playbook, the host configuration will resemble
All-in-one simplex (i.e. defaulting to the loopback interface) until it
is unlocked for the first time.

Scope of the new workflow
-------------------------
The new workflow will cover the **initial config** for all supported system
configurations in a containerized platform.

Bootstrap playbook roles and tasks (high level)
-----------------------------------------------
Below is a list of major roles and tasks. The names are deliberately long
to make them self-explanatory for review purpose. They can be renamed to
be more terse as role variables should be prefixed with role names.
During implementation, some roles and tasks will likely be decomposed or
combined.

Role: validate-config-input
   * Task: validate-config
Role: prepare-environment-for-execution
   * Task: validate-environment
   * Task: set-environment-variables
Role: cleanup-environment-after-execution
   * Task: unset-environment-variables
   * Task: remove-temp-files
Role: store-admin-password
   * Task: validate-password
   * Task: store-password
Role: apply-bootstrap-manifest
   * Task: generate-bootstrap-data
   * Task: apply-manifest
Role: populate-initial-config
   * Task: persist-keyring
   * Task: set-permanent-puppet-workdir
   * Task: set-permanent-pxe-configdir
   * Task: set-postgres-config-for-mate
   * Task: process-branding-and-banner
   * Task: populate-system-config
   * Task: populate-load-config
   * Task: populate-network-config
   * Task: populate-controller-config
   * Task: create-loopback-interface
   * Task: update-local-dns
   * Task: update-platform-config-file
   * Task: add-dns-server
Role: bring-up-kubernetes-master-and-dependent-services
   * Task: bring-up-kubernetes-master
   * Task: bring-up-tiller
   * Task: bring-up-fault-management
   * Task: bring-up-maintenance
   * Task: bring-up-vim

Playbook directory layout
-------------------------
The directory layout of the playbook initially could be as follows:

bootstrap.yml

roles/
  validate-config-input/
    tasks/
      main.yml
    handlers/
      main.yml
    files/
      <scripts, files>
    vars/
      main.yml
    defaults/
      main.yml
    meta/
      main.yml

  prepare-environment-for-execution/

  cleanup-environment-after-execution/

  store-admin-password/

  apply-bootstrap-manifest/

  popupate-initial-config/

  bring-up-Kubernetes-master-and-dependent-services/

Playbook pre_tasks and post_tasks
---------------------------------
The pre_tasks and post_tasks can be as simple as marking the start and end
of the playbook execution.

Running ``bootstrap playbook``
------------------------------
ansible-playbook bootstrap.yml -u <named-account-with-sudo-privileges>
[-K -i <config-input-file> -e <list-of-variable-value-pairs-to-overwrite>
--ask-vault-password]

The playbook should be run using wrsroot account. However, it can be run using
another account with sudo privileges if desired provided that the account has
already been setup beforehand. Many playbook tasks must be run as root.
The option -K will prompt for privilege escalation password.

Overwriting playbook defaults
-----------------------------
The ``bootstrap playbook`` will come with default variables and a
bootstrap_host.yml file packaged in StarlingX iso at location
/etc/ansible/hosts. These defaults and content of the host file are meant
for running the playbook locally and bootstrapping the initial controller
for All-in-one simplex in virtual box. In practice, some of these defaults
will need to be overwritten with user supplied values.

Variables that usually require overwriting are:

* host IP (for running the playbook remotely)
* system properties
* Management, OAM, PXE, cluster subnets
* Default DNS server

There are various ways to overwrite variables in Ansible Playbook.

**Overwrite with configuration input file**

One simple and clean option is to overwrite with -i command line parameter.
The content of the provided configuration input file must be in YAML format.

The default host file will have the following entries:

...
  hosts:
    localhost:
      ansible_user: wrsroot
      ansible_become: true

To overwrite the bootstrap host IP and user in the custom configuration input
file:

...
  hosts:
    128.224.150.81:
      ansible_user: abc
      ansible_become: true

To overwrite the role default variables, one option is to add the relevant
role sections and the list of overwritten variables:

  roles:
    populate_initial_config:
      system_mode: duplex-direct
      dns_server: 8.8.8.8

**Overwrite with role vars**

Another option is to replace main.yml file under ``vars`` directory of
the corresponding role(s) with custom one(s) before running the playbook.

**Overwrite with extra vars**

Command line -e option which has the highest precedence can also be used
to overwrite defaults. However, this method can be cumbersome if many
defaults need overwriting and the playbook is run manually.

The list of role defaults as well as the preferred method to overwrite
these defaults will be documented after the playbook has been developed.

Overwriting sensitive variables
-------------------------------
The admin password is a sensitive variable that usually needs to be
overwritten. To ensure sensitive information is encrypted, sensitive
variables and values are copied to a vault file and secure using
ansible-vault encrypt command. The corresponding defaults will need to be
mapped to the variables in vaulted file using jinja2 syntax.

The command line argument --ask-vault-pass will need to be supplied when
running the playbook with encrypted vault file.

For development/test purposes, these variables can simply be overwritten
using the command line -e option.

Validating configuration parameters
-----------------------------------
The config_controller script has extensive logic to validate config
parameters in user input file which could be leveraged in
validate-config-input role of the ``bootstrap playbook``.

Config_controller script changes
--------------------------------
Currently this complex script has multiple uses: a) perform initial
configuration required mainly to bring up the controller services,
b) backup system configuration, c) restore system configuration from
backup file, d) clone the image, and e) restore the system from a clone.

The proposed Ansible bootstrap deployment will replace the initial system
configuration aspect of the script. The script will continue to be used for
other operations. Relevant code will be removed from the script once the
implementation of the playbook is complete.

Puppet changes
--------------
The initial ``bootstrap playbook`` will leverage the existing Puppet
bootstrap.pp manifest to bring up the following services that will be
used by the playbook for the remaining tasks:

**Required services to bring up Kubernetes master:**

* docker
* etcd

**Required services for host unlock:**

* fm
* mtcAgent
* nfv-vim

The puppet .pp and in some cases .py files related to these services and
Kubernetes will require update.

Sysinv changes
--------------
Traditionally, the ``config_controller`` script is provided with all
required parameters either interactively or via a config file to perform
both bootstrap operations and host configuration. Networking and storage
provisioning using system commands beyond this point have certain
restrictions as the controller manifest has been applied.

With Ansible bootstrap deployment method, some system commands will
require changes to support manual configuration adjustments and replays of
the ``bootstrap playbook``. The ``cgtsclient`` will also need minor
modification to avoid requesting for smapi endpoint which is not yet
available in this early stage.

Maintenance changes
-------------------
Some minor tweaks to maintenance code will be required for maintenance
Client and Agent to operate properly during the bootstrap phase.

Packaging of ``bootstrap playbook`` in the ISO and SDK
------------------------------------------------------
The playbook will be packaged in the ISO as well as SDK to allow
both local and remote execution.

Alternatives
============

Additional host configuration roles to support the initial host-unlock
were considered. However, this would add much of the complex modeling of
input configuration (i.e. more upfront planning) to the intial deployment step.

Data model impact
=================

No impact to existing system inventory data model.

REST API impact
===============

At this time, no REST API impact is anticipated.

Security impact
===============

The proposal is to make use of Ansible Playbook which is a well adopted
multi-node configuration and deployment orchestration tool partly due to
Ansible secure architecture and design.

The scope of the proposed ``bootstrap playbook`` is limited to bringing the
initial controller to the state where it can be unlocked and allow other
Kubernetes nodes on an internal cluster network if configured to join.

The Playbook can only be executed remotely over SSH using a named account
with sudo privileges. Ansible vault will be used to store secrets/private
information where applicable. As such, no additional security impact is
introduced.

Other end user impact
=====================

The user will be expected to interact with the feature using
ansible-playbook and ansible-vault commands. The bootstrap deployment
method will give the user more flexibility to customize and automate
the deployment.

Once the initial controller is ready to accept system commands and
Kubernetes master is up, the user can:
* perform minimum host configurations and unlock the host
* join other Kubernetes nodes and perform more extensive custom
configurations before the unlock

The playbook can be replayed to update system properties and general
networking information. It will not be playable after the host is unlocked.

Performance Impact
==================

Ansible execution overhead is unknown at this time. However, as the
controller manifest application and services activation steps are deferred
till host-unlock, the time to bring the controller to unlock-ready state
should be significantly faster than with the traditional method.

Other deployer impact
=====================

None

Developer impact
================

See end user impact.

The developers can extend the ``bootstrap playbook`` with custom host
configuration role(s) or another playbook to suit their specific needs.

Upgrade impact
==============

None as this is the initial release of Bootstrap Deployment using
Ansible Playbook.

Implementation
==============

Assignee(s)
===========

Primary assignee:

* Tee Ngo (teewrs)

Other contributors:

* Eric McDonald (emacdona)

Repos Impacted
==============

stx-config
stx-metal

Work Items
==========

* Modify maintenance to enable maintenance operations during bootstrap
  phase.
* Modify sysinv and cgtsclient to be more flexible with configuration
  updates during bootstrap deployment using either system commands or APIs.
* Modify puppet classes and python scripts to allow launching a limited
  number of services required for bootstrap operations and initial host
  unlock.
* Create a ``bootstrap`` Playbook to bring up Kubernetes master node and
  configure the primary controller based on default and user-supplied config
  parameters.
* Package the Playbook as part of the ISO & SDK to allow both on premise
  and remote execution.
* Make other necessary changes to support primary controller configuration
  using either the playbook or traditional config_controller until the
  transition is complete. This includes lab setup tool changes.


Dependencies
============

* config_controller script
* Ansible 2.4

Testing
=======

This story changes the way StarlingX system is deployed, specifically
how the primary controller is configured, which will require changes in
existing automated installation and lab setup tools.

The system deployment tests will be limited to All-in-one simplex,
duplex standard configurations. Deployment tests for region and
distributed cloud configurations are out of scope.

Documentation Impact
====================

This story affects the StarlingX installation and configuration
documentation. Specific details of the documentation changes will be
addressed once the implementation is complete.

References
==========

.. [1]  https://docs.ansible.com/ansible/2.4/ansible-playbook.html

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - TBD
     - Introduced
