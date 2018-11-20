..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License. http://creativecommons.org/licenses/by/3.0/legalcode

..
  Many thanks to the OpenStack Nova team for the Example Spec that formed the
  basis for this document.

=========================================
Distributed Cloud - Synchronized Keystone
=========================================

| Storyboard:  https://storyboard.openstack.org/#!/story/2002842
| ( Distributed Cloud Keystone Scalability )
|

The OpenStack Edge-Computing group has defined an Edge Reference Architecture.
For Identity Management, it uses Federated Keystone to manage Identity across
all Edge Clouds.  If 'full autonomy' is required at Edge Clouds, this requires
a Distributed Identity Provider Solution with an Identity Provider (IDP)
presence at every Edge Cloud.

The Federated Keystone solution makes sense where:

* Integration with an existing IDP infrastructure is already required,
* In large deployments that would benefit from distributed IDP solutions,
* Where partial autonomy is acceptable in the presence of edge cloud isolation
  or
* The cost of hosting an IDP presence at every Edge Cloud is acceptable for
  full autonomy.

The OpenStack Edge-Computing group recognizes that there is more than a
'one-size-fits-all' architecture for the Edge.  As agreed upon within
the OpenStack Edge-Computing meetings, this specification proposes an
additional Identity solution for the Edge Reference Architecture; i.e. a
'Synchronized Keystone' solution.  In the Synchronized Keystone solution,
a Synchronization Framework synchronizes the Identity Resources of a Central
Cloud to all of the Edge Clouds.

Synchronized Keystone provides an Identity solution for the edge where :

* a simpler standalone Identity solution can be used for the edge cloud
  deployments, and
* the edge cloud sites are compute-power-limited deployments, e.g. small AIO
  simplex / duplex servers, where the cost of hosting an IDP presence
  in support of full autonomy is too high.

Problem description
===================

In a distributed edge cloud environment, with 100s or 1000s of edge cloud
sites, the centralized orchestration of cloud services across all the edge
cloud sites is imperative for operational usability.  This specification
deals specifically with the centralized orchestration of the Identity Cloud
Service across all the edge cloud sites.

For the Identity Cloud Service, in a distributed edge cloud environment, it is
desired to support the same set of Users and Projects across all edge clouds.
I.e. At any edge cloud, be able to login with the same User name and Project
name, using the same authentication credentials and getting the same
authorization capabilities and roles.

Note that for some use cases, network connectivity between the edge cloud and
the central cloud is not reliable.  The Identity Cloud Service at the edge
cloud must be fully autonomous in the event of network connectivity loss to
the central cloud.  I.e. both Service Users as well as Tenant Users must
continue to be able to authenticate and be authorized when the edge cloud is
isolated from the central cloud.

This specification also enables an optimization for orchestration scalability
in the distributed edge cloud environment.  The orchestration of services
across all edge clouds requires authentication, typically of the same user,
across 100s/1000s of edge clouds.  With the Identity Service's Users and
Projects now synchronized across all edge clouds, then by additionally
synchronizing Fernet Keys across all edge clouds, an authenticated Fernet
Token generated at the Central Cloud can be used at any or all edge clouds;
reducing the 100s or 1000s of authentication operations to a single
authentication.

Use Cases
=========

The requirement for common Identity Users and Projects across all edge clouds
applies to all Edge Computing Use Cases.

The Use Cases that require full autonomy of edge clouds (in the event of edge
cloud isolation) are Use Cases where:

* There are both

  * Remote Physical users (at a central cloud site) and
  * Local physical users (at edge cloud sites).

* All 'userids' are centrally managed for security reasons,
* At the edge cloud site,

  * When connectivity to central cloud is lost

    * local edge users must be able to manage their edge cloud and workloads on
      the edge cloud,
    * ... using their normal userid credentials.

Examples of such Use Cases are:

* Management of Retail Chains (e.g. Walmart)
* Large Hospital Campus
* Large Control Plant

These are also Use Cases where the simplicity of a standalone Identity solution
for the edge would be desirable.

Background
==========

The Distributed Cloud (DC) sub-project within StarlingX, already supports a
Synchronization Framework which is used to synchronize Nova, Neutron, Cinder
and StarlingX resources from the Central Cloud to all of the Edge Clouds.

This Synchronization Framework provides:

* Synchronization Request Management

  * Managing Synchronization Request Message Queues per Edge Cloud,
  * With retry on failure.

* The Overall Synchronization Audit Sequencing,
* Connectivity Status tracking for Edge Clouds, and
* Synchronization Status tracking for Edge Clouds.

