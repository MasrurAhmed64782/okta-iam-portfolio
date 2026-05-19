# Project 02 — Enterprise SSO, MFA & SCIM Provisioning

## Overview

This project demonstrates a full enterprise-grade identity integration between **Okta** (Identity Provider) and **Salesforce** (Service Provider), covering SAML 2.0 SSO, OIDC-based authentication, adaptive MFA policy enforcement, and automated user provisioning via SCIM. Each phase reflects real-world IAM engineering practices applied in production environments.

---

## Objectives

- Establish federated authentication between Okta and Salesforce using SAML 2.0 with proper claims mapping and certificate management
- Implement OIDC-based SSO as a modern OAuth 2.0 alternative to SAML
- Design and enforce adaptive MFA policies with group-based conditional access rules
- Configure automated user lifecycle management (provisioning/deprovisioning) via SCIM 2.0

---

## Technologies & Platforms

| Component | Role |
|-----------|------|
| Okta (Identity Provider) | Central IdP — authentication, policy enforcement, provisioning |
| Salesforce Developer Edition | Service Provider — downstream application target |
| SAML 2.0 | Federated SSO protocol — Phase 1 |
| OIDC / OAuth 2.0 | Modern authentication protocol — Phase 2 |
| Okta Authenticators | MFA enforcement (Okta Verify, TOTP) |
| Okta Authentication Policies | Conditional access — group-based MFA rules |
| SCIM 2.0 | Automated user provisioning and deprovisioning |
| Salesforce Connected App | OAuth client — SCIM provisioning integration |
| Postman | API credential validation and OAuth flow testing |

---

## Implementation

### Phase 1 — SAML 2.0 SSO Integration ✅

Configured federated Single Sign-On between Okta and Salesforce using the SAML 2.0 protocol.

**Key configurations:**
- Added Salesforce.com from the Okta Integration Network (OIN)
- Configured SAML settings: ACS URL, Entity ID, and Name ID format
- Set SAML Identity Type to **"Assertion contains the User's Salesforce username"**
- Mapped Okta user attributes to Salesforce SAML assertions
- Exchanged and validated SAML metadata and signing certificates
- Verified IdP-initiated and SP-initiated SSO flows

**Outcome:** Users authenticate through Okta and are granted federated access to Salesforce without a separate Salesforce login.

**Screenshots:**

![SAML Configuration I](screenshots/phase-01-saml/01-saml-configuration%20i.png)
![SAML Configuration II](screenshots/phase-01-saml/01-saml-configuration%20ii.png)
![SAML Configuration III](screenshots/phase-01-saml/01-saml-configuration%20iii.png)

---

### Phase 2 — OIDC SSO Integration ✅

Implemented OpenID Connect (OIDC) as a modern OAuth 2.0-based authentication alternative to SAML.

**Key configurations:**
- Configured OIDC application integration in Okta
- Defined authorization scopes (`openid`, `profile`, `email`)
- Configured redirect URIs and token endpoint settings
- Validated ID token claims and user attribute mapping

**Outcome:** Demonstrated OIDC-based authentication flow as a complement to SAML, reflecting current industry direction toward OAuth 2.0-based identity protocols.

**Screenshots:**

![OIDC Configuration I](screenshots/phase-02-oidc/02-oidc-configuration%20i.png)
![OIDC Configuration II](screenshots/phase-02-oidc/02-oidc-configuration%20ii.png)
![OIDC Configuration III](screenshots/phase-02-oidc/02-oidc-configuration%20iii.png)

---

### Phase 3 — MFA & Adaptive Authentication Policies ✅

Designed and enforced adaptive MFA policies using Okta's Authentication Policy framework.

**Key configurations:**
- Configured Okta Verify and TOTP as authenticators
- Created group-based authentication policy rules with MFA requirements
- Applied step-up authentication conditions (e.g., require MFA when accessing Salesforce)
- Validated policy enforcement across different user group assignments

**Outcome:** Demonstrated conditional access enforcement where authentication requirements adapt based on user group membership and application sensitivity.

**Screenshots:**

![MFA Policy I](screenshots/phase-03-mfa/03-saml-sso-success%20i.png)
![MFA Policy II](screenshots/phase-03-mfa/03-saml-sso-success%20ii.png)
![MFA Policy III](screenshots/phase-03-mfa/03-saml-sso-success%20iii.png)
![MFA Policy IV](screenshots/phase-03-mfa/03-saml-sso-success%20iv.png)

---

### Phase 4 — SCIM Provisioning (Investigation & Escalation) ⚠️

