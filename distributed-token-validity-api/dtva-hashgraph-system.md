% Title = "Distributed Token Validity using Hashgraph v1"
% abbrev = "dtva"
% category = "info"
% docName = "dtva-hashgraph-system-1.0"
% area = "Connect"
% workgroup = "OpenID Connect Working Group"
% date = 2017-04-21T00:00:00Z
% ipr = "none"
%
% keyword = ["distributed", "token", "id_token", "validity", "session", "hashgraph", "swirlds"]
%
% [pi]
% private = "Draft"
% compact = "yes"
% subcompact = "no"
% tocdepth = "5"
% topblock = "yes"
% comments = "no"
% iprnotified = "no"
%
% [[author]]
% initials="D."
% surname="Waite"
% fullname = "David Waite"
% role = "editor"
%   [author.address]
%   email = "david@alkaline-solutions.com"
% [[author]]
% initials="J."
% surname="Bradley"
% fullname="John Bradley"
% organization = "Ping Identity"
%   [author.address]
%   phone = "+1.202.630.5272"
%   email = "jbradley@pingidentity.com"
%   uri = "http://www.thread-safe.com/"
%   [author.address.postal]
%   street = "Casilla 177, Sucursal Talagante"
%   city = "Talagante"
%   country = "Chile"
%   region = "RM"
<!--
  NOTE:  This Markdown file using mmark syntax and any generated XML file is input used to produce the authoritative copy of an OpenID Foundation specification.  The authoritative copy is the HTML output.  This XML source file is not authoritative.  The statement ipr="none" is present only to satisfy the document compilation tool and is not indicative of the IPR status of this specification.  The IPR for this specification is described in the "Notices" section.  This is a public OpenID Foundation document and not a private document, as the private="..." declaration could be taken to indicate.
-->

.# Abstract

This document describes a mechanism for synchronizing the validity state of both OAuth 2.0 access tokens and OpenID Connect identity tokens via hashgraph, a distributed mechanism for consensus with byzantine fault tolerance.

Using this mechanism, Identity Providers and Relying Parties can deploy local systems for token validity. These local systems can be deployed to meet their own needs with respect to system latency, throughput, and security policy - rather than sharing infrastructure run by a single participant in the overall identity system.

This document allows for revocation of tokens by the token issuer as well as the ability to distribute information about token usage activity across participants such that they all still have a uniform view of the state, either to trigger revocation by the issuer or to extend the usage lifetime of tokens.

{mainmatter}

# Introduction

OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 [@!RFC6749] protocol. It enables Clients to verify the identity of the End-User based on the authentication performed by an Authorization Server by receiving an identity token, as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner.

This specification complements OpenID Connect Core 1.0 [OpenID.Core] by allowing participating Identity Providers and Relying Parties monitor whether an identity token they received is still valid. This specification also complements OAuth 2.0 allowing participating Authorization Services and Protected Resources to monitor whether access tokens are still valid.

The validity of these tokens is according to both the Identity Provider/Authorization Service who issued the token, and a policy throughout the distributed system around said tokens - both with regards to explicit events, and behavior related to things like usage timeouts.

This specification details several implementation details of a hashgraph-based distributed token validation service. Due to hashgraph coordinating not just events but signed system state, every participating deployment must have a common interpretation of events and a common representation of state. For this reason, the participants must all conform to the same version of this specification so that they are uniformly processing and signing state changes identically.

Implementations conforming with this spec may wish to also implement the Distributed Token Validity API, to provide a complete solution for token validation.

## Compatibility

This specification details several implementation details of a hashgraph-based distributed token validation service.

Hashgraph implementations MAY use co-signed state to collapse events into a signed system state. As implementations generate the canonical system state locally, every participant is expected to have a common interpretation of events applied to generate this canonical representation of state.

Modifications or extensions to this specification thus typically cannot made with forward or backward compatibility, and must be agreed upon by all participants. For this reason, the specification itself is has a version to allow for participants to agree upon the events and state representation within the system they are participating within.

## Requirements Notations and Conventions

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in RFC 2119 [@!RFC2119].

## Syntax Notation