For the existing framework, each Service being synchronized implements the
following within the Synchronization Framework:

* an API Proxy

  * For intercepting Service API calls in order to trigger immediate
    synchronization to Edge Clouds,

* a DC Orchestration Module

  * For Service-specific details of Service API Request building and auditing,
  * For managing the mapping of resources in each subcloud to the canonical
    resource in the central cloud, and
  * (in future) for dealing with any API / Schema differences between Central
    Cloud and Edge Cloud (e.g. in Software Upgrade scenario).

Currently the existing Synchronization Framework supports REST API -based
synchronization of a Service's resources.

For OpenStack Keystone, a REST API -based synchronization approach will not
work since not all details of Keystone resources are exposed thru Keystone's
REST APIs, e.g.:

* User-IDs and Project-IDs can NOT be set on POST
  (required to be synchronized so that Fernet Tokens can be used on any/all
  edge clouds)
* Revocation events, generated internally by Keystone to track events that
  affect token validity, are NOT exposed via Keystone REST API,

Proposed change
===============

Synchronization Framework Support for Keystone DB-based Synchronization
-----------------------------------------------------------------------

This specification proposes enhancing the StarlingX's Distributed Cloud's
Synchronization Framework to support DB-based synchronization of a Service's
resources.

I.e. use the existing Synchronization Framework in order to leverage the
existing retry mechanisms, audit mechanisms, synch status tracking, etc.,
but in this case, the Service Module within the 'DC Orchestration Engine'
would synchronize DB Records by:

* Directly querying/setting the Services' DB, and
* Using a new (admin-only) StarlingX DC DB SYNC Service and its REST API
  on the StarlingX Edge Cloud which exposes the DB operations remotely
  for synchronization purposes.

The Service's API Proxy triggers an immediate DB sync of the affected row(s)
of the Service's DB table(s), due to particular API request, while the
Synchronization Framework's Audit Mechanism (default every 10 mins) deals
with non-API events, unexpected events and/or errors to ensure required DB
Table(s) are in-sync.

The following Keystone resources will be synchronized with this method:
Users, Passwords, Projects, Roles, Role Assignments and Token Revocation
Events.

Synchronization of Fernet Keys
------------------------------

This specification also proposes enhancing the StarlingX's Distributed
Cloud's Synchronization Framework to support API-based synchronization of
the Fernet Key Repo.

New REST APIs for bulk synching of the Fernet Key Repo, updating the Fernet
Key Repo (on rotation of keys) and auditing of the Fernet Key Repo are
added to the STX-CONFIG service.

The Synchronization Framework will be extended to support Fernet Key Repo
synchronization thru the STX-CONFIG service; adding a Fernet Key Manager to
the STX-CONFIG DC Orchestration Module for managing the Fernet Key Repo
synchronization messaging done by the Synchronization Framework.

Alternatives
============

An alternative solution considered for synchronizing keystone would be to use
built-in DB synchronization of open-source DBs used within StarlingX for
the OpenStack Service DBs.  I.e. use the built-in DB Synchronization
capabilities of mariaDB or postgresDB, both of which support replication
of DB Tables from a single R/W Master to multiple ReadOnly Slaves.

However, the built-in DB synchronization solutions of mariaDB or postgresDB,
do NOT support the ability of handling different DB Schemas in the Central
Cloud and Edge Clouds; i.e. required for Software Upgrade scenarios, or even
just a heterogeneous mix of openstack-versioned edge clouds.

Data model impact
=================

There are no DB Model changes required to any Services.

REST API impact
===============

Synchronization Framework Support for Keystone DB-based Synchronization
-----------------------------------------------------------------------

The following REST APIs were added to the STX-DISTCLOUD service to support
DB-based synchronization of Services between the Central Cloud and the
Edge Clouds:

NOTE: These are public REST APIs in the sense that the Central Cloud
will use these REST APIs to synchronize data to the Edge Clouds.  HOWEVER
these REST APIs are NOT intended to be used by an end user.

* GET /v1.0/identity/users

  * Description:  DB SYNC List all identity users
  * Normal Reponse Codes:  200
  * Error Response Codes:  computeFault (400, 500, …),
    serviceUnavailable (503), badRequest (400), unauthorized (401),
    forbidden (403), badMethod (405), overLimit (413), badMediaType (415)
  * Response Parameters:

    * < all users of the Keystone DB Table >

      * < all the attributes of the Keystone User DB Table >

