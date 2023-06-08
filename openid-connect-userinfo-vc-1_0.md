%%%
title = "OpenID Connect UserInfo Verifiable Credentials - Draft 00"
abbrev = "UserInfo VC"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid", "verifiable credential"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-connect-userinfo-vc-1_0-00"
status = "standard"

[[author]]
initials="M."
surname="Ansari"
fullname="Morteza Ansari"
organization="Individual"
    [author.address]
    email = "mansari@robotsource.com"

[[author]]
initials="R."
surname="Barnes"
fullname="Richard Barnes"
organization="Cisco Systems"
    [author.address]
    email = "rlb@ipv.sx"

[[author]]
initials="P."
surname="Kasselman"
fullname="Pieter Kasselman"
organization="Microsoft"
    [author.address]
    email = "pieter.kasselman@microsoft.com"

[[author]]
initials="K."
surname="Yasuda"
fullname="Kristina Yasuda"
organization="Microsoft"
    [author.address]
    email = "kristina.yasuda@microsoft.com"

%%%

.# Abstract

The OpenID Connect UserInfo endpoint provides user attributes to OpenID Clients.
Providing these attributes in the form of a Verifiable Credential enables new
use cases.  This specification defines a new Verifiable Credential type
"UserInfoCredential" for this purpose, and defines a profile of the OpenID for
Verifiable Credential Issuance protocol for issuing these credentials.  We also
define an interoperable profile of the StatusList2021 credential revocation
mechanism, and a mechanism for distributing OpenID Provider JWK Sets that
enables credential verification even if the OpenID Provider is unreachable.

{mainmatter}

# Introduction

The OpenID Connect UserInfo endpoint is the channel by which an OpenID Provider
(OP) exposes user attributes to OpenID Client.  The standard claims defined in
the OpenID Connect Core specification [@!OpenID.Core] are widely supported,
forming the basis of much of today's identity ecosystem.

