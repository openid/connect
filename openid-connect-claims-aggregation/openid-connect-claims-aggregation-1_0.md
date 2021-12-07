%%%
title = "OpenID Connect Claims Aggregation Draft 02"
abbrev = "oidc-ca"
ipr = "none"
workgroup = "connect"
keyword = ["security", "openid", "w3c verifiable credential", "did", "privacy"]

[seriesInfo]
name = "OpenID-Draft"
value = "oidc-ca-01"
status = "standard"

[pi]
subcompact = "yes"
private = "Draft-02"
tocdepth = "5"
iprnotified = "no"

[[author]]
initials="N."
surname="Sakimura"
fullname="Nat Sakimura"
organization="NAT.Consulting"
    [author.address]
    email = "nat@nat.consulting"

[[author]]
initials="E."
surname="Jay"
fullname="Edmund Jay"
organization="Illumila"
    [author.address]
    email = "ejay@illumi.la"
    
[[author]]
initials = "T."
surname = "Looker"
fullname = "Tobias Looker"
#role = "editor"
organization = "MATTR Ltd"
[author.address]
email = "tobias.looker@mattr.global"

%%%

.# Abstract 

OpenID Providers within OpenID Connect assume many roles, one of these is providing End-User claims to relying parties at the consent of the End-User such as their name or date of birth. OpenID Connect defines multiple models under which claims are provided and relied upon by a relying parties, including simple, aggregated and distributed claims. This document focuses on elaborating upon the aggregated model outlined in section 5.6.2 of OpenID Connect core by defining the full life-cycle of aggregated claims and the new roles of the entities involved in an aggregated claims model.

.# Warning

This document is not an OIDF International Standard. It is distributed for review and comment. It is subject to change without notice and may not be referred to as an International Standard.

Recipients of this draft are invited to submit, with their comments, notification of any relevant patent rights of which they are aware and to provide supporting documentation.

.# Copyright notice & license

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.

.# Foreword

The OpenID Foundation (OIDF) promotes, protects and nurtures the OpenID community and technologies. As a non-profit international standardizing body, it is comprised by over 160 participating entities (workgroup participants). The work of preparing implementer drafts and final international standards is carried out through OIDF workgroups in accordance with the OpenID Process. Participants interested in a subject for which a workgroup has been established has the right to be represented in that workgroup. International organizations, governmental and non-governmental, in liaison with OIDF, also take part in the work. OIDF collaborates closely with other standardizing bodies in the related fields.

.# Introduction

OpenID Connect is a selective claims disclosure mechanism. When a set of claims included in its response is about 
an entity authentication event, it works as an entity authentication federation mechanism, but it can deliver 
other claims selected by the subject as well. 

In the OpenID Connect specification, it is assumed that there are many sources of claims. 
Each claim is associated with its authoritative source so there naturally will be many authoritative sources. 
Claim sources can be corroborative, i.e., not authoritative, as well. 
In total, there will be many Claim Set sources in OpenID Connect Framework. 
These Claim sources are called Issuing Authorities in OpenID Connect. 
Issuing-Authority (IA) is just an Identity-Agent (IdA), but it does not provide the claims about the current authentication event 
and its associated subject identifier authoritatively. Note that Issuing-Authority can act as an Identity-Agent in other transactions. 
Whether it is called IdA or IA is depending on their role in a particular transaction. 

There are four main actors in the OpenID Connect aggregated claims model.

1. End-User (Subject)
2. Issuing-Authority (IA) - Entity that is the issuer of claims about the End-User.
3. Identity-Agent (IdA) that provides claims about the subject authentication event and provides signed claim sets obtained from other Issuing Authorities
4. Claims-Consumer (CC) that verifies and relies upon the provided claims.

An IdA can provide an CC the claims by value or by reference. 

By value is the case where an IdA collects claims and their values 
from IAs and aggregate them in one package to provide to the CC. 
This model is called Aggregated Claims Model. 

By value is the case where the IdA does not collect and provide the value 
but just provide the reference and its access information to the CC. 
This model is called Distributed Claims Model. 

In either case, there has to be strong binding between the subject 
in the claim sets provided by IAs and the subject in the ID Token provided by the IdA. 
Conceptually, it can be done through Subject value correlation or 
through the correlation of signing key materials. 
Regardless of the methods, there has to be this binding. 
Otherwise, the provided claims cannot be trusted to be linked to the authenticated subject. 

This document defines how to do this binding even in the case of ephemeral subject identifier 
in which case the `sub` value of the ID Token will change every time the subject visits the CC. 
This allows anonymous attribute based authentication where an CC cannot link two visits 
by the subject without using other facilities. 

By supporting the case of ephemeral subject identifier, pairwise pseudonymous identifier 
and omni-directional identifier cases are also covered. 

Another feature that this document provides is the way to avoid multiple consent screen 
per CC authorization request. If OpenID Connect Core spec is used to build Aggregated Claims Model 
naively, it may result in many consent screens per CC request. 
For example, if four IAs and one IdA is involved in the request, then, there may be five consent screens. 
This is too onerous. This document defines a mechanism to consolidate it into one consent screen. 
This is done through one "IdA User Setup Phase" per IA that the IdA obtains the consent 
from the subject to obtain claims from the IA for the purpose of creating aggregated and distributed 
claims response for future CC requests in which IdA will collect a new consent from the subject. 