* GET /v1.0/identity/users/<UUID>

  * Description:  DB SYNC Get specific identity user
  * Normal Reponse Codes:  200
  * Error Response Codes:  computeFault (400, 500, …),
    serviceUnavailable (503), badRequest (400), unauthorized (401),
    forbidden (403), badMethod (405), overLimit (413), badMediaType (415)
  * Response Parameters:

    * < all the attributes of the Keystone User DB Table >

* POST /v1.0/identity/users

  * Description:  DB SYNC create identity user (and password)
  * Normal Reponse Codes:  201
  * Error Response Codes:  computeFault (400, 500, …),
    serviceUnavailable (503), badRequest (400), unauthorized (401),
    forbidden (403), badMethod (405), overLimit (413), badMediaType (415)
  * Request Parameters:

    * < all the attributes of the Keystone User DB Table >

* PUT /v1.0/identity/users/<UUID>

  * Description:  DB SYNC update identity user (and password)
  * Normal Reponse Codes:  202
  * Error Response Codes:  computeFault (400, 500, …),
    serviceUnavailable (503), badRequest (400), unauthorized (401),
    forbidden (403), badMethod (405), overLimit (413), badMediaType (415)
  * Request Parameters:

    * < all the attributes of the Keystone User DB Table >


... and similarly for the other Keystone DB Resources

* GET /v1.0/identity/projects
* GET /v1.0/identity/projects/<UUID>
* POST /v1.0/identity/projects
* PUT /v1.0/identity/projects/<UUID>

|

* GET /v1.0/identity/roles
* GET /v1.0/identity/roles/<UUID>
* POST /v1.0/identity/roles
* PUT /v1.0/identity/roles/<UUID>

|

* GET /v1.0/identity/assignments
* GET /v1.0/identity/assignments/<UUID>
* POST /v1.0/identity/assignments
* PUT /v1.0/identity/assignments/<UUID>

|

* GET /v1.0/identity/token-revocation-events
* GET /v1.0/identity/token-revocation-events/<UUID>
* POST /v1.0/identity/token-revocation-events
* PUT /v1.0/identity/token-revocation-events/<UUID>

Synchronization of Fernet Keys
------------------------------

The following REST APIs were added to the STX-CONFIG service to support
synchronization of Fernet Key Repo between the Central Cloud and the
Edge Clouds:

NOTE: These are public REST APIs in the sense that the Central Cloud
will use these REST APIs to synchronize data to the Edge Clouds.  HOWEVER
these REST APIs are NOT intended to be used by an end user.

* POST /v1/fernet_repo

  * Description:  Distribute fernet repo
  * Normal Reponse Codes:  201
  * Error Response Codes:  computeFault (400, 500, …),
    serviceUnavailable (503), badRequest (400), unauthorized (401),
    forbidden (403), badMethod (405), overLimit (413), badMediaType (415)
  * Request Parameters:

    * Content-Type application/json

      * Style: Plain
      * Type: Xsd:String
      * Description: The list of Fernet Keys.

* PUT /v1/fernet_repo

  * Description:  Update fernet repo with keys
  * Normal Reponse Codes:  202
  * Error Response Codes:  computeFault (400, 500, …),
    serviceUnavailable (503), badRequest (400), unauthorized (401),
    forbidden (403), badMethod (405), overLimit (413), badMediaType (415)
  * Request Parameters:

    * Content-Type application/json

      * Style: Plain
      * Type: Xsd:String
      * Description: The list of Fernet Keys.

* GET /v1/fernet_repo

  * Description:  List contents of fernet_repo (the keys)
  * Normal Reponse Codes:  200
  * Error Response Codes:  computeFault (400, 500, …),
    serviceUnavailable (503), badRequest (400), unauthorized (401),
    forbidden (403), badMethod (405), overLimit (413), badMediaType (415)
  * Response Parameters:

    * Fernet_keys

      * Style: Plain
      * Type: Xsd:List
      * Description: The list of fernet keys

Security impact
===============

This work only impacts security in a Distributed Cloud environment.

In a Distributed Cloud environment, this work directly manipulates Identity
data by synchronizing selected Keystone resources and Fernet Keys between
the Central Cloud and the Edge Clouds.

The only external impact is that in a Distributed Cloud environment,
a Token created on any Cloud (Central or Edge) can be used on any or
all Clouds (Central or Edge).

Other end user impact
=====================

This work only impacts end user in a Distributed Cloud environment.

In a Distributed Cloud environment, a user can indirectly interact with the
feature when using ANY OpenStack Service API across Edge Clouds by
leveraging the fact that a Token created on the Central Cloud can be
used on any or all Edge Clouds.

In a Distributed Cloud environment, in an edge cloud network isolation
scenario, an end user, local to the edge site, can now login / authenticate
with his normal userid and credentials and manage his workloads.

