# KB-003: New-hire account provisioning checklist

| Field | Value |
|---|---|
| **Article ID** | KB-003 |
| **Owner** | Access & Accounts (L1) |
| **Review date** | 6 months from publish |
| **Last reviewed** | *(fill in on import)* |

## Applies to

- Every new employee and long-term contractor, all departments
- Executed by the L1 service desk from the HR onboarding ticket

## Prerequisites (must be in the HR ticket before you start)

- [ ] Legal name, preferred name, personal email, phone number
- [ ] Start date, department, job title, direct manager
- [ ] Work location (office / remote) and required hardware per the
  hardware matrix
- [ ] Any special access noted by the manager (shared mailboxes, apps,
  elevated rights): if it's not in the ticket, provision the standard
  set and let the manager request extras

**Naming convention:** `firstname.lastname` for UPN and primary email
(e.g. `ana.gomez`). On collision, append the middle initial, then a
number: never reuse a departed employee's address.

## Provisioning steps (in order)

### 1. Identity and licensing
- [ ] Create the Entra ID user (cloud-only, or synced if hybrid: follow
  the site's standard)
- [ ] Assign the M365 license per role (standard vs. frontline: check the
  license matrix, don't guess)
- [ ] Set usage location (required before license assignment)
- [ ] Force password change at first sign-in; generate a Temporary Access
  Pass for day-one login

### 2. Groups and access
- [ ] Add to the department security group (drives file-share and app
  access)
- [ ] Add to the "All Staff" distribution list and any
  department/office DLs
- [ ] Add to role-based app groups per the manager's request (VPN users,
  Zendesk agents, etc.)
- [ ] Shared mailboxes: grant access, do **not** create separate
  credentials

### 3. Device
- [ ] Assign hardware per the hardware matrix; record asset tag and serial
  in the asset system, linked to the user
- [ ] Enroll in Intune / Autopilot (or image per the build standard);
  confirm compliance policy applies before handoff
- [ ] For remote hires: confirm shipment tracking and delivery date with
  the user

### 4. Service access
- [ ] VPN: create the WireGuard peer / distribute the tunnel profile
  (see KB-001 for the client setup the user will follow)
- [ ] ITSM: create the agent or end-user account as appropriate
- [ ] MFA: user registers at https://aka.ms/mfasetup on day one,
  **do not pre-register MFA for them** (see KB-002 for the reset flow
  if they get stuck)

### 5. Handoff
- [ ] Send the manager the credentials package through the approved
  channel (TAP + login instructions: never plain-text passwords over
  email)
- [ ] Send the new hire the day-one checklist: Wi-Fi/VPN setup, MFA
  enrollment, where to file tickets, who to contact for help
- [ ] Link the HR onboarding ticket and close the provisioning ticket
  only after day-one verification

## Verification (day one, with the user or manager)

- [ ] User signs in to M365 with TAP and completes MFA enrollment
- [ ] Mailbox receives mail; Teams and OneDrive open
- [ ] VPN connects (handshake check per KB-001)
- [ ] Device shows compliant in Intune
- [ ] User can reach the ticket portal and knows how to file a request

## Escalation

- **Missing prerequisites** (no start date, no manager sign-off) →
  send back to HR; do not provision on verbal requests.
- **License shortage or special access approvals** → L2 / IT lead.
- **Day-one failures** (can't sign in, no MFA, no VPN) → treat as P2
  until the user is working; link the provisioning ticket so the trail
  is complete.
