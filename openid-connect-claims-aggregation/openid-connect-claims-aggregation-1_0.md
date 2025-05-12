%%%
title = "OpenID Connect Claims Aggregation Draft 03"
abbrev = "oidc-ca"
ipr = "none"
workgroup = "connect"
keyword = ["security", "openid", "privacy"]

[seriesInfo]
name = "OpenID-Draft"
value = "openid-connect-claims-aggregation-1_0"
status = "standard"

[pi]
subcompact = "yes"
private = "Draft-03"
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

%%%

.# Abstract 

OpenID Providers within OpenID Connect assume many roles, one of these is providing End-User claims to relying parties at the consent of the End-User such as their name or date of birth. OpenID Connect defines multiple models under which claims are provided and relied upon by a relying parties, including simple, aggregated and distributed claims. This document focuses on elaborating upon the aggregated model outlined in section 5.6.2 of OpenID Connect core by defining the full life-cycle of aggregated claims and the new roles of the entities involved in an aggregated claims model.

This document specifies the methods for

* an OP acting as a client of Claims Provider (CP) to perform discovery for a Claims Provider Metadata;
* the OP to perform client registration to the Claims Provider;
* the OP to obtain claims from the Claims Provider; 
* an RP to ask for verified claims to the OP;  
* the OP to return obtained claims from CP to the RP; and 
* the RP to verify the claims.

.# Warning

This document is not an OIDF International Standard. It is distributed for review and comment. It is subject to change without notice and may not be referred to as an International Standard.

Recipients of this draft are invited to submit, with their comments, notification of any relevant patent rights of which they are aware and to provide supporting documentation.

.# Notices

Copyright (c) 2025 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer,
implementer, or other interested party a non-exclusive, royalty free,
worldwide copyright license to reproduce, prepare derivative works from,
distribute, perform and display, this Implementers Draft, Final
Specification, or Final Specification Incorporating Errata Corrections
solely for the purposes of (i) developing specifications,
and (ii) implementing Implementers Drafts, Final Specifications,
and Final Specification Incorporating Errata Corrections based
on such documents, provided that attribution be made to the OIDF as the
source of the material, but that such attribution does not indicate an
endorsement by the OIDF.

The technology described in this specification was made available
from contributions from various sources, including members of the OpenID
Foundation and others. Although the OpenID Foundation has taken steps to
help ensure that the technology is available for distribution, it takes
no position regarding the validity or scope of any intellectual property
or other rights that might be claimed to pertain to the implementation
or use of the technology described in this specification or the extent
to which any license under such rights might or might not be available;
neither does it represent that it has made any independent effort to
identify any such rights. The OpenID Foundation and the contributors to
this specification make no (and hereby expressly disclaim any)
warranties (express, implied, or otherwise), including implied
warranties of merchantability, non-infringement, fitness for a
particular purpose, or title, related to this specification, and the
entire risk as to implementing this specification is assumed by the
implementer. The OpenID Intellectual Property Rights policy
(found at openid.net) requires
contributors to offer a patent promise not to assert certain patent
claims against other contributors and against implementers.
OpenID invites any interested party to bring to its attention any
copyrights, patents, patent applications, or other proprietary rights
that may cover technology that may be required to practice this
specification.

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
These Claim sources are called Claims Providers in OpenID Connect. 
Claims Provider (CP) is just an OpenID Provider (OP) but it does not provide the claims about the current authentication event 
and its associated subject identifier authoritatively. Note that Claims Provider can act as an OpenID Provider in other transactions. 
Whether it is called OP or CP is depending on their role in a particular transaction. 

There are four main actors in the OpenID Connect aggregated claims model.

1. Subject (User)
1. OpenID Provider (OP) that provides claims about the subject authentication event and provides signed claim sets obtained from other Claims Providers
1. Claims Providers (CP) that provides claims
1. Relying Party (RP) that verifies and consumes the provided claim sets. 

An OP can provide an RP the claims by value or by reference. 

By value is the case where an OP collects claims and their values 
from CPs and aggregate them in one package to provide to the RP. 
This model is called Aggregated Claims Model. 

By reference is the case where the OP does not collect and provide the value 
but just provide the reference and its access information to the RP. 
This model is called Distributed Claims Model. 

