# DevBoard Docs

How DevBoard works. Written for someone who has never seen the code.

> **In one minute:** DevBoard is a tool for teams to plan work: teams, projects, tickets, sprints and comments.
> The backend is split into 7 small services. They talk over HTTP and one Redis stream.
> This repo explains what each service does and how they work together.

## Start here

1. Read [System overview](architecture/overview.md). It has the big picture and the architecture diagram. (5 min)
2. Pick a [service page](#pages) to see how one repo works.
3. Pick a [flow page](architecture/flows/README.md) to follow one full journey, for example "upload a file".

## What is in scope

| In scope | Out of scope |
|---|---|
| `devboard-auth` | `devboard-web` (the web app) |
| `devboard-core` | `devboard-design` (the design prototype) |
| `devboard-work` | |
| `devboard-email` | |
| `devboard-integrations` | |
| `devboard-analytics` | |
| `devboard-attachments` | |
| `devboard-infra` (Postgres, Redis, MongoDB, MinIO, Docker Compose) | |

The web app appears in diagrams as "the client". It has no page of its own.

## Pages

| Page | What you learn |
|---|---|
| [System overview](architecture/overview.md) | All services, the diagram, how they talk, the outbox, auth in 30 seconds |
| [Auth and security](architecture/auth-and-security.md) | User tokens, the service key, roles, secrets, security gaps |
| [Data map](architecture/data-map.md) | What is stored where (Postgres, MongoDB, Redis, MinIO), and why |
| [Decisions](architecture/decisions.md) | Why the system looks this way: choice, reason, cost |
| [Glossary](architecture/glossary.md) | Plain meanings for the hard words (JWT, outbox, presigned URL, and more) |
| [devboard-auth](services/auth/index.md) | Sign-up, login, tokens, rate limits |
| [devboard-core](services/core/index.md) | User profiles, roles, status |
| [devboard-email](services/email/index.md) | Sending mail from templates |
| [devboard-work](services/work/index.md) | Teams, projects, tickets, sprints, comments, the outbox |
| [devboard-attachments](services/attachments/index.md) | File uploads with presigned URLs |
| [devboard-integrations](services/integrations/index.md) | Notifications, Slack/Discord, GitHub commit links |
| [devboard-analytics](services/analytics/index.md) | Activity log and reports |
| [devboard-infra](services/infra/index.md) | PostgreSQL, Redis, MongoDB, MinIO, start scripts |
Every service page has the same parts: at a glance, a small diagram, how it checks callers, what it does, what happens when something is down, known gaps, and links to the key code.

### Flows

Each flow starts with a sequence diagram, then numbered steps, then what happens when a step fails.

| Flow | What you learn |
|---|---|
| [Sign-up and email check](architecture/flows/sign-up-and-verify.md) | New account: auth, core, email |
| [Login and refresh](architecture/flows/login-and-refresh.md) | Tokens, refresh, logout, password reset, deactivation |
| [Team, project and ticket](architecture/flows/team-project-ticket.md) | Planning work, roles, sprints |
| [Comment and @mention](architecture/flows/comment-and-mention.md) | Comments, mentions, notifications |
| [File upload](architecture/flows/file-upload.md) | Presigned upload, confirm, attach, read |
| [Events and notifications](architecture/flows/events-and-notifications.md) | Outbox, Redis stream, two readers |
| [GitHub webhook](architecture/flows/inbound-webhook.md) | Linking commits to tickets |
| [Analytics pipeline](architecture/flows/analytics-pipeline.md) | Event in, stored, report out |
| [Deletes and cleanup](architecture/flows/deletes-and-cleanup.md) | What is deleted, what stays behind |

## Coming next

- Known limitations (the gaps from every page, in one list)

## Sources

Facts come from the service READMEs and from the code. Last checked on 2026-09-21.
If a README and the code disagree, the docs follow the code.

## Rules for these docs

- The docs describe the system **as it is today**. If something is unfinished, the page says so.
- No secrets. Only variable names appear, never real values.
- Code is not copied here. Pages link to the source instead.
