# OpenID Connect Repository

This is the [**OpenID Connect Working Group (AB/Connect WG)**](https://openid.net/wg/connect/) repository for the original [**OpenID Connect 1.0**](https://openid.net/connect) specifications.
It is used for tracking issues, discussing enhancements, and maintaining the specifications that
define OpenID Connect and some extensions to it.
The full list of OpenID Connect working group repositories is at https://openid.net/wg/connect/repositories/.

---

## Specifications and Documents

These specifications are in this repository:

* [**OpenID Connect Core**](https://openid.net/specs/openid-connect-core-1_0.html) – Defines the core OpenID Connect functionality: authentication built on top of OAuth 2.0
  and the use of claims to communicate information about the End-User.
* [**OpenID Connect Discovery**](https://openid.net/specs/openid-connect-discovery-1_0.html) – Defines how clients dynamically discover information about OpenID Providers.
* [**OpenID Connect Dynamic Client Registration**](https://openid.net/specs/openid-connect-registration-1_0.html) – Defines how clients dynamically register with OpenID Providers.
* [**OpenID Connect RP-Initiated Logout**](https://openid.net/specs/openid-connect-rpinitiated-1_0.html) – Defines how a Relying Party requests that an OpenID Provider log out
  the End-User.
* [**OpenID Connect Session Management**](https://openid.net/specs/openid-connect-session-1_0.html) – Defines how to manage OpenID Connect sessions, including postMessage-based
  logout functionality.
* [**OpenID Connect Front-Channel Logout**](https://openid.net/specs/openid-connect-frontchannel-1_0.html) – Defines a front-channel logout mechanism that does not use an OP iframe
  on RP pages.
* [**OpenID Connect Back-Channel Logout**](https://openid.net/specs/openid-connect-backchannel-1_0.html) – Defines a logout mechanism that uses direct back-channel communication
  between the OP and RPs being logged out.
* [**OpenID Connect Core Error Code unmet_authentication_requirements**](https://openid.net/specs/openid-connect-unmet-authentication-requirements-1_0.html) – Defines the unmet_authentication_requirements
  authentication response error code.
* [**OAuth 2.0 Multiple Response Types**](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html) – Defines several specific new OAuth 2.0 response types.
* [**OAuth 2.0 Form Post Response Mode**](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html) – Defines how to return OAuth 2.0 Authorization Response parameters (including
OpenID Connect Authentication Response parameters) using HTML form values that are auto-submitted by the User-Agent
using HTTP POST.
* [**Initiating User Registration via OpenID Connect**](https://openid.net/specs/openid-connect-prompt-create-1_0.html) – Defines the prompt=create authentication request parameter.

The full list of OpenID Connect Working Group specifications is at https://openid.net/wg/connect/specifications/.

---

## Contributing & Issue Tracking

This repository was migrated from https://bitbucket.org/openid/connect in June 2026.

### Filing Issues
If you find a bug, inconsistency, or ambiguity in any of the active specification drafts or finalized specs:
1. Search the [Existing Issues](https://github.com/openid/connect/issues) to ensure it hasn't already been raised.
2. Open a new issue and tag it with the appropriate component label (e.g., `Core`, `Discovery`).
3. Provide explicit references to the section or line number of the spec you are referring to.

### Intellectual Property (IP) Notice
Contributions to this repository are subject to the [**OpenID Foundation Intellectual Property Policy**](https://openid.net/ipr). To contribute text, pull requests, or major revisions, you must be a member of the OpenID Foundation and have signed the **Contribution Agreement**.

* Learn more about participating in the working group at https://openid.net/wg/connect/.
