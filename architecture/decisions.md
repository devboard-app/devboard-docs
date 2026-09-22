# Decisions

> **In one minute:** Short notes on why the system looks the way it does. Each row has the choice, the reason and the cost.
> Reasons come from the service READMEs and the code. A reason marked *(inferred)* was not written down. The last section lists choices with no recorded reason.

## Services and data

| Decision | Why | Cost |
|---|---|---|
| **Each service owns its database** (five Postgres databases, one server) | No service can break another's tables. Each can change its schema alone *(inferred)* | No joins across services. Copied ids are not checked |
| **Analytics is one service** (log and reports together) | A second publisher exists (`ticket.commit_linked` comes from integrations). Core should not own a table it does not write. Reports read only this log | One more service to run |
| **MongoDB for the activity log** | Event details differ by type. A label event and a commit event share only the envelope | A second database technology. Shape is enforced in code |
| **PostgreSQL for attachments** | Uniform data. The `pending` to `stored` step needs transactions | If file details ever vary, add a `JSONB` column, not a new database |
| **Integrations has no team table** | Work stays the single owner of roles. Integrations asks work each time *(inferred)* | An extra HTTP call per request. Needs work to be up |

More: [Data map](data-map.md), [devboard-analytics](../services/analytics/index.md).

## Auth and roles

| Decision | Why | Cost |
|---|---|---|
| **Each service checks the JWT itself** | No call to auth on each request | One shared secret. Any service could forge a token |
| **Access token 5 minutes, refresh token 7 days, rotated** | A stolen access token works only briefly. A refresh token works once *(inferred)* | Role and status changes reach some services late |
| **Only hashes of refresh and reset tokens are stored** | A database leak does not give usable tokens *(inferred)* | None worth noting |
| **Rate limits refuse when Redis is down** | Never run login without limits | Redis down means no login |
| **Team roles and project roles are separate** | Being on a team must not open every project | Two checks on every request |

More: [Auth and security](auth-and-security.md).

## Events

| Decision | Why | Cost |
|---|---|---|
| **One Redis stream, one consumer group per reader** | Integrations and analytics each see every event. A new group can replay all of it | Nothing trims the stream |
| **Work uses an outbox** | The change and its event are saved together. A Redis outage does not fail the request or lose the event | Events are late by about 2 seconds. Retries with backoff (up to 150 attempts) instead of giving up fast, so a longer outage is now survivable |
| **The outbox row's own id (`outbox_id`) is sent with the event, and used as the MongoDB `_id` when present** | Redelivery is harmless even though it gets a new Redis message id — the outbox id stays the same, so analytics can still dedupe it | Readers that don't check for `outbox_id` and fall back to the Redis message id still dedupe worse |
| **Analytics renames some events** (`ticket.status_changed` is stored as `ticket.updated`) | One shape for all field changes. Reports are simpler | Queries filter on a field, not a bare action |
| **Reports replay the event log** | No change to the event format in three repos. No calls back to work | Slow on big projects — mitigated by caching the result per project (Redis, invalidated the moment a new event lands, not on a timer). The first request after a change still pays the full replay cost |

More: [Events and notifications](flows/events-and-notifications.md), [Analytics pipeline](flows/analytics-pipeline.md).

## Files

| Decision | Why | Cost |
|---|---|---|
| **Presigned upload, straight to MinIO** | The bytes never pass through a service | The service must check the file afterwards |
| **Confirm reads the real file** | The client's claims are not trusted. A zip named `cat.png` fails | One extra download per upload |
| **Work keeps only file ids. Links are made when needed** | Links expire, so they are never stored | An extra call to attachments on each read |
| **Attachments knows nothing about comments** (generic `context_type` and `context_id`) | The same service can serve tickets or avatars later | The columns are not used yet |
| **Two S3 clients** (internal and public address) | A signed link is tied to its host name. The browser needs the public one | Two settings to keep right |

More: [File upload](flows/file-upload.md), [devboard-attachments](../services/attachments/index.md).

## GitHub and integrations

| Decision | Why | Cost |
|---|---|---|
| **The event actor is a fixed system id** | Anyone can fake the commit author | Analytics cannot show who pushed |
| **`linked_commits` has a unique key on (repo, sha, ticket)** | GitHub redelivers webhooks | A failed publish loses the link |
| **Webhook URLs allow only Slack and Discord hosts** | A team admin cannot point a webhook at an internal service | New providers need a code change |

More: [GitHub webhook](flows/inbound-webhook.md).

## Running it

| Decision | Why | Cost |
|---|---|---|
| **`stack.yml` uses `include:`** | Each repo keeps its own compose file and still runs alone | No `depends_on`. Containers restart until Postgres is ready |
| **Redis runs with `--appendonly yes`** | A restart must not lose the stream or the reader positions | Slightly slower writes *(inferred)* |
| **Email has no database and no queue** | It stays small. The caller decides whether to retry | A mail is lost if SMTP is down and the caller does not retry |

More: [devboard-infra](../services/infra/index.md), [devboard-email](../services/email/index.md).
