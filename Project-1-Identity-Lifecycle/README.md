# Project 1: Identity Lifecycle Management Simulation
### Joiners, Movers, and Leavers — Microsoft Entra ID

---

## Overview

This project simulates a real-world Identity Lifecycle Management workflow using Microsoft Entra ID P2. Using a fictional company environment built on the NeurAlvo Tech tenant, I walked through the full JML process — onboarding a new hire, managing a role change, and executing a clean offboarding. Every step was documented with screenshots and Loom video walkthroughs.

**Fictional Company:** NeurAlvo Tech  
**Fictional Employee:** Nico Bellic  
**Tools Used:** Microsoft Entra ID P2 | Microsoft 365 Admin Center | Microsoft Entra Admin Center | Loom  

---

## Phase 1 — Joiner: Onboarding Nico Bellic

Nico Bellic was brought on as an IT Support Analyst. The onboarding process included creating his user account in Microsoft Entra ID, assigning him to the IT Support Team security group, and provisioning his Microsoft Entra ID P2 license.

Group assignment matters here because access in Entra ID is group-driven — putting Nico in the right group from day one means he gets exactly what he needs and nothing he doesn't. License provisioning unlocks the identity protection and governance features tied to his role.

**Steps completed:**
- Created user account in Microsoft Entra ID
- Assigned to IT Support Team security group
- Provisioned Microsoft Entra ID P2 license

> **Security principle:** Least privilege from the start. New users get access scoped to their role only.

---

## Phase 2 — Mover: Promoting Nico Bellic to IT Manager

Nico was promoted from IT Support Analyst to IT Manager. This triggered a full access review. His job title was updated in Entra ID, a new IT Management security group was created to reflect his elevated responsibilities, and he was removed from the IT Support Team group.

That last step is the one most organizations get wrong. Leaving old group memberships in place after a role change is how privilege creep happens — users accumulate access over time that no longer fits their role, and that's a real security risk. Cleaning it up during the transition is non-negotiable.

**Steps completed:**
- Updated job title to IT Manager in Entra ID
- Created IT Management security group
- Added Nico to the elevated group
- Removed Nico from IT Support Team group

> **Security principle:** Role changes require both adding new access and removing old access. One without the other creates risk.

---

## Phase 3 — Leaver: Offboarding Nico Bellic

When Nico's tenure ended, the offboarding followed a deliberate sequence. First, his account was disabled by unchecking Account Enabled in Entra ID — cutting off sign-in access immediately. Next, active sessions were revoked to invalidate any existing tokens. Then group memberships were removed, followed by license revocation.

The order matters. You disable first, revoke sessions second, clean up third. If you go straight to deleting group memberships without blocking sign-in, there's a window where the user could still access resources. That window is the risk.

**Steps completed:**
- Disabled account — blocked sign-in immediately
- Revoked active sessions — invalidated all existing tokens
- Removed group memberships
- Revoked Microsoft Entra ID P2 license

> **Security principle:** Offboarding is a sequence, not a checklist. Order of operations directly impacts your security posture.

---

## Documentation

Full Loom video series covering each phase and sub-step end to end.

| Video | Phase | Description |
|-------|-------|-------------|
| Phase 1 #1 | Joiner | Carl Johnson user created |
| Phase 1 #2 | Joiner | Niko Bellic user created |
| Phase 1 #3 | Joiner | Assign users to security groups in Entra ID |
| Phase 1 #4 | Joiner | Adding Carl to the Marketing Group |
| Phase 1 #5 | Joiner | Assigning a license to the users |
| Phase 2 #1 | Mover | Niko's promotion to IT Manager |
| Phase 2 #2 | Mover | Creating and adding Niko to elevated group |
| Phase 3 #1 | Leaver | Offboarding of Niko Bellic |

---

## Key Takeaways

- Identity lifecycle management is not just an IT task — it is a security function
- Least privilege must be enforced at onboarding and maintained through every role change
- Privilege creep is one of the most common and preventable identity security risks
- Offboarding sequence order is critical — disable first, revoke sessions second, clean up third

---

*Project completed as part of SC-300 exam preparation | Samuel Orji Jr | github.com/SamuelOrjiJr*
