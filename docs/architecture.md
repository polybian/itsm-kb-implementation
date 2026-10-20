# Architecture & Design Decisions

## Stack

- **osTicket** (community Docker image, PHP + web server) as the ITSM app:
  ticket intake, queues, workflows, SLA plans, and staff/agent panels.
- **MariaDB 10.11** as the backing store. MariaDB has no published port —
  only the osTicket container can reach it.
- **Named volumes** (`osticket-data`, `db-data`) for persistence; `./exports`
  (git-ignored) as the drop-in point for SQL exports and restore drills.

## Why osTicket as the stand-in for Zendesk / Jira Service Management

osTicket's concepts map nearly 1:1 onto the SaaS platforms, so skills
transfer directly:

| SaaS concept (Zendesk / JSM) | osTicket equivalent |
|---|---|
| Queues / request types | Departments |
| Ticket forms / issue types | Help topics (with custom form fields) |
| Groups | Teams |
| Macros | Canned responses |
| SLA policies / JSM SLA goals | SLA plans (per priority, per department) |
| Triggers / automation rules | Filters + auto-assign + banlists (simpler, but the same idea) |
| Guide / Confluence KB | Built-in Knowledge Base (internal + public articles) |

The mechanics differ (less automation than Zendesk triggers), but the
**design discipline** — queues, categories, priorities, escalation, SLA math —
is identical, and that's the part interviews test.

## Why osTicket over Zammad / GLPI

- **Zammad** is the more modern app, but its data model (groups, overviews,
  triggers) maps cleanly onto *Zendesk only*. osTicket's
  departments/help-topics model sits between Zendesk and Jira, which matches
  this lab's dual purpose: Zendesk-style support queues *and* the Jira→Zendesk
  migration mapping doc.
- **GLPI** leans ITSM-asset-heavy (CMDB, inventory) — valuable, but a
  different lab. This lab is about the ticket + KB workflow.
- osTicket is a single PHP/MySQL app with well-worn community Docker images —
  it starts in one `docker compose up` and the whole state lives in two
  volumes.

## Data model (what lives in MariaDB)

- **End users** (client portal accounts) and **staff** (agents, with
  department/team memberships and permission levels).
- **Tickets**: subject, help topic, priority, SLA plan, status, department;
  **threads** carry every message/note/attachment in chronological order.
- **Departments** = queues (e.g. "Access & Accounts"); **help topics** =
  ticket forms (e.g. "MFA reset"), each with its own default priority and
  SLA plan.
- **SLA plans** bind priority → response/resolution targets and can auto-
  escalate overdue tickets to a different department or team.
- **KB articles** (categories + FAQs) stored in the same database, importable
  from the Markdown source of truth in `kb/`.

## KB-as-docs approach

The knowledge base lives in `kb/` as Markdown and is **versioned in git**,
not authored inside the app:

- Articles are diff-able, reviewable in pull requests, and portable — the
  same files feed a SaaS Guide, Confluence, or a static site.
- The osTicket KB is the *published* copy; `kb/` is the *source of truth*.
  Updates flow one way: edit Markdown → paste into the Admin panel (or a
  future import script).
- Each article has a fixed SOP structure (Title, Applies to, Symptoms,
  Cause, Resolution steps, Verification, Escalation) so agents — including
  new ones — can execute it without tribal knowledge.
- Stale-article prevention is a process decision, not a tool feature: every
  article names an owner and a review date; see the taxonomy doc.

## Hardening notes

- **Installer hygiene:** complete the web installer, then delete the
  `setup/` directory; change the default admin password immediately and
  create named admin/agent accounts instead of sharing one.
- **Network:** the lab binds `127.0.0.1:8080` only. For anything beyond a
  laptop, terminate TLS at a reverse proxy and restrict `/scp` to the
  office/VPN IP ranges.
- **Secrets:** database and admin credentials live in `.env` (git-ignored)
  or a secret manager — never in the compose file, docs, or chat logs.
- **Backups:** dump MariaDB on a schedule (`mysqldump` into `./backups`,
  off-box copy). The `db-data` volume is not a backup strategy by itself.
- **Updates:** pin the osTicket image tag; test upgrades on a copy of the
  volumes before touching the live stack. Keep MariaDB on a supported
  minor line.
