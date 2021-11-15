# Distributed Token Validity API

This folder contains the contribution "Distributed Token Validity API", formerly "Distributed Session Management".

The system described contains an API leveraging a JWT token session identifier to check for validity. This is distinguished from the OAuth Token Introspection Endpoint in several ways, the most important being:

- The request only needs to provide the identifier and not the entire token
- The response indicates only validity and not contents
- The process can be applied to both access tokens and id_tokens
- The API is designed to encourage local caching of responses by the RP, for example by invoking the OP endpoint via a local caching HTTP Proxy.

In addition to the core API, there is a document describing how this could be leveraged using a distributed consensus process. This document describes how to leverage a distributed application running on the Hashgraph system, which is the precursor to the Hedera Network. A prototype implementation was created at https://github.com/pingidentity/dtva-reference .

The Hashgraph system was leveraged for secure gossipping of updates as well as for consensus ordering being driven by a consensus-derived clock for events.

The specification details approaches for rectifying some of the issues with using decentralized systems, such as an approach to handle requests for validity on new tokens which have outpaced consensus updates. It also defines an update mechanism, by which the distributed system can support a policy for token expiration due to inactivity.
