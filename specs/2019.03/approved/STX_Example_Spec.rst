..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

..
  Many thanks to the OpenStack Nova team for the Example Spec that formed the
  basis for this document.

=======================
StarlingX: Example Spec
=======================

Some notes about the Spec process:

* The aim of this document is first to define the problem we need to solve, and
  second agree on the overall approach to solve that problem.

* This is not intended to be extensive documentation for a new feature. For
  example, there is no need to specify the exact configuration changes, nor the
  exact details of any DB model changes. But you should still define that such
  changes are required, and be clear on how that will affect upgrades.

* You should aim to get your spec approved before writing your code. While you
  are free to write prototypes and code before getting your spec approved, its
  possible that the outcome of the spec review process leads you towards a
  fundamentally different solution than you first envisaged.

* But, API changes are held to a much higher level of scrutiny. As soon as an
  API change merges, we must assume it could be in production somewhere, and as
  such, we then need to support that API change forever. To avoid getting that
  wrong, we do want lots of details about API changes upfront.

Some notes about using this template:

* Your spec should be in ReSTructured text, like this template.

* Please wrap text at 79 columns.

* The filename in the git repository should include the StoryBoard number and
  name, for example a Story at
  https://storyboard.openstack.org/#!/story/1234567 should be named
  [category]_1234567-feature-name.rst (refer to instructions.rst for guidelines
  on a suitable category name)

* Please do not delete any of the sections in this template. If you have
  nothing to say for a whole section, just write: None

* For help with syntax, see http://sphinx-doc.org/rest.html

* To test out your formatting, build the docs using tox and see the generated
  HTML file in doc/build/html/specs/<path_of_your_file>

* If you would like to provide a diagram with your spec, ascii diagrams are
  required. http://asciiflow.com/ is a very nice tool to assist with making
  ascii diagrams. The reason for this is that the tool used to review specs is
  based purely on plain text. Plain text will allow review to proceed without
  having to look at additional files which can not be viewed in gerrit. It will
  also allow inline feedback on the diagram itself.


Example Spec - The title of your blueprint
==========================================

Include the URL of your Storyboard Story:

Storyboard:
https://storyboard.openstack.org/#!/story/list?status=active&project_group_id=86

Introduction paragraph -- why are we doing anything? The essential "Why" or
motivation is key to laying the ground for the work ahead. It provides contexts
for all involved in the work. A single paragraph of prose that operators can
understand. The title and this first paragraph should be used as the subject
line and body of the commit message respectively.

Problem description
===================

A detailed description of the problem. What problem is this spec addressing?

Use Cases
=========

What use cases does this address? What impact on actors does this change have?
Ensure you are clear about the actors/personas in each use case: Developer, End
User, Deployer etc.

Proposed change
===============

Here is where you cover the change you propose to make in detail. How do you
propose to solve this problem?

If this is one part of a larger effort make it clear where this piece ends. In
other words, what's the scope of this effort?

At this point, if you would like to just get feedback on if the problem and
proposed change fit in StarlingX, you can stop here and post this for review to
get preliminary feedback. If so please say: Posting to get preliminary feedback
on the scope of this spec.

Alternatives
============

What other ways could we do this thing? Why aren't we using those? This doesn't
have to be a full literature review, but it should demonstrate that thought has
been put into why the proposed solution is an appropriate one.

Data model impact
=================

Changes which require modifications to the data model often have a wider impact
on the system. The community often has strong opinions on how the data model
should be evolved, from both a functional and performance perspective. It is
therefore important to capture and gain agreement as early as possible on any
proposed changes to the data model.

Questions which need to be addressed by this section should include:

* What new data objects and/or database schema changes is this going to
  require?

* What database migrations will accompany this change.

* How will the initial set of new data objects be generated.

REST API impact
===============

Each API method which is either added or changed should have the following

* Specification for the method : As best as can be determined at the definition
  stage.

  * Parameters which can be passed via the url

* Example use case including typical API samples for both data supplied by the
  caller and the response

* Discuss any policy changes, and discuss what things a deployer needs to think
  about when defining their policy.

Note that the schema should be defined as restrictively as possible. Parameters
which are required should be marked as such and only under exceptional
circumstances should additional parameters which are not defined in the schema
be permitted (eg additionaProperties should be False).

Reuse of existing predefined parameter types such as regexps for passwords and
user defined names is highly encouraged.

Security impact
===============

Describe any potential security impact on the system. Some of the items to
consider include:

* Does this change touch sensitive data such as tokens, keys, or user data?

* Does this change alter the API in a way that may impact security, such as a
  new way to access sensitive information or a new way to login?

