%%%
title = "IPSIE AL1 SCIM 2.0 Profile"
abbrev = "IPSIE AL1 SCIM"
ipr = "trust200902"
area = "Applications and Real-Time"
workgroup = "IPSIE"
keyword = ["scim", "ipsie", "provisioning", "identity", "oauth"]

[seriesInfo]
name = "Internet-Draft"
value = "draft-schreiber-ipsie-scim-al1-profile-latest"
stream = "IETF"
status = "informational"

[[author]]
initials = "J."
surname = "Schreiber"
fullname = "Jen Schreiber"
organization = "SGNL"
  [author.address]
  email = "jen@sgnl.ai"

[[author]]
initials = "M."
surname = "Maguire"
fullname = "Mark Maguire"
organization = "Aujas Cybersecurity"
  [author.address]
  email = "mark.maguire@aujas.com"

[[author]]
initials = "P."
surname = "Valarezo"
fullname = "Pablo Valarezo"
organization = "Independent"
  [author.address]
  email = "pxvalarezo@gmail.com"
%%%

.# Abstract

This document defines a profile for SCIM 2.0 to meet the security and interoperability requirements for identity lifecycle management within enterprises. Within the context of SCIM, The profile establishes requirements for provisioning, account management, client authentication, and identity synchronization.

{mainmatter}

# Discussion Venues

This note is to be removed before publishing as an RFC.

Source for this draft and an issue tracker can be found at
<https://github.com/jischr/ipsie-scim-al1-profile>.

# Introduction

This document defines the IPSIE Account Lifecycle 1 (AL1) Profile for SCIM 2.0. It provides a clear reference for SCIM deployments that require a well-defined security baseline meeting best practices for interoperable enterprise identity management.

The profile addresses critical aspects of secure identity management, with particular emphasis on:

* Client authentication
* Retrieve, add, and modify Users
* Retrieve, add, and modify Groups
* Synchronization of data from the Identity Service to the Application

By adhering to this profile, organizations can implement SCIM-based integrations that meet stringent security requirements while ensuring interoperability across different implementations.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all capitals, as shown here.

## Terminology

SCIM

> The System for Cross-domain Identity Management as defined in [@!RFC7643] and [@!RFC7644]

SCIM Client

> An application that uses the SCIM protocol to manage identity data maintained by the service provider (SP). The client initiates SCIM HTTP requests to a target service provider. To clarify the relationship in terms of identity management, the Identity Provider (IdP) is the SCIM Client and initiates requests to the SP, which is the SCIM Server.

SCIM Server (also referred to as SCIM Service Provider)

> An HTTP web application that provides identity information via the SCIM protocol. To clarify the relationship in terms of identity management, the SP is the SCIM Server and receives the requests from the IdP, which is the SCIM Client.

Role

> A set of permissions.  Any user or account within the same role receives the predefined ability to access a resource and/or perform an action.

Identity Service or Identity Provider (IdP)

> Acts as the SCIM client, initiating all provisioning, updates, and deprovisioning operations.

Application

> Acts as the SCIM server or service provider, hosting SCIM endpoints and processing all provisioning requests.

Note: When SCIM is applied to the context of IPSIE, the Identity Service acts as the SCIM client and the Application acts as the SCIM service provider. The document will use the Role terms below for consistency across IPSIE Profiles.

# Profile

## Authentication and Authorization {#authn-authz}

The Identity Service and Application MUST use OAuth 2.0 [@!RFC6749] for authentication and authorization of SCIM protocol.

> **Editor's Note:** This section should be expanded and may need to reference the IPSIE Session Lifecycle 1 (SL1) profile.

The following requirements ensure  consistent and secure handling of access tokens and authorization server configuration:

