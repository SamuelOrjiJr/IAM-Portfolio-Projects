# Lab C: App Registrations, Consent, and App Roles

**Microsoft Entra ID | App Registrations | Enterprise Applications | Workload Identities**
Built October 28 to November 2, 2026 by Samuel Orji Jr as hands-on preparation for the SC-300 (Microsoft Identity and Access Administrator) exam.

---

## Overview

I registered an application in Microsoft Entra ID and took it through its full identity lifecycle: the app object and its service principal, delegated and application permissions, admin consent, a client secret, a custom app role, user assignment, and "Assignment required." I proved each piece with evidence: the audit log, decoded tokens, and real sign-in attempts that succeeded and failed.

**Scenario:** Marketing wants an internal app, **LabC-Marketing-Reports**, that does two things:
- a dashboard where signed-in users see their own profile (**delegated** permission)
- a nightly job that reads the user directory with nobody signed in (**application** permission)

Only approved users should be able to sign in, and viewers should get a specific in-app role.

### Environment

| Item | Detail |
|---|---|
| Tenant | Lab tenant with Microsoft Entra ID P2 |
| App | LabC-Marketing-Reports, single tenant |
| Assigned user | Franklin Clinton, Marketing Analyst (reused from Projects 3 and 4) |
| Unassigned test users | Monica Thompson, Carl Johnson |
| Token viewer | jwt.ms (Microsoft's token decoder) |

### Skills demonstrated

- App registration vs enterprise application (application object vs service principal)
- Delegated vs application permissions on Microsoft Graph
- Admin consent, and where consent is recorded
- Client secrets and credential hygiene
- App roles, user assignment, and the roles claim
- "Assignment required" as an access control
- Audit log and sign-in log investigation using Correlation IDs
- Reading ID tokens (aud, roles, oid, tid, nonce)

---

## Design

| Setting | Value | Why |
|---|---|---|
| Supported account types | This organization only (single tenant) | Internal app, internal users only |
| Delegated permission | Microsoft Graph User.Read | Signed-in users read their own profile |
| Application permission | Microsoft Graph User.Read.All | Nightly job reads the directory with no user; least privilege instead of Directory.Read.All |
| Consent | Admin consent, tenant-wide | Application permissions always require admin consent |
| Credential | Client secret, 90 days | Shortest standard lifetime; deleted at cleanup |
| App role | Marketing Report Viewer (value `Reports.View`), Users/Groups | RBAC inside the app |
| Assignment required | Yes | Only assigned users can get a token |

> **Production note:** For anything running in Azure, I would use a **managed identity** instead of a client secret, so there is no credential to store or rotate. Outside Azure, I would use a **certificate** over a secret, track expiration dates, and assign the app role to a **group** rather than to individual users.

---

## Step-by-step build

### 1. Registered the app
App registrations > New registration. Name LabC-Marketing-Reports, single tenant, no redirect URI yet.

![App registered](screenshots/LabC-step1-app-registered.png)

### 2. Found the service principal
The Overview link "Managed application in local directory" opens the enterprise application. Same **Application ID** (`5658de84-...`), different **Object ID** (`5cad20b5-...` for the app registration vs `9231db7f-...` for the service principal). One app, two objects: the blueprint and the house built from it in this tenant.

![Enterprise app service principal](screenshots/LabC-step2-enterprise-app-service-principal.png)

### 3. Added permissions (not yet granted)
User.Read (Delegated) showed Admin consent required = No. User.Read.All (Application) showed Admin consent required = Yes and "Not granted." Adding a permission only means the app is asking for it.

![Permissions not granted](screenshots/LabC-step3-api-permissions-not-granted.png)

### 4. Granted admin consent
Both permissions moved to "Granted." The grant is recorded on the **enterprise application**, not the app registration.

![Admin consent granted](screenshots/LabC-step4a-admin-consent-granted.png)
![Enterprise app permissions](screenshots/LabC-step4b-enterprise-app-permissions.png)

### 5. Created a client secret
Description LabC-nightly-job, 90 day expiry. The value is shown only once and is blurred in the screenshot.

![Client secret created](screenshots/LabC-step5-client-secret-created.png)

### 6. Created an app role
Marketing Report Viewer, value `Reports.View`, allowed member type Users/Groups. Defined on the app registration.

![App role created](screenshots/LabC-step6-app-role-created.png)

### 7. Assigned Franklin to the role
Enterprise application > Users and groups > Add user/group. With only one role defined, the portal selected it automatically.

![Franklin assigned](screenshots/LabC-step7-franklin-assigned-role.png)

### 7b. Investigated an unexpected assignment
My admin account also appeared on the list with **Default Access**, which I never assigned. I traced it in the enterprise app's audit log (see findings below).

![Audit log assignments](screenshots/LabC-step7b-audit-log-assignments.png)
![Audit log detail](screenshots/LabC-step7b-audit-log-detail.png)
![Audit log targets](screenshots/LabC-step7b-audit-log-detail-2.png)
![Audit log modified properties](screenshots/LabC-step7b-audit-log-detail-3.png)

### 8. Added a redirect URI for token inspection
Web redirect URI `https://jwt.ms`, with ID tokens enabled so the token could be decoded.

![Redirect URI](screenshots/LabC-step8-redirect-uri-jwtms.png)
![ID tokens enabled](screenshots/LabC-step8-redirect-uri-jwtms-2.png)

### 9. Signed in as Franklin and read his token
Franklin's ID token contained `"roles": ["Reports.View"]`. My admin token, with only Default Access, contained no roles claim.

![Franklin token](screenshots/LabC-step9-franklin-token-decoded.png)
![Admin token, no roles](screenshots/LabC-step9b-admin-token-no-roles.png)

### 10. Turned on "Assignment required" and tested it
Set Assignment required to Yes, removed my own Default Access assignment, and tested sign-ins with unassigned users.

![Assignment required](screenshots/LabC-step10a-assignment-required-yes.png)
![Carl blocked by risk](screenshots/LabC-step10b-carl-blocked-53004.png)
![Removed Default Access](screenshots/LabC-step10c-removed-default-access.png)
![Monica blocked, AADSTS50105](screenshots/LabC-step10d-unassigned-user-blocked.png)
![Owners check, app registration](screenshots/LabC-step10e-owners-check.png)
![Owners check, enterprise app](screenshots/LabC-step10e-owners-check-2.png)
![Admin sign-in log](screenshots/LabC-step10f-admin-signin-log.png)
![Admin sign-in Conditional Access](screenshots/LabC-step10f-admin-signin-log-2.png)

### 11. Cleanup
Deleted the unused client secret. Kept the app, its role assignment, and the redirect URI for later practice.

![Secret deleted](screenshots/LabC-step11a-secret-deleted.png)

---

## What I found along the way

| Issue | What I found | How I handled it |
|---|---|---|
| Unexpected Default Access | My admin account was assigned to the app with Default Access. | The audit log showed the assignment at 3:59:01 PM, the same second as "Consent to application," with the **same Correlation ID**. Granting admin consent in the portal assigned me. Modified Properties showed AppRole.Id as all zeros, which is how Entra stores Default Access. |
| Permissions logged differently | User.Read.All logged as "Add app role assignment to service principal"; User.Read logged as "Add delegated permission grant." | Application permissions on Graph are app roles that Graph defines for applications. Delegated permissions are stored as a separate grant. |
| Test users blocked by risk | Carl and Monica got error **53004**: "Cannot configure MFA methods due to suspicious activity." | My VPN gave them an unfamiliar IP, so the sign-in was rated risky. The sign-in risk policy required MFA, and they had none registered, which Entra will not allow during a risky sign-in. With the VPN off, Monica reached the assignment check and got the expected error. The Request ID and Correlation ID on the error page matched the sign-in log. |
| Session reuse | The sign-in URL kept signing me in as the previous user. | Added `&prompt=login` to force a fresh sign-in. This is single sign-on working as designed. |
| Admin not blocked (open finding) | After removing my assignment, my Global Administrator account still received a token, while unassigned standard user Monica was blocked with AADSTS50105. | I ruled out saved settings, propagation delay (fresh token days later), assignment, ownership on both objects, and Conditional Access (no policy applied, and CA can only restrict access, not grant it). The cause is still open. Lesson: verify a control against non-admin accounts, not only admins. |
| Password change mid-lab | Changing my admin password invalidated existing browser sessions (AADSTS50133) and left one browser stuck on a portal error. | Signed in from a clean browser. This reinforced why every tenant needs break-glass accounts. |

---

## Results and proof

### Token claims (Franklin, assigned with Marketing Report Viewer)

| Claim | Value | Meaning |
|---|---|---|
| roles | `Reports.View` | The app role's Value, from Steps 6 and 7 |
| aud | `5658de84-...` | Token issued for this app (its client ID) |
| oid | Franklin's object ID | Which user signed in |
| tid | Tenant ID | Which tenant issued it |
| nonce | `labc123` | Matches the request, protecting against replay |

My admin token, with only Default Access, had **no roles claim**.

### Assignment required test

| User | Assigned? | Result |
|---|---|---|
| Franklin Clinton | Yes, Marketing Report Viewer | Token issued with `Reports.View` |
| Monica Thompson | No | **AADSTS50105**, blocked; sign-in log shows Failure |
| Carl Johnson (VPN on) | No | 53004, blocked earlier by sign-in risk |
| Lab admin (Global Administrator) | No (removed) | Token issued; open finding |

---

## Key concepts reinforced

**App registration vs enterprise application**
- App registration = the application object, the global definition: client ID, requested permissions, credentials, app roles. Lives in the home tenant.
- Enterprise application = the service principal, the local instance in each tenant: assignments, granted consent, sign-in settings, logs.
- A vendor's app appears only under Enterprise applications in your tenant.

**Permissions and consent**
- Delegated = app acts for a signed-in user. Application = app acts as itself, no user, and always needs admin consent.
- An app can never consent for itself.
- Admin consent for Graph application permissions requires Privileged Role Administrator or Global Administrator.
- Consent is recorded on the service principal; revoke it there.

**Credentials**
- Secret value is shown once; the Secret ID is not the secret.
- Expired secrets are a top cause of integration outages.
- Prefer certificates, and managed identities for Azure resources.

**App roles and assignment**
- Define roles on the app registration; assign them on the enterprise application.
- The role's Value appears in the token's roles claim.
- No roles defined = Default Access. Group assignment requires P1 or P2, and nested groups are not supported.
- Assignment required = Yes blocks unassigned users with AADSTS50105. "Visible to users" only controls the My Apps tile.

**Troubleshooting**
- Audit logs answer "who changed what"; sign-in logs answer "who signed in and did it work."
- Correlation IDs tie related events together across logs and error pages.
- Common errors: AADSTS50105 (not assigned), AADSTS50011 (redirect URI mismatch), 53004 (cannot register MFA during a risky sign-in), AADSTS50133 (session invalid after password change).
- `prompt=login` forces re-authentication instead of reusing a session.

---

## Screenshot index

The client secret value and public IP addresses are blurred.

| # | File | What it shows |
|---|---|---|
| 1 | LabC-step1-app-registered | App registration Overview with IDs |
| 2 | LabC-step2-enterprise-app-service-principal | Same Application ID, different Object ID |
| 3 | LabC-step3-api-permissions-not-granted | User.Read and User.Read.All, not granted |
| 4 | LabC-step4a-admin-consent-granted | Both permissions granted |
| 5 | LabC-step4b-enterprise-app-permissions | Grants recorded on the enterprise app |
| 6 | LabC-step5-client-secret-created | Secret created, value blurred |
| 7 | LabC-step6-app-role-created | Marketing Report Viewer, Reports.View |
| 8 | LabC-step7-franklin-assigned-role | Franklin with Marketing Report Viewer |
| 9 | LabC-step7b-audit-log-assignments | Audit log timeline |
| 10 | LabC-step7b-audit-log-detail | Assignment activity and Correlation ID |
| 11 | LabC-step7b-audit-log-detail-2 | Targets: service principal and admin user |
| 12 | LabC-step7b-audit-log-detail-3 | Modified properties: all-zeros Default Access role |
| 13 | LabC-step8-redirect-uri-jwtms | Web redirect URI https://jwt.ms |
| 14 | LabC-step8-redirect-uri-jwtms-2 | ID tokens enabled |
| 15 | LabC-step9-franklin-token-decoded | roles: Reports.View |
| 16 | LabC-step9b-admin-token-no-roles | Admin token without roles claim |
| 17 | LabC-step10a-assignment-required-yes | Assignment required set to Yes |
| 18 | LabC-step10b-carl-blocked-53004 | Risk-based block, error 53004 |
| 19 | LabC-step10c-removed-default-access | Admin assignment removed |
| 20 | LabC-step10d-unassigned-user-blocked | Monica blocked with AADSTS50105 |
| 21 | LabC-step10e-owners-check | No owners on the app registration |
| 22 | LabC-step10e-owners-check-2 | No owners on the enterprise app |
| 23 | LabC-step10f-admin-signin-log | Admin sign-in, Success, single factor |
| 24 | LabC-step10f-admin-signin-log-2 | No Conditional Access policy applied |
| 25 | LabC-step11a-secret-deleted | Client secrets (0) |

---

## About this lab series

This lab is part of a series of hands-on Microsoft Entra ID labs covering Conditional Access, Privileged Identity Management, app registrations, Identity Protection, Defender for Cloud Apps, and hybrid identity. Each lab is built in a live tenant and documented with evidence.

**Samuel Orji Jr** | IAM and Cybersecurity | [github.com/SamuelOrjiJr](https://github.com/SamuelOrjiJr)
