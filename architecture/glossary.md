# Glossary

> **In one minute:** Plain meanings for the words used in these docs. Find the group you need, then the word.
> Each meaning is one line. Where a page explains the idea in full, the meaning links to it.

## Login and security

| Word | Meaning |
|---|---|
| **Access token** | The JWT the client sends with each request. Lives 5 minutes. |
| **Active / inactive** | A user's status in core. An inactive user is blocked in core and work. |
| **Bearer** | The way to send a token: the header `Authorization: Bearer <token>`. |
| **bcrypt** | The way auth scrambles passwords before saving them. |
| **Constant-time compare** | Comparing two secrets in the same time, however alike they are. `hmac.compare_digest`. |
| **Hash** | A one-way scramble. It cannot be turned back. Tokens and passwords are saved as hashes. |
| **HMAC** | A signature made from the data plus a shared secret. GitHub uses it to sign webhooks. |
| **HS256** | How the JWT is signed: with one shared secret (`JWT_SECRET`). |
| **JWT** | A signed pass that says who the user is. A service checks the signature itself. See [Auth and security](auth-and-security.md). |
| **Rate limit** | A maximum number of tries in a time window. Auth uses it on login and sign-up. |
| **Refresh token** | A random string that lives 7 days. It is swapped for a new pair, and works once. |
| **Service key** | The shared secret in the header `X-Service-Key`. It proves the caller is a DevBoard service. |
| **Signature** | A stamp that proves a token or a webhook was not changed. |
| **Verified** | The user clicked the link in the "verify your email" mail. |

## Events

| Word | Meaning |
|---|---|
| **Ack (`XACK`)** | "I finished this message." Without it, the message stays pending. |
| **Consumer** | One reader inside a consumer group. In the code it has a fixed name. |
| **Consumer group** | A named reader of a stream. Each group gets every message. |
| **Dead letter** | The place for messages that failed too often: the `failed_events` table or collection. |
| **Event** | A small message that says something happened, like `ticket.assigned`. |
| **Idempotent** | Doing it twice gives the same result as doing it once. |
| **Outbox** | A table in work. A change and its event are saved together, and the event is sent later. See [Events and notifications](flows/events-and-notifications.md). |
| **Payload** | The data inside an event or a request. |
| **Pending** | Read by a consumer but not acked yet. |
| **Reclaim** | Taking back pending messages that sat idle too long. |
| **Recipient** | The user who gets a notification (`recipient_id`). |
| **Redis stream** | A list of events in Redis. The DevBoard stream is `devboard:events`. |
| **Relay** | The container that reads the outbox and sends each event. |
| **Replay** | Reading old events again. Analytics uses it to rebuild the log. |
| **Webhook** | A URL that a system calls when something happens. GitHub calls integrations. Integrations calls Slack. |
| **Worker** | A container that runs a background loop. It does not answer HTTP. |
| **`XADD`** | The Redis command that adds an event to a stream. |

## Files

| Word | Meaning |
|---|---|
| **Bucket** | The top-level folder in MinIO or S3. Ours is `devboard-attachments`. |
| **Content type** | What kind of file it is, like `image/png`. |
| **Context** | What a file belongs to: `context_type` and `context_id`. |
| **MinIO** | The file storage. It speaks the same language as S3. |
| **Object** | One stored file. |
| **Pending / stored** | The two states of an upload. `pending` is not confirmed yet. `stored` is checked. |
| **Presigned URL** | A temporary link that lets the browser upload or download one file with no login. See [File upload](flows/file-upload.md). |
| **S3** | Amazon's file-storage service and its API. MinIO copies the API. |
| **Storage key** | The path of an object in the bucket: `{id}/{filename}`. |

## Data

| Word | Meaning |
|---|---|
| **AOF** | Redis saves every write to disk. Started with `--appendonly yes`. |
| **Cascade** | Deleting a parent row also deletes its children. |
| **Foreign key** | A database rule that links two tables. There are none across services. |
| **Hard delete / soft delete** | Hard: the row is gone. Soft: the row stays with a "deleted" flag. Work uses hard deletes. |
| **Index** | An extra structure that makes a query faster. |
| **Migration** | A script that changes a database's tables. Alembic or Django runs it. |
| **MongoDB** | The document database. Analytics keeps its activity log there. |
| **PostgreSQL** | The main database. One server, five databases. See [Data map](data-map.md). |
| **Redis** | A fast store. Here it holds the event stream, rate limits and web sessions. |
| **Transaction** | Many database changes that all succeed or all fail together. |
| **UUID** | A long random id, like `3f2b8c1e-...`. |
| **Volume** | Docker storage that keeps data after a container restarts. |

## Running it

| Word | Meaning |
|---|---|
| **Commit (git)** | A saved snapshot of the code. "Key code" links point to one commit. |
| **Container** | One running program, packed with what it needs. |
| **Docker Compose** | The tool that starts many containers from a file. |
| **Endpoint / route** | One URL a service answers, like `POST /auth/login/`. |
| **Health check** | A route (`/health`) that says the service is up. |
| **Internal route** | A route for other services only. It needs the service key. |
| **Reverse proxy** | A server in front of the services. It decides which routes the public can reach. |
| **SMTP** | The protocol for sending email. |
| **`stack.yml`** | The one file that starts every service. In `devboard-infra`. |

## Work terms

| Word | Meaning |
|---|---|
| **Backlog** | Tickets that are not in a sprint. |
| **Board** | Tickets grouped by status. |
| **Burndown** | Work left in a sprint, day by day. |
| **Cycle time** | How long a ticket was really worked on. |
| **Epic** | A big ticket that groups other tickets. |
| **Lead time** | From a ticket created to done. |
| **Lead / contributor** | The two project roles. A lead can do more. |
| **Mention** | `@username` in a comment. The person gets a notification. |
| **Sprint** | A time box of work. One active sprint per project. |
| **Story points** | A size estimate for a ticket. |
| **Ticket key** | The short name of a ticket, like `DEV-12`. |
| **Velocity** | Story points finished per sprint. |

## HTTP status codes

| Code | Meaning here |
|---|---|
| `401` | Not logged in, or the token is bad or expired. |
| `403` | Logged in, but not allowed. Also a wrong service key. |
| `404` | Not found. |
| `409` | Conflict: it already exists, or a rule blocks it. |
| `413` | The file is too big. |
| `415` | The file is the wrong type. |
| `422` | The input is not valid. |
| `429` | Too many requests. The rate limit stopped it. |
| `502` | A service this one calls failed. |
| `503` | A service is not available, or a needed part is down. |

Next: go back to the [README](../README.md) and pick a service page or a flow.