Another feature that this document provides is the way to avoid multiple consent screen 
per RP authorization request. If OpenID Connect Core spec is used to build Aggregated Claims Model 
naively, it may result in many consent screens per RP request. 
For example, if four CPs and one OP is involved in the request, then, there may be five consent screens. 
This is too onerous. This document defines a mechanism to consolidate it into one consent screen. 
This is done through one "OP User Setup Phase" per CP that the OP obtains the consent 
from the subject to obtain claims from the CP for the purpose of creating aggregated 
claims response for future RP requests in which OP will collect a new consent from the subject. 

The mechanism used for this is to obtain an access token and a refresh token that corresponds 
to a suitably wide scope for the purpose. 

The claims are returned from the Clams Provider's UserInfo endpoint and are always signed.

Note: Refresh tokens are optional in this specification, but profiles and trust frameworks may require the use of
refresh tokens.


There are four phases defined in this document. 

1. CP Discovery Phase: OP discovers CP metadata. 
1. OP Registration Phase: OP registers to CP as an RP. 
1. Setup Phase: OP obtains the access and refresh tokens from CP by the permission of the subject. 
1. RP Phase: 
    1. RP makes authentication and claims request, 
	1. OP fetches relevant claim sets from CPs, 
	1. OP respond to the RP
	1. the RP verifies the response. 

Intended reader of this document is the developer and systems architect who builds attributes and claims base systems. 



.# Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP
14 [RFC2119] [RFC8174] when, and only when, they appear in all
capitals, as shown here.

{mainmatter}

# Scope

This document specifies the methods for

* an OP acting as a client of Claims Provider (CP) to perform discovery for a Claims Provider Metadata;
* the OP to perform client registration to the Claims Provider;
* the OP to obtain claims from the Claims Provider; 
* the OP to return obtained claims from CP to the RP; and 
* the RP to verify the claims.

# Normative references
The following referenced documents are indispensable for the application of this document. For dated references, only the edition cited applied. For undated references, the latest edition of the referenced document (including any amendments) applies.

[BCP14] - Key words for use in RFCs to Indicate Requirement Levels
[BCP14]: https://tools.ietf.org/html/bcp14

[RFC6749] - The OAuth 2.0 Authorization Framework
[RFC6749]: https://tools.ietf.org/html/rfc6749

[RFC6750] - The OAuth 2.0 Authorization Framework: Bearer Token Usage
[RFC6750]: https://tools.ietf.org/html/rfc6750

[OIDC] - OpenID Connect Core 1.0 incorporating errata set 1
[OIDC]: https://openid.net/specs/openid-connect-core-1_0.html

[OpenID.Discovery] - OpenID Connect Discovery 1.0
[OpenID.Discovery]: https://openid.net/specs/openid-connect-discovery-1_0.html

[OpenID.Registration] - OpenID Connect Registration 1.0
[OpenID.Registration]: http://openid.net/specs/openid-connect-registration-1_0.html

[JWT] - JSON Web Token
[JWT]: http://tools.ietf.org/html/draft-ietf-oauth-json-web-token

[JWS] - JSON Web Signature
[JWS]: http://tools.ietf.org/html/draft-ietf-jose-json-web-signature

[JWE] - JSON Web Encryption
[JWE]: http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption

[JWA] - JSON Web Algorithms
[JWA]: http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms


# Terms and definitions
For the purpose of this document, the terms defined in [RFC6749], [RFC6750], [RFC7636], [OpenID Connect Core][OIDC] and the following apply.

**Claim Set** – a set of claims issued by a Claims Provider


# Symbols and abbreviated terms

**RP** – Relying Party

**OP** – OpenID Provider

**CP** – Claims Provider

**HTTP** - Hyper Text Transfer Protocol

**TLS** - Transport Layer Security

# Actors

In this document, there are four main actors. 

1. Subject (User)
1. CP
1. OP 
1. RP that verifies and consumes the provided claim sets. 

They are topologically connected as in the following diagram.

!---
~~~ ascii-art
               +---------+
    +----------| Subject |--------+
    | grants   +---------+        |
    v               |             |
