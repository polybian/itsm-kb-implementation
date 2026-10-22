# ITSM + Knowledge Base Implementation — osTicket on Docker

I led the move from Jira to Zendesk at Kime Capital. We did a hard cutover and lost
the links to old tickets; next time I'd run both systems side by side for a while
first. This repo rebuilds the same workflows and knowledge base structure on
open-source osTicket, with all company data removed.

It's a working ticket queue with help topics, SLA plans, and teams, backed by
MariaDB, plus a knowledge base kept as versioned Markdown.

**What this demonstrates:** ITSM platform administration (queues, ticket
forms, workflows, SLA design), support-ticket taxonomy design, L1/L2/L3
escalation modeling, knowledge-base authoring and lifecycle management, and
Docker Compose deployment of a real PHP/MySQL stack.

```
 ┌──────────────┐   web (8080)    ┌───────────────────────────────┐
 │ end users    │ ───────────────▶│            osTicket           │
 │ (client portal)                │  departments (queues)         │
 └──────────────┘                 │  help topics (ticket forms)   │
                                  │  SLA plans  ·  teams          │
 ┌──────────────┐   /scp (8080)   │  canned responses (macros)    │
 │ agents       │ ───────────────▶└──────────────┬────────────────┘
 │ (staff panel)                                 │ SQL
 └──────────────┘                                ▼
                                  ┌───────────────────────────────┐
                                  │            MariaDB            │
                                  │  tickets · threads · users    │
                                  │  KB articles (imported)       │
                                  └───────────────────────────────┘

 ┌────────────────────────────────────────────────────────────────┐
 │ KB-as-docs: kb/ markdown (git versioned) → imported into       │
 │ osTicket's Knowledge Base via Admin panel (source of truth)    │
 └────────────────────────────────────────────────────────────────┘
```

## Repo layout

| Path | Purpose |
|---|---|
| `docker-compose.yml` | osTicket web app + MariaDB; named volumes, placeholder secrets |
| `.env.example` | Template for all secrets — copy to `.env`, never commit |
| `docs/architecture.md` | Design decisions: why osTicket, data model, KB-as-docs |
| `docs/ticket-taxonomy-and-workflows.md` | Category scheme, priority matrix, statuses, SLAs, L1/L2/L3 rules, Jira→Zendesk migration mapping |
| `kb/KB-001-vpn-connection-troubleshooting.md` | SOP article: WireGuard client won't connect |
| `kb/KB-002-mfa-reset-and-reregistration.md` | SOP article: lost phone / MFA reset + re-register |
| `kb/KB-003-new-hire-account-checklist.md` | SOP article: new-hire account provisioning checklist |
| `.gitignore` | `.env`, `exports/`, `backups/` excluded |

## Quick start

```bash
# 1. Configure secrets (placeholders only — real values live here, uncommitted)
cp .env.example .env
# edit .env and set DB_PASSWORD, DB_ROOT_PASSWORD, admin creds

# 2. Start the stack
docker compose up -d

# 3. First login
#    Client portal: http://localhost:8080
#    Staff panel:   http://localhost:8080/scp
#    Run the osTicket web installer on first visit (needs a browser on the host).
#    Use the DB host "db", database/user/password from your .env.

# 4. Load the category scheme (manual, in the staff panel — Admin panel)
#    Recreate the help topics in docs/ticket-taxonomy-and-workflows.md
#    (Admin panel → Manage → Help Topics), set priorities and SLA plans
#    to match the design tables, and create the department queues.

# 5. Import the knowledge base
#    Admin panel → Knowledge Base → Categories, then paste each kb/KB-*.md
#    article in as an internal article (staff-only) — the kb/ folder stays
#    the versioned source of truth.
```

## Security notes

- **No real credentials anywhere in this repo.** `docker-compose.yml` and
  `.env.example` carry only placeholders (`changeme` defaults). Real values
  live in `.env` (git-ignored, per `.gitignore`) or your secret manager.
- osTicket ships with a default admin account during install — change the
  password immediately and delete the installer directory after setup.
- This implementation binds osTicket to `localhost:8080` only; put a reverse proxy with
  TLS in front before exposing it anywhere real.
- MariaDB has no published port in this compose file — the database is only
  reachable from the osTicket container.

## Validation

- `docker-compose.yml`: parsed with `python3 -c "yaml.safe_load(...)"` — **no
  errors** (compose schema check, not a full `docker compose config`, since
  Docker is not available locally).
- `.env.example` / `.gitignore`: manually reviewed — no real secrets present.
- KB articles and design docs: careful review pass (structure, step order,
  terminology) — no host available to run them end to end.
- **Needs a real host/browser to complete:** `docker compose pull`
  (verify the `campbellsoftwaresolutions/osticket` tag exists at pull time),
  `docker compose up`, and the osTicket web installer (database connection,
  admin account creation, and KB import are manual UI steps).