Performance Impact
==================

This work only impacts performance in a Distributed Cloud environment.

Overall there is a reduced amount of synchronization messaging between
the Central Cloud and the Edge Clouds in a Distributed Cloud Environment.

Logically more data is being synchronized; i.e. Fernet Keys and selected
Keystone DB Resources, in addition to the existing selected STX, Nova,
Neutron and Cinder DB Resources.  However with the ability to use a
single Token, generated on the Central Cloud, for ALL Edge Cloud
synchronization messages, this drastically reduces the Synchronization
Framework messaging.

Other deployer impact
=====================

There are no deployer impacts with this work.

Developer impact
=================

In a Distributed Cloud environment, developers implementing new services
that orchestrate across all Edge Clouds should leverage the fact that
a Token created on the Central Cloud can be used on ANY / ALL Edge Clouds,
in order to reduce their messaging impact on the system.


Upgrade impact
===============

In a Distributed Cloud environment, there are upgrade impacts with this work;
i.e. when upgrading from OpenStack Version N to OpenStack Version N+1.

This work is sensitive to any Keystone DB Model changes.  However the
architecture of the DB-based synchronization within the StarlingX
Distributed Cloud Synchronization Framework does support the ability
to manage DB Schema changes between the Central Cloud and the Edge Cloud.
This was one of the major reasons for choosing this approach.

The plan for Software Upgrades (from one OpenStack Version to another), in
a Distributed Cloud environment, is that the Central Cloud will be
upgraded first to version N+1, and then the Edge Clouds.

If the Keystone DB Schema changes between version N and version N+1,
the N+1 version of Distributed Cloud Synchronization Framework must
implement the Keystone DB Schema conversions between N+1 and N,
for all synchronization messages during the Rolling Software Upgrade
across the entire Distributed Cloud system.

Implementation
==============

Assignee(s)
===========

Primary assignee:
  Andy Ning

Other contributors:
  Tao Liu

Repos Impacted
==============

Repositories in StarlingX that are impacted by this spec:

* stx-distcloud

Work Items
===========

Synchronization Framework Support for Keystone DB-based Synchronization
-----------------------------------------------------------------------

* Introduce dbsync agent/api on sub cloud, and add it to starlingx as a new
  service,
* REST APIs between dcorch engine and dbsync agent (POST/PUT/GET),
* Implement dbsync client to wrap dbsync APIs into python functions,
* Enhance identity module within dcorch engine to do DB based resource
  synchronization,
* Enhance identity module within dcorch engine to do DB based resource audit,
* Add new resources to be synced (token revocation events),

  *  NOTE: that current code is synching users, passwords, projects, roles and
      role assignments ... albeit using API-based synchronization,

* Deployment and configuration of new StarlingX DistCloud Services,
* Unit test.


Synchronization of Fernet Keys
------------------------------

* Add new stx-config APIs (POST) for central cloud to distribute fernet repo
  including RPC between stx-config API and conductor,
* Add new stx-config APIs (GET) for central cloud to audit existing keys
  including RPC between stx-config API and conductor,
* Add new stx-config APIs (PUT) for central cloud to update repo with keys
  including RPC between stx-config API and conductor,
* stx-config internally, safely retrieve and update fernet keys,
* Enhance stx-distcloud orch engine (or cron job) to rotate keys and
  call stx-config APIs to distribute new keys,
* Enhance stx-distcloud orch engine to audit fernet keys across managed
  sub clouds, and call stx-config APIs to distribute keys if mis-matches found,
* Enhance dc manager to trigger key distribution when a sub cloud becomes
  managed,
* Add logic to stx-config to empty and re-setup fernet repo locally when
  receive an empty POST,
* stx-config/stx-metal/stx-distcloud unit test (Tox),
* Manifest for fernet repo and keys creation during deployment may not need
  any changes on both central cloud and sub clouds.

Dependencies
============

There are no external dependencies for this work.

I.e. there are NO requirements on changes to OpenStack Keystone.

Testing
=======

Need to do explicit testing of Fernet Token synchronization and Keystone
DB Resource synchronization between Central Cloud and Edge Clouds.

Need to do COMPLETE regression of StarlingX Distributed Cloud (DC)
functionality.

Should qualitatively evaluate performance / messaging scalability
improvements before and after this work.

Need to do a SANITY regression of StarlingX in an NON-DC environment.

Documentation Impact
====================

Currently there is no documentation on the StarlingX Distributed Cloud
functionality.  When this documentation is created, the work of this
specification should be described at a functional level.

References
==========

None.


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - 19.03
     - Introduced