* OAuth 2.0 interactions MUST comply with JWT Client Authentication as defined in [@!RFC7523]
* The token SHALL exchange a signed JWT for an access token and present that token in the {Authorization: Bearer} header on all subsequent SCIM requests.
* The token MUST contain a "token_endpoint" value which is the URL of the Identity Service's OAuth 2.0 token endpoint.
* The Acess Token MUST include the "scim" scope and not grant broader permissions.
* All Authorization Server parameters SHOULD be discovered from OAuth Authorization Server metadata as defined in [@!RFC8414].
* The Identity Service SHOULD expose a jwks_uri to allow the Application to perform signature verification

## SCIM Interoperability Requirements

### General Requirements

* The Identity Service SHALL implement the required functionality of a SCIM client as defined in [@!RFC7643] and [@!RFC7644].
* The Application SHALL implement the required functionality of a SCIM service provider as defined in [@!RFC7643] and [@!RFC7644].
* All SCIM operations SHALL be authenticated and authorized via OAuth 2.0 as specified in (#authn-authz).
* Local modifications to Users or Groups in the Application are prohibited.

### User Provisioning Operations

The Application MUST provide support all User provisioning operations defined in this section.

#### Create User (POST /Users)

User creation is performed by the SCIM operation POST /Users.

In addition to the user attributes required by [@!RFC7643], the following attributes are required to be part of the User schema:

> **Editor's Note:** Should specify exact attribute names (e.g., "email" vs "emails") rather than keeping this vague.

* An attribute which contains a unique identifier used by the enterprise to distinguish the owner of the account, such as "externalId."
* An attribute which contains the primary email address of the user, such as "email"

#### Update User (PATCH /Users/{id})

User updates are performed by the SCIM operation PATCH /Users/{id}

#### Deactivate or Reactivate User (PATCH /Users/{id})

Changes to the user activation status, such deactivation and reactivation, are performed by the SCIM operation PATCH /Users/{id}

The Identity Service SHOULD propagate user deactivation events to the Application within 5 minutes of the user being deactivated.

The Application SHOULD respond to user deactivation events by revoking the ability for the user to continue accessing the Application, including the revocation of currently active sessions. The revocation mechanism outside the scope of this profile. Revocation SHOULD occur within 5 minutes of receiving the deactivation request.

When a user account is deactivated, all access mechanisms and authorizations associated with that account MUST also be deactivated. This includes, but is not limited to:

* Web sessions
* API tokens
* Refresh tokens
* Personal access tokens
* SSH keys associated with the user
* Device-based authentication credentials

The Application MUST allow reactivation of a deactivated user.

#### Delete User (DELETE /Users/{id})

User deletions are performed by the SCIM operation DELETE /Users/{id}

After a user is deleted, the Application MUST allow the creation of a new user with the same username.

> **Editor's Note:** Need to clarify implications for maintaining user data and avoiding conflicts when recreating users with the same username.

#### Get All Users (GET /Users)

Get all users in the system is performed by the SCIM operation GET /Users

This endpoint ensures that all users are managed by the Identity Service.

To ensure that large amounts of data can be read from the Application, the application must support with index-based or cursor-based pagination for the GET /Users request. To ensure system stability and prevent abuse, the Application SHALL enforce rate limits on this endpoint and must respond with appropriate headers, such as "429 Too Many Requests" and "Retry-After," when limits are exceeded.

#### Get User By ID (GET /Users/{id})

User searches by id are performed by the SCIM operation GET /Users/{id}

#### List Users By Alternate Identifier (GET /Users?)

User searches by alternate identifier are performed via the SCIM operation: GET /Users?filter={filterExpression}

Application Providers MUST support the following filter expressions:

* username eq \{username\}
* externalId eq \{externalId\}
* emails[value eq \{email\}]

### Group (Role) Provisioning Operations

The Application MUST provide support all Group provisioning operations defined in this section.

**Note**: Within the IPSIE standard, Application permissions are referred to as "Roles." Within SCIM, Application permissions are referred to as "Groups." The term "Role" in IPSIE is functionally equivalent to the term "Group" in SCIM.

#### Create Group (POST /Groups)

Group creation is performed by the SCIM operation POST /Group.

> **Editor's Note:** This section needs additional details about required Group attributes and schema requirements.

#### Get All Groups (GET /Groups)

A search for all groups in the system is performed by the SCIM operation GET /Groups

This endpoint ensures that all groups are managed by the Identity Service.

To ensure that large amounts of data can be read from the Application, the application MUST support with index-based or cursor-based pagination for the GET /Groups request. To ensure system stability and prevent abuse, the Application SHALL enforce rate limits on this endpoint and MUST respond with appropriate headers, such as "429 Too Many Requests" and "Retry-After," when limits are exceeded.

#### Get Group By ID (GET /Group/{id})

Group searches by id are performed by the SCIM operation GET /Group/{id}?excludedAttributes=members

#### List Groups By Alternate Identifier (GET /Groups?)

User lookups by alternate identifier are performed by the SCIM operation GET /Groups?filter={filterExpression}&excludedAttributes=members

Application Providers MUST support the following filter expressions: TBD

#### Add or Remove Group Members (PATCH /Group/{id})

Members are added or removed from Groups via the SCIM operation PATCH /Groups/{id}

For each Operation in the PATCH:

The op attribute MUST contain either "add" or "remove".

* When the op is "add":
  * The path attribute MUST be "members".
  * The value attribute MUST be an array of Group member elements, as defined in Section 4.2 of the SCIM Core Schema [@!RFC7643]. Each member MUST contain a value subattribute with the id of the resource being added to the group.

* When the op is "remove":
  * The path attribute MUST be either:
    * "members" (to remove all members)
    * "members[value eq \{id\}]" (to remove a single member)
  * The value attribute MUST be unspecified.

## Metadata Endpoints

### ResourceTypes

Application MUST host a /ResourceTypes endpoint, as defined in Section 4 of [@!RFC7644].

The supported ResourceTypes MUST include Users and Groups.

### ServiceProviderConfig

Application Providers MUST host a /ServiceProviderConfig endpoint to describe the operations they support, as defined in Section 4 of [@!RFC7644]

The operations MUST include, at minimum, the set of SCIM capabilities required for compatibility with this IPSIE profile.

### Schemas

Application Providers MUST host a /Schemas endpoint to describe the supported schemas, as defined in Section 4 of [@!RFC7644]. There must be a schema for both Users and Groups. The schemas must include all required attributes from RFC 7643 and from Section 3.2.3 (Create User).

# Security Considerations

For SCIM security considerations, see [@!RFC7643] and [@!RFC7644]

Additionally, the following requierements are included to address security considerations.

* **Transport Security**: All endpoints SHALL enforce TLS 1.2 or later with strong cipher suites and certificate validation.
* **Error Handling**: Error responses SHALL use the SCIM error format and SHALL NOT leak internal details.
* **Replay Resistance**: Access tokens SHALL expire and nonces SHALL be validated to prevent replay.
* **Auditing**: All provisioning actions and responses SHALL be logged for audit and troubleshooting.
* **Rate Limiting**: All endpoints SHALL enforce rate limiting and must respond with "429 Too Many Requests" when limits are exceeded.

# IANA Considerations

This document has no IANA actions.

# Compliance Statement

Implementation of all mandatory requirements in this profile will result in a SCIM 2.0 deployment that satisfies IPSIE Identity Lifecycle Level 1 (IL1). Specifically:

* **Identity Service (SCIM client)**
  * SHALL initiate all CRUD operations for Users and Groups.
  * SHALL adhere to the security considerations above.

* **Application (SCIM service provider)**
  * SHALL host all SCIM endpoints with full support for User and Group provisioning.
  * SHALL prevent local modifications outside of SCIM.
  * SHALL enforce OAuth 2.0 JWT Profile for Authentication [@!RFC7523].
  * SHALL adhere to the security considerations above.

By conforming to this profile, implementations will achieve a consistent, secure, and interoperable baseline for enterprise identity lifecycle management.

{backmatter}