This specification uses the CBOR Data Definition Language of [@!I-D.greevenbosch-appsawg-cbor-cddl#9]. [#grammar] shows the collected grammar across all parts of this specification.

Note that this specification gives additional format and processing requirements. Therefore, validation against the CDDL grammar is insufficient for messages to be processed as valid per this specification.

## Terminology

This specification uses the terms "Authorization Server" and "Client" defined by OAuth 2.0 [@!RFC6749], the terms "Hard Expiry", "Interaction Timeout", "Token Issuer", and "Token Validator" as defined by Distributed Token Validation API, and the terms defined by OpenID Connect Core 1.0 [OpenID.Core].

"Session Identifier" is to be tracked to an appropriate external document.

"Hashgraph", "Participant", and "Transaction" are described by the paper [The Swirlds Hashgraph Consensus Algorithm](http://www.swirlds.com/downloads/SWIRLDS-TR-2016-01.pdf)

- **Validity Key**

# Overall System Description

All participants in the hashgraph are considered Token Validators, while certain participants are also considered Token Issuers. The issued tokens are not exchanged directly within this system. Instead the system coordinates token validity in terms of a session identifier which is embedded or associated with one or more access or identity tokens.

The system described is meant to contain and coordinate no personally identifying information, with correlation of a particular subject and accessing user agent happening only once a relying party or client receives an issued token associated with the session identifier.

Session identifiers issued by this system are structured, and the data within them can be used to determine a validity key, the actual value tracked by the hashgraph.

Transactions within the system exist to register a validity keys, associate with them information on user activity, or mark them as invalidated by the token issuer. These transactions are processed in consensus order by a consistent set of rules, allowing each participant to maintain a consistent view of the overall state of the system.

A validity key **MAY** be associated (via session identifiers) with tokens issued to multiple independent parties, assuming the overall system security and privacy policy allows for it. The [#Security Considerations] section details this further.

Transaction ordering in hashgraph is ultimately determined by all participants establishing a "consensus time" that the transaction was first seen by a significant portion of the network. Because the system of token validity is fundamentally tied to the time domain, processing of transactions is done (nearly exclusively) using this consensus time rather than system time.

# Validity Key

As described in [@distributed-token-validity-api], a token's lifetime is defined by several values including a hard expiry and an optional interactivity timeout.

Validity keys are variable-length binary keys used to indirectly track one or more token lifetimes.  Validity keys are compound keys, containing within them the configurable values that control a token's lifetime.  These values are thus considered invariant over the lifetime
of the validity key and associated tokens.

The compound key is a binary value expressed via CBOR [@!RFC7049].

``` cddl

;;; Types

; time expressed as an integer number seconds after posix epoch time
ptime = uint

; integer number of seconds
duration = uint

; hard expiry time of token validity, after which associated tokens
; must not be accepted
hard-expiry-time = ptime

; integer index corresponding to the issuer name. Before the issuer
; can create tokens, the issuer name must be part of consensus state
issuer-index = uint

; the maximum time allowed in-between user activity events, or nil if
; interactivity is not being tracked for associated token validity. A
; token which goes longer than the interactivity time without user
; interaction reported will be considered expired
interactivity-timeout = duration .min 1 / nil

; used to allow multiple validity keys with the same configuration to
; be issued, for example on behalf of multiple users authenticated
; with the same policy during a sub-second window.
nonce = uint

; a compound key used as the system name for representing token
; validity. Various transactions reference this key to update the
; token validity state.
validity-key = [
  hard-expiry-time,
  issuer-index,
  interactivity-timeout,
  nonce
]
```

The following is an example of a validity key decomposed into hexadecimal-formatted octets: `0x841a58c33e001019038410`

``` ascii-art
84              -- 4 element array
   1a 58c33e00  -- expiry at midnight, March 11th, 2017 (UTC)
   10           -- issuer index 0
   19 0384      -- 15 minute inactivity timeout
   10           -- marker of 0

```

## Session Identifier

Establishing consensus has several additional steps and happens over a different channel (hashgraph communication) than the transmission of the tokens. Because of this, it is possible that requests for token validity via session identifier may be attempted before the local system has a consensus view of the token issuer and validity requirements.

Rather than requiring the system to require token validators to block until consensus is established, the minimal information is provided embedded within a structured *session identifier token*. The validity key is included within the session identifier, which includes both information on how long the token is to be considered valid and the information needed to communicate about the token within the hashgraph system.

In addition, a "Consensus Grace" time can be set to limit how long implementations may assume a token is valid outside of consensus. If a validation request is received with a session identifier where the validity key is not yet known, the consensus grace **MAY** be used to provide a non-authoritative answer.

``` cddl
; time the grace period expires for participants to accept queries
; around tokens without consensus established yet, or nil if no grace
; period has been provided
consensus-grace-time = ptime / nil

; the session identifier embedded in / associated with a given token.
; the value is composed of the compound key as well as a grace period
; to indicate processing should succeed for a short period time in
; the absence of consensus state about the token session
session-identifier-token = [
  compound-key,
  consensus-grace-time
]
```

The format of the session identifier token is a two element array. The first element is the compound key as detailed above, while the second value is  the `Consensus Grace` timeout, which is encoded as a negative offset from the hard expiry, in seconds, or `null` if no grace necessary (such as if a preexisting key is being used in a new token)

``` ascii-art
82               -- 2 element array
  84             -- 4 element array
    1a 58c33e00  -- expiry at midnight, March 11th, 2017 (UTC)
    00           -- issuer index 0
    19 0384      -- 15 minute inactivity timeout
    00           -- marker of 0
  39 7044        -- negative offset of 7 hours, 59 minutes
                 --  from hard expiry
```

## Time interpretation

The hard expiry time is set by the token issuer and corresponds most likely to the issuer's local time and not an estimate of consensus time. However, a participating system SHOULD NOT modify the hard expiry time, as the particular time may also be coordinated with other processing instructions in the token.

The hard expiry time is most likely computed by the issuer in terms of a token lasting a certain number of minutes, hours, or days, while it is expected the difference between local time and consensus time will be at most seconds. Because of the relative difference, this spec considers the hard expiry acceptable to process as if it was specified in consensus time.

The consensus grace is set by the participating system of the token issuer. This grace is represented within local time, and is never an element for system consensus. It is expected this value will represent both an appropriate margin for establishing consensus on the creation transaction as well as to allow for clock drift in local time across all participants.

# Hashgraph Constitution

## Configuration

- **Maximum Expiry Duration** - Participants joining a hashgraph agree that no token by any issuer can live longer than the maximum duration. This is mostly done for data and cache expiry, as even invalidated tokens continue to have records until they reach their hard expiry time. This value applies to the creation transaction, detailed below.

## Participation

Token issuers and/or validators can participate in a hashgraph in order to have a local copy of consensus data. Participants are marked as whether they have permission to be a token issuer; this merely allows them to register issuer names.

# Consensus State

## Data Model

There are two sets of data within the system, representable via the following abstract data types:

The issuer names are represented as a List in registration order. Associated with each issuer name is the participating token issuer who registered it.

Token validity is represented as an associative array of validity keys to a record of mutable data. The validity keys which have passed their hard expiry time are not represented within this associative array.

This record two values, last interactivity time and invalidation time. The last interactivity time is set initially to consensus creation time on creation, and if interactivity is tracked for the validity key will be updated by interactivity transactions. The invalidation time records if the token issuer requested the validity key be explicitly invalidated earlier than the hard expiry, and when that transaction was received.

Because the validity key contains the hard expiry time, and even on invalidation is used through that hard expiry time, it is not possible for a valid validity key to be reused later.

## Binary Representation

A consistent binary representation of state may be used by the system implementing hashgraph to optimize or discard historical transactional data. In these systems, the binary representation is independently generated and cosigned by each participant of the system.

Within this system, the binary state up to a given transaction is represented as a two element CBOR array.

The first element is the issuer array in registration order. Each element is a pair of NFC-normalized issuer names as text strings and the hashgraph participant.

The second element is an array of validity keys in binary sort order. Each array element contains the validity key, the interactivity time, and invalidation time recorded within it.

``` cddl
; name of issuer as used within tokens. This string must be NFC-
; normalized per Unicode specs
issuer-name = tstr

; common identifier of a participant across all instances of the
; hashgraph
participant = uint

issuer-table-item = [
  issuer-name,
  participant
]

; table of issuers in order of registration
issuer-table = [* issuer-table-item]

; time of last reported interactivity as seconds since epoch, or
; consensus creation time of record if no interactivity reported or
; interactivity tracking is disabled
last-interactivity-time = ptime

; time of invalidation by the token issuer, or nil if token was not
; explicitly invalidated
invalidation-time = ptime / nil

; mutable state of each record
per-key-record = (
  last-interactivity-time,
  invalidation-time
)

; recorded information against each session
session-record = [
  validity-key,
  per-key-record
]

; session state representation at a particular point in the
; transaction history. Some hashgraph implementations may share
; co-signed state to collapse the transaction history
session-state = [
  issuer-table,
  [* session-record]
]
```

## Additional Consensus Data

A participant **MAY** have a consensus data model which goes above and beyond the model used for state synchronization indicated above. However, the participant **MUST** still be capable of representing the above state for synchronization purposes. Note if the hashgraph implementation uses signed state to collapse history, such an implementation may develop gaps in its data if it is disconnected or otherwise partitioned from graph participation for a long enough period of time.

## Local Pre-Consensus State

A participant **MAY** have an additional local data based on transactions processed before consensus has been established. Such local state **MUST NOT** affect the ability to generate a signed state based on consensus, and **MUST** be represented to any consuming local systems as non-authoritative information. 

A common use of this is token invalidation, where reporting tokens as invalid to local systems immediately is worth the added complexity of representing non-consensus state. Token invalidation thus **MAY** be expedited, acted upon immediately on receiving a cryptographically valid transaction and represented as temporary local state, without waiting for consensus ordering to be established. Since the system will eventually represent this transaction as part of the consensus-ordered transactions, this does not break eventual consistency.

Creation of a validity key MAY also be tracked via non-consensus state, specifically if a participant does not wish to honor the "consensus grace" value within session identifier tokens. In addition to normal processing, an implementation **MUST** require any referenced issuer name to be previously established via consensus state. An implementation **MUST** also be prepared to stop recognizing the validity key if consensus is not established after a certain period of local time.

## Local view of token validity

The local participant is responsible for representing time when querying token validity. For this reason, it is **RECOMMENDED** that local system time be synchronized across participant infrastructure.

If validity state is meant to be cached (such as via HTTP caching headers), such caching **MUST** have time-based limits to support reasonable recognition of invalidated token sessions.

If validity is reported based on non-consensus data, such as the "consensus grace" value or recognition of creation/invalidation transactions before consensus is established, such values **MUST** be represented as non-authoritative data, and **MUST** have appropriate time-based limits on any caching to encourage clients to move to authoritative data in a reasonable amount of time.

# Hashgraph Transactions

## Transaction semantics

Transactions within this system are binary data, represented by a leading byte value and transaction-specific data. It is assumed that the hashgraph system itself uses appropriate signatures to authenticate the sending participant and integrity-protect the data.

Except for the special additional case of invalidation transactions, transactions are processed on consensus, with the participant and consensus time as additional inputs. Transactions are thus processed in order, based solely on the transaction and an existing consensus view of state, leading all systems to maintain eventual consistency.

Transactions are represented in this system by a two element CBOR array. The first element is an integer transaction identifier, while the second element is the data of the transaction.

``` cddl
transaction = [
  tx-kind: uint
  data: any
]
```

Transaction processing has three outcomes:

- **Accepted** : an accepted transaction will lead to update system state.
- **Ignored** : an ignored transaction does not update system state, and behaves as if the transaction had not been received
- **Rejected** : a rejected transaction does not update system state, and may also lead to escalation. Currently escalation is RECOMMENDED to be indicated in some administrative-level reporting.

## Register Issuer Name Transaction

Participants with issuer permission have the ability to register unlimited issuer names. There is no dispute resolution process currently, with issuer names being considered first-come-first-serve by consensus order.

The format of this transaction is two element CBOR array, with the first element being a transaction type represented by the integer 0, and the second element being a text string of a stripped, NFC-normalized Unicode issuer name in UTF-8.

```cddl
; transaction to register a validity key within the system
register-issuer-tx = [
  tx-kind: 0,
  issuer-name
] .within transaction
```

The transaction **MUST** be rejected if any of the following are true:

- The participant does not have issuer privilege
- The issuer name is already registered
- The issuer name is not NFC-normalized Unicode
- The issuer name has leading or trailing whitespace characters
- The issuer name contains control characters
- The issuer name is empty

Note that several of the checks above are done to limit the ability to add visually similar issuer names to the table. There are other more complex cases in Unicode strings (such as degenerate cases with a single combining mark, or use of private character spaces) which are not currently prohibited.

An example transaction registering the issuer name `https://example.com`:

``` ascii-art
82                         -- 2 element array
  00                       -- transaction type 0
  73 68 74 74 70 73 3a 2f  -- 19 byte text string of "https://example.com"
     2f 65 78 61 6d 70 6c
     65 2e 63 6f 6d
```

## Create Validity Key

Participants with issuer permission have the ability to create unlimited validity keys. Validity keys are valid from the point of consensus time on their creation, to the requested hard expiry. Externally, tokens contain session identifiers formed from the validity key.

The format of this transaction is a two element CBOR array, with the first element being a transaction type represented by the integer 1, and the second element being the validity composed of requested values.

``` cddl
; transaction to create a new validity key, with creation parameters
; embedded within the compound key
validity-key-create-tx = [
  tx-kind: 1,
  validity-key
) .within transaction
```

The transaction **MUST** be rejected if any of the following are true:

- the participant does not have issuer privilege
- the validity key is already known by consensus state
- the hard expiry in the past compared to consensus time
- the hard expiry that is more than the configured `maximum expiry duration` seconds after consensus time
- the interactivity timeout is zero
- the issuer index is unknown
- the issuer index corresponds to a name registered to a different participant

Otherwise, the session identifier is accepted and represented within the state of the overall system. Creation is considered the first interactivity.

## Update Interactivity for Session Identifier

Any participant has the ability to update interactivity for a session identifier which is set to track interactivity. Interactivity is considered at consensus time of the transaction, causing either the session identifier to be considered active or invalidated across all participants consistently.

The format of this transaction is a two element CBOR array, with the first element being a transaction type represented by the integer 2, and the second element being the compound key composed of requested values.

```cddl
; transaction to update the interactivity time associated with a
; validity key
update-validity-key-tx = [
  tx-kind: 2,
  issuer-name
] .within transaction
```

An example of this transaction 
The transaction **MUST** be ignored if any of the following are true:
- the validity key is not known to the system (such as being received after hard expiry)
- the validity key represents a session which has already been invalidated
- the validity key represents session which is not tracking interactivity
- consensus time is more than `interactivity timeout` seconds since last accepted interactivity for the validity key

Otherwise, the last interactivity time associated with a session identifier is updated to consensus time.

## Invalidate Session Identifier

The issuing participant of a given session identifier has the capability to invalidate that session identifier. Invalidation is considered at consensus time of the transaction, and if accepted will invalidate that transaction with respect to all future transactions.

The format of this transaction is a two element CBOR array, with the first element being a transaction type represented by the integer 3, and the second element being the compound key composed of requested values.

```cddl
; transaction to update the interactivity time associated with a
; validity key
invalidate-validity-key-tx = [
  tx-kind: 2,
  issuer-name
] .within transaction
```

The transaction **MUST** be rejected if any of the following are true:
- The participant was not the issuer of the original validity key
- the validity key has already been invalidated
- the validity key is unknown, but has not yet reached hard expiry time

Otherwise, the transaction **MUST** be ignored if the validity key is unknown and has passed hard expiry

# Security Concerns

To complete:

- Time synchronization in local environment and between participants
- Correlation
  - Statistical user behavior gathering
  - User interaction with other relying parties

<reference anchor='distributed-token-validity-api' target='./'>
    <front>
        <title>Distributed Token Validity API</title>
        <author initials='D.' surname='Waite' fullname='David Waite'/>
        <author initials="J." surname='Miller' fullname='Jeremie Miller'/>
        <date year='2017'/>
    </front>
</reference>

{backmatter}

# CDDL

To gather

# Acknowledgements

The author wishes to thank all those poor friends who were kindly forced to read this document and that provided some nifty comments.

# Notices

Copyright (c) 2017 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.