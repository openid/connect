%%%
title = "Security and Trust in OpenID for Verifiable Credentials"
abbrev = "openid-4-vc-security-and-trust"
ipr = "none"
workgroup = "connect"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-4-vc-security-and-trust-1_0-00"
status = "standard"

[[author]]
initials="D."
surname="Fett"
fullname="Daniel Fett"
organization="yes.com"
    [author.address]
    email = "mail@danielfett.de"


[[author]]
initials="T."
surname="Lodderstedt"
fullname="Torsten Lodderstedt"
organization="yes.com"
    [author.address]
    email = "torsten@lodderstedt.net"

%%%

.# Abstract

This specification describes the trust architecture in OpenID Connect for Verifiable Credentials (VCs) and outlines security considerations for the use of VCs in OpenID Connect. 

{mainmatter}

# Introduction


# Terminology


**TODO: Check if all of the following terms are actually used.**


This specification uses the terms "Access Token", "Authorization Request", "Authorization Response", "Authorization Server", "Client", "Client Authentication", "Client Identifier", "Grant Type", "Response Type", "Token Request" and "Token Response" defined by OAuth 2.0 [@!RFC6749], the terms "End-User", "Entity", "Request Object", "Request URI" as defined by OpenID Connect Core [@!OpenID.Core], the term "JSON Web Token (JWT)" defined by JSON Web Token (JWT) [@!RFC7519], the term "JOSE Header" and the term "Base64url Encoding" defined by JSON Web Signature (JWS) [@!RFC7515], and the term "Response Mode" defined by OAuth 2.0 Multiple Response Type Encoding Practices [@!OAuth.Responses].

This specification also defines the following terms. In the case where a term has a definition that differs, the definition below is authoritative.

Credential:
:  A set of one or more claims about a subject made by a Credential Issuer. Note that this definition of a term "Credential" in this specification is different from that in [@!OpenID.Core].

Verifiable Credential (VC):
:  An Issuer-signed Credential whose authenticity can be cryptographically verified. Can be of any format used in the Issuer-Holder-Verifier Model, including, but not limited to those defined in [@VC_DATA], [@ISO.18013-5] (mdoc) and [@Hyperledger.Indy] (AnonCreds).

W3C Verifiable Credential:
:  A Verifiable Credential compliant to the [@VC_DATA] specification.

Presentation:
:  Data that is shared with a specific Verifier, derived from one or more Verifiable Credentials that can be from the same or different issuers.

Verifiable Presentation (VP):
:  A Holder-signed Credential whose authenticity can be cryptographically verified to provide Cryptographic Holder Binding. Can be of any format used in the Issuer-Holder-Verifier Model, including, but not limited to those defined in [@VC_DATA], [@ISO.18013-5] (mdoc) and [@Hyperledger.Indy] (AnonCreds).

Credential Issuer:
:  An entity that issues Verifiable Credentials. Also called Issuer.

Holder:
:  An entity that receives Verifiable Credentials and has control over them to present them to the Verifiers as Verifiable Presentations.

Verifier:
:  An entity that requests, receives and validates Verifiable Presentations. During presentation of Credentials, Verifier acts as an OAuth 2.0 Client towards the Wallet that is acting as an OAuth 2.0 Authorization Server. The Verifier is a specific case of OAuth 2.0 Client, just like Relying Party (RP) in [@OpenID.Core].

Issuer-Holder-Verifier Model:
:  A model for claims sharing where claims are issued in the form of Verifiable Credentials independent of the process of presenting them as Verifiable Presentation to the Verifiers. An issued Verifiable Credential can (but must not necessarily) be used multiple times.

Holder Binding: 
: Ability of the Holder to prove legitimate possession of a Verifiable Credential. 

Wallet:
:  An entity used by the Holder to receive, store, present, and manage Verifiable Credentials and key material. There is no single deployment model of a Wallet: Verifiable Credentials and keys can both be stored/managed locally, or by using a remote self-hosted service, or a remote third-party service. In the context of this specification, the Wallet acts as an OAuth 2.0 Authorization Server (see [@!RFC6749]) towards the Credential Verifier which acts as the OAuth 2.0 Client.


# Trust Model

Verifiable Credentials facilitate an indirect model to obtain and present authoritative claims about a subject. 

**TODO: add diagram**

In this model, the Issuer (the entity making an authoritative claim)
creates a Verifiable Credential and delivers it to the End-User (or her
Wallet, respectively). The Wallet/End-User can then present the Verifiable
Credential to a Verifier (the entity that needs to know the claim).

A Credential can be presented various times during its lifecycle and
multiple Credentials (or subsets of the claims in those Credentials) can
be presented to a Verifier in a single transaction. 

The End-User then decides what Credential to use with which Verifier.
The Verifier decides what Credential to accept from the End-User.

The involved parties need to access data about identifiers, keys, and
Credentials as well as data about the acting parties themselves in order
to securely implement the Credential exchange. The details of how this is achieved are defined in the Trust Framework.

All parties communicate with each other using a set of protocols, or
just "protocol" in the following. In the case of OpenID 4 VP, the protocols are
OpenID 4 Verifiable Credential Issuance and OpenID 4 Verifiable Presentations, with an
optional use of SIOP v2.

## Trust in the Issuer-Holder-Verifier Model

Trust in the Issuer-Holder-Verifier Model means different things to different parties. 

 * For Verifiers:
    * Issuer Identification: The Verifier can identify the Issuer to
      such a degree that it can trust that the Issuer is the entity it
      claims to be.
    * Authenticity of Claims: A Verifier can trust that the set of
      presented claims was issued by a specific Issuer and not altered
      by a malicious party.
    * Identity of Presenter: A Verifier can trust that the party presenting
      the claims is (controlled by) the subject of the claims. This is
      typically achieved by enforcing Holder Binding and trusting the
      Issuer to issue Credentials only to the End-Users they are about
      (see below).
    * Integrity of Interaction: When interacting with a Wallet, the
      Verifier can trust that it receives whatever data this Holder
      wants to release, not forged data from a third party. If there is
      an existing session between the Verifier and the Holder, the
      Verifier can trust that the presented claims are intended for this
      session.
 * For Holders/End-Users:
    * Privacy (I): A Holder can trust that no party learns any claims
      except Verifiers to whom the Holder has explicitly released the
      claims. 
    * Privacy (II): A Holder can trust that a Verifier only learns the
      claims that it intends to release to the Verifier and not more.
    * Context: A Holder can trust that if a Credential presentation is
      started in a user session with a website (acting as Verifier),
      this Website and this session is the receiver of the presented
      claims. 
      **TODO: Refine**
    * Correctness of Claims: A Holder can trust that the claims in a
      Credential are correct and not altered or influenced by a
      malicious party.
 * For Issuers:
   * ?