+------+            | instructs   | allows
|  CP  |----+       |             |
+------+    |       v             v
   .        |    +------+     +------+
   .        |----|  OP  |-----|  RP  | 
   .        |    +------+     +------+
+------+    |
|  CP  |----+
+------+  

~~~
Figure: Relationships among actors


## Subject (User)

Subject is the entity that grants access to the claims at CPs and the OP. 
In this system, the Subject grants CP to provide OP the Claims for 
the purpose of providing those claims with other claims to potentially 
unspecified RPs under the Subject's direction. 

This request from the OP to the CP is sent by the Subject's instruction. 
The Subject also allows OP to potentially store the obtained claims. 

The Subject also allows RP to make a claims request to the OP, 
typically for the Subject to receive some services from the RP. 

## RP

RP is an actor that typically provides some service to the Subject. 
To perform the service, the RP obtains some claims about the Subject from OP. 
The basis for the processing of the Subject's claims by the RP can be 
performance of contract, consent, and other lawful basis. 

## CP

CP, Claims Provider, is a role assumed by an OpenID Provider 
that supports signed UserInfo response and has the OP as a client.  

The provision for the Claims Provider are as follows: 

1. It MUST support signed UserInfo response. 
1. It SHOULD support the Discovery Metadata extension defined by this document. 
1. It SHOULD support the registration of the OPs with extensions defined in this document. 
1. It SHOULD support the registration of the OPs through Dynamic Registration. 

### UserInfo Endpoint
The UserInfo Endpoint is described in 5.3 of OpenID Connect 1.0 [OIDC] as an OAuth 2.0 Protected Resource that returns Claims about the authenticated Subject.



## OP

OP is an entity that acts as an OpenID Provider to the RP. 
Also, OP acts as a relying party to CPs. 

The provision for the OP is as follows: 

1. It MUST support OpenID Connect Aggregated Claims as an OpenID Provider. 
1. It MUST act as an OpenID Connect relying party to CPs to fetch claims from CPs according to instructions given by the Subject. 
1. As an OpenID Provider, OP MUST implement mandatory to implement extensions that this document defines. 
1. It MAY store the signed claims obtained from CPs with appropriate safeguarding controls. 
1. To the authenticated Subject, it MUST provide a user interface to show what claims about the subject it stores. 
1. It MUST NOT provide claims to RPs without the Subject's permission. 



# Discovery Phase

Before registering itself as an OpenID Connect Client to a CP, the OP needs to obtain 
configuration information from the CP, 
including its Authorization Endpoint, Token Endpoint, and UserInfo Endpoint locations. 

This information is obtained via Discovery, as described in OpenID Connect Discovery 1.0 [OpenID.Discovery], or may be obtained via other mechanisms.

The following OpenID Connect Discovery 1.0 [OpenID.Discovery] parameters are required in the Claims Provider Metadata:

`userinfo_endpoint`
: **Required**. Claims Endpoint. URL at the Issuing-Authority that provides signed claims.

`userinfo_signed_response_alg`
: **Optional**. JSON array containing a list of the  JWS [JWS] signing algorithms (alg values) JWA [JWA] supported by the UserInfo Endpoint to encode the Claims in a  JWT [JWT]. The value *none* MUST NOT be included.


The following optional OpenID Connect Discovery 1.0 [OpenID.Discovery] parameters are now required in the OpenID Provider Metadata:

- `claim_types_supported`. The Claims Types supported by the Claims Provider. It MUST contain the value *aggregated*.


# Registration Phase

Before starting to make requests to a CP, the OP MUST register itself to the CP. 
The registration SHOULD be performed 
via Dynamic Registration, as described in OpenID Connect Dynamic Client Registration 1.0 [OpenID.Registration].  

The OP MUST register the following client parameters :

- `userinfo_signed_response_alg`. Signing algorithm supported by the Claims Provider.

In addition, the OP MUST NOT register `userinfo_encrypted_response_alg` value.


# Setup Phase

In this phase, the OP obtains an access token (and optionally refresh token) 
that is bound to the current user so that the OP can obtain the claims about the current user 
from the CP subsequently without taking the user to the CP and show them the consent dialogue for every RP requests.

