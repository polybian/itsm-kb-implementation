# KB-002 — MFA reset and re-registration (lost or replaced phone)

| Field | Value |
|---|---|
| **Article ID** | KB-002 |
| **Owner** | Access & Accounts (L1, with Security oversight) |
| **Review date** | 90 days from publish |
| **Last reviewed** | *(fill in on import)* |

## Applies to

- Microsoft 365 / Entra ID accounts using Microsoft Authenticator
  (or other TOTP authenticator apps) for MFA
- Situations: lost or stolen phone, broken phone, new phone without
  migrating the authenticator, authenticator app deleted

## Symptoms

- User cannot complete MFA sign-in: no approval prompt arrives, or the
  authenticator app shows no account for the company
- User is locked out of M365, VPN, or other SSO apps that require MFA

## ⚠️ Security verification FIRST — before touching anything

An MFA reset hands account access to whoever is asking. **Verify identity
before resetting, no exceptions.** Do not reset MFA based on an email
request alone.

1. **Verify the requester:** call the user back at the phone number on file
   in HR/the directory (not a number they provide in chat), or verify via
   video with badge, or get written confirmation from their direct manager
   via a known channel. Document *how* you verified in the ticket.
2. **Check for red flags:** sign-in logs showing impossible travel, MFA
   prompts the user didn't initiate, or password changes they don't
   recognize → stop and escalate to Security as a suspected compromise
   (P2 minimum) instead of doing a routine reset.
3. **Lost/stolen device:** if the phone is lost or stolen (not just
   replaced), revoke all active sessions first (Entra admin center → user →
   Revoke sessions) so the old device can't be used.

## Resolution steps

1. Complete the identity verification above and record the method in the
   ticket.
2. In the Entra admin center, open the user → **Authentication methods** →
   select **Require re-register MFA**. This clears their existing methods
   and forces setup at next sign-in. (Do not delete methods one by one —
   re-register is the clean reset.)
3. If the user needs access *before* they can re-register (e.g. they're at
   a new device with no working MFA at all), issue a **Temporary Access
   Pass** (one-time, short lifetime — 1 hour) and give it to the user
   through the verified channel. Never email a TAP to an unverified
   address.
4. Direct the user to **https://aka.ms/mfasetup** and walk them through:
   - Sign in (with the TAP if issued)
   - Add method → Authenticator app → follow the QR-code pairing
   - Complete the test approval/push to confirm it works
   - Set the authenticator as the default method
5. Have the user sign out and sign back in once to confirm the full loop
   works end to end.

## Verification

- [ ] Identity verification method documented in the ticket
- [ ] User completed a fresh MFA sign-in without the TAP
- [ ] Old device sessions revoked (if lost/stolen)
- [ ] Sign-in logs show no unfamiliar locations or failed MFA bursts
- [ ] User confirms access to mail, Teams, and VPN

## Escalation

Escalate to **Security (L2/L3)** if: any red flags in sign-in logs,
the user reports prompts they didn't trigger, the account shows password
changes they didn't make, or the requester can't be identity-verified.
Include: verification attempts made, relevant sign-in log entries
(timestamps, IPs, locations), and whether sessions were revoked. When in
doubt, treat as compromise first and reset second.