### Note on Holder Binding

Depending on the use case, the Verifier also needs to trust the End-User
to be the legitimate holder of a certain Credential. Accepting a
Credential from every bearer is a security risk:

 - A malicious party that has gained access to a Credential (e.g., data
   leaked from a connection or from a Verifier) can present it to a
   Verifier and thus impersonate the legitimate End-User.
 - A Verifier that acts in a benign way towards an End-User but misuses
   the presented Credentials can impersonate a legitimate End-User.

So anyone getting in possession of a Credential, e.g. through a security
breach, could pose as the legitimate End-User for a long time.

Therefore, Credentials are usually bound to the legitimate End-User,
also known as "holder binding". There are three ways to bind the
Credential to its legitimate holder:

 * biometric traits, e.g. a passport photograph. This method is well
   known from physical Credentials such as passports or id cards. If
   someone presents the Credential, the Verifier needs to compare the
   data in the Credential with the respective biometric traits of the
   presenter.  
 * holder claims, e.g. a diploma might be bound to a person named "Erika
   Mustermann". This binding requires a primary Credential to prove the
   presenter’s identity which in turn is bound to the legitimate holder
   using either biometric or cryptographic holder binding or both.  
 * cryptographic binding, i.e. the Credential is directly or indirectly
   bound to key material under the control of the legitimate holder.
   Only if the presenter can prove possession of the private key
   material, it is considered the legitimate holder of that Credential.
   Cryptographic binding is well suited for digital use cases and is
   more privacy preserving than biometric or claims-based binding since
   no or few (arbitrary) PII is required in order to implement the
   binding.


**TODO: incorporate this into the text:**

Holder Binding can mean that

 - the party that presents a credential (sends a presentation) is the
   subject of the credential, or
 - the party that presents a credential (sends a presentation) is a
   legitimate possessor of the credential.
 
Only with the assumption that an Issuer only issues a credential to the
subject of the credential, the first definition is equivalent to the
second one. 

> **Security Requirement:** Issuers must only issue Credentials to the
subject of the respective Credential. 

**TODO: incorporate this into the text - taken from SD-JWT:** Verifiers
MUST decide whether Holder Binding is required for a particular use case
or not before verifying a credential. This decision can be informed by
various factors including, but not limited to the following: business
requirements, the use case, the type of binding between a Holder and its
credential that is required for a use case, the sensitivity of the use
case, the expected properties of a credential, the type and contents of
other credentials expected to be presented at the same time, etc.

This can be showcased based on two scenarios for a mobile driver's
license use case for SD-JWT:

Scenario A: For the verification of the driver's license when stopped by
a police officer for exceeding a speed limit, Holder Binding may be
necessary to ensure that the person driving the car and presenting the
license is the actual Holder of the license. The Verifier (e.g., the
software used by the police officer) will ensure that a Holder Binding
JWT is present and signed with the Holder's private key. Claims-based
Holder Binding may be used as well, e.g., by including a first name,
last name and a date of birth that matches that of an insurance policy
paper.

Scenario B: A rental car agency may want to ensure, for insurance
purposes, that all drivers named on the rental contract own a
government-issued driver's license. The signer of the rental contract
can present the mobile driver's license of all named drivers. In this
case, the rental car agency does not need to check Holder Binding as the
goal is not to verify the identity of the person presenting the license,
but to verify that a license exists and is valid.

It is important that a Verifier does not make its security policy
decisions based on data that can be influenced by an attacker or that
can be misinterpreted. For this reason, when deciding whether Holder
binding is required or not, Verifiers MUST NOT take into account

whether an Holder qBinding JWT is present or not, as an attacker can
remove the Holder Binding JWT from any Presentation and present it to
the Verifier, or whether Holder Binding data is present in the SD-JWT or
not, as the Issuer might have added the key to the SD-JWT in a
format/claim that is not recognized by the Verifier. If a Verifier has
decided that Holder Binding is required for a particular use case and
the Holder Binding is not present, does not fulfill the requirements
(e.g., on the signing algorithm), or no recognized Holder Binding data
is present in the SD-JWT, the Verifier will reject the presentation, as
described in (TODO).



## Security and Privacy Requirements

To implement a secure trust chain, a number of Security and Privacy
Requirements must be met. In the following, the Security Requirements
are described along the trust chain from Verifier to Issuer, then
separately from the view of the End-User. All requirements are numbered
for reference and are prefixed with the respective party or component
that needs to implement the requirement:

  * V: Verifier
  * I: Issuer
  * W: Wallet

  * CF: Credential Format
  * TF: Trust Framework
  * P: Protocols (both for Issuance and Verification)

Trust Framework here refers to the design of relationships of parties
with each other, their roles and permissions within an ecosystem, the
verification of their real-world identities and the mechanisms selected
and provided for key creation and distribution.

### Prerequisites

As a prerequisite for operating in an Verifiable Credential ecosystem,
Verifier, Issuer and Wallet need to implement the protocols and the
credential formats securely and correctly, that is, according to the
respective specifications, respecting the security and privacy
requirements of the specifications, and following the best practices for
application security.

This includes, for example, the use of secure implementations of
cryptographic algorithms, the proper verification of signatures and
credentials wherever required by the protocols or credential formats,
the use of secure random number generators, the secure use of
hardware-based storage, and protection against injection attacks.


> **Security Requirement V-00:** The Verifier must implement the
> protocol securely and correctly.

> **Security Requirement V-01:** The Verifier must implement the
> credential format securely and correctly.

> **Security Requirement I-00:** The Issuer must implement the protocol
> securely and correctly.

> **Security Requirement I-01:** The Issuer must implement the
> credential format securely and correctly.