* Does this change involve cryptography or hashing?

* Does this change require the use of sudo or any elevated privileges?

* Does this change involve using or parsing user-provided data? This could be
  directly at the API level or indirectly such as changes to a cache layer.

* Can this change enable a resource exhaustion attack, such as allowing a
  single API interaction to consume significant server resources? Some examples
  of this include launching subprocesses for each connection, or entity
  expansion attacks in XML.

For more detailed guidance, please see the OpenStack Security Guidelines as a
reference (https://wiki.openstack.org/wiki/Security/Guidelines). These
guidelines are a work in progress and are designed to help you identify
security best practices. For further information, feel free to reach out to the
OpenStack Security Group at openstack-security@lists.openstack.org.

Other end user impact
=====================

Aside from the API, are there other ways a user will interact with this
feature?

* Does this change have an impact on python-client? What does the user
  interface there look like?

Performance Impact
==================

Describe any potential performance impact on the system, for example how often
will new code be called, and is there a major change to the calling pattern of
existing code.

Examples of things to consider here include:

* A periodic task might look like a small addition but if it calls conductor or
  another service the load is multiplied by the number of nodes in the system.

* Any impacts to the deployment performance

* A small change in a utility function or a commonly used decorator can have a
  large impacts on performance.

* Calls which result in a database queries (whether direct or via conductor)
  can have a profound impact on performance when called in critical sections of
  the code.

* Will the change include any locking, and if so what considerations are there
  on holding the lock?

Other deployer impact
=====================

Discuss things that will affect how you deploy and configure OpenStack that
have not already been mentioned, such as:

* What config options are being added? Should they be more generic than
  proposed? Are the default values ones which will work well in real
  deployments?

* Is this a change that takes immediate effect after its merged, or is it
  something that has to be explicitly enabled?

* If this change is a new binary, how would it be deployed?

* Please state anything that those those upgrading from the previous release,
  need to be aware of. Also describe any plans to deprecate configuration
  values or features. Consider the potential implications of automated
  deployment technologies.

Developer impact
=================

Discuss things that will affect other developers working on StarlingX.

Upgrade impact
===============

Describe any potential upgrade impact on the system, such as:

* StarlingX supports N-1 version for rolling upgrades. Does the proposed change
  need to consider older code running that may impact how the new change
  functions, for example, by changing or overwriting global state in the
  database? This is generally most problematic when making changes that involve
  multiple compute hosts, like move operations such as migrate, resize,
  unshelve and evacuate.


Implementation
==============

Assignee(s)
===========

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Repos Impacted
==============

List repositories in StarlingX that are impacted by this spec.

Work Items
===========

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.


Dependencies
============

* Include specific references to specs in StarlingX, or in other projects, that
  this one either depends on or is related to.

* If this requires functionality of another project that is not currently used
  by StarlingX document that fact.

* Does this feature require any new library dependencies or code otherwise not
  included in OpenStack? Or does it depend on a specific version of library?


Testing
=======

Please discuss the important scenarios needed to test here, as well as specific
edge cases we should be ensuring work correctly. For each scenario please
specify if this requires specialized hardware, a full openstack environment, or
can be simulated inside the project tree.

Please discuss how the change will be tested. We especially want to know what
tempest tests will be added. It is assumed that unit test coverage will be
added so that doesn't need to be mentioned explicitly, but discussion of why
you think unit tests are sufficient and we don't need to add more tests would
need to be included.

Is this untestable in gate given current limitations (specific hardware /
software configurations available)? If so, are there mitigation plans (3rd
party testing, gate enhancements, etc).


Documentation Impact
====================

Which audiences are affected most by this change, and which documentation
titles for StarlingX should be updated because of this change? Don't repeat
details discussed above, but reference them here in the context of
documentation for multiple audiences. For example, the End User Guide would
need to be updated if the change offers a new feature available through the CLI
or dashboard. If a config option changes or is deprecated, note here that the
documentation needs to be updated to reflect this specification's change.

References
==========

Please add any useful references here. You are not required to have any
reference. Moreover, this specification should still make sense when your
references are unavailable. Examples of what you could include are:

* Links to mailing list or IRC discussions

* Links to notes from a summit session

* Links to relevant research, if appropriate

* Related specifications as appropriate (e.g. if it's an EC2 thing, link the
  EC2 docs)

* Anything else you feel it is worthwhile to refer to


History
=======

Optional section intended to be used each time the spec is updated to describe
new design, API or any database schema updated. Useful to let reader understand
what's happened along the time.

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Stein
     - Introduced
