# Signal Desk

A compact shared support inbox for the systeme.io support triage assessment. The workspace starts empty so reviewers can import the provided backlog and observe the triage process themselves.

## Demo links

- **Signal Desk app:** https://signal-desk-j4mf.onrender.com
- **Public intake form:** https://gabrielpernia.app.n8n.cloud/form/signal-desk-intake
- **Source repository:** https://github.com/betterenglishacademy1-hue/signal-desk-demo

The hosted app is protected by a demo password. Share the password separately with the reviewer; never commit it here.

## Architecture and services

| Layer | Technology/platform | Role |
| --- | --- | --- |
| User interface | HTML, CSS, vanilla JavaScript | Inbox, analytics, settings, request detail, editing, filters, and AI triage controls. |
| Application/API server | Node.js (`server.js`) | Serves the app, protects the demo, and exposes request, ingestion, and AI classification endpoints. No external npm dependencies are required. |
| Hosting | Render Web Service | Runs the Node.js server and supplies runtime environment variables. |
| AI triage | OpenAI API (`gpt-4o-mini` by default) | Classifies messages by team, priority, language, confidence, summary, and rationale. The API key stays on the server. |
| Shared storage | Supabase PostgreSQL via the Supabase REST API | Persists requests, assignments, statuses, notes, and change history so demo users share the same data. |
| Intake automation | n8n Cloud | Hosts the public form and sends each submission to the protected Signal Desk ingestion endpoint. |
| Source control | GitHub | Stores the application source and documentation. |

### Request flows

- **CSV import:** Import a CSV from the inbox. Newly imported rows are automatically sent to AI triage as that batch; existing requests are not reprocessed. If the provider request fails, each affected row is marked failed with an explanation and can be retried; if no API key is configured, the app uses its labeled rules fallback.
- **Public form:** A reviewer submits the n8n form. The active n8n workflow sends only that new message to `POST /api/ingest` with a bearer token. The Node.js server stores it immediately as “Analyzing”, classifies that one message in the background, then saves the result to Supabase. Critical requests are surfaced in the app.
- **Manual entry:** Create a request in the app; AI triage starts automatically for that new request only.

## Features

- Empty initial workspace; reviewers bring the assessment CSV from Slack or use another CSV.
- Searchable inbox with workspace, team, status, priority, assignee, source, customer, and sort filters. **Newest first** is the default; each browser remembers its selected sort.
- Team views, My queue, unassigned, resolved, spam, recently deleted, analytics, and activity/history views.
- Request assignment and status updates, inline edits, internal notes, reversible soft deletion, permanent deletion from Recently deleted, and change history with restore/undo actions.
- Bulk selection and actions; manual request creation; CSV import preview and duplicate detection.
- AI-assisted team, priority, language, confidence, summary, and classification rationale.
- Critical and low-confidence cases are visible for human review. AI output is a recommendation; agents remain responsible for decisions.
- Shared Supabase persistence with a five-second refresh between browsers. This is shared demo state, not a live collaborative editing channel.

## n8n workflow

The active **Signal Desk · New request intake** workflow provides the public intake form. It calls the protected `POST /api/ingest` endpoint and routes the result by team. Retry handling is enabled for the API call.

Team email and needs-review email nodes are intentionally disabled placeholders. The error workflow is active, but its email node is also disabled. No outgoing team or failure notification emails are sent. Critical requests are surfaced in-app.

The form accepts a name, email, and message. Do not submit real customer data to this demo without authorization.

## Configuration

Copy `.env.example` to `.env` for local development and set values locally. Keep `.env` out of Git.

- `OPENAI_API_KEY`: server-side OpenAI API key.
- `OPENAI_MODEL`: optional model override; defaults to `gpt-4o-mini`.
- `SUPABASE_URL`: Supabase project URL.
- `SUPABASE_SECRET_KEY`: Supabase secret key for server-side database access. The server also accepts legacy `SUPABASE_SERVICE_ROLE_KEY`.
- `DEMO_PASSWORD`: required for the temporary hosted demo.
- `N8N_INGEST_TOKEN`: shared bearer token between n8n and `POST /api/ingest`.
- `AI_RUNS_PER_HOUR`: optional per-IP AI triage limit; defaults to 5.
- `PORT`: optional port; defaults to 4173 locally. Render supplies its own port.

Production values belong in Render's environment settings and n8n credentials, never in browser code, workflow exports, screenshots, or Git. The browser does not receive the OpenAI or Supabase server secrets. If shared Supabase storage is unavailable, the app falls back to browser-local storage.

## Run locally

Requires Node.js 18 or newer.

1. Clone the repository and copy `.env.example` to `.env`.
2. Set the environment variables you need. OpenAI and Supabase are optional for local UI testing; without them, the app uses rules-based triage and local browser storage.
3. Run `npm start`.
4. Open http://localhost:4173.

There are no external npm packages to install.

## Demo limitations and test notes

- The hosted demo uses one shared password, not individual accounts or role-based access.
- Shared data refreshes every five seconds; there is no real-time collaboration channel.
- The public n8n intake form is live. Email/chat inbox integrations are not connected.
- Outgoing team and failure notification emails are disabled placeholders; no emails are sent. Critical requests are surfaced in-app.
- OpenAI triage is configured server-side. New messages are sent to OpenAI automatically during triage. Existing requests are re-analyzed only when the user selects Run AI triage.
- The workspace is intended to be empty before reviewer testing. Import the CSV from Slack to see per-row AI triage, submit the public form to test automated intake, or create a request manually.