The mechanism used for this is to obtain an access token and a refresh token that corresponds 
to a suitably wide scope for the purpose. While the claims at the time of an CC request can be 
obtained from the UserInfo endpoint, this document defines a new endpoint called claims endpoint. 
It is almost the same as the UserInfo endpoint, but there are a few important differences: 

1. It allows collection minimization by supporting claims parameter. 
1. It allows an identifier to correlate the claims it is returning and the ID Token the IdA provides. 
1. It includes the `iss` claim. (Userinfo Endpoint does not)
1. It returns signed response. 
1. It allows multiple schema types for its response. (e.g, JWT and W3C Verifiable Credentials formats)

Note that while Userinfo Endpoint was conceived to support multiple response types, 
e.g., support for SCIM schema, 
the exact method was not specified at the time of the publication. 

This new Claims Endpoint allows the specification of the response schema and media type, 
e.g., OIDC JWT, OIDC4IDA JWT, W3C Verifiable Claims in JWT and JSON-LD, SCIM 2.0 in JWT, etc. 
It is done so in an extensible manner (using registry tbd). 

This implies that there will be an impact to the authentication and claims request that an CC makes. 
It may optionally want to specify its preferred format for the Claim Sets to be received. 
Therefore, this document also defines a new parameter to express its preference. 

There are four phases defined in this document. 

1. IA Discovery Phase: IdA discovers IA metadata. 
1. IdA Registration Phase: IdA registers to IA as an CC. 
1. Setup Phase: IdA obtains the access and refresh tokens from IA by the permission of the subject. 
1. CC Phase: 
    1. CC makes authentication and claims request, 
	1. IdA fetches relevant claim sets from IAs, 
	1. IdA respond to the CC
	1. the CC verifies the response. 

Note that distributed claims model is out of scope of this document. 

Intended reader of this document is the developer and systems architect who builds attributes and claims base systems. 

.# Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP
14 [RFC2119] [RFC8174] when, and only when, they appear in all
capitals, as shown here.

In the .txt version of this document, values are quoted to indicate that they are to be taken literally. When using these values in protocol messages, the quotes MUST NOT be used as part of the value. In the HTML version of this document, values to be taken literally are indicated by the use of this fixed-width font.

All uses of JSON Web Signature (JWS) JWS and JSON Web Encryption (JWE) JWE data structures in this specification utilize the JWS Compact Serialization or the JWE Compact Serialization; the JWS JSON Serialization and the JWE JSON Serialization are not used.



{mainmatter}

# Scope

This document specifies the methods for

* an Identity-Agent acting as a client of Issuing-Authority to perform discovery for an Issuing-Authority Metadata;
* the Identity-Agent to perform client registration to the Issuing-Authority;
* the Identity-Agent to obtain claims from the Issuing-Authority; 
* a Claims-Consumer to ask for verified claims to the Identity-Agent;  
* the Identity-Agent to return obtained claims from Issuing-Authority to the Claims-Consumer; and 
* the Claims-Consumer to verify the claims.

# Normative references

The following referenced documents are indispensable for the application of this document. For dated references, only the edition cited applied. For undated references, the latest edition of the referenced document (including any amendments) applies.

# Terms and definitions

For the purpose of this document, the terms defined in [RFC6749], [RFC6750], [RFC7636], [OpenID Connect Core][OIDC] apply. In addition, following terms are defined as a shorthand for the definition text that follows. 

**3.1**  
**Issuing-Authority**  
party that issues attested claims

**3.2**  
**Claims-Consumer**  
party that verifies and consumes the provided claim sets

**3.3**  
**Identity-Agent**  
party that acts as an RP to Issuing-Authorities and OP to Claims-Consumers

Note to entry: A Digital Identity Wallet is a type of IdA. 

# Symbols and abbreviated terms

**IA** – Issuing-Authority

**IdA** – Identity-Agent

**CC** – Claims-Consumer

**DID** - Decentralized Identifier

**HTTP** - Hyper Text Transfer Protocol

**TLS** - Transport Layer Security

**VC** - W3C Verifiable Credential

**VP** - W3C Verifiable Presentation

**W3C** - World Wide Web Consortium

# Actors

In this document, there are four main actors. 

1. Subject (User)
1. Issuing-Authority (IA)
1. Identity-Agent (IdA) 
1. Claims-Consumer(CC)  

They are topologically connected as in the following diagram.

!---
~~~ ascii-art
                +---------+
    +-----------| Subject |--------+
    | grants    +---------+        |
    v                |             |
+------+             | instructs   | allows
|  IA  |----+        |             |
+------+    |        v             v
   .        |    +-------+     +------+
   .        |----|  IdA  |-----|  CC  | 
   .        |    +-------+     +------+
+------+    |
|  IA  |----+
+------+  

~~~
!---
Figure: Relationships among actors


## Subject (User)

Subject is the entity that grants access to the claims at IAs and the IdA. 
In this system, the Subject grants IA to provide IdA the Claims for 
the purpose of providing those claims with other claims to potentially 
unspecified CCs under the Subject's direction. 

This request from the IdA to the IA is sent by the Subject's instruction. 
The Subject also allows IdA to potentially store the obtained claims. 

The Subject also allows CC to make a claims request to the IdA, 
typically for the Subject to receive some services from the CC. 

## Claims-Consumer (CC)

CC is an actor that typically provides some service to the Subject. 
To perform the service, the CC obtains some claims about the Subject from IdA. 
The basis for the processing of the Subject's claims by the CC can be 
performance of contract, consent, and other lawful basis. 

