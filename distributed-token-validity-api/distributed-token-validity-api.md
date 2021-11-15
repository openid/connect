% Title = "Distributed Token Validity API"
% abbrev = "dtva"
% category = "std"
% docName = "distributed-token-validity-api"
% area = "Connect"
% workgroup = "OpenID Connect Working Group"
% date = 2017-04-21T00:00:00Z
% ipr = "none"
%
% keyword = ["distributed", "token", "id_token", "validity", "session"]
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
% fullname="David Waite"
% role = "editor"
%   [author.address]
%   email = "david@alkaline-solutions.com"
%
% [[author]]
% initials="J."
% surname="Miller"
% fullname="Jeremie Miller"
%   [author.address]
%   email = "jeremie@jabber.org"
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

OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol. It enables Clients to verify the identity of the End-User based on the authentication performed by an Authorization Server, as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner.

OpenID Connect adds the concept of an identity token (`id_token`) to OAuth token grant responses. Unlike the OAuth 2.0 Access Token, which represents the authorization of a client to a protected resource, the `id_token` represents the authentication of the resource owner to the client.

This document describes an HTTP-based API for management and validation of these tokens that can be deployed in multiple distributed locations/RPs along with a coordination service to maintain a unified view of validity state. It describes a mechanism to dynamically manage and introspect the lifetime and validity for both access and identity tokens. This allows for revocation of tokens by the IDP, as well as the ability to share information around token usage activity to trigger revocation or extend the usage lifetime of tokens.

This document may be used stand-alone or accompanied with others that describe a common coordination service for multiple instances of this API.  When the coordination service can guarantee an eventually consistent view of token validity then the API described in this document can act as a complete solution for token validation.

{mainmatter}

# Introduction

OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 [@!RFC6749] protocol. It enables Clients to verify the identity of the End-User based on the authentication performed by an Authorization Server by receiving an identity token, as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner.

This specification complements OpenID Connect Core 1.0 [OpenID.Core] by allowing a Relying Party to monitor whether the identity token they received is still valid according to the OpenID Provider without relying directly on a front- or back-channel polling mechanism to the OP.  It can also provide a common mechanism for protected resources to monitor whether the OAuth 2.0 access tokens they received are still valid.

The expectation is that any invalidated token will only lead a Relying Party to attempt to retrieve a new replacement token.  By providing a local service for indicating tokens are invalid and forcing a Relying Party and/or any protected resources to fetch new tokens, the OpenID Provider can then utilize a coordination service to dynamically enforce authentication policy and to update authorizations.

This enables an Identity Provider to actively maintain its relationship between a user and the relying party or the client and protected resource over the entire lifetime. Examples of Identity Provider behaviors in this regard include issuing a new token with different authentication/authorization or attribute information, challenging the user for additional proof of identity, or immediately refusing to issue a new token for the user or relying party based on a change in IDP policy.  No matter what behavior the Identity Provider wishes to provide, the signal mechanism of invalidating a token remains the same.

This can be compared against existing session management and logout mechanisms in various Single Sign On (SSO) protocols. Previous protocols have focused predominantly in terminating browser sessions as a user or administrator action, with behavior to take in response often established out-of-band. Example behaviors would include deleting cookies, deleting cached data, preventing automatic re-authentication, and/or defining landing pages containing user information.

These previous protocols have also represented such logout requests as a token representing the action, without delivery guarantees. While a temporary failure in allowing SSO to occur may keep valid users out, a temporary failure in processing logout messages could result in invalid users sessions remaining active.

These previous protocols have generally implemented one or both of the following mechanisms:

1. A "front" channel, where the logout action is indicated through browser actions such as specially-formulated redirects. This channel requires user participation, being inappropriate for cases such as administrator-initiated logout or as part of an access de-provisioning system. Being delivered via the browser, messages can be feasibly blocked by malware, leaving the user agent authenticated.The protocols provide only the most minimal capabilities to catch or diagnose communication issues between the identity provider and relying party. Finally, many deployments due to implementation complexity only delete artifacts such as cookies from the user agent cache as part of logout. If a malicious party captured the appropriate cookies, a logout via this mechanism does nothing to revoke a malicious party's access.
1. A "back" channel, where the logout action is indicated through a direct request/response protocol between the identity provider and relying party. This channel potentially solves the administrative-initiated logout, malware blocking, and connectivity diagnosis problems. However, it is significantly more difficult to develop as well as to deploy, resulting in very few known deployments. Most notably, must be some shared state to allow the back channel communication to affect the front channel user agent actions, making environments where state is maintained purely by cookies impossible to integrate with "back" channel mechanisms. Also, the protocols typically do not define logic to retry transmission of logout actions on temporary failures, still allowing invalidated user sessions to retain access. This model is also only usable by relying parties which can support a back channel communication. Mobile applications, for instance, typically cannot support such a mechanism.