> **Security Requirement W-00:** The Wallet must implement the protocol
> securely and correctly.

> **Security Requirement W-01:** The Wallet must implement the
> credential format securely and correctly.

### Trust Chain from Verifier to Issuer

The Verifier receives a presentation that is cryptographically tied to a
credential that itself is cryptographically tied to a specific
Credential Issuer. One of the most basic security requirements is that
the Verifier can verify that the presentation is tied to the correct
Issuer.

> **Security Requirement CF-10/TF-10:** For any presentation, the
credential format and trust framework must be designed such that there
is a secure way to determine the Issuer and to check that the original
credential was issued by this issuer. (E.g., by using a cryptographic
signature.)

The data within the credential must be protected against tampering by
an attacker. The Verifier must be able to verify that the data in the
presentation is the same as the data in the original credential.

> **Security Requirement CF-20:** For any presentation, the credential
format must ensure that the data contained therein that is tied to the
original credential cannot be not altered. (E.g., by using a
cryptographic signature.)

The root of trust from the perspective of a Verifier is the Credential
Issuer. The Verifier decides, based on policy, regulation, conformity
assessment, and/or contracts, to trust a certain Issuer to issue
Credentials of a certain type with the correct data to the legitimate
End-Users. This requires that the Issuer is identified uniquely and
unambiguously.

> **Security Requirement TF-20:** The trust framework must ensure that the
identification of an Issuer is unique and unambiguous. If there are
multiple instances of the same Issuer sharing the same key material, the
Verifier must trust all instances equally.

For example, a mobile app as issuer which holds the key material in the
app itself is most likely a bad idea - not all instances of this app are
equally trustworthy, some might be hacked and the key material leaked to
malicious parties.

While the way in which the Verifier determines the trustworthiness of
the Issuer defined in the trust framework is out of scope of this
specification, this decision must not be influenced by a malicious
party.

> **Security Requirement TF-30:** The way in which the Verifier
determines the trustworthiness of the Issuer defined in the trust
framework must be secured from influence by a malicious party that can,
for example, introduce untrustworthy entities into a directory.

Any information that is used by the Verifier to determine the
trustworthiness of the Issuer must be kept up to date. When an Issuer
leaves the ecosystem or key material needs to be revoked or rotated,
appropriate mechanisms must be in place.

> **Security Requirement TF-40:** The trust framework must ensure that
there is a way for Verifiers to keep their information on trusted
Issuers up to date and that there is a way to revoke trust in an Issuer.

The Verifier trusts in the Authenticity of Claims as described above.
This means:

> **Security Requirement I-10:** The Issuer must authenticate/identify the
End-User properly according to the expectations of the Verifier (which
may be defined in a specification, trust framework, or by convention).

The Verifier trusts in the Integrity of Claims as described above. It is
obvious that the Issuer must not issue false claims about the End-User.

> **Security Requirement I-20:** The Issuer must only put correct and
up-to-date claims about the End-User into the Credential where verified
data is expected.

(By convention, specification or trust framework,
self-declared or otherwise "unverified" claims may be allowed in some
places in some credentials.)

#### Requirements on Issuance

To ensure basic privacy and prevent abuse of stolen credentials on a
very basic level, it must be ensured that credentials cannot be read by
third parties during the issuance process.

> **Security Requirement P-10:** The protocol must ensure that no third
> party can extract the credential issued by the Issuer.

It must be ensured that an attacker cannot introduce credentials into a
Wallet that are not intended for the End-User.

> **Security Requirement P-20:** The protocol must ensure that no third
> party can interfere with the issuance process such that the Issuer
> issues Credentials for the third party to the End-User.

Finally, an Issuer that learns about potential abuse of a credential
must be able to revoke it.

> **Security Requirement I-30:** The Issuer must revoke a Credential
once the Issuer learns about potential abuse of the Credential.

#### Requirements on Presentation Process

The Verifier trusts in the Integrity of the Interaction with the Wallet,
as described above. This means:

> **Security Requirement P-30:** The protocol must ensure that the
interaction between the Wallet and Verifier is protected such that no
third party can interfere with the interaction by modifying the
information transmitted.

An attacker could extract messages of the presentation process and
forward them to an End-User. The End-User could be confused about the
purpose of the interaction and complete the presentation process on the
attacker's behalf, giving the attacker access to resources or a session
that was authenticated using the End-Users identity. 

> **Security Requirement P-40:** The protocol must ensure that the
> interaction between an attacker and a Verifier cannot be forwarded to
> and successfully completed by a user.

Likewise, an attacker interacting directly with a Verifier must not be
able to re-use parts extracted from an End-User's session to
successfully complete the interaction.

> **Security Requirement P-41:** The protocol must ensure that an
> attacker cannot successfully forward an interaction between a Wallet
> and a Verifier to a Verifier under his own control.


### Holder Binding

If Holder Binding is required by the use-case of the Verifier, the
Verifier must trust the Issuer to bind credentials to the End-User. This
means:

> **Security Requirement I-40:** The Issuer must only include
holder-binding data into the credential that is tied to the actual
End-User (and not, e.g., include a cryptographic key under control by a
third party).  

It must be ensured that during issuance, the correct holder-binding data
ends up in the credential.

> **Security Requirement P-50:** The protocol must ensure that third
> parties cannot interfere with the binding process.

For cryptographic holder binding, the security largely depends on the
quality of the key management, e.g. whether private keys can be
extracted from the user's device and copied to another device. So the
Verifier needs to trust in the wallet to implement the key management
securely. 

> **Security Requirement W-10:** The Wallet must implement the key
management securely such that only the legitimate Holder can use a
credential.

This can be achieved, for example by using a secure enclave or similar technology.

### Ensuring Secure Storage of Credentials

This can be achieved in one of two ways:

 * Direct: The Verifier checks the trustworthiness of the wallet
   directly by, for example, requiring an attested key management.
 * Indirect: The Verifier trusts the Issuer to only issue credentials to
   wallets that implement the key management securely. 

#### Direct Model

> **Security Requirement V-10 (conditional):** The Verifier must ensure
that the credential was stored in a secure wallet.

#### Indirect Model