1. This has to be done at least once for each CP that a user of an OP who wishes to use the facility this document explains.
1. To obtain the grant, the OP MUST use an OpenID Connect Authentication Request. 
1. The Claims to be granted SHOULD be specified with the`scope` parameter and/or the `userinfo` member of the `claims` parameter. 
The CP MUST show a dialogue to the Subject explaining that the OP will be 
getting signed claims set from this CP as appropriate to provide claims to RPs as directed 
by the Subject. 

The dialogue MUST provide the link to the `policy_url` provided by the OP during its registration. 

The actual act of granting MUST involve active user interaction. 

The grant that is to be obtained in this phase SHOULD be sufficiently large so that it will reduce the 
number of times that OP needs to take the Subject to the CP to obtain additional grants. 

# Delivery Phase (RP Phase)

In Delivery Phase, the claims are delivered to RP. 
To do so, it typically goes through the following steps: 

1. (Claims Request) An RP makes an OpenID Connect Authentication Request with extension parameters defined in this document to the OP. 
1. (Request Verification) The OP verifies if the request is valid. 
1. (Subject Granting) The OP shows dialogue to the Authenticated Subject if it grants the access to the claims and obtains grant from the Subject. 
1. (Claims Collection) The OP accesses relevant Claims Provider's UserInfo Endpoints with Access Tokens to collect signed claims
1. (Claims Verification) The RP verifies the received response. 

Claims Collection MAY be done out of sync. That is, the signed claim sets can be obtained before the RP requests. 


## Claims Request by RP to OP

For an RP to request claims according to this document, the RP 

1. MUST use the OpenID Connect Authentication Request with extension parameters defined in this document to the OP. 


## RP authentication and the request verification

Upon receipt of the request, the OP 

1. MUST verify that the request is not tampered and is from a valid registered RP if the request is signed; and 
1. MUST at least verify that the client_id is valid and make sure 
   that the authorization code that it is going to return will be bound to this client_id if the request is not signed. 

NOTE: RP MUST be authenticated at one point or another before completion of the transaction. 

## Subject Granting

After verifying the request, the OP 

1. MUST authenticate the Subject if it has not yet been; 
1. MUST show the Subject what is being requested from the RP; 
1. MUST show the Subject the link to the RP provided policy_url; and 
1. MUST obtain grant from the Subject through explicit action. 

## Claims Collection

The OP collects the required claims from the relevant UserInfo Endpoint. 
This process can be performed before the RP's request. 

To minimize the information returned from the UserInfo Endpoint, the OP SHOULD downscope the access token to return only the claims requested by the RP's authorization request. This OP retrieves a downscoped access token by making a request to the CP's Token Endpoint as described in 12 of OpenID Connect 1.0 [OIDC]. The `scope` parameter value SHOULD only allow access to the claims requested by the RP at the CP.  


### UserInfo Endpoint Request

For claims collection, the OP

1. MUST send OAuth protected resource request to the UserInfo Endpoint using previously obtained Access Token as described in 5.3.1 of OpenID Connect 1.0 [OIDC];

This process will be repeated for as each Claims Providers as necessary.

## Claims Delivery

Once the necessary claim sets were collected, 
the OP creates the Aggregated Claims response to be returned as described in 5.6.2 of OpenID Connect 1.0 [OIDC] . 

## Claims Verification

For Claims Verification,  

1. the RP MUST verify the signature of the Aggregate Claim according to  [JWS] using the algorithm specified in the JWT  *alg*  Header Parameter the keys provided by the Issuer of the Aggregate Claim for the signature verification; 
1. the RP MUST extract the signed claims from JWT and other relevant members and verify according to their verification rule; 
1. the RP MUST verify that issuers of the signed claims in the aggregated claims are the ones it trusts; 
1. the RP MUST verify that the *aud* (audience) Claim is that of the OP's Issuer Identifier;
1. The RP MUST reject the response if any of the verification above fails. 

# Security Considerations

The security considerations of OpenID Connect 1.0 [OIDC] and OpenID Connect Dynamic Client Registration 1.0 [OpenID.Registration] apply.

# Privacy Considerations

Unless ephemeral identifiers are used at CPs, colluding RPs CAN correlate the subject using the `sub` value in the JWT included in the aggregated claim.

{backmatter}