In comparison, this method does not replace the role of those existing protocols, but instead provides a local stateful system where applications can ask if the tokens which make up the authentication and authorization foundation of the user session are valid. If the tokens are not valid, this does not indicate that the application performs any form of cleanup, but instead that actions requested by the user agent must fail or delay until new, valid tokens are retrieved. The stateful system exposing that API is meant to be resilient in the face of temporary connectivity problems by exposing at least a partial view of sessions, and can independently coordinate state changes across many local instances with eventual consistency.

By providing an API, the stateful system can be used either directly by JavaScript or other client logic, or by back-end systems as part of returning results or rendering content. By providing this API locally, multiple components of a domain (such as mobile clients or microservices) can utilize the API with the domain owner able to take responsibility for the resources to manage the reliability and performance characteristics required.

## Requirements Notations and Conventions

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in RFC 2119 [@!RFC2119].

## Terminology

This specification uses the terms "Authorization Endpoint", "Authorization Server", "Client", and "Client Identifier" defined by OAuth 2.0 [@!RFC6749], the term "User Agent" defined by [@!RFC7230], and the terms defined by OpenID Connect Core 1.0 [OpenID.Core].

This specification also defines the following term:

- Token Issuer
  - A party responsible for issuing cross-domain tokens, such as access tokens and identity tokens.
- Token Validator
  - A party responsible for validating that a token is still valid, such as a protected resource or relying party.
- Coordination Service
  - A service that facilitates coordination of state changes between a Token Issuer and distributed Token Validators which can provide guarantees about eventual consistency

# Endpoint Discovery

As the API is local within an identity provider or relying party, there is currently no required endpoint discovery defined for communication between identity provider and relying party for this specification.

In the case where the local API is both protected by and provided as part of an Identity Provider or OAuth 2.0 AS, the following key is defined for use with [@!I-D.ietf-oauth-discovery]:

- **validity_endpoint**: The endpoint that local token validators can look up information on an existing token, and local issuers can create a new token validity record. Should be an absolute, secure HTTP URL.

Otherwise, this value is expected to be communicated out-of-band as an implementation detail for the local environment that the token issuer or token validator is part of.

# Token Lifecycle Model

Token validity is tracked based on the combination of the token issuer and session identifier included in the token.
If multiple tokens contain the session identifier, then they will share token validity state.

This document defines all tokens which support validity as having a limited lifetime, with a lifecycle defined by:

1. The token validity begins when the token issuer requests a new session identifier, for the purposes of embedding within a token
1. Tokens have an issuance time, before which the token could not have been valid
1. The token validity has a hard expiry time, which cannot be extended. After the hard expiry time, the token must be considered invalid
1. The token validity may have events against it which affect whether the token is considered valid. As an example, this specification defines an inactivity timeout mechanism which will cause a token to be considered invalid without periodic user activity.
1. A token may be invalidated by the token issuer. After this invalidation time, the token is no longer considered valid

A token may be made invalid for varied business logic reasons by the token issuer. The reasoning for invalidating a token is not shared with the token validator by this specification. Example reasons for invalidating a token include:

- Wanting user interaction via new code flow request
- Wanting to represent new attributes or authorizations via new tokens
- Needing to revoke tokens (for a single user agent, a user, or a relying party) due to some security event

## Token Interactivity

There may be communications specifically around token usage and lifetime, meant to either directly influence the interpretation of whether a token is valid, or indirectly affect the token issuer in deciding a token is invalid. Within this specification, there is an inactivity timeout mechanism for tokens. However, since tokens may be used to represent both process and user-initiated actions, the inactivity mechanisms here are designed specifically around representing the concept of user interactivity.

- Token interactivity timeout, the maximum time a token can go without reported user interaction before being considered invalid by the issuer.
- Last interactivity, the time of the last user interaction of a token.

Not every application usage of a token can be considered to be interactive. For example, a mail client periodically asking for any new mail message headers would be non-interactive, while a request for a mail message body based on a user click would be considered interactive. This document specifies mechanisms for both interactive and non-interactive usage of a token.

When using an access token against protected resources, whether those protective resources indicate that the client activity is interactive or non-interactive is considered part of the design of the system being protected by OAuth 2.0, and is out of scope for this document.