Assuming this trust can be established by way of conformity assessment
of some sort, the Issuer needs to make sure that it issues Credentials
into a wallet it trusts and the Verifier needs to make sure that it
accepts presentations only from wallets it trusts. 

In order to make the life of Verifiers easier, this concept obliges the
Issuer to only issue Credentials to trustworthy wallets and to make its
requirements transparent to Verifiers via its policy so this becomes
part of the rules a Verifier needs to understand and accept. 

From the Verifier’s standpoint it is now sufficient to ensure it accepts
Credentials from this Issuer since, by conclusion, if it gets presented
a valid Credential issued by this Issuer it must have come from a
trustworthy wallet. This change significantly simplifies the protocol
and fosters privacy since the presentation process does not need to
authenticate the wallet towards the Verifier. 

> **Security Requirement I-50 (conditional):** The Issuer must ensure that the
credential was stored in a secure wallet.

Then, the Verifier must check that the Issuer actually adheres to this
policy.

> **Security Requirement V-20 (conditional):** The Verifier must ensure that
the credential was issued by an issuer that only issues credentials to
trustworthy wallets.

### End-User's Perspective

**TODO: Expand this section**

The End-User trusts the Issuer, the Wallet, and the Verifier to treat
her data securely and only exchange data as needed preserving her
privacy. The End-User trusts the wallet to validate the authenticity of
Issuers and Verifiers and provide the End-User with trustworthy
information about those.

Therefore, the Wallet must provide trustworthy information about Issuers:

> **Security Requirement W-20:** The Wallet must provide trustworthy and
> complete information about Issuers to the End-User.

Even more important for Verifiers:

> **Security Requirement W-30:** The Wallet must provide trustworthy and
> complete information about Verifiers to the End-User.

Credentials must not be used without the End-Users consent.

> **Privacy Requirement W-40:** The Wallet must ask the End-User for
> meaningful consent before a Credential is used. The Wallet must
> provide the End-User the opportunity to review any data that is shared
> with a Verifier.

The credential format must allow selective disclosure of data.

> **Privacy Requirement CF-30:** The Credential Format must ensure that
> there is a robust mechanism to ensure that data that is not to be
> released to a Verifier cannot be extracted by the Verifier (selective
> disclosure).

The End-User trusts the Issuer and the wallet provider to protect her
from abuse of her Credentials. This mainly means that the implementation
of the selected key management and protocols must be secure, but
specifically the communication protocols must be designed such that an
attacker cannot exfiltrate PII.

> **Security/Privacy Requirement P-60:** The protocol must ensure that
> during an interaction with a Verifier, an attacker cannot exfiltrate
> PII.

> **Security/Privacy Requirement P-70:** The protocol must ensure that
> during an interaction with an Issuer, an attacker cannot exfiltrate
> PII.

If an Issuer, Wallet, or Verifier is compromised, the risk for End-Users
must be minimized.

> **Security/Privacy Requirement W-50:** The Wallet must ensure that the
> credentials and private keys are protected from unauthorized access.

Since data leaks are hard to avoid completely in large ecosystems, the
fallout of a compromise must be minimized.

> **Security/Privacy Requirement TF-50:** The Trust Framework must
> ensure that lifecycles of keys, certificates, and credentials are
> designed such that the impact of a compromise is minimized.

On the other hand, the End-User does not want the Issuer to learn where
she uses the Credentials. This must be supported by the Issuer, the
Wallet, and the Trust Framework.

> **Privacy Requirement P-80:** The protocol must ensure that the Issuer
> cannot learn where the End-User uses the Credential.

> **Privacy Requirement W-60:** The Wallet must ensure that the Issuer
> cannot learn where the End-User uses the Credential.

> **Privacy Requirement TF-60:** The Trust Framework must ensure that
> the Issuer cannot learn where the End-User uses the Credential.

When using a Credential at multiple Verifiers, the End-User might not
want that the Verifiers learn that the same End-User is using their
services (correlation). The exception to this is when the End-Users
shares data with the Verifiers that allow for a unique identification,
e.g., a name and birth date, or a unique identifier, e.g., a social
security number.

> **Privacy Requirement W-70:** The Wallet must ensure that the Verifier
> cannot learn that the same End-User is using other Verifiers.

> **Privacy Requirement TF-70:** The Trust Framework must support
> correlation protection.

> **Privacy Requirement CF-40:** The Credential Format must support
> correlation protection.

# Security and Privacy Requirements on the Credential Format {#security-requirements-on-the-credential-format}

| Security Requirement | How to achieve                       |
| -------------------- | ------------------------------------ |
| CF-10                | Requirement on the credential format |
| CF-20                | Requirement on the credential format |
| CF-30                | Requirement on the credential format |
| CF-40                | Requirement on the credential format |

# Security and Privacy Requirements on the Trust Framework {#security-requirements-on-the-trust-framework}

