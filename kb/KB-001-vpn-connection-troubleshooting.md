# KB-001 — VPN client won't connect (WireGuard)

| Field | Value |
|---|---|
| **Article ID** | KB-001 |
| **Owner** | Network support (L2) |
| **Review date** | 90 days from publish |
| **Last reviewed** | *(fill in on import)* |

## Applies to

- Company-managed and BYOD devices running the WireGuard client app
  (Windows 10/11, macOS 13+)
- The company remote-access tunnel profile (peer `vpn-gateway`, network
  `10.8.0.0/24`)
- Users who previously connected successfully, or first-time setups

## Symptoms

- WireGuard tunnel shows "Active" but no internal resources load, **or**
- Tunnel fails to activate / immediately deactivates, **or**
- User reports "VPN is connected but nothing works"

## Quick triage (2 minutes)

1. Ask the user to open the WireGuard app and read the **"Latest handshake"**
   timestamp for the tunnel.
   - **Handshake within the last 2 minutes** → the tunnel is up; the problem
     is routing or DNS (skip to step 4 of Resolution).
   - **Handshake old/absent, or "N/A"** → the tunnel never established;
     the problem is connectivity, keys, or the endpoint (start at step 1).
   - This single check splits the problem in thirds: handshake vs. routing
     vs. DNS. Do it first, every time.

## Resolution steps

### 1. If there is no recent handshake

1. **Confirm the basics:** device has working internet (browse any public
   site), and the correct tunnel profile is selected and toggled on.
   On mobile hotspots or captive portals (hotel/airport Wi-Fi), have the
   user complete the captive-portal login in a browser first.
2. **Check the endpoint:** in the tunnel config, the `Endpoint` must be the
   current gateway address and port `51820/udp`. If the gateway address
   changed recently, push the updated profile — don't have users hand-edit
   it.
3. **Check the keys:** the user's **public** key on file must match the
   peer entry on the server (compare in the admin panel; never ask for or
   handle the user's *private* key). If the user reinstalled the app or got
   a new device, their keypair changed — generate a new peer entry and send
   a fresh profile. Mismatched keys are the #1 cause of "suddenly stopped
   working after reinstall."
4. **Check local firewall/security software:** third-party firewalls and
   some EDR products block outbound UDP. Have the user temporarily allow
   the WireGuard app, or check its logs for blocked `51820/udp` traffic.

### 2. If the handshake is fresh but nothing works

5. **Check routing:** the profile should be full-tunnel
   (`AllowedIPs = 0.0.0.0/0`). If it shows only `10.8.0.0/24` (split
   tunnel) and the user needs general internet through the VPN, they have
   the wrong profile — send the correct one.
6. **Check DNS:** with the tunnel up, internal names (e.g.
   `fileserver.lab.local`) must resolve via the tunnel DNS (`10.8.0.1`).
   On Windows, run `nslookup fileserver.lab.local 10.8.0.1`; on macOS,
   check that the tunnel's DNS server is listed first in
   `scutil --dns`. If public sites work but internal names don't, it's a
   DNS ordering issue — toggle the tunnel off/on once to re-apply DNS.

### 3. If it worked before and broke today

7. **Ask what changed:** OS update, new security software, new router, or
   travel to a new network. OS updates frequently reset firewall rules and
   DNS settings — re-verify steps 4 and 6.
8. **Server side (L2):** check the gateway is reachable and the user's peer
   entry still exists (`wg show` on the server should list their public
   key with a recent handshake). If the peer is missing, it was likely
   pruned — re-add it.

## Verification

- [ ] WireGuard app shows a handshake within the last 2 minutes
- [ ] `ping 10.8.0.1` succeeds (gateway reachable through the tunnel)
- [ ] Internal hostname resolves (e.g. `nslookup fileserver.lab.local`)
- [ ] User can open one internal resource (intranet page or file share)

## Escalation

Escalate to **L2 — Network** if: no handshake after key re-issue, suspected
gateway outage (multiple users affected → treat as P1/P2 per the priority
matrix), or ISP-level UDP blocking. Include in the escalation: handshake
status, client app version, OS version, network type (office/home/hotel),
and whether other users on the same network are affected. **Never include
private keys** — only public keys and timestamps.
