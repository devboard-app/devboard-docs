# Known limitations

> **In one minute:** This is an index of what is not right yet. Each problem is listed once, with links to the pages that explain it.
> Nothing here breaks the system today. The full text, with the reason and the effect, is in the **Known gaps** section of each linked page.

Pages are short names: **overview**, **auth**, **core**, **email**, **work**, **attachments**, **integrations**, **analytics**, **infra**, and the flows.

## Events and the outbox

| Problem | Where |
|---|---|
| The stream, the outbox and the `failed_events` tables only grow | [data map](data-map.md), [events](flows/events-and-notifications.md), [deletes](flows/deletes-and-cleanup.md) |
| Some actions send no event, so analytics does not log them | [team flow](flows/team-project-ticket.md), [analytics flow](flows/analytics-pipeline.md) |

## Files

| Problem | Where |
|---|---|
| The ownership check on file links is optional | [attachments](../services/attachments/index.md), [file upload](flows/file-upload.md) |
| No route to list your own files | [attachments](../services/attachments/index.md) |
| MinIO still uses root credentials for everything (no bucket policy or scoped key) | [attachments](../services/attachments/index.md), [data map](data-map.md), [infra](../services/infra/index.md) |

## Auth and security

| Problem | Where |
|---|---|
| One shared `INTERNAL_API_KEY` for all services | [auth and security](auth-and-security.md), [overview](overview.md) |
| One shared HS256 secret: any service could forge a token | [auth and security](auth-and-security.md) |
| Internal routes are reachable if you have the key and the port | [auth and security](auth-and-security.md) |
| A deactivated user has up to 5 minutes in some services | [auth](../services/auth/index.md), [login flow](flows/login-and-refresh.md), [auth and security](auth-and-security.md) |
| A role change reaches services up to 5 minutes late | [auth](../services/auth/index.md), [login flow](flows/login-and-refresh.md), [auth and security](auth-and-security.md) |
| IP rate limits may still be shared by all users in production (no gateway sets `X-Forwarded-For` there yet; local dev is fixed) | [sign-up flow](flows/sign-up-and-verify.md), [login flow](flows/login-and-refresh.md), [auth and security](auth-and-security.md) |

## Coupling and speed

| Problem | Where |
|---|---|
| Role and status live in two places (auth and core) | [core](../services/core/index.md) |

## Integrations and notifications

| Problem | Where |
|---|---|
| The webhook publishes straight to Redis, with no outbox | [webhook flow](flows/inbound-webhook.md) |
| One repo can link to only one project | [integrations](../services/integrations/index.md), [webhook flow](flows/inbound-webhook.md) |
| A core outage silently drops @mentions | [comment flow](flows/comment-and-mention.md) |

## Infra and running it

| Problem | Where |
|---|---|
| One Postgres for five services | [data map](data-map.md), [infra](../services/infra/index.md) |
| Redis does several jobs in one instance | [data map](data-map.md) |
| App service ports (8001-8008) are still open on the host. Shared infra ports (Postgres, Redis, Mongo, MinIO) are now bound to `127.0.0.1` only | [infra](../services/infra/index.md), [auth and security](auth-and-security.md) |
| `stack.yml` does not run migrations automatically (now documented in its own header — run `migrate.bat` after) | [infra](../services/infra/index.md) |
| The scripts run on Windows only | [infra](../services/infra/index.md) |
| No start order (minor: containers restart until ready) | [infra](../services/infra/index.md) |
| Email has no retry (declined on purpose: the caller waits on this synchronously, so a retry means a slower request, not a faster recovery) | [email](../services/email/index.md) |

## Small cleanups

| Problem | Where |
|---|---|
| `get_recent_events` in analytics has no route | [analytics](../services/analytics/index.md) |
| There is no route to delete a user | [deletes](flows/deletes-and-cleanup.md) |
| Cascade deletes send no per-ticket events | [deletes](flows/deletes-and-cleanup.md) |
| Copied ids across services are not checked | [data map](data-map.md) |

## How to keep this page true

1. When you fix a problem, delete its row here and its ⚠️ line on the pages.
2. When you find a new problem, add the ⚠️ line to the page first, then one row here.
3. This page holds titles only. Do not copy the explanation.

Next: open the page in the **Where** column of the row you care about most, and read its **Known gaps** section.
