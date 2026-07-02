# OpenID Connect Repository

This is the [**OpenID Connect Working Group (AB/Connect WG)**](https://openid.net/wg/connect/) repository for the original [**OpenID Connect 1.0**](https://openid.net/connect) specifications.
It is used  tracking issues, discussing enhancements, and maintaining the specifications that
define OpenID Connect and some extensions to it.
The full list of OpenID Connect working group repositories is at https://openid.net/wg/connect/repositories/.

---

## 📄 Specifications & Documents

The specifications managed by this working group include, but are not limited to:

* **OpenID Connect Core** – Defines the core OpenID Connect functionality: authentication built on top of OAuth 2.0
  and the use of claims to communicate information about the End-User.
* **OpenID Connect Discovery** – Defines how clients dynamically discover information about OpenID Providers.
* **OpenID Connect Dynamic Client Registration** – Defines how clients dynamically register with OpenID Providers.
* **OpenID Connect RP-Initiated Logout** – Defines how a Relying Party requests that an OpenID Provider log out
  the End-User.
* **OpenID Connect Session Management** – Defines how to manage OpenID Connect sessions, including postMessage-based
  logout functionality.
* **OpenID Connect Front-Channel Logout** – Defines a front-channel logout mechanism that does not use an OP iframe
  on RP pages.
* **OpenID Connect Back-Channel Logout** – Defines a logout mechanism that uses direct back-channel communication
  between the OP and RPs being logged out.
* **OpenID Connect Core Error Code unmet_authentication_requirements** – Defines the unmet_authentication_requirements
  authentication response error code.
* **OpenID Connect Relying Party Metadata Choices 1.0** – this specification extends the OpenID Connect Dynamic Client
Registration 1.0 specification to enable RPs to express a set of supported values for some RP metadata parameters,
rather than just single values.
* **OpenID Federation 1.1** – Protocol-independent functionality enabling parties within a federation to establish trust
with one another.
* **OAuth 2.0 Multiple Response Types** – Defines several specific new OAuth 2.0 response types.
* **OAuth 2.0 Form Post Response Mode** – Defines how to return OAuth 2.0 Authorization Response parameters (including
OpenID Connect Authentication Response parameters) using HTML form values that are auto-submitted by the User-Agent
using HTTP POST.
* **Initiating User Registration via OpenID Connect** – Defines the prompt=create authentication request parameter.

> 🌐 **Official Specifications:** The final, approved versions of specifications are published at [https://openid.net/developers/specs/](https://openid.net/developers/specs/).

---

## 🛠 Contributing & Issue Tracking

Historically migrated from Bitbucket, this GitHub repository handles active issue triage and community enhancements for the suite of OpenID Connect specifications.

### Filing Issues
If you find a bug, inconsistency, or ambiguity in any of the active specification drafts or finalized specs:
1. Search the [Existing Issues](https://github.com/openid/connect/issues) to ensure it hasn't already been raised.
2. Open a new issue and tag it with the appropriate component label (e.g., `Core`, `Discovery`).
3. Provide explicit references to the section or line number of the spec you are referring to.

### Intellectual Property (IP) Notice
Contributions to this repository are subject to the **OpenID Foundation Intellectual Property Policy**. To contribute text, pull requests, or major revisions, you must be a member of the OpenID Foundation and have signed the **Contribution Agreement**.

* Learn more about joining the working group at https://openid.net/wg/connect/.

---

## 👥 Getting Involved

The Connect Working Group is open to new participants. You can engage with the community through the following channels:

* **Mailing List:** Join the discussion on the [OpenID Specs AB/Connect Mailing List](https://lists.openid.net/mailman/listinfo/openid-specs-ab).
* **Working Group Calls:** The WG hosts regular conference calls to review issues and pull requests. Meeting schedules and agendas are posted on the mailing list.
* **Website:** Visit the [OpenID Connect WG Home Page](https://openid.net/wg/connect/) for charter details and progress updates.