## Issuing-Authority (IA)

IA, Issuing-Authority, is a role assumed by an Identity-Agent 
that supports signed claims according to this document.  
Specifically, it supports an extension to bind the 
authentication response that CCs received from the IdA 
to the claim sets that it provides. 

The provision for the Issuing-Authority are as follows: 

1. It MUST implement Claims Endpoint. 
1. It MUST support the issuance of Access Tokens and Refresh Tokens for the Claims Endpoint. 
1. It MUST support the Discovery Metadata extension defined by this document. 
1. It MUST support the registration of the IdAs with extensions defined in this document. 
1. It SHOULD support the registration of the IdAs through Dynamic Registration. 

### Claims Endpoint
The Claims Endpoint is an OAuth 2.0 Protected Resource that returns Claims about the authenticated Subject. 
To obtain the requested Claims about the Subject, 
the IdA acting as a Client makes a request to the Claims Endpoint of the IA using an Access Token obtained through OpenID Connect Authentication. 
These Claims can be represented in variety of format as requested. 

This document defines the following request parameters for the Claims Endpoint request:

- *uid*  String value which identifies the end-user at the Client. The value is described in 5.3.2 of this specification. If this parameter is not supplied, the *uid* claims value MUST be supplied in the *claims* parameter.
- *claims*  This parameter is used to request that specific Claims be returned. This is a JSON object with only the *c_token* top-level member defined in 5.3.2 of this specification. The *c_token* member requests that the listed individual Claims be returned from the Claims Endpoint. The requested claims MUST only be a subset of the claims bounded to the Access Token as requested in the Authentication Request's *scope*, *claims*, or *request* or *request_uri* parameters. The *c_token* member MUST contain the *uid* claim value if the *uid* request parameter is not supplied.
- *aud*  JSON object containing a string array of client identifiers that will be added as additional *aud*  (audience) claims for the resulting JWT response from this endpoint. (Editor's NOTE: This point needs more discussion.)

** Editor's NOTE ** Is there a way to specify the CCs that are registered to the IdA? 

The provision for the Claims Endpoint are as follows: 

1. It MUST implement `uid`, `claims`, and `aud` as defined above. 
1. It MUST implement OIDC4IDA response encoded in JWS signed JWT as its response. 
1. It SHOULD support W3C Verified Credentials in JWT or JSON-LD representation as its response. 
1. It MAY support other response format as advertised in the Discovery response. 
1. It MUST enforce TLS for request and response. See  [Section 16.17](https://openid.net/specs/openid-connect-core-1_0.html#TLSRequirements)  for more information on using TLS.
1. It MUST support the use of the HTTP  GET  and HTTP  POST  methods defined in  [RFC 2616](https://openid.net/specs/openid-connect-core-1_0.html#RFC2616)  [RFC2616].
1. It MUST accept Access Tokens as  [OAuth 2.0 Bearer Token Usage](https://openid.net/specs/openid-connect-core-1_0.html#RFC6750)  [RFC6750].
1. It SHOULD support the use of  [Cross Origin Resource Sharing (CORS)](https://openid.net/specs/openid-connect-core-1_0.html#CORS)  [CORS] and or other methods as appropriate to enable JavaScript Clients to access the endpoint.

** EDITOR'S NOTE ** I guess there are other provisions. The above probably needs to be tweaked as well. 


## Identity-Agent (IdA)

IdA is an entity that acts as an Identity-Agent to the CC. 
Also, IdA acts as a Claims-Consumer to IAs. 

The provision for the IdA is as follows: 

1. It MUST support OpenID Connect Aggregated Claims as an Identity-Agent. 
1. It MUST act as an OpenID Connect Claims-Consumer to IAs to fetch claims from IAs according to instructions given by the Subject. 
1. As an Identity-Agent, IdA MUST implement mandatory to implement extensions that this document defines. 
1. It MAY store the signed claims obtained from IAs with appropriate safeguarding controls. 
1. To the authenticated Subject, it MUST provide a user interface to show what claims about the subject it stores. 
1. It MUST NOT provide claims to CCs without the Subject's permission. 
1. It MUST implement `c_token` authentication request parameter as a mechanism to request claims to IAs. 

** EDITOR'S NOTE ** I guess there are other provisions. 

### c_token 

`c_token` Authorization Request parameter lists individual Claims 
that the IdA asks the IA to be returned from the Claims Endpoint. 
This top-level member is a JSON object with the names of the individual Claims being requested 
as the member names, and the values are defined as in 5.5.1 of OpenID Connect 1.0 [OIDC].

# Discovery Phase

Before registering itself as an OAuth Client to an IA, the IdA needs to obtain 
configuration information from the IA, 
including its Authorization Endpoint and Token Endpoint locations. 

This information is obtained via Discovery, as described in OpenID Connect Discovery 1.0 [OpenID.Discovery], or may be obtained via other mechanisms.

This document adds the following Identity-Agent Metadata to the OpenID Connect Discovery 1.0 [OpenID.Discovery] response: 

* `claims_endpoint` **Required**. Claims Endpoint. URL at the Issuing-Authority that provides signed claims.
* `claims_signing_alg_values_supported` **Optional**. JSON array containing a list of the  JWS [JWS] signing algorithms (alg values) JWA [JWA] supported by the Claims Endpoint to encode the Claims in a  JWT [JWT]. The value *none* MUST NOT be included.
* `claims_encryption_alg_values_supported` **Optional**. JSON array containing a list of the  JWE [JWE] encryption algorithms (alg values) JWA [JWA] supported by the Claims Endpoint to encode the Claims in a JWT [JWT]. 
* `claims_encryption_enc_values_supported` **Optional**. JSON array containing a list of the  JWE [JWE] encryption algorithms (enc values) JWA [JWA] supported by the Claims Endpoint to encode the Claims in a JWT [JWT]. 

Additionally, the following optional OpenID Connect Discovery 1.0 [OpenID.Discovery] parameters are now required in the Issuing-Authority Metadata:

`claim_types_supported`
: The JSON array MUST contain the values *normal*, *distributed*, ... .

`claims_supported`
: A JSON array containing a list of the Claim Names of the Claims that the Identity-Agent MAY be able to supply values for.

`claims_parameter_supported`
: The value MUST be *true* to support the *claims* request parameter.

`request_parameter_supported`
: The value MUST be *true* to support the *request* request parameter.

`request_uri_parameter_supported`
: The value MUST be *true* to support the *request_uri* request parameter.

`claimset_supported`
: Boolean value indicating that the Issuing-Authority supports the claimset flows.




`claimset_formats_supported`
: A JSON array of strings identifying the resulting format of the claimset issued at the end of the flow.

`claimset_claims_supported`
: A JSON array of strings identifying the claim names supported within an issued claimset.

`claimset_name`
: A human-readable string to identify the name of the claimset offered by the provider.

`dids_supported`
: Boolean value indicating that the OpenID provider supports the resolution of [decentralized identifiers](https://w3c.github.io/did-core/).

`did_methods_supported`
: A JSON array of strings representing [Decentralized Identifier Methods](https://w3c-ccg.github.io/did-method-registry/) that the OpenID provider supports resolution of.

If the IA supports OpenID Connect for Identity Assurance 1.0 [OpenID.IDA], 
the supported OpenID Connect for Identity Assurance 1.0 [OpenID.IDA] features MUST be published 
as specified in section 7 of OpenID Connect for Identity Assurance 1.0 [OpenID.IDA].

If the IA supports W3C Verifiable Credentials, the IA MUST advertise it with the following metadata: 

** Editors Note: Tobias, could you fill in here? **

The following is a non-normative example of the relevant entries in the openid-configuration metadata for an OpenID Provider supporting the claimset issuance flow

```
{
  "dids_supported": true,
  "did_methods_supported": [
    "did:ion:",
    "did:elem:",
    "did:sov:"
  ],
  "claimset_supported": true,
  "claimset_endpoint": "https://server.example.com/credential",
  "claimset_formats_supported": [
    "w3cvc-jsonld",
    "jwt"
  ],
  "claimset_claims_supported": [
    "given_name",
    "last_name",
    "https://www.w3.org/2018/credentials/examples/v1/degree"
  ],
  "claimset_name": "University Credential"
}
```

If the IA supports mDL format, the IA MUST advertise it with the following metadata: 

** Editors Note: Tony or Kristina, could you fill in here? **

If the IA supports SCIM format, the IA MUST advertise it with the following metadata: 

** Editors Note: SCIM experts, could you fill in here? **

# Registration Phase

Before starting to make requests to an IA, the IdA MUST register itself to the IA. 
The registration SHOULD be performed 
via Dynamic Registration, as described in OpenID Connect Dynamic Client Registration 1.0.  

This document adds the following Client Metadata to the OpenID Connect Dynamic Client Registration :

`claims_signed_response_alg` **Required**. JWS `alg` algorithm JWA [JWA] REQUIRED for signing Claims Responses. The value *none* MUST NOT be used. If this is specified, the response will be JWT [JWT] serialized, and signed using JWS. The default, if omitted, is `RS256`.

`claims_encrypted_response_alg` **Optional**. JWE `alg` algorithm JWA [JWA] REQUIRED for encrypting Claims responses to the client. If both signing and encryption are requested, the response will be signed then encrypted, with the result being a Nested JWT, as defined in JWT [JWT]. The default, if omitted, is that no encryption is performed.

`claims_encrypted_response_enc` **Optional**. JWE `enc` algorithm JWA [JWA] REQUIRED for encrypting Claims responses. If `claims_encrypted_response_enc` is specified, the default for this value is `A128CBC-HS256`. When `claims_encrypted_response_enc` is included, `claims_encrypted_response_alg` MUST also be provided.

Authentication requests to the Issuing-Authority's Authorization Endpoint should be signed or signed and encrypted. In order to support a more diverse set of claims, requests for claims should be made using  Request Objects which are signed or signed and encrypted by registering the appropriate values for the following Client Metadata registration parameters:

- `request_object_signing_alg`
- `request_object_encryption_alg`
- `request_object_encryption_enc` 


# Setup Phase

## Overview

In this phase, the IdA obtains an access token (and optionally refresh token) 
that is bound to the current user so that the IdA can obtain the claims about the current user 
from the IA subsequently without taking the user to the IA and show them the consent dialogue for every CC requests.

1. This has to be done at least once for each IA that a user of an IdA who wishes to use the facility this document explains.
1. To obtain the grant, the IdA MUST use OpenID Connect Authentication Request. 

** Editor's NOTE:** Originally, it had: The Claims to be granted MUST be specified with `c_token` parameter. 

## Authorization request

A Signed Claim Set Request uses the OpenID and OAuth2.0 request parameters as outlined 
in section [3.1.2.1](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest) of 
OpenID Connect core, except for the following additional constraints.

claimset_format
: REQUIRED. Determines the format of the signed claim set returned at the end of the flow, values supported by the IA are advertised in their openid-configuration metadata, under the `claimset_formats_supported` attribute.

sub_jwk
: OPTIONAL. Used when making a Signed Claimset Request, defines the key material the IdA is requesting the claim set to be bound to the key responsible for signing the request object. The value is a JSON Object that is a valid [JWK](https://tools.ietf.org/html/rfc7517).

** Editors Note: ** DISCUSS the following paragraph. the parameter `did` should be sent in the claims collection phase rather than here? 

did
: OPTIONAL. Defines the relationship between the key material the IdA is requesting the claimset to be bound to and a [decentralized identifier](https://w3c.github.io/did-core/). Processing of this value requires the IA to support the resolution of [decentralized identifiers](https://w3c.github.io/did-core/) which is advertised in their openid-configuration metadata, under the `dids_supported` attribute. The value of this field MUST be a valid [decentralized identifier](https://w3c.github.io/did-core/).

** Editors Note: ** DISCUSS the following paragraph. Normal client authentication should be a primary choice instead? 

Public private key pairs are used by a requesting IdA to establish a means of binding to the resulting signed claim set. 
An IdA making a Claims Request to an IA MUST prove control over this binding mechanism during the request, 
this is accomplished through the extended usage of a [signed request](https://openid.net/specs/openid-connect-core-1_0.html#SignedRequestObject) defined in OpenID Connect Core.

The IA MUST show a dialogue to the Subject explaining that the IdA will be 
getting signed claims set from this IA as appropriate to provide claims to CCs as directed 
by the Subject. 

The dialogue MUST provide the link to the `policy_url` provided by the IdA during its registration. 

The actual act of granting MUST involve active user interaction. 

The grant that is to be obtained in this phase SHOULD be sufficiently large so that it will reduce the 
number of times that IdA needs to take the Subject to the IA to obtain additional grants. 

## Token Endpoint Response

Successful and Error Authentication Response are in the same manner 
as [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html) 
with the `code` parameter always being returned with the Authorization Code Flow. ** DISCUSS ** 

On Request to the Token Endpoint the `grant_type` value MUST be `authorization_code` inline with the Authorization Code Flow and the `code` value included as a parameter.

The following is a non-normative example of a response from the token endpoint, whereby the `access_token` authorizes the Claimset Holder to request a `credential` from the claimset endpoint.

```
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
  "token_type": "bearer",
  "expires_in": 86400,
  "id_token": "eyJodHRwOi8vbWF0dHIvdGVuYW50L..3Mz"
}
```

# Delivery Phase (CC Phase)

## Overview

In Delivery Phase, the claims are delivered to CC. 
To do so, it typically goes through the following steps: 

1. (Claims Request) An CC makes an OpenID Connect Authentication Request with extension parameters defined in this document to the IdA. 
1. (Request Verification) The IdA verifies if the request is valid. 
1. (Subject Granting) The IdA shows dialogue to the Authenticated Subject if it grants the access to the claims and obtains grant from the Subject. 
1. (Claims Collection) The IdA accesses relevant Claims Endpoints with Access Tokens to collect signed claims. 
1. (Claims Delivery) The IdA delivers the collected claims through ID Token, UserInfo endpoint, and/or Claims Endpoint if it supports. 
1. (Claims Verification) The CC verifies the received response. 

Claims Collection MAY be done out of sync. That is, the signed claim sets can be obtained before the CC requests. 
This is typically the case when claims are provided through the W3C Verifiable Credentials container. 

## Claims Request by CC to IdA

For an CC to request claims according to this document, the CC 

1. MUST use the OpenID Connect Authentication Request with extension parameters defined in this document to the IdA; and 
1. MAY use Pushed Authorization Request. 

## CC authentication and the request verification

Upon receipt of the request, the IdA 

1. MUST verify that the request is not tampered and is from a valid registered CC if the request is signed; and 
1. MUST at least verify that the client_id is valid and make sure 
   that the authorization code that it is going to return will be bound to this client_id if the request is not signed. 

NOTE: CC MUST be authenticated at one point or another before completion of the transaction. 

## Subject Granting

After verifying the request, the IdA 

1. MUST authenticate the Subject if it has not yet been; 
1. MUST show the Subject what is being requested from the CC; 
1. MUST show the Subject the link to the CC provided policy_url; and 
1. MUST obtain grant from the Subject through explicit action. 

## Claims Collection

The IdA collects the required claims from relevant Claims Endpoints. 
This process can be performed before the CC's request, 
in which case IdA stores the obtained signed claim set for a later day use. 

The Claims Endpoint is an OAuth 2.0 Protected Resource that when called, 
returns Claims about the authenticated Subject in the form of a signed claim set. 
To obtain a claim set on behalf of the Subject, the IdA makes a request to the Claims Endpoint using 
an Access Token obtained through OpenID Connect Authentication stated above.

Communication with the Credential Endpoint MUST utilize TLS. See Section TBD for more information on using TLS.

The Claims Endpoint MUST support the use of HTTP POST methods defined in RFC 2616 [RFC2616].

The Claims Endpoint SHOULD support the use of Cross Origin Resource Sharing (CORS) [CORS] and or other methods as appropriate to enable JavaScript Clients to access the endpoint. 

The Claims Endpoint MUST support either MTLS or DPOP. 


### Claims Endpoint Request

For claims collection, the IdA 

1. MUST send OAuth protected resource request to the Claims Endpoint using previously obtained Access Token; 
1. MUST send `claims` parameter to minimize the collected claims to what is necessary; 
1. MUST send `uid` parameter if binding between the claim set and the response to the CC is sought; and 
1. SHOULD send `aud` parameter. 

Additionally, 

1. The OAuth protected resource request SHOULD be protected by MTLS. 
1. It is RECOMMENDED that the request use the HTTP POST method, and the Access Token be sent using the Authorization header field.

The following is a non-normative example of a Claims Endpoint Request:

```
      POST /claims HTTP/1.1      
      HTTP/1.1
      Host: server.example.com
      Authorization: Bearer SlAV32hkKG
      claims={"uid":"id8837395937","email":{"essential":true},"email_verified":{"essential":true}}&aud={["client1234"]}
```

** Editor's Note ** An alternative way. 

A non-normative example of a Signed Credential request.

```
      POST /claims HTTP/1.1
      Host: https://issuer.example.com
      Authorization: Bearer <access-token>
      Content-Type: application/json
{
  "request": <signed-jwt-request-obj>
}
```

where the decoded payload of the request parameter is as follows:

```
{
  "aud": "https://issuer.example.com",
  "iss": "https://wallet.example.com",
  "sub": "urn:uuid:dc000c79-6aa3-45f2-9527-43747d5962a5",
  "sub_jwk" : {
    "crv":"secp256k1",
    "kid":"YkDpvGNsch2lFBf6p8u3",
    "kty":"EC",
    "x":"7KEKZa5xJPh7WVqHJyUpb2MgEe3nA8Rk7eUlXsmBl-M",
    "y":"3zIgl_ml4RhapyEm5J7lvU-4f5jiBvZr4KgxUjEhl9o"
  },
  "claimset_format": "w3cvc-jwt",
  "nonce": "43747d5962a5",
  "iat": 1591069056,
  "exp": 1591069556
}
```

The process will be repeated to as many Claims Endpoints as necessary. 

### Signed Claimset Request using a Decentralized Identifier

**Usage of Decentralized Identifiers** 

TODO improve this section

[Decentralized identifiers](https://w3c.github.io/did-core/) are a resolvable identifier to a set of statements about the [did subject](https://w3c.github.io/did-core/#dfn-did-subjects) including a set of cryptographic material (e.g. public keys). Using this cryptographic material, a [decentralized identifier](https://w3c.github.io/did-core/) can be used as an authenticatable identifier in a claimset, rather than using a public key directly.


An IdA submitting a signed Claim Set Request can request, 
that the resulting claim set be bound to the IdA through the usage of [decentralized identifiers](https://w3c.github.io/did-core/) by using the `did` field.

An IdA prior to submitting a claim set request SHOULD validate that the IA supports the resolution of decentralized identifiers 
by retrieving their openid-configuration metadata to check if an attribute of `dids_supported` has a value of `true`.

The IdA SHOULD also validate that the IA supports the [did method](https://w3c-ccg.github.io/did-method-registry/) 
to be used in the request by retrieving their openid-configuration metadata to check if an attribute of `did_methods_supported` contains the required did method.

An IA processing a claim set request featuring a [decentralized identifier](https://w3c.github.io/did-core/) MUST follow the following additional steps to validate the request.

1. Validate the value in the `did` field is a valid [decentralized identifier](https://w3c.github.io/did-core/).
2. Resolve the `did` value to a [did document](https://w3c.github.io/did-core/#dfn-did-documents).
3. Validate that the key in the `sub_jwk` field of the request is referenced in the [authentication](https://w3c.github.io/did-core/#authentication) section of the [DID Document](https://w3c.github.io/did-core/#dfn-did-documents).

If any of the steps fail then the IA MUST respond to the request with the Error Response parameter, 
[section 3.1.2.6.](https://openid.net/specs/openid-connect-core-1_0.html#AuthError) with Error code: `invalid_did`.

The following is a non-normative example of requesting the issuance of a claimset that uses a decentralized identifier.

```
{
  "response_type": "code",
  "client_id": "IAicV0pt9co5nn9D1tUKDCoPQq8BFlGH",
  "sub_jwk" : {
    "crv":"secp256k1",
    "kid":"YkDpvGNsch2lFBf6p8u3",
    "kty":"EC",
    "x":"7KEKZa5xJPh7WVqHJyUpb2MgEe3nA8Rk7eUlXsmBl-M",
    "y":"3zIgl_ml4RhapyEm5J7lvU-4f5jiBvZr4KgxUjEhl9o"
  },
  "did": "did:example:1234",
  "redirect_uri": "https://Client.example.com/callback",
  "claimset_format": "w3cvc-jsonld"
}
```



### Claims Endpoint Response

Upon receipt of the request, the Claims Endpoint Response is returned in the top level member of the 
JSON payload named `claimset`, 
of which the format is indicated by another top level member`format`. 

format : REQUIRED. The proof format the claimset was returned in. For example `oidc-jws` or `w3cvc-jsonld` or `w3cvc-jwt`.
claimset : REQUIRED. A cryptographically verifiable proof in the defined proof `format`. Most commonly a Linked Data Proof or a JWS.

The following is a non-normative example: 

```
{
  "format": "w3cvc-jsonld",
  "claimset": <claimset>
}
```

#### Claimset in OIDC JWS format

If the claimset is provided as oidc-jws, the claimset 

1. MUST be signed or signed and encrypted;
1. MUST NOT contain other claims than asked; 
1. MAY omit claims if not appropriate or available in which case the claim name MUST be omitted;
1. MUST provide correct content-type of the HTTP response; and 
1. MUST contain `aud` claim with is value a subset of what was in the request; 
1. MUST contain iss that is set as the IA's Issuer Identifier URL; 
1. MUST contain *op_iss* claim whose value is the IdA's issuer identifier URL registered to the IA; 
1. MUST contain *sub* claim that is set to the *uid* claim value if it was in the request; 

NOTE: the combination of *op_iss* and *sub* is used to correlated to the IdA response to the CC later. 

The following is a non-normative example of such:

```
  To be provided. 

```

#### Claimset in W3C VC format

Formats of the `claimset` can vary, examples include JSON-LD or JWT based Credentials, 
the IA SHOULD make their supported claimset formats available at their openid-configuration metadata endpoint.

The following is a non-normative example of a claim set issued as a [W3C Verifiable Credential 1.0](https://www.w3.org/TR/vc-data-model/) compliant format in JSON-LD.

```
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  "id": "http://example.gov/credentials/3732",
  "type": ["VerifiableCredential", "UniversityDegreeCredential"],
  "issuer": "did:key:z6MkjRagNiMu91DduvCvgEsqLZDVzrJzFrwahc4tXLt9DoHd",
  "issuanceDate": "2020-03-10T04:24:12.164Z",
  "credentialSubject": {
    "id": "urn:uuid:dc000c79-6aa3-45f2-9527-43747d5962a5",
    "jwk": {
      "crv":"secp256k1",
      "kid":"YkDpvGNsch2lFBf6p8u3",
      "kty":"EC",
      "x":"7KEKZa5xJPh7WVqHJyUpb2MgEe3nA8Rk7eUlXsmBl-M",
      "y":"3zIgl_ml4RhapyEm5J7lvU-4f5jiBvZr4KgxUjEhl9o"
    },
    "givenName": "John",
    "familyName": "Doe",
    "degree": {
      "type": "BachelorDegree",
      "name": "Bachelor of Science and Arts"
    }
  },
  "proof": {
    "type": "Ed25519Signature2018",
    "created": "2020-04-10T21:35:35Z",
    "verificationMethod": "did:key:z6MkjRagNiMu91DduvCvgEsqLZDVzrJzFrwahc4tXLt9DoHd#z6MkjRagNiMu91DduvCvgEsqLZDVzrJzFrwahc4tXLt9DoHd",
    "proofPurpose": "assertionMethod",
    "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..l9d0YHjcFAH2H4dB9xlWFZQLUpixVCWJk0eOt4CXQe1NXKWZwmhmn9OQp6YxX0a2LffegtYESTCJEoGVXLqWAA"
  }
}
```

The following is a non-normative example of a claims endpoint response for the request shown above.

```
{
  "format": "w3cvc-jsonld",
  "credential": {
    "@context": [
      "https://www.w3.org/2018/credentials/v1",
      "https://www.w3.org/2018/credentials/examples/v1"
    ],
    "id": "http://example.gov/credentials/3732",
    "type": ["VerifiableCredential", "UniversityDegreeCredential"],
    "issuer": "https://issuer.edu",
    "issuanceDate": "2020-03-10T04:24:12.164Z",
    "credentialSubject": {
      "id": "did:example:1234",
      "degree": {
        "type": "BachelorDegree",
        "name": "Bachelor of Science and Arts"
      }
    },
    "proof": {
      "type": "Ed25519Signature2018",
      "created": "2020-04-10T21:35:35Z",
      "verificationMethod": "https://issuer.edu/keys/1",
      "proofPurpose": "assertionMethod",
      "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..l9d0YHjcFAH2H4dB9xlWFZQLUpixVCWJk0eOt4CXQe1NXKWZwmhmn9OQp6YxX0a2LffegtYESTCJEoGVXLqWAA"
    }
  }
}
```

The following is a non-normative example of a Claim Set issued as a [JWT](https://tools.ietf.org/html/rfc7519)

```
ewogICJhbGciOiAiRVMyNTYiLAogICJ0eXAiOiAiSldUIgp9.ewogICJpc3MiOiAiaXNzdWVyIjogImh0dHBzOi8vaXNzdWVyLmVkdSIsCiAgInN1YiI6ICJkaWQ6ZXhhbXBsZToxMjM0NTYiLAogICJpYXQiOiAxNTkxMDY5MDU2LAogICJleHAiOiAxNTkxMDY5NTU2LAogICJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy9leGFtcGxlcy92MS9kZWdyZWUiOiB7CiAgICAgImh0dHBzOi8vd3d3LnczLm9yZy8yMDE4L2NyZWRlbnRpYWxzL2V4YW1wbGVzL3YxL3R5cGUiOiAiQmFjaGVsb3JEZWdyZWUiLAogICAgICJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy9leGFtcGxlcy92MS9uYW1lIjogIkJhY2hlbG9yIG9mIFNjaWVuY2UgYW5kIEFydHMiCiAgfQp9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

And the decoded Claim Set of the JWT

```
{
  "iss": "https://issuer.example.com",
  "sub": "urn:uuid:dc000c79-6aa3-45f2-9527-43747d5962a5",
  "sub_jwk" : {
    "crv":"secp256k1",
    "kid":"YkDpvGNsch2lFBf6p8u3",
    "kty":"EC",
    "x":"7KEKZa5xJPh7WVqHJyUpb2MgEe3nA8Rk7eUlXsmBl-M",
    "y":"3zIgl_ml4RhapyEm5J7lvU-4f5jiBvZr4KgxUjEhl9o"
  },
  "iat": 1591069056,
  "exp": 1591069556,
  "https://www.w3.org/2018/credentials/examples/v1/degree": {
    "https://www.w3.org/2018/credentials/examples/v1/type": "BachelorDegree",
    "https://www.w3.org/2018/credentials/examples/v1/name": "Bachelor of Science and Arts"
  }
}
```


### Claims Endpoint Error Response
 When an error condition occurs, the Claims Endpoint returns an Error Response as defined in Section 3 of  [OAuth 2.0 Bearer Token Usage](https://openid.net/specs/openid-connect-core-1_0.html#RFC6750)  [RFC6750]. (HTTP errors unrelated to RFC 6750 are returned to the User Agent using the appropriate HTTP status code.)

The following is a non-normative example of a Claims Endpoint Error Response:

```
    HTTP/1.1 401 Unauthorized
      WWW-Authenticate: error="invalid_token",
        error_description="The Access Token expired"
```

### Claims Endpoint Verification

Upon receipt of the response, the IdA 

1. MUST verify ... 
 
## Claims Delivery

Once the necessary claim sets were collected, 
the IdA creates the Aggregated Claims response to be returned. 

The response can be returned as ID Token or Userinfo Response ( or Claims Endpoint Response). 

The aggregated claims response is constructed as follows: 

1. The overall container format complies to what is described in 5.6.2 of OpenID Connect 1.0 [OIDC].
1. If the claim set was obtained as JWT, then it MUST be stored in the corresponding "JWT" member of the aggregated claims. 
1. If the claim set was obtained as W3C Verifiable Credential, then it MUST be stored in the corresponding "verifiable_presentations" member of the aggregated claims. 
1. If the claim set was obtained in other format, then ...


## Claims Verification

For Claims Verification,  

1. the CC MUST verify the signature of the Aggregate Claim according to  [JWS] using the algorithm specified in the JWT  *alg*  Header Parameter the keys provided by the Issuer of the Aggregate Claim for the signature verification; 
1. the CC MUST extract the signed claims from JWT and other relevant members and verify according to their verification rule; 
1. the CC MUST verify that issuers of the signed claims in the aggregated claims are the ones it trusts; 
1. the CC MUST verify that `op_iss` and `uid` values in the signed claims match the `iss` and `sub` value of the response; 
1. the CC MUST verify that the  aud  (audience) Claim contains its  client_id  value registered at the Issuer identified by the  iss  (issuer) Claim as an audience; 
1. The CC MUST verify that the aud claim does not contain claim values not trusted by the CC; and 
1. The CC MUST reject the response if any of the verification above fails. 

# Security Considerations

TBD

# Privacy Considerations

TBD

# References

## Normative references
The following referenced documents are indispensable for the application of this document. For dated references, only the edition cited applied. For undated references, the latest edition of the referenced document (including any amendments) applies.

[BCP14] - Key words for use in RFCs to Indicate Requirement Levels
[BCP14]: https://tools.ietf.org/html/bcp14

[RFC6749] - The OAuth 2.0 Authorization Framework
[RFC6749]: https://tools.ietf.org/html/rfc6749

[RFC6750] - The OAuth 2.0 Authorization Framework: Bearer Token Usage
[RFC6750]: https://tools.ietf.org/html/rfc6750

[RFC7636] - # Proof Key for Code Exchange by OAuth Public Clients
[RFC7636]: https://tools.ietf.org/html/rfc76360

[BCP212] - OAuth 2.0 for Native Apps
[BCP212]: https://tools.ietf.org/html/bcp212

[RFC6819] - OAuth 2.0 Threat Model and Security Considerations
[RFC6819]: https://tools.ietf.org/html/rfc6819

[BCP195] - Recommendations for Secure Use of Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS)
[BCP195]: https://tools.ietf.org/html/bcp195

[OIDC] - OpenID Connect Core 1.0 incorporating errata set 1
[OIDC]: https://openid.net/specs/openid-connect-core-1_0.html

[OpenID.Discovery] - OpenID Connect Discovery 1.0
[OpenID.Discovery]: https://openid.net/specs/openid-connect-discovery-1_0.html

[OpenID.Registration] - OpenID Connect Registration 1.0
[OpenID.Registration]: http://openid.net/specs/openid-connect-registration-1_0.html

[OpenID.IDA] - OpenID Connect for Identity Assurance 1.0
[OpenID.IDA]:https://openid.net/specs/openid-connect-4-identity-assurance-1_0-ID1.html

[MTLS] - OAuth 2.0 Mutual TLS Client Authentication and Certificate Bound Access Tokens
[MTLS]: https://tools.ietf.org/html/draft-ietf-oauth-mtls

[JWT] - JSON Web Token
[JWT]: http://tools.ietf.org/html/draft-ietf-oauth-json-web-token

[JWS] - JSON Web Signature
[JWS]: http://tools.ietf.org/html/draft-ietf-jose-json-web-signature

[JWE] - JSON Web Encryption
[JWE]: http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption

[JWA] - JSON Web Algorithms
[JWA]: http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms

{backmatter}