However, the bearer token structure of OpenID Connect means that each party that
wants to authenticate a user's attributes needs to be independently registered
as a Client of the OP and needs to do an independent OpenID Connect interaction
to authenticate the user.  This constraint is not too onerous in the cases where
OpenID Connect is typically deployed today, but it rules out other use cases.
For example, in the End-to-End Identity use case discussed in
(#end-to-end-identity), each individual user would have to be registered as a
Client of every OP involved in a session, which clearly does not scale.

Verifiable Credentials provide a more flexible model for distributing
information about a user [@!W3C.vc-data-model].  In this model, an Issuer that
knows a user's identity attributes issues a Verifiable Credential to a Holder
that acts on behalf of the user.  The Holder can then present the Verifiable
Credential to any Verifier that trusts the issuer, without the Verifier or the
verification transaction having to be known to the Issuer.

``` aasvg
+--------+               +--------+                   +----------+
| Issuer |--(issuance)-->| Holder |--(presentation)-->| Verifier |
+--------+               +--------+                   +----------+
    ^                                                      .
    .                                                      .
    .......................(trust)..........................
```

This specification defines the interfaces required for an OpenID Provider to
expose the information provided by the UserInfo endpoint in the form of a
Verifiable Credential:

* A new "UserInfoCredential" Verifiable Credential type that carries UserInfo claims
* A profile of the OpenID for Verifiable Credential Issuance [@!OpenID4VCI]
* A profile of the StatusList2021 mechanism for credential revocation [@!StatusList2021]
* A new mechanism for asynchronously distributing an OpenID Provider's JWK Set

Together, these interfaces allow the issuance of Verifiable Credentials
containing OpenID Connect claims with the same degree of simplicity and
interoperability as OpenID Connect itself.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 [@!RFC2119]. 

We use the terminology defined in [@!OpenID.Core], [@!W3C.vc-data-model], and
[@!OpenID4VCI].

This document introduces the following terms:

UserInfo Verifiable Credential (UserInfo VC):
: A Verifiable Credential carrying a set of claims that would be provided by the
OpenID Connect UserInfo endpoint, as specified in
(#userinfo-verifiable-credential-type)

Signed JWK Set:
: A JWK Set for an OpenID Provider, encapsulated on a JWT as described in
(#signed-openid-provider-jwk-sets).  Distributing an OP's keys in a Signed JWK
Set allows them to be trusted even when the OP's discovery endpoint is
unreachable.


# Overview

Since this specification is mainly a profile of OpenID for Verifiable Credential
Issuance [@!OpenID4VCI], the overall flow of the protocol is the same.
(#fig-overview) shows how an OP is involved in the issuance and verification of
a UserInfo Verifiable Credential: 

1. The Client (Holder in the VC model) obtains the OP's server metadata, which
   indicates support for issuance of UserInfo VCs.

1. The Client issues an authorization request including the
   `userinfo_credential` scope to request authorization to obtain UserInfo VCs,
   as well as the `openid` scope and other scopes to request access to UserInfo
   information (e.g., `profile`, `email`).

1. The OP authenticates the user and obtains the user's consent to credential
   issuance.

1. The Authorization Response provides the Client with an authorization code

1. The Client sends a Token Request containing the authorization code

1. The Token Response provides the Client with:
    * An access token that can be used to access the credential endpoint
    * A nonce that can be used to prove possession of a private key

1. The Client sends a Credential Request specifying that it desires a UserInfo
   VC, together with a proof that it controls the private key of a signature key
   pair.

1. The Credential Response contains a UserInfo VC that attests to the following
   attributes of the Holder:
    * The claims that would have been provided by the UserInfo endpoint
    * The public key corresponding to the private key used to compute the proof
      in the Credential Request

1. The Client sends the credential to a Verifier in some presentation protocol
   (outside the scope of this document)

1. The Verifier fetches the OP's JWK Set

1. The Verifier uses a public key from the OP's JWK Set to verify the signature
   on the credential.

``` aasvg
+--------+                                    +--------+              +----------+
| Client |                                    |   OP   |              |          | 
|   ==   |                                    |   ==   |              | Verifier |
| Holder |                                    | Issuer |              |          |
+---+----+                                    +---+----+              +-----+----+
    |                                             |                         |
    | 1. Obtain Issuer server metadata            |                         |
    |    (types=UserInfoCredential)               |                         |
    |<--------------------------------------------+                         |
    |                                             |                         |
    | 2. Authorization Request                    |                         |
    |    (scope=userinfo_credential)              |                         |
    +-------------------------------------------->|                         |
    |                                             |                         |
    |~~~~~~ 3. User authentication / consent ~~~~~|                         |
    |                                             |                         |
    | 4. Authorization Response (code)            |                         |
    |<--------------------------------------------+                         |
    |                                             |                         |
    | 5. Token Request (code)                     |                         |
    +-------------------------------------------->|                         |
    |                                             |                         |
    | 6. Token Response (access_token, c_nonce)   |                         |
    |<--------------------------------------------+                         |
    |                                             |                         |
    | 7. Credential Request                       |                         |
    |    (type=UserInfoCredential, proof)         |                         |
    +-------------------------------------------->|                         |
    |                                             |                         |
    | 8. Credential Response (credential)         |                         |
    |<--------------------------------------------+                         |
    |                                             |                         |
    |                                                                       |
    |~~~~~~~~~~~~~~~~~ 9. Presentation protocol (credential) ~~~~~~~~~~~~~~~|
    |                                                                       |
    |                                             |                         |
    |                                             | 10. Obtain Issuer JWKS  |
    |                                             +------------------------>|
    |                                             |                         |
    |                                             | 11. Verify credential   |
    |                                             |                 .-------+
    |                                             |                |        |
    |                                             |                 '------>|
    |                                             |                         |
```
Figure: Issuance, presentation, and verification of a UserInfo VC
{#fig-overview}


# Out of Scope

The focus of this specification is on the Verifiable Credential issuance
interaction, and on the processes to verify credentials so issued.  We do not
address the presentation interaction.  This step can be accomplished with a
standard presentation protocol such as OpenID for Verifiable Presentations
[@!OpenID4VP], or via integration with another protocol that provides
proof-of-possession mechanics, such as MLS or TLS [@!I-D.ietf-mls-protocol]
[@!RFC8446].

This specification is intended as a minimal profile of the more general OpenID
for VC Issuance mechanism, and places no constraints on how an OP implements VC
Issuance for credential types other than `UserInfoCredential`.


# UserInfo Verifiable Credential Type

A UserInfo Verifiable Credential enapsulates the claims that the OP offers via
the UserInfo endpoint as a Verifiable Credential that can be presented to a
third-party Verifier.  Having the UserInfo endpoint and the Credential Endpoint
return the same set of claims allows for the re-use of existing mechanisms for
negotiating which claims are provided to a client (e.g., the `profile` and
`email` scopes).  A UserInfo VC is distinguished from other Verifiable
Credentials by including the `UserInfoCredential` value in its list of types.

A UserInfo VC issued by an OpenID Provider MUST satisfy the following
requirements:

* An UserInfo VC MUST be represented as a JWT-formatted VC, as specified in
  Section 6.3.1 of [@!W3C.vc-data-model].
  
    * The `alg`, `kid`, and `typ` fields in the JWT header and the `exp`, `iss`,
      `nbf`, and `jti` claims MUST be populated as specified in that section.

    * The `sub` claim at the top level MUST be omitted. The `id` field in the
      `credentialSubject` object is used instead, as describe below.

    * The corresponding subfields of the `vc` claim SHOULD be omitted.

* The `kid` field in the JWT header MUST be set to the identifier of the public
  key in the OP's JWK Set that should be used to verify the credential.

* The `iss` claim MUST be set to the OP's Issuer Identifier.

* The `aud` claim MAY be omitted.  If present, it MUST contain the OAuth 2.0
  `client_id` of the Client, just as in an OpenID Connect ID Token.  Note that
  this value represents the Holder of the VC, not the Verifier to whom it will
  be presented.

* In the `vc` claim, the `@context` field MUST be a JSON array with the
  following single entry:
  * `"https://www.w3.org/2018/credentials/v1"`

* In the `vc` claim, the `type` field MUST be a JSON array with the following
  two entries, in order:
  * `"VerifiableCredential"`
  * `"OpenIDCredential"`

* In the `vc` claim, the `credentialSubject` field MUST be a JSON object.
  * The `id` field of this object a MUST be a Decentralized Identifier with DID
    method `jwk` [@!W3C.did-core] [@!W3C.did-spec-registries], reflecting the
    public key that the credential subject presented in their credential request
    (see (#credential-request)).
  * The other fields in this object MUST include all of the claims that would
    be returned on a successful UserInfo request made with the access token
    that was used in the Credential Request.

* In the `vc` claim, the `credentialStatus` field MAY be populated.  If it is
  populated, then it MUST meet the requirements of (#revocation-information).

An UserInfo VC is thus a JWT that can be signed and verified in largely the same
way as the other JWTs produced by OpenID Connect (e.g., ID tokens), but using
the VC syntax to present a public key for the credential subject in addition to
the claims provided by the OP.

A non-normative example of a UserInfo VC is shown below:

```
JWT Header:
{
  "alg": "ES256",
  "typ": "JWT",
  "kid": "XTSGmh734_J6fOWUbI7BNim7wyvj5LWx8GzuIH7WHw8"
}

JWT Payload:
{
  "vc": {
    "@context": [
      "https://www.w3.org/2018/credentials/v1"
    ],
    "type": [
      "VerifiableCredential",
      "UserInfoCredential"
    ],
    "credentialSubject": {
      "id": "did:jwk:eyJrdHkiOiJFQyIsInVzZSI6InNpZyIsImNydiI6IlAtMjU2Iiwi
             eCI6InFpR0tMd1hSSm1KUl9BT1FwV09IWExYNXVZSWZ6dlB3RHVyV3ZtWkJ3
             dnciLCJ5IjoiaXA4bnl1THBKNU5wcmlaekNWS2lHMFR0ZXFQTWtyemZOT1VR
             OFl6ZUdkayIsImFsZyI6IkVTMjU2In0",
      "sub": "248289761001",
      "name": "Jane Doe",
      "given_name": "Jane",
      "family_name": "Doe",
      "preferred_username": "j.doe",
      "email": "janedoe@example.com",
      "picture": "http://example.com/janedoe/me.jpg",
      "phone_number": "+1 202 555 1212"
    }
    "credentialStatus": {
      "id": "https://server.example.com/credentials/status/3#94567"
      "type": "StatusList2021Entry",
      "statusPurpose": "revocation",
      "statusListIndex": "94567",
      "statusListCredential": "https://server.example.com/credentials/status/3"
    }
  },
  "iat": 1667575982,
  "exp": 1668180782,
  "iss": "https://server.example.com"
}
```
Figure: The contents of an OpenID Verifiable Credential


## Revocation Information

If present, credential status information in a UserInfo VC MUST use the
StatusList2021 mechanism [@!StatusList2021].  This mechanism provides a concise
list of credentials revoked by an OP in a "status list credential". Status list
credentials for OIDC VCs MUST meet the following requirements, in addition to
the requirements of [@!StatusList2021]:

* A status list credential MUST be represented as a JWT-formatted VC, as
  specified in Section 6.3.1 of [@!W3C.vc-data-model].  The `alg`, `kid`, and
  `typ` fields in the JWT header and the `exp`, `iss`, `nbf`, `jti`, and `sub`
  claims MUST be populated as specified in that section.  The corresponding
  subfields of the `vc` claim SHOULD be omitted.

* The `iss` claim MUST be equal to the `iss` claim of the credential being
  validated.

* The `jti` claim, if present, MUST be equal to the `statusListCredential` field
  of the credential being validated.

* The `vc` claim MUST NOT contain a `credentialStatus` field.

* The `statusPurpose` value in the `credentialSubject` object MUST be `revocation`.
  Suspension of UserInfo VCs is not supported.


```
JWT header = {
  "alg": "ES256",
  "kid": "50615383-48AA-454D-B1E8-8721FBB7D7D1",
  "typ": "JWT"
}

JWT payload = {
  "iss": "https://server.example.com/",
  "iat": 1617632860,
  "exp": 1618237660,
  "jti": "https://server.example.com/credentials/status/3",
  "sub": "https://server.example.com/status/3#list"

  "vc": {
    "@context": [
      "https://www.w3.org/2018/credentials/v1",
      "https://w3id.org/vc/status-list/2021/v1"
    ],
    "type": [
      "VerifiableCredential",
      "StatusList2021Credential"
    ],
    "credentialSubject": {
      "type": "StatusList2021",
      "statusPurpose": "revocation",
      "encodedList": "H4sIAAAAAAAAA-3BMQEAAADCoPVPbQwfoAAAAAAAAAAAAAAAAAAAAIC3AYbSVKsAQAAA"
    },
  },
}
```
Figure: A status list credential


## UserInfo VC Verification

A Verifier processing an OIDC VC MUST validate it using the following steps:

1. Verify that the `alg` value represents an algorithm supported by the
   Verifier.

1. Obtain a JWK Set for the Issuer from a trusted source.  For example:

    * Obtain and verify a Signed JWK Set (see (#signed-openid-provider-jwk-sets))
    * Extract the OP's Issuer ID from the `iss` claim in the payload of the VC,
      and use OpenID Connect Discovery [@!OpenID.Discovery]:
        * Send a Discovery request for the specified Issuer Identifier
        * Fetch the JWK Set referenced by the `jwks_uri` field in the provider metadata
        * Identifies the key in the JWK Set corresponding to the `kid` field in the VC

1. Identify a key in the JWK Set that has the same `kid` value as the JWT Header
   of the UserInfo VC.

1. Use the identified key to verify the signature on the UserInfo VC.

1. Verify that the current time is after the time represented in the `nbf` claim
   (if present) and before the time represented by the `exp` claim.

1. If the `vc` claim has a `credentialStatus` field, verify the VC's
   revocation status as follows:
    1.  Fetch the status list credential from URL in the `statusListCredential`
        field of the `credentialStatus` object.

    1. Verify that the credential meets the criteria above.

    1. Verify the signature and expiration status of the status list credential.

    1. Perform the "Validate Algorithm" defined in [@!StatusList2021].

    1. If the final step returns `true`, then the Verifier MUST reject the
       certificate as revoked (depending on the `statusPurpose`).

    1. If any step in revocation status checking fails, then the Verifier SHOULD
       reject the credential.


# Profile of OpenID for VC Issuance

An OP implementing this specification MUST support OpenID for Verifiable
Credential Issuance [@!OpenID4VCI], supporting at least profile defined
in this section.  The Wallet-initiated, authorized code flow MUST be supported,
since this corresponds to the most common usage pattern for OpenID Connect.

Endpoints that are optional in the general OpenID for Verifiable Credential
Issuance specification are also optional here.

The remainder of this section specifies additional requirements for a
single, interoperable flow for issuing UserInfo VCs.


## Server Metadata

The server's metadata MUST have a `supportedCredentials` entry for the format
`jwt_vc_json`, with the following properties:

  * `cryptographic_suites_supported` MUST be present.
  * `cryptographic_binding_methods_supported` MUST be present and MUST include `jwk`.
  * `types` MUST include the values `VerifiableCredential` and `UserInfoCredential`.

A non-normative example server metadata object is shown below:

```
{
  // ... other server metadata fields ...
  "credential_endpoint": "https://server.example.com/connect/credential",
  "credentials_supported": [{
    "format": "jwt_vc_json",
    "types": [
        "VerifiableCredential",
        "UserInfoCredential"
    ],
    "cryptographic_binding_methods_supported": [
        "jwk"
    ],
    "cryptographic_suites_supported": [
        "ES256"
    ],
  }]
}
```


## Authorization Endpoint

A Client requests authorization to obtain UserInfo VCs by including the scope
value `userinfo_credential` in its authorization request.  The OP MUST support
this scope value.

A non-normative example authorization request is shown below:

```
GET /auth?client_id=C6pfRp679ez9HvDhg3TgI
   &scope=openid%20email%20profile%20userinfo_credential
   &response_type=code
   &redirect_uri=https%3A%2F%2Foidc-client.invalid%3A4000%2Fcb
   &code_challenge=7slr54gqLAj4gAc_FHYo9xx9pcFrACc-DSyofu7SjMk
   &code_challenge_method=S256 HTTP/1.1
```


## Token Endpoint

If the Client has been granted the `userinfo_credential` scope, then the Token
Response MUST include an `access_token` value that can be used to access the
Credential Endpoint.

The Token Response SHOULD include `c_nonce` and `c_nonce_expires_in` fields.
This avoids the need for a client to make an additional request to the
Credential Endpoint to obtain a nonce.

A non-normative example token request and response are shown below:

```
POST /token HTTP/1.1
Accept: application/json
Authorization: Basic /* base64(client_id + ':' + client_secret)*/

grant_type=authorization_code
&code=L8_66PBAv_nIO4LsXCW1mj2P9M2j5J8MXBQQxdQW6mC
&redirect_uri=https%3A%2F%2Foidc-client.invalid%3A4000%2Fcb
&code_verifier=aipxCdREzMCkTnBZVjLUG8mHNSXErrfQ9P6YqzT5hfU

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "access_token": "S_-WDgsXG8s3V7qrTbI3kZHf4mIcTByLPU2Rd5FoDib",
  "expires_in": 3600,
  "id_token": "eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IjFQVzVyT3pDWTNI
               RFIzb2FvX3ZsSTBDY2JEN1J2WXVCTFJjdEJtZ24wQWsifQ.eyJzdWIiOiJiY
               WNrZW5kLjExMzMwNDE4ODMwNjI2NzA4NzMyNCIsImF0X2hhc2giOiItNkhkZ
               EU5aE1TQnNFSlFtS2tjdERRIiwiYXVkIjoiQzZwZlJwNjc5ZXo5SHZEaGczV
               GdJIiwiZXhwIjoxNjY3NTc4MTYxLCJpYXQiOjE2Njc1NzQ1NjEsImlzcyI6I
               mh0dHBzOi8vbG9jYWxob3N0OjMwMDAifQ.IJfXA64d--xEt-TATIZnFLdSMd
               H6LZu_glRAGw_qLFyQ8kXnxlNFeMGQqQ2PSA6vtO9zDyH3CHzJSfEAHYUQ2Q",
  "scope": "openid email profile userinfo_credential",
  "c_nonce": "N7cbtWrD1Uoq3TOxS87QiQ",
  "c_nonce_expires_in": "86400",
}
```

Note that the `id_token` field is included here because this is an OpenID
Connect token request.  It is not necessary for VC issuance, but may be used by
the Client in the same way as an ID token normally would be.


## Credential Endpoint

### Credential Request

Consistent with the metadata requirements in (#server-metadata), the OP's
Credential Endpoint MUST support Credential Requests with `format` field set to
`jwt_vc_json`, and a `type` field set to an array including
`VerifiableCredential` and `UserCredential`.  For such requests, the OP MUST
require the client to prove possession of a private key.  The OP MUST support
the `jwt` proof type, with the client's public key presented in the `jwk` header
field in the proof JWT.

A non-normative example credential request is shown below:

```
POST /credential HTTP/1.1
Authorization: Bearer S_-WDgsXG8s3V7qrTbI3kZHf4mIcTByLPU2Rd5FoDib
Content-Type: application/json

{
  "format": "jwt_vc_json"
  "type": ["VerifiableCredential", "UserInfoCredential"],
  "proof": {
    "proof_type": "jwt",
    "jwt": "eyJhbGciOiJFUzI1NiIsImp3ayI6eyJrdHkiOiJFQyIsImNydiI6IlAtMjU2
            IiwieCI6InFpR0tMd1hSSm1KUl9BT1FwV09IWExYNXVZSWZ6dlB3RHVyV3Zt
            WkJ3dnciLCJ5IjoiaXA4bnl1THBKNU5wcmlaekNWS2lHMFR0ZXFQTWtyemZO
            T1VROFl6ZUdkayIsImFsZyI6IkVTMjU2In19.eyJhdGgiOiJHVXF5LWpySFV
            Rdll3QXkwQXZ4RGRtOTYweVYxdXJCZWExejl4UGo1UGZvIiwiaXNzIjoiYk9
            4OFI0NFhoSUhpM0E5NjV6QkVuIiwiYXVkIjoiaHR0cHM6Ly9zZXJ2ZXIuZXh
            hbXBsZS5jb20iLCJub25jZSI6Ik43Y2J0V3JEMVVvcTNUT3hTODdRaVEiLCJ
            pYXQiOjE2Njc1NzU5ODIsImp0aSI6IlhGbHpQWEFmSWxYR3ExMHFZRmVUejh
            Zak5EbjhSUzNwRHMzZVJPVlJ4VkUifQ.NRHs_6_j1SR45FQJpdapyxAYSJch
            qnOFiRSbsicfhNWFppbWwOxvkpDPhfxb9a1Usxi7kmUnPEpV278QkiddHw"

  }
}
```

The content of the `proof` JWT is as follows:

```
JWT Header:
{
  "alg": "ES256",
  "jwk": {
    "kty": "EC",
    "crv": "P-256",
    "x": "qiGKLwXRJmJR_AOQpWOHXLX5uYIfzvPwDurWvmZBwvw",
    "y": "ip8nyuLpJ5NpriZzCVKiG0TteqPMkrzfNOUQ8YzeGdk",
    "alg": "ES256"
  }
}


JWT Payload:
{
  "ath": "GUqy-jrHUQvYwAy0AvxDdm960yV1urBea1z9xPj5Pfo",
  "iss": "bOx8R44XhIHi3A965zBEn",
  "aud": "https://server.example.com",
  "nonce": "N7cbtWrD1Uoq3TOxS87QiQ",
  "iat": 1667575982,
  "jti": "XFlzPXAfIlXGq10qYFeTz8YjNDn8RS3pDs3eROVRxVE"
}
```


### Credential Response

A successful Credential Response to a Credential Request for a UserInfo VC MUST
contain a `credential` field.  Deferred credential issuance MUST NOT be used.

A non-normative example credential response is shown below:

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "format": "jwt_vc_json",
  "credential": "eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IlhUU0dtaDczNF9K
                 NmZPV1ViSTdCTmltN3d5dmo1TFd4OEd6dUlIN1dIdzgifQ.eyJ2YyI6eyJAY
                 29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFsc
                 y92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiVXNlckluZ
                 m9DcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImlkIjoiZGlkO
                 mp3azpleUpyZEhraU9pSkZReUlzSW5WelpTSTZJbk5wWnlJc0ltTnlkaUk2S
                 WxBdE1qVTJJaXdpZUNJNkluRnBSMHRNZDFoU1NtMUtVbDlCVDFGd1YwOUlXR
                 XhZTlhWWlNXWjZkbEIzUkhWeVYzWnRXa0ozZG5jaUxDSjVJam9pYVhBNGJub
                 DFUSEJLTlU1d2NtbGFla05XUzJsSE1GUjBaWEZRVFd0eWVtWk9UMVZST0ZsN
                 lpVZGtheUlzSW1Gc1p5STZJa1ZUTWpVMkluMCIsInN1YiI6IjI0ODI4OTc2M
                 TAwMSIsIm5hbWUiOiJKYW5lIERvZSIsImdpdmVuX25hbWUiOiJKYW5lIiwiZ
                 mFtaWx5X25hbWUiOiJEb2UiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJqLmRvZ
                 SIsImVtYWlsIjoiamFuZWRvZUBleGFtcGxlLmNvbSIsInBpY3R1cmUiOiJod
                 HRwOi8vZXhhbXBsZS5jb20vamFuZWRvZS9tZS5qcGciLCJwaG9uZV9udW1iZ
                 XIiOiIrMSAyMDIgNTU1IDEyMTIifX0sImlhdCI6MTY2NzU3NTk4MiwiZXhwI
                 joxNjY4MTgwNzgyLCJpc3MiOiJodHRwczovL3NlcnZlci5leGFtcGxlLmNvb
                 SJ9.4kg6Pi8Xz8B85qUiXkEVlTRQpfqUpr725ZIq0YMx1yMIgE7otkTvXdAp
                 pbMHbLbO9-nItJHLd7x-WR5g8rqksQ"
}
```

The contents of the credential in the example are as follows:

```
JWT Header:
{
  "alg": "ES256",
  "typ": "JWT",
  "kid": "XTSGmh734_J6fOWUbI7BNim7wyvj5LWx8GzuIH7WHw8"
}

JWT Payload:
{
  "vc": {
    "@context": [
      "https://www.w3.org/2018/credentials/v1"
    ],
    "type": [
      "VerifiableCredential",
      "UserInfoCredential"
    ],
    "credentialSubject": {
      "id": "did:jwk:eyJrdHkiOiJFQyIsInVzZSI6InNpZyIsImNydiI6IlAtMjU2Iiwi
             eCI6InFpR0tMd1hSSm1KUl9BT1FwV09IWExYNXVZSWZ6dlB3RHVyV3ZtWkJ3
             dnciLCJ5IjoiaXA4bnl1THBKNU5wcmlaekNWS2lHMFR0ZXFQTWtyemZOT1VR
             OFl6ZUdkayIsImFsZyI6IkVTMjU2In0",
      "sub": "248289761001",
      "name": "Jane Doe",
      "given_name": "Jane",
      "family_name": "Doe",
      "preferred_username": "j.doe",
      "email": "janedoe@example.com",
      "picture": "http://example.com/janedoe/me.jpg",
      "phone_number": "+1 202 555 1212"
    }
  },
  "iat": 1667575982,
  "exp": 1668180782,
  "iss": "https://server.example.com"
}
```


### Priming Requests

The OP's Credential Endpoint MUST also support "priming" requests, in which the
`format` field is set to `jwt_vc_json`, and a `type` field set to an array
including `VerifiableCredential` and `UserCredential`, but no `proof` field is
provided. 

The response to such requests MUST be a standard credential error response with
status code 400 and `error_code` set to `missing_proof`.  `c_nonce` and
`c_nonce_expires_in` fields MUST be provided.

Priming requests are used by clients to obtain a fresh nonce, e.g., when one is
not provided by the token endpoint.

An non-normative example priming request and error response are shown below:

```
POST /credential HTTP/1.1
Authorization: Bearer S_-WDgsXG8s3V7qrTbI3kZHf4mIcTByLPU2Rd5FoDib
Content-Type: application/json

{
  "format": "jwt_vc_json"
  "type": ["VerifiableCredential", "UserInfoCredential"],
}

HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8

{
  "format": "jwt_vc",
  "c_nonce": "7mVx0A6QRgYPlXC0SkEtNw",
  "c_nonce_expires_in": 86400
}
```


## Client Nonce Handling

This section is non-normative.

Because of the requirement for proof of possession, clients will need to have a
fresh nonce before they can successfully request a credential.  The OP can
provide that nonce either in a successful Token Response or in a success or
error response from the Credential Endpoint. 

When a client receives a nonce via any of these channels, it should store the
nonce and use it to construct a proof in the next credential request it makes.
Each nonce should only be used once.  Expired nonces should be deleted.

If a client wishes to make a Credential Request and does not have a nonce
(e.g., because all of its nonces expired), then it should send a priming
request of the form described in (#priming-requests).  The response to this
request will be an error response, but will contain a nonce that can be used to
construct a proof for a credential request.


# Signed OpenID Provider JWK Sets

One benefit of verifiable credentials is a looser coupling between the
credential Issuer (here the OP) and the Verifier.  The Verifier only needs to
know how to verify signatures from a trusted Issuer; the Issuer don't need to know
anything about the Verifier.

Verifiers can discover the JWK Set for a given Issuer OP using OpenID Connect
Discovery [@!OpenID.Discovery].  However, this risks introducing a requirement
that the Issuer's discovery endpoint be online at the time of verification.  In
order to avoid such a requirement, this section defines a mechanism for an OP to
sign its JWK Set to prove its authenticity to verifiers.  Such a signed JWK Set
can be provided to a Verifier by an untrusted party (for example, the party
presenting a credential), not just the OP.

A Verifier requests a signed JWK Set by sending an HTTP GET request to the OP's
JWKS URL with the value `application/jwt` in the HTTP `Accept` header field.

An OP provides a signed JWK Set in a response to such a request by sending a
response containing a JWT of the following form:

* The `x5c` field of the JWT header MUST be populated with a certificate chain
  that authenticates the domain name in the OP's Issuer Identifier.  The host
  name in the Issuer Identifier MUST appear as a `dNSName` entry in the
  `subjectAltName` extension of the end-entity certificate.

* The `alg` field of the JWT header MUST represent an algorithm that is
  compatible with the subject public key of the certificate in the `x5c`
  parameter.

* The `iss` claim of the JWT MUST contain the OP's Issuer ID.

* The JWT SHOULD NOT contain an `aud` claim.

* The JWT MUST contain an `exp` claim.

* The JWT MUST conatin a `jwks` claim, whose value is the OP's JWK Set.

A non-normative example of a request and response for a Signed JWK Set is shown
below:

```
GET /jwks HTTP/1.1
Host: server.example.com
Accept: application/jwt

HTTP/1.1 200 OK
Content-Type: application/jose

eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsIng1YyI6WyJNSUlDNXpDQ0Fs
QUNBUUV3RFFZSktvWklodmNOQVFFRkJRQXdnYnN4SkRBaUJnTlZCQWNURzFa
aGJHbERaWEowSUZaaGJHbGtZWFJwYjI0Z1RtVjBkMjl5YXpFWE1CVUdBMVVF
Q2hNT1ZtRnNhVU5sY25Rc0lFbHVZeTR4TlRBekJnTlZCQXNUTEZaaGJHbERa
WEowSUVOc1lYTnpJRElnVUc5c2FXTjVJRlpoYkdsa1lYUnBiMjRnUVhWMGFH
OXlhWFI1TVNFd0h3WURWUVFERXhob2RIUndPaTh2ZDNkM0xuWmhiR2xqWlhK
MExtTnZiUzh4SURBZUJna3Foa2lHOXcwQkNRRVdFV2x1Wm05QWRtRnNhV05s
Y25RdVkyOXRNQjRYRFRrNU1EWXlOakF3TVRrMU5Gb1hEVEU1TURZeU5qQXdN
VGsxTkZvd2dic3hKREFpQmdOVkJBY1RHMVpoYkdsRFpYSjBJRlpoYkdsa1lY
UnBiMjRnVG1WMGQyOXlhekVYTUJVR0ExVUVDaE1PVm1Gc2FVTmxjblFzSUVs
dVl5NHhOVEF6QmdOVkJBc1RMRlpoYkdsRFpYSjBJRU5zWVhOeklESWdVRzlz
YVdONUlGWmhiR2xrWVhScGIyNGdRWFYwYUc5eWFYUjVNU0V3SHdZRFZRUURF
eGhvZEhSd09pOHZkM2QzTG5aaGJHbGpaWEowTG1OdmJTOHhJREFlQmdrcWhr
aUc5dzBCQ1FFV0VXbHVabTlBZG1Gc2FXTmxjblF1WTI5dE1JR2ZNQTBHQ1Nx
R1NJYjNEUUVCQVFVQUE0R05BRENCaVFLQmdRRE9PbkhLNWF2SVdaSlYxNnZZ
ZEE3NTd0bjJWVWRaWlVjT0JWWGM2NWcyUEZ4VFhkTXd6empzdlVHSjdTVkND
U1JyQ2w2emZOMVNMVXptMU5aOVdsbXBaZFJKRXkwa1RSeFFiN1hCaFZRNy9u
SGswMXhDK1lEZ2tSb0tXemsyWi9NL1ZYd2JQN1JmWkhNMDQ3UVN2NGRrK05v
Uy96Y253Yk5EdSs5N2JpNXA5d0lEQVFBQk1BMEdDU3FHU0liM0RRRUJCUVVB
QTRHQkFEdC9VRzl2VUpTWlNXSTRPQjlMK0tYSVBxZUNnZllyeCtqRnp1ZzZF
SUxMR0FDT1RiMm9XSCtoZVFDMXUrbU5yMEhaRHpUdUlZRVpvREpKS1BURWps
YlZValA5VU5WK21Xd0Q1TWxNL010c3EyYXpTaUdNNWJVTU1qNFFzc3hzb2R5
YW1Fd0NXL1BPdVo2bGNnNUt0ejg4NWhabytMN3RkRXk4VzlWaUgwUGQiXX0.
eyJpYXQiOjE2Njc1NzU5ODIsImV4cCI6MTY2ODE4MDc4MiwiaXNzIjoiaHR0
cHM6Ly9zZXJ2ZXIuZXhhbXBsZS5jb20iLCJqd2tzIjp7ImtleXMiOlt7Imt0
eSI6IkVDIiwiY3J2IjoiUC0yNTYiLCJhbGciOiJFUzI1NiIsImtpZCI6IlhU
U0dtaDczNF9KNmZPV1ViSTdCTmltN3d5dmo1TFd4OEd6dUlIN1dIdzgiLCJ4
IjoicWlHS0x3WFJKbUpSX0FPUXBXT0hYTFg1dVlJZnp2UHdEdXJXdm1aQnd2
dyIsInkiOiJpcDhueXVMcEo1TnByaVp6Q1ZLaUcwVHRlcVBNa3J6Zk5PVVE4
WXplR2RrIn1dfX0.w32l5L84A6OVo-_pFKI3eWYExpNDAWdwOuyw9LDQo2Kh
Nqz0KU_m4VdrPbmipITDrhhhkxwkkv-e-9w0cXsh1Q
```

The contents of the Signed JWK Set in the example are as follows (omitting the
full certificate chain):

```
JWT Header:
{
  "alg": "ES256",
  "typ": "JWT",
  "x5c": ["MII..."]
}

JWT Payload:
{
  "iat": 1667575982,
  "exp": 1668180782,
  "iss": "https://server.example.com",
  "jwks": {
    "keys": [{
      "kty": "EC",
      "crv": "P-256",
      "alg": "ES256",
      "kid": "XTSGmh734_J6fOWUbI7BNim7wyvj5LWx8GzuIH7WHw8",
      "x": "qiGKLwXRJmJR_AOQpWOHXLX5uYIfzvPwDurWvmZBwvw",
      "y": "ip8nyuLpJ5NpriZzCVKiG0TteqPMkrzfNOUQ8YzeGdk"
    }]
  }
}
```

A Verifier that receives such a signed JWK Set validates it by taking the
folloinwg steps:

1. If this Signed JWK Set was looked up using an Issuer ID, verify that the
   Issuer ID in the `iss` claim is identical to the one used to discover it.

1. Verify that the JWT has not expired, according to its `exp` claim.

1. Verify that the certificate chain in the `x5c` field is valid from a trusted
   certificate authority (see [@!RFC5280][@!RFC6125]).

1. Verify that the end-entity certificate matches the Issuer ID.

1. Verify the signature on the JWS using the subject public key of the
   end-entity certificate


# Security Considerations {#security-considerations}

TODO

# Implementation Considerations

TODO

# Privacy Considerations

TODO

{backmatter}

# Use Cases

## End-to-End Identity

Many applications today provide end-to-end encryption, which protects against
inspection or tampering by the communications service provider.  Current
applications, however, have only very manual techniques for verifying the
identity of other users in a communication, for example, verifying key
fingerprints in person.  E2E encryption without identity verification is like
HTTPS with self-signed certificates – vulnerable to impersonation attacks.

When appropriately integrated in an E2E encryption system, OpenID Verifiable
Credentials could eliminte the risk of impersonation attacks.  A participant in
an E2E-secure session would be able to present identity information that the
other participants could verify as coming from a trusted OP, and thus
protected from tampering by the application's servers.

In this regard, the OP would be the Issuer, and the Holder and Verifier roles
would be played by the user agent software in the E2E encrypted application.  A
user agent would act as Holder when proving its user's identity to others, and
as a Verifier when authenticating other participants in a session.

## OpenID for Verifiable Presentations

The OpenID for Verifiable Presentations specification defines a general
mechanism by which OpenID mechanisms are used to implement the VC presentation
interaction between a Holder and a Verifier.  Through this interaction, a Holder
presents one or more credentials to a Verifier and proves that the Holder is the
legitimate subject of those credentials.  As a result, the Verifier can
associate the attributes in the credentials with the Holder.

UserInfo VCs are a specific type of Verifiable Credential, so they can be used
in this protocol to present OpenID claims to a Verifier.  For example, a
UserInfo VC containing a `email` claim could tell the Verifier that the Holder
legitimately represents that email address (assuming the Verifier trusts the OP
to make such claims).


<reference anchor="StatusList2021" target="https://www.w3.org/TR/vc-data-model">
  <front>
    <title>Status List 2021</title>
    <author fullname="Manu Sporny">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Dave Longley">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Orie Steele">
      <organization>Transmute</organization>
    </author>
    <author fullname="Mike Prorock">
      <organization>mesur.io</organization>
    </author>
    <author fullname="Mahmoud Alkhraishi">
      <organization>Mavennet</organization>
    </author>
   <date day="16" month="Jun" year="2022"/>
  </front>
</reference>

<reference anchor="OpenID.Core" target="http://openid.net/specs/openid-connect-core-1_0.html">
  <front>
    <title>OpenID Connect Core 1.0 incorporating errata set 1</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="M." surname="Jones" fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
    <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
      <organization>Google</organization>
    </author>
    <author initials="C." surname="Mortimore" fullname="Chuck Mortimore">
      <organization>Salesforce</organization>
    </author>
   <date day="8" month="Nov" year="2014"/>
  </front>
</reference>

<reference anchor="OpenID.Discovery" target="https://openid.net/specs/openid-connect-discovery-1_0.html">
  <front>
    <title>OpenID Connect Discovery 1.0 incorporating errata set 1</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="M." surname="Jones" fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
    <author initials="E." surname="Jay" fullname="Edmund Jay">
      <organization>Illumila</organization>
    </author>
   <date day="8" month="Nov" year="2014"/>
  </front>
</reference>

<reference anchor="OpenID4VP" target="https://openid.net/specs/openid-4-verifiable-presentations-1_0.html">
      <front>
        <title>OpenID for Verifiable Presentations</title>
        <author initials="O." surname="Terbu" fullname="Oliver Terbu">
         <organization>ConsenSys Mesh</organization>
        </author>
        <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
          <organization>yes.com</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>Microsoft</organization>
        </author>
        <author initials="A." surname="Lemmon" fullname="Adam Lemmon">
          <organization>Convergence.tech</organization>
        </author>
        <author initials="T." surname="Looker" fullname="Tobias Looker">
          <organization>Mattr</organization>
        </author>
       <date day="20" month="June" year="2022"/>
      </front>
</reference>

<reference anchor="OpenID4VCI" target="https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html">
      <front>
        <title>OpenID for Verifiable Credential Issuance</title>
        <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
          <organization>yes.com</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>Microsoft</organization>
        </author>
        <author initials="T." surname="Looker" fullname="Tobias Looker">
          <organization>Mattr</organization>
        </author>
       <date day="6" month="September" year="2022"/>
      </front>
</reference>

# IANA Considerations

TBD

# Acknowledgements {#Acknowledgements}

TBD

# Notices

Copyright (c) 2022 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.

# Appendix

# Document History

   [[ To be removed from the final specification ]]

   -00 

   *  initial revision

