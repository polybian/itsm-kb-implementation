# Ticket Taxonomy, Workflows & SLA Design

Design template for the lab's osTicket instance — and the same structure
used to plan the Jira Service Management → Zendesk migration. Implement
these in the staff panel under Admin panel → Manage (Help Topics,
Departments, SLA Plans) and Teams.

## 1. Help topics (ticket categories)

Short list, user-facing. Each help topic maps to a department queue, a
default priority, and an SLA plan.

| Help topic | Department (queue) | Default priority | Examples |
|---|---|---|---|
| Access & Accounts | Access & Accounts | P3 | Password reset, MFA reset, new-hire provisioning, terminations, group membership |
| Hardware | End-User Devices | P3 | Laptop won't boot, broken screen, peripherals, loaner requests |
| Software | End-User Devices | P4 | App install, license request, app crashes, update failures |
| Network / VPN | Network | P3 | Wi-Fi issues, WireGuard client won't connect, slow office network |
| M365 & Collaboration | Access & Accounts | P3 | Mailbox issues, Teams/SharePoint access, OneDrive sync, distribution lists |
| Security Incident | Security | P2 | Phishing report, suspected compromise, lost/stolen device |
| Procurement | Procurement | P4 | New hardware request, software purchase, renewals |
| General / Other | Service Desk | P4 | Anything not covered — reviewed monthly; a fat "Other" bucket means the taxonomy is wrong |

**Design rule:** 6–10 topics. If agents can't pick the right one in under
5 seconds, you have too many.

## 2. Priority matrix — impact × urgency

|  | High urgency | Normal urgency | Low urgency |
|---|---|---|---|
| **High impact** (many users / business-critical) | **P1** Critical | **P2** High | **P2** High |
| **Medium impact** (one team / degraded) | **P2** High | **P3** Normal | **P3** Normal |
| **Low impact** (one user / workaround exists) | **P3** Normal | **P3** Normal | **P4** Low |

Security incidents override the matrix: suspected compromise is always ≥ P2.

## 3. Status workflow

```
New → Assigned → In Progress → Pending User → Resolved → Closed
                  │               │
                  └── Escalated ──┘   (Escalated is a flag/state, not a dead end:
                                      ticket moves between departments but keeps its thread)
```

- **New:** un-triaged. Target: triaged within the P-level response time.
- **Assigned:** an agent owns it.
- **In Progress:** actively being worked.
- **Pending User:** waiting on the requester; the clock pauses, but the agent
  follows up on a schedule (24h, then 72h) before auto-resolving.
- **Resolved:** fix delivered, awaiting requester confirmation (auto-closes
  after 5 business days of silence).
- **Closed:** confirmed or auto-closed. Reopening creates a linked follow-up,
  not a zombie ticket.

## 4. SLA targets (design template — tune to the business)

These are *design starting points*, not measured results. Set them in osTicket
as SLA plans per priority.

| Priority | First response | Resolution target | Escalation trigger |
|---|---|---|---|
| P1 Critical | 15 min | 4 h | Notify lead at 30 min; L2 engaged at 1 h |
| P2 High | 1 h | 8 h | L2 review if untouched at 2 h |
| P3 Normal | 4 h | 2 business days | L2 consult if stuck at 1 day |
| P4 Low | 8 h | 5 business days | Review at 3 days |

Business-hours vs. 24/7 coverage is declared in the SLA plan itself — P1
outside hours follows the on-call rotation, not the queue.

## 5. Escalation rules — L1 → L2 → L3

| Level | Who | Owns | Timebox before escalating |
|---|---|---|---|
| **L1** Service desk | Generalist agents | Triage, KB-known issues, resets, provisioning, < ~15 min fixes | P1: 30 min · P2: 2 h · P3: 1 day |
| **L2** Systems / Network | Specialists | Root cause, config changes, server/network/M365 admin | P1: 2 h · P2: 1 day · P3: 2 days |
| **L3** Engineering / Vendor | Engineers, vendor TAC | Code bugs, firmware, vendor cases | Engaged by L2 with a complete case file |

**Warm-handoff rule:** an escalation must include a summary, steps already
tried, relevant logs/screenshots, and what the requester was last told.
Escalations without that context get bounced back — politely, once.
**KB rule:** whoever solves it writes or updates the article, so next time
it's an L1 resolution.

## 6. Jira Service Management → Zendesk migration mapping

Planning template: every row in the JSM configuration needs a Zendesk home
before cutover. osTicket equivalents in parentheses for this lab.

| Jira (source) | Zendesk (target) | Notes / decisions |
|---|---|---|
| Issue types: Incident, Service Request, Problem, Change | Ticket types + Help topics (`osTicket: help topics`) | Jira's Problem/Change become dedicated help topics with their own workflow |
| Workflow statuses: To Do, In Progress, In Review, Waiting, Done | New, Open, Pending, On hold, Solved (`osTicket: New → … → Resolved → Closed`) | "In Review" + "Waiting" collapse into Pending/On hold; nuance preserved in internal notes |
| Priorities: Highest, High, Medium, Low | P1–P4 via impact × urgency matrix above | Jira's 5 levels remapped; document the old→new mapping for reporting continuity |
| Queues (JSM project queues) | Departments (`osTicket: departments`) | 1:1 where queues were healthy; merged where queues had < 5 tickets/month |
| Custom fields (e.g. Asset tag, Office location) | Ticket fields / tags (`osTicket: custom form fields per help topic`) | Fields with no equivalent become tags + a migration note on the ticket |
| Automation rules | Triggers + SLA plans (`osTicket: filters, auto-assign, SLA plans`) | Rebuild, don't port: Zendesk/osTicket automation is simpler — keep only rules that fired in the last 90 days |
| Confluence KB | Guide articles (`osTicket: Knowledge Base`) | Rewritten into the SOP template (owner + review date), not copy-pasted |
| Customer portal | Help center (`osTicket: client portal`) | Request forms rebuilt per help topic; test the full submit→triage path in pilot |
| Users / agents | End users / Agents + Teams (`osTicket: users, staff, teams`) | Export, de-duplicate, map Jira groups → Zendesk groups/teams |

**Migration order:** ① export & map (this table) → ② build target config →
③ pilot one category for 2 weeks on live tickets → ④ fix gaps →
⑤ full cutover over a weekend → ⑥ retrain (short hands-on sessions +
one-page "where your Jira thing lives now" cheat sheet per team) →
⑦ 30-day hypercare with the mapping table open for corrections.