~~~ ascii-art

 Issuance                                   Hard Expiry
    .-.                                         +-+
   (   )--------------------------------------->| |
    `.'            +-+                          +-+
    ( )----------->| |
     '      .      +-+    +-+
           ( )----------->| |
            '          .  +-+        +-+
                      ( )----------->| |
                       '             +-+
                     Last       Interactivity
                Interactivity      Timeout

  ----------------------------------------------------->
                        Time
~~~

# HTTP API

## System Overview

The API provides the ability for token issuers to issue and revoke tokens, and token validators to both check the validity of a token and to indicate interaction based on a token for the purpose of token keepalive.

Within the tokens, the session identifier ("sid") attribute is used to correlate an issued token to token validity. The API itself does not know the user attributes or authorizations associated with a token. Instead, the "sid" attribute is used to correlate a token with the validity management being done by the system.

No user information/PII is tracked associated with a period within this system - a session identifier is only correlated with user identity, attributes and authorizations via a traditional token

## Communication

Communication as defined here is over HTTP, between a client and a set of Distributed Token Validity API (DTVA) resources.

## Authorization to API

It is RECOMMENDED that the HTTP communication between a token validator to the local API be protected via authorization. It is REQUIRED that HTTP communication between a token issuer and the local API be protected via authorization.

One such authentication mechanism would be OAuth 2 bearer tokens. It MAY be considered appropriate to have the access to the local API be considered distinct from the authorization provided by the Identity Provider's access tokens, that a distinct AS is used to control access to the local API as a protected resource.

### Token Validity Location

Requests for a particular token validity are performed initially by HTTP GET against the local token validity URL with an appropriate query attached. The query portion is defined by the request parameters below, and encoded by [application/x-www-form-urlencoded](https://www.w3.org/TR/html5/forms.html#application/x-www-form-urlencoded-encoding-algorithm):

- **iss**: (optional) the identifier of the issuer of the token. If more than one token issuer is supported by the system, this value is required
- **sid**: the subject identifier of the token

The validity of a particular token MAY be represented by a permanent URL. A request for the token validity URL MUST result in a 308 redirect to a permanent URL, if exists. Implementations SHOULD keep track of the permanent location of the token validity in order to optimize further validations of the given token. The permanent URL for a token validity MAY contain a separate authority from that of the token validity URL. In this case, that authority MUST accept the same authentication as the token validity URL.

## HTTP Response Status Codes

The following list of HTTP status codes are represented within this document as response statuses for the various HTTP requests. Implementations MAY give other error codes as responses which are not covered by this list.

- 200 **OK**
  - Responses which contain authoritative information. Modifications to token validity (POST or DELETE) may return this status if the returned data represents a state with the change applied.
- 201 **Created**
  - Requests which create a token validity record immediately. The authoritative token validity is represented within the response. The `Location` header MUST contain either the token validation location URL with appropriate parameters encoded, or the permanent URL of any entity created to represent this token validity.
- 202 **Accepted**
  - Requests which create or modify the token validity return this response to represent that the process of creation/modification has been acknowledged, but not yet completed. The `Location` header MUST contain either the token validation location URL, or the permanent URL of any entity created to represent the token validity. Further requests against that location for MUST return either a 200 to indicate authoritative information or 203 to indicate non-authoritative information, from the point of this return to the point where the hard expiry of the token has been reached. Further attempts to modify the record may fail.
- 203 **Non-Authoritative Information**
  - Responses which have known stale or incomplete responses may return this status rather than blocking until authoritative data is available
- 304 **Not Modified**
  - GET/HEAD requests containing cache validators such as If-Modified-Since may return this response to indicate the validity record is up-to-date
- 308 **Permanent Redirect** [@!RFC7538]
  - Token validator requests against the validity endpoint for token information may result to redirection to a resource representation of a particular token validity.
- 400 **Bad Request**
  - May be returned for one of the following reasons:
     1. Token validity entity submitted does not contain all the necessary information
     1. Token validity creation submitted contains unknown or invalid information
     1. Attempt to change invariant or unknown values of token validity
- 401 **Authorization Required**
  - Authorization required for use of this endpoint. It is RECOMMENDED that OAuth 2 be supported as an authentication mechanism
- 403 **Forbidden**
  - Authorization is insufficient for the requested use of this endpoint.
- 404 **Not Found**
  - May be returned for requests for unrecognized token session identifier, including against permanent URLs which do not appear to coordinate to a token session identifier. Tokens which have gone past their hard expiry time MAY return a 404.
- 409 **Conflict**
  - POST update for a token which conflicts with current token validity state
- 410 **Gone**
  - GET/POST/DELETE on an invalidated token

## Token Validity Response

The response body of successful token validity requests, including creation and modification, will be a JSON [@!RFC7159] document.

- **sexp**: The date/time of hard expiry as enforced by the token validity service. The value is string formatted as a [@!RFC3339] internet date/time in Z/Zulu timezone offset.

- **sid**: The session identifier of token(s) this validity refers to.

- **issuer**: identifier of the issuer of the token(s) this validity refers to.

- **interactivity_timeout**: (optional) A positive integer number of seconds after the time of last user activity at which a token will be considered irrevocably invalid, if specified

The following HTTP caching headers SHOULD be used, and have the indicated meanings:

- **Last-Modified**: the time that the token record was created
- **Expires**: the time after which a token issuer or validator MUST check the system before assuming a token is still valid. This must be less than or equal to the `exp` time. This must also be less than or equal to the interactivity timeout plus time of last activity. The API implementation SHOULD adjust this time to reduce the potential of stale validity data being cached by intermediaries.

Invalid tokens as well as errors are RECOMMENDED to use JSON problem details for the body, as defined by [@?RFC7807].

## Creating a new token validity record

### Creation Request

A new record is created by a token issuer POST request against the validity URL. The request document is a JSON document or URL-encoded form containing the following values:

- **sexp**: The date/time of hard expiry as enforced by the token validity service, after which a token based on the session identifier returned from this create request will never be considered valid for use. The value is string-formatted as a [@!RFC3339] internet date/time in Z/Zulu timezone offset. 

  If a token indicates a period for which tokens are valid for processing (such as the `exp` attribute in JWT [@?RFC7519]), tokens MUST NOT be valid for processing after the `sexp` time. If a token indicates that claims are valid for use for a period of time, it is RECOMMENDED that time be less or equal to the `sexp` time, and this token validity spec MUST be considered authoritative.

**interactivity_timeout**: (optional) A positive integer number of seconds after the time of last user interactivity at which a token will be considered irrevocably invalid.

**iss**: (optional) The name of the token issuer which will be used by the token validation API. If more than one token issuer name is valid within the system, this value MUST be specified. If specified, the token issuer MUST be authenticated, and token issuer authorization MUST be verified to allow them to issue tokens with the specified name.

### Creation Response

If the request was valid, the response will have a status of:

- 201 **Created**
- 202 **Accepted**

## Validate Token

Validating a token allows a system to determine whether or not it should still use a token. This is similar to updating a token validity, which not only returns whether the token is valid but also allows providing additional information and can be used to report user activity.

Validation MAY use a cached HTTP response, in particular if the time specified in the `Expires` header has not yet passed.

### Validation Request

In addition, a request may also include the following caching header:

- **If-Modified-Since**: the time reported for Last-Modified on a previous token validity response

The validity of a particular token MAY be represented by a permanent URL. In this case, a request for the token validity URL may result in a redirect to the authoritative location. Implementations MAY keep track of the permanent location of the token validity if they wish to optimize the process of validating tokens in the future.

### Validation Response

If the request was valid, the response should be either:

- 200 **OK**
- 203 **Non-Authoritative Information**

If the token validation has a permanent URL, the response MUST be:
- 308 **Moved Permanently**

at which point, the request should be repeated against the permanent URL.

Invalid requests or requests corresponding to invalid tokens may be represented by:

- 400 **Bad Request**
- 404 **Not Found**
- 408 **Gone**

## Update Token Validity

This document defines a single type of update to tokens - indicating that user interactivity was detected locally. Future specifications may define other types of updates.

### Update Request

An update request is made via POST to the token validation location, as documented above. The request document MUST be a JSON object, containing at least one key.

The parameters during creation are considered immutable, and MUST NOT be updatable. The following single parameter is defined for updates by this document:

- **interaction_detected** : indicate that a end user was detected interacting via a user agent with the local environment. If present, must be the boolean value `true`.

### Update Response

An update may respond successfully with:
If the request was valid, the response should be either:

- 200 **OK**
- 202 **Accepted**
- 203 **Non-Authoritative Information**

If the token validation has a permanent URL, the response MUST be:
- 308 **Moved Permanently**

at which point, the request should be repeated against the permanent URL.

Invalid requests may be represented by:

- 400 **Bad Request**
- 404 **Not Found**
- 409 **Conflict**
- 410 **Gone**

The body of the response is a token validation response. Note that the update process MAY be eventually consistent based on the implementation of the API, and may not represent the requested update having been applied. The requested update may also eventually fail to be applied at some point in the future (such as an interactivity seen after the implementation decided the token was invalid). 

The response should not be taken to indicate the parameters were or were not taken into account in deciding validity, only that from the perspective of the API that the token continues to be valid for the time being.

## Indicate Token is Invalid

The token issuer can indicate one or more tokens should no longer be used by expiring the corresponding `sid`.

### Invalidation Request

An invalidation request is made via DELETE to the token validation location, as documented above. No other headers or entity is required for this specification.

### Invalidation Response

On success, the response status will be one of:

- 200 **OK**
- 202 **Accepted**

The following response statuses are defined for errors:
- 404 **Not Found**
- 410 **Gone**

The body of the response is a token validation response.

# Interaction with other specifications

## OAuth2 Access Token Bearer

Access tokens represent the authorizations of a client against one or more protected resources.  As access tokens already MUST be verified by the protected resource, and OAuth 2 defines that access tokens may be opaque to the client, a client typically will not use this specification to verify access tokens.  Instead, the protected resource will check the validity of the tokens, either by extracting `sid` from the token and using the API specified in this document, or by using token introspection.

## Token introspection

In an environment where the AS is participating in distributed token validity, the [token introspection endpoint](https://tools.ietf.org/html/rfc7662) response MUST indicate the token is not `active` if the token validity API returns that the token is invalid.  Protected resources using token introspection MAY skip checking token validity via other mechanisms.  The token introspection response MUST also include the `sid` property in its response.

If caching is used for the token introspection response, it is RECOMMENDED that the cache duration match the cache duration returned by the token validity API.

## OpenID Connect

Identity Tokens (id_token) are messages to the client by the Identity Provider about the user.  As the client is expected to maintain authentication based on the validity of this token, a client MUST use either this API or an alternative mechanism to verify the validity of the token.

## Front-Channel Logout, Back-Channel Logout

The [OpenID Connect Front-Channel Logout](http://openid.net/specs/openid-connect-frontchannel-1_0.html) and
 [OpenID Connect Back-Channel Logout](http://openid.net/specs/openid-connect-backchannel-1_0-04.html) draft specifications deal not with the invalidation of a token, but with an action revoking user agent access to the application, either temporarily or permanently.  This includes recommended steps such as session cookie deletion and cache clearing.

This specification does not provide an equivalent explicit signal that access is revoked, only that access requires acquiring a new token - that the token expired prematurely.  The Identity Provider determines what steps, if any, will allow a client to resume access.  The choice to distinctly separate token validity from logout was intentional, enabling Identity Providers a means to manage many different types of token lifecycle events as well as adopt distributed technologies to provide reliable coordination services.

As the signaling to revoke user agent access and expire caching can fail in both of the existing logout specifications, at this time it is suggested instead that such signaling happen out-of-band with this specification. For example, managed devices or applications may receive a message directly telling them to revoke access or even uninstall a client which should no longer have access, which could be triggered during a re-auth after a token invalidation.

## Session management

The [OpenID Connect Session Management 1.0](http://openid.net/specs/openid-connect-session-1_0.html) draft defines a Javascript mechanism using frames and posted messages for monitoring a session at the Identity Provider. The connotation for an expired session is re-authentication and not cache invalidation, which pairs well with this specification.

This specification provides three additional mechanisms for enhancing Session Management.

1. Rather than using a frame and posted messages, an AJAX API can be used to check token validity.
1. Rather than loading a remote frame from an identity provider into an application, session management can be implemented directly in the application with its own local javascript and without frames.
1. HTTP caching headers can be used to enforce validity in supported User-Agents.

## Security Event Tokens

The Security Event Token (SET) [@?I-D.ietf-secevent-token] draft defines common semantics for a JWT that can carry session-related events as an underlying common format for any OpenID or other specifications that require communication of modifications to an existing state.

This specification is one step removed from the format of these events, as it only communicates _about_ the _cached validity state_ of an existing token, not the contents of the token nor any changes to it.  Therefore it is supportive of and additive to systems which have adopted SETs.

Any events that are communicated between an OpenID Provider and Relying Parties using SETs SHOULD be examined for potential modifications to the cache validity of all related tokens.  When the tokens are destroyed as the result of a logout in a SET (either front- or back-channel) then all API requests for the `sid` MUST fail.

# Security Concerns

Lorem Ipsum Time synchronization etc etc.

{backmatter}

# Acknowledgements

The author wishes to thank all those poor friends who were kindly forced to read this document and that provided some nifty comments.

# Notices

Copyright (c) 2017 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.