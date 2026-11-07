# Domain Guard

Domain Guard monitors a portfolio of domains and investigates problems with an AI agent. It is built from n8n workflows and a [Raia](https://raia2.com) agent:

- An hourly monitor and a daily expiry sweep check every domain listed in a Google Sheet.
- Problems go to an incident router, which opens or updates an incident, asks the Raia agent to diagnose it, and logs the result.
- The agent calls five check tools (HTTP status, DNS, SSL certificate, redirects, domain registration), which are n8n webhooks.
- Fixes that need a human go out by email as Approve and Reject links.
- A weekly report summarizes incidents, approvals and upcoming domain expiries, and a failure-alert workflow emails you if any of these workflows breaks.

The workflow files are in [`workflows/`](workflows). Test notes from the agent's evaluation are in [`docs/test-results.md`](docs/test-results.md).

## Placeholders to replace

The exports were scrubbed before publishing. After importing, replace these in the workflows:

| Placeholder | Replace with | Where |
|---|---|---|
| `you@example.com` | The address that receives alerts, approvals and reports | Email nodes in the incident router, weekly report and failure alerts |
| `YOUR_GOOGLE_SHEET_ID` | The ID of your Domain Guard spreadsheet | Every Google Sheets node (or pick the sheet from the node's list) |
| `your-instance.app.n8n.cloud` | Your n8n address | `Send to incident router` in the monitor and sweep, and the approval links in the incident router |

No secrets are included. Credentials appear only by name, and you create your own (see *Restoring into another n8n account* below).


Exported from n8n Cloud on 3 Nov 2026, and the five `check_*` files again on 7 Nov 2026. Each file was verified against the live, published workflow with a SHA-256 hash of its nodes, connections and settings.

## Check tool security

The five check webhooks (`check_http`, `check_dns`, `check_ssl`, `check_redirects`, `check_whois`) require the `X-Domain-Guard-Key` header and answer `403` without it. Each of the five Raia Functions sends that header, with the same value as the n8n **Header Auth account** credential. If you change the secret, update it in that credential and in all five Raia Functions, or Raia's checks will fail.

| File | Workflow | Webhook path |
|---|---|---|
| `incident_router.json` | Domain Guard - incident router | `/webhook/domain-guard-incident`, `/webhook/domain-guard-thread-end`, `/webhook/domain-guard-approval` (GET), `/webhook/domain-guard-approval-confirm` (POST) |
| `check_http.json` | Domain Guard - check_http | `/webhook/domain-guard-check-http` |
| `check_dns.json` | Domain Guard - check_dns | `/webhook/domain-guard-check-dns` |
| `check_ssl.json` | Domain Guard - check_ssl | `/webhook/domain-guard-check-ssl` |
| `check_redirects.json` | Domain Guard - check_redirects | `/webhook/domain-guard-check-redirects` |
| `check_whois.json` | Domain Guard - check_whois | `/webhook/domain-guard-check-whois` |
| `scheduled_monitor.json` | Domain Guard - scheduled monitor | none (runs every hour, on the hour) |
| `expiry_sweep.json` | Domain Guard - expiry sweep | none (runs daily at 9:00) |
| `weekly_report.json` | Domain Guard - weekly report | none (runs Mondays at 8:00) |
| `failure_alerts.json` | Domain Guard - failure alerts | none (runs when another Domain Guard workflow fails) |

## Watched domains

The scheduled monitor, the expiry sweep and the weekly report all read the domain list from the **Domains** tab of the Domain Guard sheet: one domain per row under a `Domain` header, without `https://`. To add or remove a domain, edit that tab. The next run of each workflow picks it up.

## Incident lifecycle

Every alert that reaches the incident router is attached to an incident in the **Incident Log** tab, one row per incident. If the domain already has an incident that isn't `Resolved`, the alert updates that row. Otherwise the router opens a new one with an ID like `INC-20261103-ONXG`. Raia is told whether the alert is new or an update, and what happened so far. The row records the status, when it opened, the number of updates, every event type seen, and Raia's latest diagnosis. When Raia replies `STATUS: Resolved` (for example after the monitor reports `RECOVERED` and Raia verifies the site), the row is closed with `Resolved At` and `Minutes Open`. The next alert for that domain opens a new incident. Approval rows and email subjects carry the incident ID. The older **Active Incidents** and **Incidents** tabs still get one row per event as a history.

The router needs the **Incident Log** tab to exist. In a new spreadsheet, add an empty tab with that name. The first incident writes its header row.

## Weekly report

Every Monday at 8:00 (the n8n account's timezone) the weekly report emails a summary of the last 7 days: incidents opened, resolved and still open, the median time to resolve, domains with repeat incidents, approvals waiting on you, and every domain whose registration expires within 60 days. It reads the **Incident Log** and **Approvals** tabs and looks up each domain's registration (about a minute of lookups).

## Failure alerts

Every Domain Guard workflow has **Domain Guard - failure alerts** set as its error workflow (Workflow settings → Error workflow). When a run fails, for example because a Google sign-in expired or Raia is down, it emails the workflow name, the failed step, the error and a link to the failed run. The same error from the same workflow is emailed at most once every 6 hours.

## Approvals

When Raia ends with `STATUS: Awaiting Approval`, the incident router adds a row to the **Approvals** tab of the Domain Guard sheet and emails Approve and Reject links. Each link opens a confirmation page, and nothing is recorded until you confirm there, so email scanners that pre-open links can't approve anything. Confirming checks the row's token and that the row is still `Pending`, then sets `Status` to `Approved` or `Rejected` with the time and an optional note. A second confirmation, a wrong token or an unknown ID gets a "Not recorded" page. The two approval webhooks have no header auth because they are opened from a browser. The token in the link is what protects them. After you reject a request, the confirmation page says that no change will be made.

## Restoring into another n8n account

1. In n8n, create a new workflow, open the **⋯** menu, choose **Import from file**, and pick one JSON file. Repeat for each file.
2. **Reconnect the logins.** Exports contain only credential names, never secrets. Recreate these and select them on the nodes that use them:
   - **Raia account** (Raia API key): the `raia` node
   - **Gmail account** (Google sign-in): the email nodes in the incident router, the weekly report and the failure alerts
   - **Google Sheets account** (Google sign-in): every Google Sheets node (logging, approvals, incidents and the `Read domains` nodes)
   - **Header Auth account**: name `X-Domain-Guard-Key`, value = your secret. Used by the `Monitor alert` and `Raia thread end` webhooks, by `Send to incident router` in the scheduled monitor, and by the webhook of each of the five check workflows.
3. **Install the Raia node** if n8n asks (community package `n8n-nodes-raia`).
4. **Sheet tabs.** The Domain Guard spreadsheet needs these tabs: **Domains** (a `Domain` column with one domain per row), **Incident Log**, **Approvals**, **Active Incidents**, **Incidents** and **Conversations**.
5. **Failure alerts.** Workflow IDs change on import, so open each workflow's settings and pick **Domain Guard - failure alerts** as the error workflow.
6. **Publish** each workflow.
7. **Point everything at your n8n address** (replace `your-instance.app.n8n.cloud`):
   - The five Raia Functions (Check HTTP Status, Check DNS Records, Check SSL Certificate, Check Redirects, Check Domain Registration). Keep their `X-Domain-Guard-Key` header.
   - Raia's n8n webhook (onThreadEnd)
   - The `Send to incident router` URL in the scheduled monitor and in the expiry sweep
   - The approval link URLs in `Send approval request` and the form address in `Build confirm page`
8. **The monitor's and the sweep's memory starts empty** in a new account, so their first runs alert on any domain that has a problem at that moment.