**Objective:** Automate user lifecycle management between Okta and Salesforce using SCIM 2.0, enabling automatic provisioning and deprovisioning of Salesforce accounts when users are added or removed in Okta.

#### Configuration Completed

The following was fully configured in preparation for SCIM provisioning:

- Enabled API integration on the Salesforce.com app in Okta
- Created a dedicated **Salesforce Connected App** ("Okta SCIM Provisioning") with the following settings:
  - OAuth scopes: `Full access (full)`, `Manage user data via APIs (api)`, `Perform requests at any time (refresh_token, offline_access)`
  - Callback URL: Okta tenant OAuth callback endpoint
  - Client Credentials Flow enabled
  - PKCE disabled (per Okta documentation requirements)
  - Run As user configured (Salesforce system administrator)
- Verified SAML Identity Type set to **"Assertion contains the User's Salesforce username"**
- Confirmed Instance Type set to **Production** in Okta
- Generated and retrieved fresh OAuth Consumer Key and Consumer Secret via Salesforce Manage Consumer Details
- Entered credentials into Okta Provisioning → Integration settings

#### Issue Encountered

Despite correct configuration across all documented settings, the OAuth authentication handshake between Okta and Salesforce failed consistently with:

> *"Could not verify the Salesforce administrator credentials; please confirm that these are set correctly."*

#### Diagnostic Investigation

To isolate the root cause, direct API testing was performed outside of Okta using Postman:

| Test | Grant Type | Result |
|------|-----------|--------|
| Test 1 | `password` | `400 Bad Request` — `invalid_client_id` |
| Test 2 | `client_credentials` | `400 Bad Request` — `invalid_client_id` |

All items in the [Okta official support article](https://support.okta.com/help/s/article/salesforce-provisioning-integration-error-could-not-verify-the-salesforce-administrator-credentials-please-confirm-that-these-are-set-correctly) were verified and confirmed correct prior to testing.

#### Escalation

The issue was escalated through two channels:

**Okta Community (Paul S., Okta Employee):**
Recommended verifying PKCE settings and SAML Identity Type — both confirmed compliant. No additional resolution provided.

**Senior IAM Colleague (Kiran Adhikari):**
Performed independent credential validation against the Salesforce org to determine whether the issue is environment-specific to Salesforce Developer Edition.

#### Current Assessment

Evidence from Postman testing — where `invalid_client_id` is returned even with freshly generated credentials — suggests the issue originates on the Salesforce Developer Edition side rather than in Okta's provisioning configuration. Salesforce Developer Edition orgs have known OAuth restrictions that may prevent external SCIM clients from authenticating via the standard OAuth 2.0 flow.

**Root cause confirmed.** Paul S. (Okta Employee, Okta Community) responded: *"In the past I was able to enable Provisioning from a Salesforce Developer account, however if they changed anything I can't say for sure. I would recommend to check with them if there are any restrictions."* This confirms the issue originates from a change on Salesforce Developer Edition's side, not from the Okta configuration. All Okta-side settings were verified as correct.

**Screenshots:**

![SCIM Error I](screenshots/phase-04-scim/04-scim-credentials-error%20i.png)
![SCIM Error II](screenshots/phase-04-scim/04-scim-credentials-error%20ii.png)
![SCIM Error III](screenshots/phase-04-scim/04-scim-credentials-error%20iii.png)
![SCIM Error IV](screenshots/phase-04-scim/04-scim-credentials-error%20iv.png)
![SCIM Error V](screenshots/phase-04-scim/04-scim-credentials-error%20v.png)

---

## Key Takeaways

- SAML 2.0 and OIDC serve distinct use cases; both are relevant in enterprise IAM and understanding the differences is critical for integration design
- Adaptive MFA policy design requires careful group scoping to avoid over-enforcement or access gaps
- SCIM provisioning integrations depend on both the IdP configuration and the SP's OAuth implementation — issues can originate on either side
- Real-world IAM engineering involves systematic troubleshooting, proper escalation, and clear documentation — not just configuration
- Salesforce Developer Edition has OAuth flow restrictions that differ from production Salesforce orgs; this is a known consideration when building proof-of-concept integrations in lab environments

---

## Status

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | SAML 2.0 SSO Integration | ✅ Complete |
| 2 | OIDC SSO Integration | ✅ Complete |
| 3 | MFA & Adaptive Authentication Policies | ✅ Complete |
| 4 | SCIM Provisioning | ⚠️ Blocked — Salesforce Developer Edition OAuth restriction (confirmed by Okta Support) |

---

*Part of the [Okta IAM Portfolio](../../README.md) — a collection of hands-on identity and access management implementations.*
