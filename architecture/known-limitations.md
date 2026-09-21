# Known limitations

> **In one minute:** This is an index of what is not right yet. Each problem is listed once, with links to the pages that explain it.
> Nothing here breaks the system today. The full text, with the reason and the effect, is in the **Known gaps** section of each linked page.

Pages are short names: **overview**, **auth**, **core**, **email**, **work**, **attachments**, **integrations**, **analytics**, **infra**, and the flows.

## Events and the outbox

| Problem | Where |
|---|---|
| The outbox gives up after about 10 seconds | [work](../services/work/index.md), [events](flows/events-and-notifications.md), [team flow](flows/team-project-ticket.md) |
| An event can be sent twice, and analytics saves both | [work](../services/work/index.md), [events](flows/events-and-notifications.md), [analytics](../services/analytics/index.md), [analytics flow](flows/analytics-pipeline.md) |
| The stream, the outbox and the `failed_events` tables only grow | [data map](data-map.md), [events](flows/events-and-notifications.md), [deletes](flows/deletes-and-cleanup.md) |
| One worker per reader only (fixed consumer name) | [overview](overview.md), [integrations](../services/integrations/index.md), [analytics](../services/analytics/index.md), [events](flows/events-and-notifications.md) |
| A wrong event name fails silently | [integrations](../services/integrations/index.md), [events](flows/events-and-notifications.md) |
| Some actions send no event, so analytics does not log them | [team flow](flows/team-project-ticket.md), [analytics flow](flows/analytics-pipeline.md) |

## Files

| Problem | Where |
|---|---|
| Files of deleted comments stay forever | [work](../services/work/index.md), [attachments](../services/attachments/index.md), [file upload](flows/file-upload.md), [deletes](flows/deletes-and-cleanup.md) |
| The `context_type` and `context_id` columns are never used | [attachments](../services/attachments/index.md) |
| The per-context file limit can be skipped | [attachments](../services/attachments/index.md), [file upload](flows/file-upload.md) |
| The ownership check on file links is optional | [attachments](../services/attachments/index.md), [file upload](flows/file-upload.md) |
| No route to list your own files | [attachments](../services/attachments/index.md) |
| MinIO is dev-only: root credentials, no bucket policy, no backup | [attachments](../services/attachments/index.md), [data map](data-map.md), [infra](../services/infra/index.md) |

## Auth and security

| Problem | Where |
|---|---|
| One shared `INTERNAL_API_KEY` for all services | [auth and security](auth-and-security.md), [overview](overview.md) |
| The key is not compared the same way everywhere | [auth and security](auth-and-security.md) |
| One shared HS256 secret: any service could forge a token | [auth and security](auth-and-security.md) |
| Internal routes are reachable if you have the key and the port | [auth and security](auth-and-security.md) |
| A deactivated user has up to 5 minutes in some services | [auth](../services/auth/index.md), [login flow](flows/login-and-refresh.md), [auth and security](auth-and-security.md) |
| A role change reaches services up to 5 minutes late | [auth](../services/auth/index.md), [login flow](flows/login-and-refresh.md), [auth and security](auth-and-security.md) |
| IP rate limits may be shared by all users | [sign-up flow](flows/sign-up-and-verify.md), [login flow](flows/login-and-refresh.md), [auth and security](auth-and-security.md) |
| Sign-up is not one step (auth saves, then calls core) | [auth](../services/auth/index.md), [sign-up flow](flows/sign-up-and-verify.md) |

## Coupling and speed

| Problem | Where |
|---|---|
| Work asks core on every request. If core is down, work is down | [core](../services/core/index.md), [work](../services/work/index.md) |
| Role and status live in two places (auth and core) | [core](../services/core/index.md) |
| Every authenticated request writes `last_active` | [core](../services/core/index.md) |
| Reports replay the whole project log, with no cache | [analytics](../services/analytics/index.md), [analytics flow](flows/analytics-pipeline.md) |
| Events lost in a long MongoDB outage end in `failed_events` | [analytics flow](flows/analytics-pipeline.md) |

## Integrations and notifications

| Problem | Where |
|---|---|
| Email notifications are not built | [integrations](../services/integrations/index.md), [email](../services/email/index.md), [overview](overview.md) |
| A failed publish loses the commit link for good | [integrations](../services/integrations/index.md), [webhook flow](flows/inbound-webhook.md) |
| GitHub always gets `200`, and a skipped commit is not retried | [webhook flow](flows/inbound-webhook.md) |
| The webhook publishes straight to Redis, with no outbox | [webhook flow](flows/inbound-webhook.md) |
| One repo can link to only one project | [integrations](../services/integrations/index.md), [webhook flow](flows/inbound-webhook.md) |
| Slack and Discord messages are never retried | [integrations](../services/integrations/index.md) |
| Assigning a ticket to yourself still notifies you | [team flow](flows/team-project-ticket.md), [comment flow](flows/comment-and-mention.md) |
| A core outage silently drops @mentions | [comment flow](flows/comment-and-mention.md) |

## Infra and running it

| Problem | Where |
|---|---|
| One Postgres for five services | [data map](data-map.md), [infra](../services/infra/index.md) |
| Redis does several jobs in one instance | [data map](data-map.md) |
| Dev ports are open on the host | [infra](../services/infra/index.md), [auth and security](auth-and-security.md) |
| `stack.yml` does not run migrations | [infra](../services/infra/index.md) |
| The scripts run on Windows only | [infra](../services/infra/index.md) |
| No start order (minor: containers restart until ready) | [infra](../services/infra/index.md) |
| Email has no retry, and `APP_URL` is missing from `.env.example` | [email](../services/email/index.md) |

## Small cleanups

| Problem | Where |
|---|---|
| `POST /events/` in analytics has no caller | [analytics](../services/analytics/index.md) |
| `get_recent_events` in analytics has no route | [analytics](../services/analytics/index.md) |
| There is no route to delete a user | [deletes](flows/deletes-and-cleanup.md) |
| Cascade deletes send no per-ticket events | [deletes](flows/deletes-and-cleanup.md) |
| Copied ids across services are not checked | [data map](data-map.md) |

## How to keep this page true

1. When you fix a problem, delete its row here and its ⚠️ line on the pages.
2. When you find a new problem, add the ⚠️ line to the page first, then one row here.
3. This page holds titles only. Do not copy the explanation.

Next: open the page in the **Where** column of the row you care about most, and read its **Known gaps** section.
