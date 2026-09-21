# Flows

A flow follows one user journey across services. Each page has a picture first, then numbered steps, then what happens when a step fails.

## The flows

| Flow | Question it answers | Services |
|---|---|---|
| [Sign-up and email check](sign-up-and-verify.md) | How does a new user get an account? | web, auth, core, email |
| [Login and refresh](login-and-refresh.md) | How do tokens work? How does a user stay logged in? What happens on deactivation and password reset? | web, auth, core, work |
| [Team, project and ticket](team-project-ticket.md) | How does work get planned? Who may do what? | web, work, core, email |
| [Comment and @mention](comment-and-mention.md) | What happens when someone comments and mentions a teammate? | web, work, core, attachments, integrations |
| [File upload](file-upload.md) | How does a file get from the browser into a comment? | web, attachments, MinIO, work |
| [Events and notifications](events-and-notifications.md) | How does a change in work become a notification or a Slack message? | work, Redis, integrations |
| [GitHub webhook](inbound-webhook.md) | How does a commit get linked to a ticket? | GitHub, integrations, work, Redis |
| [Analytics pipeline](analytics-pipeline.md) | How does an event become a report? | work, Redis, analytics, MongoDB |
| [Deletes and cleanup](deletes-and-cleanup.md) | What gets deleted, what stays, and what cleans up after itself? | work, attachments, auth, integrations |

## How to read a flow page

1. Look at the picture. It shows who calls whom, in order.
2. Read the numbered steps. Each step matches an arrow.
3. Read "If a step fails" to see what breaks and what does not.
4. Follow the links to the service pages and the code.

## Two ideas that appear in many flows

- **The web app is the client.** The browser talks only to `devboard-web`. The web app keeps the user's tokens in a server-side session and calls the backend services. It has no page of its own here. See [System overview](../overview.md).
- **Events go through an outbox.** Work saves an event in its own database together with the change. A relay container sends it to Redis later. See [devboard-work](../../services/work/index.md).

Next: start with [Sign-up and email check](sign-up-and-verify.md).
