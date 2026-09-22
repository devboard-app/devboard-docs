# Flow: events and notifications

> **In one minute:** When something changes in `devboard-work`, work writes an **event** next to the change.
> A relay sends the event to a Redis stream. Two readers take it from there: integrations (notifications, Slack, Discord) and analytics (the activity log).

## The picture

```mermaid
sequenceDiagram
    autonumber
    participant K as devboard-work
    participant P as PostgreSQL work_db
    participant R as outbox relay
    participant X as Redis stream devboard:events
    participant I as integrations worker
    participant N as analytics worker
    participant D as integrations_db
    participant S as Slack or Discord

    K->>P: One transaction: the change and an outbox row
    loop Every 2 seconds
        R->>P: Read unsent rows (oldest first)
        R->>X: XADD event
        R->>P: Set delivered_at
    end
    par Group devboard-integrations-group
        X->>I: Read new message
        alt Event has a recipient_id
            I->>D: Save notification
        else sprint.started or sprint.completed
            I->>S: POST message to the team webhook
        end
        I->>X: XACK
    and Group devboard-analytics-group
        X->>N: Read new message
        N->>N: Translate and save (see Analytics pipeline)
        N->>X: XACK
    end
```

## Step by step

1. **A change and its event are saved together.** Work uses `write_with_outbox`. One database transaction saves the change and a row in `outbox_events`. If the transaction fails, neither exists.
2. **The relay reads the outbox.** The container `devboard-work-outbox-relay` runs `python manage.py drain_outbox`. Every 2 seconds it reads up to 100 unsent rows, oldest first. Redis rows go before email rows.
3. **The relay sends each row** with `XADD` to the stream `devboard:events`, including the outbox row's own id as `outbox_id`. Every value in the event is a string. On success it sets `delivered_at`.
4. **Two consumer groups read the same stream.** Each group has its own position, so each sees **every** message.

**What integrations does with an event:**

| Event | Result |
|---|---|
| `ticket.assigned`, `ticket.status_changed`, `comment.created`, `comment.mentioned` | A row in `notifications` for the `recipient_id`. The status-change and comment events are skipped when there is no `recipient_id`, and `ticket.assigned`/`ticket.status_changed` are also skipped when `recipient_id == actor_id` — you don't get notified about your own action. |
| `sprint.started`, `sprint.completed` | A message to the team's Slack and Discord webhooks. Sent **only if** the URL is set **and** that trigger is `true` in `enabled_triggers`. Retried up to 3 times with a growing delay for timeouts, connection errors and `5xx`; a `4xx` isn't retried. |
| An event in `IGNORED_EVENTS` (analytics-only) | Acked and dropped, on purpose. |
| Any other unmatched event | Logged as an **error** and acked — a typo in an event name is no longer silent. |

**What analytics does:** see [Analytics pipeline](analytics-pipeline.md).

**How a reader handles a failure:**

1. If a handler raises, the message is **not** acked. It stays *pending*.
2. Every loop, the reader takes back messages that were idle too long (60 seconds in integrations, 30 in analytics — reclaiming up to 5000 at a time in analytics, matched to what `xautoclaim` can actually reclaim in one pass).
3. After **3 deliveries** the message goes to the `failed_events` table or collection, with the raw data, and is acked.
4. **Analytics only:** the worker checks MongoDB is reachable before reclaiming or reading anything each loop. While Mongo is down, nothing is processed and nothing's delivery count climbs — so a MongoDB outage doesn't burn through a healthy message's 3 attempts the way it used to.

**Events that exist today:**

- Tickets (work): `ticket.created`, `ticket.updated`, `ticket.assigned`, `ticket.unassigned`, `ticket.status_changed`, `ticket.deleted`
- Ticket links (work): `ticket.epic_linked`, `ticket.epic_unlinked`, `ticket.sprint_added`, `ticket.sprint_removed`
- Labels (work): `label.applied`, `label.removed`
- Sprints (work): `sprint.started`, `sprint.completed`
- Comments (work): `comment.created`, `comment.updated`, `comment.deleted`, `comment.mentioned`
- Commits (integrations): `ticket.commit_linked`. See [GitHub webhook](inbound-webhook.md).

## Why an outbox

The request must not fail just because Redis is slow, and the event must not be lost if work stops right after saving. Writing the event into the same database transaction makes both true. The relay does the risky part later.

## If a step fails

| Step | What happens |
|---|---|
| 3, Redis is down | The row stays unsent. The relay retries with a growing delay (4s, doubling up to a 300s cap) instead of failing fast. |
| 3, sending still fails after 150 attempts | The row is left as failed (roughly half a day of retrying). See the gap below. |
| Reader is stopped | Events wait in the stream. The reader catches up when it starts again. |
| Redis restarts | The stream and the group positions survive, because Redis runs with `--appendonly yes`. |
| Slack or Discord fails | Retried up to 3 times (2s, then 4s) for timeouts, connection errors and `5xx`. A `4xx` isn't retried. Still lost if all retries are exhausted. |
| Handler bug on one event | Retried 3 times, then saved in `failed_events`. Other events keep flowing. |

## ⚠️ Known gaps

- **The stream is never trimmed.** `XADD` has no size limit. The stream grows for ever. (It also means a new group can replay the full history.)
- **Delivered outbox rows are never deleted.** The table grows.
- **Rows that exhaust all 150 attempts stay unsent.** Nothing retries them further; fixing requires resetting `attempts` by hand.

## Key code

| Where | What is there |
|---|---|
| [work `outbox/writer.py`](https://github.com/devboard-app/devboard-work/blob/0beba51/outbox/writer.py) | `write_with_outbox` |
| [work `outbox/management/commands/drain_outbox.py`](https://github.com/devboard-app/devboard-work/blob/0beba51/outbox/management/commands/drain_outbox.py) | The relay loop |
| [work `outbox/dispatch.py`](https://github.com/devboard-app/devboard-work/blob/0beba51/outbox/dispatch.py) | `dispatch` (`XADD` or HTTP to email) |
| [work `work/infrastructure/events.py`](https://github.com/devboard-app/devboard-work/blob/0beba51/work/infrastructure/events.py) | `build_payload` |
| [integrations `app/consumer/worker.py`](https://github.com/devboard-app/devboard-integrations/blob/694a98f/app/consumer/worker.py) | The read loop and retry count |
| [integrations `app/consumer/handlers.py`](https://github.com/devboard-app/devboard-integrations/blob/694a98f/app/consumer/handlers.py) | `HANDLERS` |
| [analytics `app/consumer/worker.py`](https://github.com/devboard-app/devboard-analytics/blob/637ba5e/app/consumer/worker.py) | The other reader |

Next: [GitHub webhook](inbound-webhook.md).
