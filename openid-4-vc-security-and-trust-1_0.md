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

W3C Verifiable Presentation:
:  A Verifiable Presentations compliant to the [@VC_DATA] specification.

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

Cryptographic Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proving control over the same private key during the issuance and presentation. Mechanism might depend on the Credential Format. For example, in `jwt_vc_json` Credential Format, a VC with Cryptographic Holder Binding contains a public key or a reference to a public key that matches to the private key controlled by the Holder. 

Claim-based Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proofing certain claims, e.g. name and date of birth, for example by presenting another Verifiable Credential. Claim-based Holder Binding allows long term, cross device use of a Credential as it does not depend on cryptographic key material stored on a certain device. One example of such a Verifiable Credential could be a Diploma.

Biometrics-based holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by demonstrating a certain biometric trait, such as finger print or face. One example of a Verifiable Credential with biometric holder binding is a mobile drivers license [@ISO.18013-5], which contains a portrait of the holder.

VP Token:
: An artifact defined in this specification that contains a single Verifiable Presentation or an array of Verifiable Presentations as defined in (#response-parameters).

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
    * Integrity of Interaction: When interacting with a Holder, the
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
      claims **TODO: Refine**. 
    * Correctness of Claims: A Holder can trust that the claims in a
      Credential are correct and not altered or influenced by a
      malicious party.
 * For Issuers:
   * ?


### Note on Holder Binding

Depending on the use case, the Verifier also needs to trust the End-User
to be the legitimate holder of a certain Credential. Accepting a
Credential from every bearer is a security risk, especially in the
indirect model since those Credentials are supposed to have a long
lifetime (more likely years then seconds). So anyone getting in
possession of a Credential, e.g. through a security breach, could pose
as the legitimate End-User for a long time.

That’s why Credentials are usually bound to the legitimate End-User,
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
subject of the respective Credential. (User A does not get a credential
about User B with Holder Binding on User A.)

## Security Requirements

To implement a secure trust chain, a number of Security Requirements must be met. In the following, the Security Requirements are described along the trust chain from Verifier to Issuer. All Requirements are numbered for reference and are prefixed with the respective party or component that needs to implement the requirement:

  * V: Verifier
  * I: Issuer
  * W: Wallet

  * CF: Credential Format
  * TF: Trust Framework
  * P: Protocol

### Prerequisites

> **Security Requirement V-00:** The Verifier must implement the protocol securely and correctly.

> **Security Requirement V-01:** The Verifier must implement the credential format securely and correctly.

It is assumed that the previous two requirements include, for example, a proper verification of signatures on credentials.

> **Security Requirement I-00:** The Issuer must implement the protocol securely and correctly.

> **Security Requirement I-01:** The Issuer must implement the credential format securely and correctly.

> **Security Requirement W-00:** The Wallet must implement the protocol securely and correctly.

> **Security Requirement W-01:** The Wallet must implement the credential format securely and correctly.

### Trust Chain from Verifier to Issuer

The Verifier receives a presentation that is cryptographically tied to a
credential that itself is cryptographically tied to a specific
Credential Issuer.

> **Security Requirement CF-10/TF-10:** For any presentation, the credential format
and trust framework must such that there is a secure way to determine
the Issuer and to check that the original credential was issued by this
issuer. (E.g., by using a cryptographic signature.)

> **Security Requirement CF-20:** For any presentation, the credential
format must ensure that the data contained therein that is tied to the
original credential cannot be not altered. (E.g., by using a
cryptographic signature.)

The root of trust from the perspective of a Verifier is the Credential
Issuer. The Verifier decides, based on policy, regulation, conformity
assessment, and/or contracts, to trust a certain Issuer to issue
Credentials of a certain type with the correct data to the legitimate
End-Users.

> **Security Requirement TF-20:** The trust framework must ensure that the
identification of an Issuer is unique and unambiguous. If there are
multiple instances of the same Issuer sharing the same key material, the
Verifier must trust all instances equally.

For example, a mobile app as issuer which holds the key material in the
app itself is most likely a bad idea - not all instances of this app are
equally trustworthy, some might be hacked and the key material leaked to
malicious parties.

> **Security Requirement TF-30:** The way in which the Verifier
determines the trustworthiness of the Issuer defined in the trust
framework must be secured from influence by a malicious party that can,
for example, introduce untrustworthy entities into a directory.

> **Security Requirement TF-40:** The trust framework must ensure that
there is a way for Verifiers to keep their information on trusted
Issuers up to date and that there is a way to revoke trust in an Issuer.

The Verifier trusts in the Authenticity of Claims as described above.
This means:

> **Security Requirement I-10:** The Issuer must authenticate/identify the
End-User properly according to the expectations of the Verifier (which
may be defined in a specification, trust framework, or by convention).

> **Security Requirement I-20:** The Issuer must only put correct and
up-to-date claims about the End-User into the Credential where verified
data is expected.

(By convention, specification or trust framework,
self-declared or otherwise "unverified" claims may be allowed in some
places in some credentials.)

> **Security Requirement P-10:** The protocol must ensure that no third
> party can extract the credential issued by the Issuer.

> **Security Requirement P-20:** The protocol must ensure that no third
> party can interfere with the issuance process such that the Issuer
> issues Credentials for the third party to the End-User.

> **Security Requirement I-30:** The Issuer must revoke a Credential
once the Issuer learns about potential abuse of the Credential.

The Verifier trusts in the Integrity of the Interaction with the Holder,
as described above. This means:

> **Security Requirement P-30:** The protocol must ensure that the
interaction between the Holder and Verifier is protected such that no
third party can interfere with the interaction by modifying the
information transmitted.

> **Security Requirement P-40:** The protocol must ensure that the
> interaction between an attacker and a Verifier cannot be forwarded to
> and successfully completed by a user.


### Holder Binding

If Holder Binding is required by the use-case of the Verifier, the
Verifier must trust the Issuer to bind credentials to the End-User. This
means:

> **Security Requirement I-40:** The Issuer must only include
holder-binding data into the credential that is tied to the actual
End-User (and not, e.g., include a cryptographic key under control by a
third party).  

> **Security Requirement P-50:** The protocol must ensure that third
> parties cannot interfere with the binding process.

For cryptographic holder binding, the security largely depends on the
quality of the key management, e.g. whether private keys can be
extracted from the user's device and copied to another device. So the
Verifier needs to trust in the wallet to implement the key management
securely. 

> **Security Requirement W-10:** The Wallet must implement the key
management securely such that only the legitimate Holder can use a
credential. (E.g., by using a secure enclave or similar technology.)

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

> **Security Requirement V-20 (conditional):** The Verifier must ensure that
the credential was issued by an issuer that only issues credentials to
trustworthy wallets.

### End-User's Perspective

**TODO: Expand this section**

The End-User trusts the Issuer, the wallet, and the Verifier to treat
her data securely and only exchange data as needed preserving her
privacy. The End-User trusts the Issuer and the wallet provider to
protect her from abuse of her Credentials. On the other hand, the
End-User does not want the Issuer to learn where she uses the
Credentials. 

The End-User trusts the wallet to validate the authenticity of Issuers
and Verifiers and provide the End-User with trustworthy information
about those.

> **Security Requirement W-20:** The Wallet must provide trustworthy and
> complete information about Issuers to the End-User.

> **Security Requirement W-30:** The Wallet must provide trustworthy and
> complete information about Verifiers to the End-User.

> **Security Requirement W-40:** The Wallet must ask the End-User for
> meaningful consent before a Credential is used. The Wallet must
> provide the End-User the opportunity to review any data that is shared
> with a Verifier.

> **Security Requirement P-50:** The protocol must ensure that during an
> interaction with a Verifier, an attacker cannot exfiltrate PII.

> **Security Requirement P-60:** The protocol must ensure that during an
> interaction with an Issuer, an attacker cannot exfiltrate PII.


## Distribution of Identifiers and Keys {#distribution-of-identifiers-and-keys}


The involved parties need to access data about identifiers, keys, and
Credentials as well as data about the acting parties themselves in order
to securely implement the Credential exchange. The way this works is out
of scope for OpenID 4 Verifiable Credential Issuance.


# Security Requirements



# Security Properties

Authenticity of Receiver: The presented receiver of the claims is the actual receiver.


# Resistance against Attacks

Three types of attack are relevant here: 

 1. Authentication Request MITM (attacker replays the initial request between itself and a Verifier in a flow between the attacker and a user)
 1. CSRF (attacker uses a credential issued for themselves and tries to inject it into a flow on a user's device between the user and a Verifier),
 1. the exfiltration of personal data without the user's (proper) consent,
 1. injection (attacker uses a credential issued for a user and tries to inject it into a flow on the attacker's own device to mislead a Verifier).

Note that exfiltration of a presentation on a user's device is a prerequisite for the second and third type of attack.

Let's go through these one by one:

## CSRF

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

## Exfiltration of Personal Data

The target of this attack is for an attacker to get access to a
presentation containing a user's data (without the user consenting to it).

To avoid this, it is important that the redirect URI does not point to an 
attacker-controlled endpoint. 

There are two sources for the redirect URI:

    1. The Verifier specifies the redirect URI in the request to the wallet. The request is signed using a key that the wallet can verify and show the proper name of the Verifier to the user. 
    2. The redirect URI is registered beforehand and therefore guaranteed to be authentic.


## Credential Bound to Wrong Key

> **Security Requirement:** The Issuer must get the correct key for binding the credential to it.

## Session Context Attacks


## Request Encryption?

## Wallet Provided Nonce?

## Difference Same-Device Cross-Device

-> Security Considerations in OpenID 4 VP
