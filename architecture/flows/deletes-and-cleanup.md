# Flow: deletes and cleanup

> **In one minute:** DevBoard has **no soft delete** and **no cross-service delete**. Work deletes rows for good, and it tells nobody.
> Exactly **one** job cleans up after itself: the attachments cleanup loop. Everything else stays until someone removes it by hand.

This page is mostly about what does **not** happen. That is on purpose: it is the messy part.

## The picture

```mermaid
flowchart TB
    del["User deletes a team, project,<br/>ticket or comment in devboard-work"]
    cascade["Django cascade in work_db<br/>rows are deleted for good"]
    stays["Stays behind, nobody is told"]

    del --> cascade
    cascade -.-> stays

    stays --> f["Stored files in MinIO<br/>and rows in attachments_db"]
    stays --> a["Events in MongoDB activity_db"]
    stays --> n["Team settings, repo links and<br/>notifications in integrations_db"]

    pending["Upload started but never confirmed<br/>(pending row and object)"]
    loop["attachments cleanup loop<br/>every 15 min"]
    pending --> loop
    loop -->|"deletes rows older than 1 hour<br/>and their files"| gone["Removed"]
```

## What gets deleted, and how

| What | Who deletes | How |
|---|---|---|
| Team | Work | `DELETE /api/teams/<id>/`. Django **cascades** to memberships, projects, and everything below. |
| Project | Work | Cascades to project members, tickets, comments, labels and sprints. |
| Ticket | Work | Cascades to its comments. Sends `ticket.deleted`. |
| Comment | Work | Author, or any project lead. Sends `comment.deleted`. |
| Sprint | Work | Only a sprint that has not started. Tickets in it stay, without a sprint. |
| Notification | The user | `DELETE /api/notifications/<id>/`. |
| Attachment | The owner | `DELETE /attachments/<id>/`. Removes the row and the file. |
| User | **Nobody** | There is no delete-user route. An admin can only set the user to `inactive`. |

All deletes in work are **hard deletes**. No model has a "deleted" flag.

## The cleanups that do exist

| Cleanup | Where | What it does |
|---|---|---|
| **Pending uploads** | attachments, loop every 15 minutes | Deletes `pending` rows older than **1 hour**, and their files. This is the only scheduled cleanup job. |
| **Failed confirm** | attachments | If the file is missing, too big or the wrong type, the row and the file are removed at once. |
| **Failed sign-up** | auth | If core cannot create the profile, auth deletes the new user again. |
| **Half-made team or project** | work | If the owner or lead membership cannot be saved, work deletes the new team or project again. |
| **Deactivation** | auth | Revokes all refresh tokens of the user. |

Revoked tokens are marked, not deleted.

## What is left behind

| What stays | Why |
|---|---|
| **Stored files of deleted comments** | Work never tells attachments. Cleanup only removes `pending` rows. The files stay in MinIO. |
| **Events of deleted projects and tickets** | Analytics keeps its log. A cascade deletes tickets **without** sending `ticket.deleted` for each one. Nobody can read the reports of a deleted project, because the role check in work fails. |
| **Team settings, repo links and notifications** of a deleted team | Integrations has its own database and no link to work's rows. Nothing tells it. |
| **Delivered outbox rows** | The relay marks them delivered and never deletes them. |
| **Outbox rows that failed 5 times** | They stay unsent. Nothing retries them. |
| **The event stream** | `XADD` has no size limit. The stream is never trimmed. |
| **`failed_events`** in integrations and analytics | Nothing reads them and nothing empties them. |
| **Revoked and expired tokens** in auth | Only marked. There is no purge job. |

## Why it looks like this

Each service owns its own database. That is the rule: no service reads or writes another one's tables. So there are no foreign keys across services, and a delete in work cannot cascade into attachments or integrations by itself. Someone has to send a message. Today nobody does.

The events already exist as the way to tell other services. Work sends `ticket.deleted` and `comment.deleted`. But no service reacts to them by deleting files or rows. And `comment.deleted` carries only the comment id, not the ids of the files.

## ⚠️ Known gaps

- **Orphan files.** This is the biggest one. Deleting a comment leaves its files in MinIO for ever.
- **Cascades are silent.** Deleting a project sends no per-ticket events.
- **Growing tables and streams.** The outbox, the event stream and the failure tables only grow.
- **No way to delete a user.** The PRD lists it, but no route exists.

## Ideas, not built

These are not in the code. They are listed only as directions:

- Add the file ids to `comment.deleted` and send it to attachments (through the outbox), so it can delete the files.
- Add a job that deletes old delivered outbox rows.
- Trim the stream, for example with a maximum length.

## Key code

| Where | What is there |
|---|---|
| [attachments `app/cleanup.py`](https://github.com/devboard-app/devboard-attachments/blob/0921514/app/cleanup.py) | `cleanup_pending` |
| [attachments `docker-compose.yml`](https://github.com/devboard-app/devboard-attachments/blob/0921514/docker-compose.yml) | The loop: `while true; do python -m app.cleanup; sleep 900; done` |
| [auth `app/services/auth.py`](https://github.com/devboard-app/devboard-auth/blob/b53e136/app/services/auth.py) | `register` (the rollback), `update_user_status` |
| [work `projects/services.py`](https://github.com/devboard-app/devboard-work/blob/0beba51/projects/services.py) | `create_project_with_lead` (the rollback), `delete_project` |
| [work `teams/services.py`](https://github.com/devboard-app/devboard-work/blob/0beba51/teams/services.py) | `create_team_with_owner` (the rollback) |
| [work `comments/services.py`](https://github.com/devboard-app/devboard-work/blob/0beba51/comments/services.py) | `delete_comment` |

Next: go back to the [flow index](README.md), or open a [service page](../../README.md#pages).