| Security Requirement | How to achieve                                                                         |
| -------------------- | -------------------------------------------------------------------------------------- |
| TF-10                | Selection of credential format                                                         |
| TF-20                | Requirement on management of issuers                                                   |
| TF-30                | Requirement on management of trust                                                     |
| TF-40                | Requirement on management of trust                                                     |
| TF-50                | Requirement on lifecycle management; also see (#note-on-storage-of-credentials) below. |
| TF-60                | Requirement on lifecycle management and key distribution                               |
| TF-70                | Requirement on lifecycle management and key distribution                               |


## Distribution of Identifiers and Keys {#distribution-of-identifiers-and-keys}


The involved parties need to access data about identifiers, keys, and
Credentials as well as data about the acting parties themselves in order
to securely implement the Credential exchange. The way this works is out
of scope for OpenID 4 Verifiable Credential Issuance.


# Security and Privacy Requirements on the Protocol {#security-requirements-on-the-protocol}

| Security Requirement | How to achieve             |
| -------------------- | -------------------------- |
| P-10                 | see (#security-requirement-p-10) |
| P-20                 | see (#security-requirement-p-20) |
| P-30                 | see (#security-requirement-p-30) |
| P-40                 | see (#security-requirement-p-40) |
| P-41                 | see (#security-requirement-p-41) |
| P-50                 | see (#security-requirement-p-50) |
| P-60                 | see (#security-privacy-requirement-p-60) |
| P-70                 | see (#security-privacy-requirement-p-70) |
| P-80                 | see (#privacy-requirement-p-80) |

# Security and Privacy Requirements on the Verifier {#security-requirements-on-the-verifier}

| Security Requirement | How to achieve                             |
| -------------------- | ------------------------------------------ |
| V-00                 | Implementation requirement                 |
| V-01                 | Implementation requirement                 |
| V-10                 | TODO add more details in appropriate place |
| V-20                 | TODO add more details in appropriate place |

# Security and Privacy Requirements on the Wallet {#security-requirements-on-the-wallet}

| Security Requirement | How to achieve                                                      |
| -------------------- | ------------------------------------------------------------------- |
| W-00                 | Implementation requirement                                          |
| W-01                 | Implementation requirement                                          |
| W-10                 | Out of scope                                                        |
| W-20                 | Implementation and operation requirement                            |
| W-30                 | Implementation and operation requirement                            |
| W-40                 | Implementation and operation requirement                            |
| W-50                 | Implementation and operation requirement  (TODO see secure storage) |
| W-60                 | Implementation and operation requirement                            |
| W-70                 | Implementation and operation requirement                            |

# Security and Privacy Requirements on the Issuer {#security-requirements-on-the-issuer}

| Security Requirement | How to achieve                                         |
| -------------------- | ------------------------------------------------------ |
| I-01                 | Implementation requirement                             |
| I-00                 | Implementation requirement                             |
| I-10                 | Implementation and End-User authentication requirement |
| I-20                 | Implementation and End-User authentication requirement |
| I-30                 | Implementation and operation requirement               |
| I-40                 | Implementation and operation requirement               |
| I-50                 | TODO add more details in appropriate place             |


# Attacker Model

For this analysis, the Web Attacker and Network Attacker models defined
as (A1) and (A2) in the OAuth Security BCP are assumed. In the context
of this specification, the Web Attacker model in particular means that
arbitrary Issuers, Wallets, and Verifiers can be under the control of an
attacker. The security of all other participants must be ensured
nonetheless. For example, the security of a non-compromised Wallet that
interacts with multiple Issuers and Verifiers must not be affected by a
single compromised Issuer or Verifier.

Special considerations are made where requests are forwarded between
devices (cross-device) and between mobile apps on the same device
(app-to-app).

## Malicious Credential Issuer Metadata Configurations {#malicious-credential-issuer-metadata-configurations}

Since it is assumed that the attacker can operate Issuers, the attacker
can attempt to perform mix-up type attacks by configuring the credential
issuer metadata (cf. Section 10.2. in OID4VCI) under their control
according to the following list:

**TODO**: Deferred Credential Endpoint metadata not defined in OID4VCI spec.

No Mix-Up:

- `credential_issuer`: Attacker's Credential Issuer Identifier
- `authorization_server`: Attacker's Authorization Server
- `credential_endpoint`: Attacker's Credential Endpoint
- `batch_credential_endpoint`: Attacker's Batch Credential Endpoint

Mix-Up Type 1:

- `credential_issuer`: Attacker's Credential Issuer Identifier
- `authorization_server`: Honest Issuer's Authorization Server
- `credential_endpoint`: Attacker-controlled Credential Endpoint
- `batch_credential_endpoint`: Attacker-controlled Batch Credential
  Endpoint

**TODO** 1a, Variant of Type 1: Only deferred endpoint it attacker-controlled.

Mix-Up Type 2:

 - `credential_issuer`: Attacker's Credential Issuer Identifier
 - `authorization_server`: Attacker-controlled Authorization Server
 - `credential_endpoint`: Honest Issuer's Credential Endpoint
 - `batch_credential_endpoint`: Honest Issuer's Batch Credential Endpoint

Note that when an attacker-controlled Authorization Server is used, the
attacker can also perform regular OAuth Mix-Up attacks unless prevented by
the `iss` parameter according to [@RFC9207].

These configurations will be used in the attack descriptions below.

# Protocol Analysis OpenID 4 VC

OpenID 4 VC consists of three protocols:

 * OpenID for Verifiable Credential Issuance – Defines an API and
   corresponding OAuth-based authorization mechanisms for issuance of
   Verifiable Credentials
 * OpenID for Verifiable Presentations – Defines a mechanism on top of
   OAuth 2.0 to allow presentation of claims in the form of Verifiable
   Credentials as part of the protocol flow
 * Self-Issued OpenID Provider v2 – Enables End-Users to use OpenID
   Providers (OPs) that they control

The following analysis aims to show that OpenID 4 VC meets the security
and privacy requirements defined in the previous section. This is an
informal analysis that may be incomplete or contain mistakes. A formal
analysis to confirm the results is yet to be done.

## Security Requirement P-10 {#security-requirement-p-10}

> The issuance protocol must ensure that no third party can extract the
> credential issued by the Issuer.

On the network level, this is achieved by TLS encrypted connections. 

On top of TLS, the protocol uses an OAuth 2.0 flow to authorize the
release of a credential.

Credentials can be released at one of three issuance endpoints:

* The credential endpoint can send a credential response containing a
  credential,
* the batch credential endpoint can send a batch credential response
  containing multiple credentials, or
* the deferred credential endpoint can send a a credential response
  containing a credential.

The deferred credential endpoint requires the use of an acceptance token
as a bearer token. The acceptance token can be obtained only from the
credential endpoint or batch credential endpoint. **TODO** Is the
endpoint a protected resource? Can DPoP or MTLS be enforced?

**ATTACK**: An attacker can configure Mix-Up 1a. The attacker receives
an acceptance token from the Wallet's request to the deferred credential
endpoint and  uses it to obtain a credential from the deferred credential
endpoint of the honest server.

The credential endpoint and batch credential endpoint can only be
accessed by using an access token. **TODO** Can DPoP or MTLS be
enforced?

**ATTACK**: With Mix-Up 1, the attacker can obtain a credential from the
honest server by using the access token it receives via the call to its own
credential endpoint.

An access token can only be obtained from the token endpoint using one
of the existing OAuth grant types or the new grant type
`urn:ietf:params:oauth:grant-type:pre-authorized_code` defined in the
specification.

### Pre-Authorized Code Grant

For the **pre-authorized code grant**, the Wallet obtaining the token
needs to possess a pre-authorized code. This code is received directly
from the credential issuer after an interaction between the user and the
credential issuer, for example, via a QR code or a deeplink. The
pre-authorized code MUST (by definition) be short-lived and single-use.

RECOMMENDATION: Measures must be in place to avoid that the
pre-authorized code is intercepted by a third party. For a QR code, it
is recommended that the code is not displayed to the user in public
environments. Deeplinks should, if possible, aim to target a specific
application in order to avoid interception by third party applications. 

An additional PIN that is transferred from the issuer to the verifier
can be used to protect the pre-authorized code.

### Authorization Code Grant

For all **other grant types**, the considerations in the OAuth Security
BCP apply, as explicitly mentioned in the specification.

## Security Requirement P-20 {#security-requirement-p-20}

> The protocol must ensure that no third party can interfere with the
> issuance process such that the Issuer issues Credentials for the third
> party to the End-User.

In order to receive credentials originally created for a different
End-User, the Wallet would need to receive it from one of the issuance
endpoints listed above.

### Honest Credential Issuer

If the Wallet is using a Credential Issuer identifier that is used by an
honest party, it can be assumed that all endpoints used in the flow
(authorization server, token endpoint, issuance endpoints) are provided
by the same, honest party. 

In this case, the latest point in the flow where an attacker can
interfere with the process is the authorization response in the case of
the authorization code flow and the pre-authorized code in case of the
pre-authorized code flow. 

#### Authorization Code Flow

In case of the authorization code flow, an attacker can attempt to
inject their own response into the flow instead of the original
authorization response. This is a well-known attack vector on OAuth
flows that is prevented by the use of PKCE, as recommended in the OAuth
Security BCP.

#### Pre-Authorized Code Flow

In this case, an attacker can attempt to forward a pre-authorized code
created for his End-User identity to a different End-User. Mitigations
against this are discussed in Section 11.3. of the OID4VCI
specification.

**TODO** The spec does not distinguish between replay (attacker forwards
code to other wallet/end-user) and stealing the code (attacker scans
code intended for other user). This needs to be fixed. 

### Compromised Credential Issuer

In case the Wallet uses a Credential Issuer Identifier under the control
of an attacker, the attacker can configure the endpoints of the  Issuer
as described in (#malicious-credential-issuer-metadata-configurations)
above.

In the first case (No Mix-Up), the attacker has full control over the
flow. The attacker can return, for example, a credential created for 
an End-User obtained from a completely different Issuer. 

RECOMMENDATION: The Wallet MUST verify that the Issuer of the received
credential is the expected Credential Issuer. This means that the
credential format needs to encode sufficient information to allow the
Wallet to verify this relationship.

In the second case (Mix-Up), other attacks are possible, but regarding
interference with the issuance process, the same considerations as in
the previous case apply.

## Security Requirement P-30 {#security-requirement-p-30}

> The protocol must ensure that the interaction between the Wallet and
> Verifier is protected such that no third party can interfere with the
> interaction by modifying the information transmitted.

The target of this attack is the user's device. The attacker needs to be
able to inject a presentation into a flow between the user and a
Verifier. For this to work, the user has to have started a flow with the
Verifier. This means that the Verifier then expects a presentation with
a specific audience and nonce.

Unless the attacker has access to the initial request between the
Verifier and the wallet, the attacker cannot know the nonce that is
expected in the flow. This works similar to the state value in classic
OAuth flows.

If the attacker does have access to the initial request, the attacker
would be able to replay the request on his own device and capture the
presentation. The attacker could then use this presentation to inject it
into the flow between the user and the Verifier. Similar attacks can
happen in classic OAuth flows and are not mitigated by the use of
"nonce" or state. However, for this (relatively weak) attack to succeed,
a (relatively strong) attacker with access to the communication between
the Verifier and the wallet is required. The likelihood of a successful
attack can be reduced when, for the wallet, claimed URLs are used and
custom schemes and endpoints on localhost addresses are avoided. 

Note: An additional encryption of the request to the wallet at the
application layer would not prevent this attack as the attacker could
still replay the encrypted request to the wallet.


## Security Requirement P-40 {#security-requirement-p-40}

> The protocol must ensure that the interaction between an attacker and
> a Verifier cannot be forwarded to and successfully completed by a
> user.

For this attack, an attacker starts a presentation flow with a Verifier.
The Verifier generates an authentication request that the attacker can
now forward to their victim. The victim might be tricked into thinking
that the completion of the flow is necessary and the attacker may or may
not impersonate the Verifier for this purpose. The victim authenticates,
authorizes the request and a presentation is created and sent to the
Verifier. Without further countermeasures, Verifier would accept the
presentation and apply it to the session between the attacker and the
Verifier. The attacker can now impersonate the Victim. **TODO** More
details for description.

The countermeasures against this attack are different in the same-device
flow and the cross-device flow.

### Same-Device Flow

In the same-device flow, the authentication response would be received
by the Verifier in the victim's browser. The Verifier should maintain a
mapping between user sessions and the nonce that is expected in the
flow. The Verifier should only accept a presentation if the nonce in the
presentation matches the nonce that is expected for the user session.
With this countermeasure, the Verifier can detect if a presentation is
sent to it that was not intended for the user session or if no user
session exists at all, preventing the attack.

RECOMMENDATION: While this is expected from OAuth implementations, the
importance of this measure should be highlighted in the specification.

**Note on web-to-app flows:** In the case of web-to-app flows (i.e., the
Verifier is a website, but the wallet is an app), the authentication
response needs to go back to the same browser where the user started the
flow. This may require extra steps in the implementation.


### Cross-Device Flow

In the cross-device flow, the authentication response cannot be sent
back to the same browser where the Verifier runs. Instead, the
authentication response is sent using the direct post mode in the
backend. This means that no session context exists for the Verifier to
assure the binding between the browser where the user started the flow
and the presentation that is sent to the Verifier.

There is currently no widely-available, convenient way to fix this
problem. A more detailed discussion including potential countermeasures
can be found in the Cross-Device Security BCP.


## Security Requirement P-41 {#security-requirement-p-41}

> The protocol must ensure that an attacker cannot successfully forward
> an interaction between a Wallet and a Verifier to a Verifier under his
> own control.

A prerequisite for a successful attack of this kind is that the attacker
has access to some messages between the Wallet and the Verifier. 

The attacker might have access to the authentication response. In this
case, the attacker has access to the presentation contained in the VP
Token. 

The attacker can now start an interaction with the Verifier under his
control. The attacker can then send the presentation to the Verifier
instead of a presentation created by his own Wallet. The Verifier would
then accept the presentation and apply it to the session between the
attacker and the Verifier. The attacker can now impersonate the
End-User.

This is prevented by the nonce contained in the authorization request
and the presentation. The Verifier can verify that the nonce in the
presentation matches the nonce in the authorization request. However,
the Verifier must also check that the audience of the presentation
matches the Verifier's identifier. Without this check, an attacker could
convince the End-User to start a new, independent flow with a Verifier
und the attacker's control and re-use the nonce from the original flow
in the authentication request. By checking the audience, this attack is
prevented.

(In the same-device flow, the Verifier usually associates the attacker's
session with the selected nonce. In the cross-device flow, the nonce
might be used as the single identifier for the flow to which a
presentation belongs. In both cases, a nonce that is not the nonce the
Verifier selected for the flow with the attacker would not work.)

-> Injection
-> MITM?

## Security Requirement P-50 {#security-requirement-p-50}

> The protocol must ensure that third parties cannot interfere with the
> binding process.

**TODO** Is the Wallet-provided nonce required?

-> MITM?
-> Redirect URIs?

## Security/Privacy Requirement P-60 {#security-privacy-requirement-p-60}

> The protocol must ensure that during an interaction with a Verifier,
> an attacker cannot exfiltrate PII.

As above, it can be assumed that all network connections are secured by TLS.

On the level of the OAuth-based protocol, the PII in the form of the
presentation is transferred during the authentication response (or,
based on an authorization code sent in the authentication response,
later in the token response).

It is therefore important that the authentication response is not
sent to an attacker-controlled endpoint.

The authentication response is sent to the redirect URI specified in
the request to the Wallet. 

There are two options for the verification of the redirect URI:

 1. The Verifier specifies the redirect URI in the request to the
    wallet. The request is signed using a key that the wallet can verify
    and show the proper name of the Verifier to the user. 
 2. The redirect URI is registered beforehand and therefore guaranteed
    to be authentic.

If the authorization code flow is used, it must be ensured that the
token endpoint URL is authentic as well (i.e., belongs to the same
Issuer).

**TODO** Check if the OID4VCI spec ensures authenticity of the token
endpoint URL.

## Security/Privacy Requirement P-70 {#security-privacy-requirement-p-70}

> The protocol must ensure that during an interaction with an Issuer, an
> attacker cannot exfiltrate PII.

-> Encryption
-> MITM?
-> Redirect URIs?

## Privacy Requirement P-80 {#privacy-requirement-p-80}

> The protocol must ensure that the Issuer cannot learn where the
> End-User uses the Credential.

The interaction between the Wallet and the Verifier is not inherently
tied to any interaction between either the Wallet and the Issuer or the
Verifier and the Issuer, with the following (potential) exceptions:

 * The Wallet may need to retrieve a new credential from the Issuer
   before presenting it to the Verifier. Reasons can include that the
   credential has expired, that the Wallet has removed the credential,
   or that the Wallet needs a new credential to present to the Verifier
   in order to avoid correlation. The point in time when such an
   interaction happens may leak information about the Wallet's actions,
   but is usually a weak correlation vector. 
 * The Verifier may need to retrieve a signature verification key,
   revocation information or other data from the Issuer before verifying
   the credential. This is not a privacy concern, but may leak
   information about the Wallet's actions.

RECOMMENDATION: Wallets can be designed to avoid this by, for example,
by refreshing credentials at a random point in time.

RECOMMENDATION: The Verifier can be designed to avoid this by, for
example, by caching the data for a random period of time.


**TODO** To be discussed: Difference Same-Device Cross-Device where not discussed already. 

**TODO** Move sections to security Considerations in OpenID 4 VP/VCI where applicable




# Note on Storage of Credentials and Presentations {#note-on-storage-of-credentials}

**TODO: Link this better to the requirements above.**

Wherever End-User data is stored, it represents a potential target for
an attacker. This target can be of particularly high value when the data
is signed by a trusted authority like an official national identity
service. For example, in OpenID Connect, signed ID Tokens can be stored
by Relying Parties. In the Issuer-Holder-Verifier model, Wallets have to
store signed credentials and associated data, and Issuers and Verifiers
may decide to store credentials or presentation as well.

Not surprisingly, a leak of such data risks revealing private data of
End-Users to third parties. Signed End-User data, the authenticity of
which can be easily verified by third parties, further exacerbates the
risk. Leaked credentials or presentations may also allow attackers to
impersonate Holders unless Holder Binding is enforced and the attacker
does not have access to the Holder's cryptographic keys. Altogether,
leaked credentials and presentations may have a high monetary value on
black markets.

Due to these risks, systems implementing the Issuer-Holder-Verifier
model SHOULD be designed to minimize the amount of data that is stored.
All involved parties SHOULD store credentials or presentations only for
as long as needed, including in log files.

Issuers SHOULD NOT store credentials after issuance.

Holders SHOULD store credentials and associated data only in encrypted
form, and, wherever possible, use hardware-backed encryption in
particular for the private Holder Binding key. Decentralized storage of
data, e.g., on End-User devices, SHOULD be preferred for End-User
credentials over centralized storage. Expired credentials and associated
data SHOULD be deleted as soon as possible.

Verifiers SHOULD NOT store presentations after verification. It may be
sufficient to store the result of the verification and any End-User data
that is needed for the application.

If reliable and secure key rotation and revocation is ensured, Issuers
may MAY opt to publish expired or revoked private signing keys (after a
grace period that ensures that the keys are not cached any longer at any
Verifier). This reduces the value of any leaked credentials as the
signatures on them can no longer be trusted to originate from the
Issuer.

{backmatter}


<reference anchor="VC_DATA" target="https://www.w3.org/TR/vc-data-model">
  <front>
    <title>Verifiable Credentials Data Model 1.0</title>
    <author fullname="Manu Sporny">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Grant Noble">
      <organization>ConsenSys</organization>
    </author>
    <author fullname="Dave Longley">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Daniel C. Burnett">
      <organization>ConsenSys</organization>
    </author>
    <author fullname="Brent Zundel">
      <organization>Evernym</organization>
    </author>
    <author fullname="David Chadwick">
      <organization>University of Kent</organization>
    </author>
   <date day="19" month="Nov" year="2019"/>
  </front>
</reference>

<reference anchor="SIOPv2" target="https://openid.bitbucket.io/connect/openid-connect-self-issued-v2-1_0.html">
  <front>
    <title>Self-Issued OpenID Provider V2</title>
    <author fullname="Kristina Yasuda">
      <organization>Microsoft</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
    <author fullname="Tobias Looker">
      <organization>Mattr</organization>
    </author>
   <date day="20" month="Jul" year="2021"/>
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

<reference anchor="DIF.PresentationExchange" target="https://identity.foundation/presentation-exchange">
        <front>
          <title>Presentation Exchange 2.0.0</title>
		  <author fullname="Daniel Buchner">
            <organization>Microsoft</organization>
          </author>
          <author fullname="Brent Zundel">
            <organization>Evernym</organization>
          </author>
          <author fullname="Martin Riedel">
            <organization>Consensys Mesh</organization>
          </author>
          <author fullname="Kim Hamilton Duffy">
            <organization>Centre Consortium</organization>
          </author>
        </front>
</reference>

<reference anchor="DID-Core" target="https://www.w3.org/TR/2021/PR-did-core-20210803/">
        <front>
        <title>Decentralized Identifiers (DIDs) v1.0</title>
        <author fullname="Manu Sporny">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="Amy Guy">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="Markus Sabadello">
            <organization>Danube Tech</organization>
        </author>
        <author fullname="Drummond Reed">
            <organization>Evernym</organization>
        </author>
        <date day="3" month="Aug" year="2021"/>
        </front>
</reference>

<reference anchor="TRAIN" target="https://oid2022.compute.dtu.dk/index.html">
        <front>
          <title>A novel approach to establish trust in Verifiable Credential
issuers in Self-Sovereign Identity ecosystems using TRAIN</title>	  
           <author fullname="Isaac Henderson Johnson Jeyakumar">
            <organization>University of Stuttgart</organization>
          </author>
          <author fullname="David W Chadwick">
            <organization>Crossword Cybersecurity</organization>
          </author>
          <author fullname="Michael Kubach">
            <organization>Fraunhofer IAO</organization>
          </author>
   <date day="8" month="July" year="2022"/>
        </front>
</reference>

<reference anchor="OpenID-Discovery" target="https://openid.net/specs/openid-connect-discovery-1_0.html">
  <front>
    <title>OpenID Connect Discovery 1.0 incorporating errata set 1</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
      <organization>Google</organization>
    </author>
    <author initials="E." surname="Jay" fullname="Edmund Jay">
      <organization> Illumila </organization>
    </author>
   <date day="8" month="Nov" year="2014"/>
  </front>
</reference>

<reference anchor="OpenID.Registration" target="https://openid.net/specs/openid-connect-registration-1_0.html">
        <front>
          <title>OpenID Connect Dynamic Client Registration 1.0 incorporating errata set 1</title>
		  <author fullname="Nat Sakimura">
            <organization>NRI</organization>
          </author>
          <author fullname="John Bradley">
            <organization>Ping Identity</organization>
          </author>
          <author fullname="Michael B. Jones">
            <organization>Microsoft</organization>
          </author>
          <date day="8" month="Nov" year="2014"/>
        </front>
 </reference>

<reference anchor="Hyperledger.Indy" target="https://www.hyperledger.org/use/hyperledger-indy">
        <front>
          <title>Hyperledger Indy Project</title>
          <author>
            <organization>Hyperledger Indy Project</organization>
          </author>
          <date year="2022"/>
        </front>
</reference>

<reference anchor="JARM" target="https://openid.net/specs/oauth-v2-jarm-final.html">
        <front>
          <title>JWT Secured Authorization Response Mode for OAuth 2.0 (JARM)</title>
		  <author fullname="Torsten Lodderstedt">
            <organization>yes.com</organization>
          </author>
          <author fullname="Brian Campbell">
            <organization>Ping Identity</organization>
          </author>
          <date day="9" month="Nov" year="2022"/>
        </front>
 </reference>

<reference anchor="ISO.18013-5" target="https://www.iso.org/standard/69084.html">
        <front>
          <title>ISO/IEC 18013-5:2021 Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL)  application</title>
          <author>
            <organization> ISO/IEC JTC 1/SC 17 Cards and security devices for personal identification</organization>
          </author>
          <date year="2021"/>
        </front>
</reference>

<reference anchor="OAuth.Responses" target="https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html">
        <front>
        <title>OAuth 2.0 Multiple Response Type Encoding Practices</title>
        <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
            <organization>Google</organization>
        </author>
        <author initials="M." surname="Scurtescu" fullname="M. Scurtescu">
            <organization>Google</organization>
        </author>        
        <author initials="P." surname="Tarjan" fullname="Facebook">
            <organization>Evernym</organization>
        </author>
        <author initials="M." surname="Jones" fullname="Michael B. Jones">
            <organization>Microsoft</organization>
        </author>
        <date day="25" month="Feb" year="2014"/>
        </front>
</reference>

<reference anchor="OpenID.VCI" target="https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html">
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
          <date day="20" month="June" year="2022"/>
        </front>
</reference>

<reference anchor="OpenID.Federation" target="https://openid.net/specs/openid-connect-federation-1_0.html">
        <front>
          <title>OpenID Connect Federation 1.0 - draft 17></title>
		  <author fullname="R. Hedberg, Ed.">
            <organization>Independent</organization>
          </author>
          <author fullname="Michael B. Jones">
            <organization>Microsoft</organization>
          </author>
          <author fullname="A. Solberg">
            <organization>Uninett</organization>
          </author>
          <author fullname="S. Gulliksson">
            <organization>Schibsted</organization>
          </author>
          <author fullname="John Bradley">
            <organization>Yubico</organization>
          </author>
          <date day="9" month="Sept" year="2021"/>
        </front>
 </reference>
