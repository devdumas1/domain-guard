# Domain Guard

Domain Guard watches a list of domains and, when one breaks, has an AI agent work out why. It checks every site each hour and every registration each day. When something changes, it opens an incident, has the agent diagnose it with live checks, and either logs the result, emails you an escalation, or emails you Approve and Reject links for a proposed fix. A weekly email summarizes everything.

It is built from [n8n](https://n8n.io) workflows (scheduling, checks, logging, email) and a [Raia](https://raia2.com) agent (diagnosis). A Google Sheet holds the domain list and all the logs.

## How it works

- **Scheduled monitor:** requests `https://<domain>` for every domain in the sheet. It alerts only when a site goes down (unreachable or HTTP 400+) or comes back up, so a site that stays down isn't reported every hour.
- **Expiry sweep:** looks up each domain's registration (RDAP) and alerts when its problems change: not registered, expired, expiring within 30 days, on hold, and so on.
- **Incident router:** keeps one incident per domain in the **Incident Log** tab, updating it on repeat alerts. It sends the alert and the incident history to the Raia agent and acts on the `STATUS:` line that ends the agent's reply.
- **Check tools:** five small webhook workflows the agent calls for live evidence. They require a shared secret header.
- **Approvals:** each link opens a confirmation page, so email scanners that pre-open links can't approve anything. Decisions are recorded in the **Approvals** tab. Domain Guard has no automated fixes, so an approved change is carried out by a person.
- **Weekly report:** incidents opened, resolved and still open, median time to resolve, repeat offenders, pending approvals, and domains expiring within 60 days.
- **Failure alerts:** set as the error workflow on every other workflow. It emails the failed workflow, step and error, at most once every 6 hours per error.

All personal details were replaced with placeholders, and no secrets are included. Credentials appear only by name, and you create your own.

## Requirements

- An n8n instance (n8n Cloud or self-hosted) that can install community nodes
- A Raia account and agent
- A Google account for Sheets and Gmail

## Setup

### 1. Create the Google Sheet

Make a spreadsheet with these tabs:

| Tab | Contents |
|---|---|
| **Domains** | A `Domain` header in A1, then one domain per row, without `https://` |
| **Incident Log** | Empty. The first incident writes its header row. |
| **Approvals** | Empty |
| **Active Incidents** | Empty (per-event history) |
| **Incidents** | Empty (per-event history) |
| **Conversations** | Empty (one row per finished agent conversation) |

Copy the spreadsheet ID from its URL (the long string between `/d/` and `/edit`).

### 2. Set up n8n

1. **Install the Raia node:** community package `n8n-nodes-raia`.
2. **Create four credentials:**
   - **Google Sheets account** (Google sign-in)
   - **Gmail account** (Google sign-in)
   - **Raia account** (your Raia API key)
   - **Header Auth account:** name `X-Domain-Guard-Key`, value = a long random secret you make up. Keep a copy, because Raia needs it too.
3. **Import the workflows.** For each file in `workflows/`, create a workflow, open the **⋯** menu, choose **Import from file**, and select the credential on any node that asks for one.
4. **Replace the placeholders:**

   | Placeholder | Replace with | Where |
   |---|---|---|
   | `YOUR_GOOGLE_SHEET_ID` | Your spreadsheet ID | Every Google Sheets node (or pick the sheet from the node's list) |
   | `you@example.com` | The address that should get alerts, approvals and reports | Email nodes in the incident router, weekly report and failure alerts |
   | `your-instance.app.n8n.cloud` | Your n8n host | `Send to incident router` in the monitor and the sweep; `Send approval request` and `Build confirm page` in the incident router |

5. **Set the error workflow.** Workflow IDs change on import, so open each workflow's **Settings** and set **Error workflow** to **Domain Guard - failure alerts**.
6. **Publish** every workflow.

### 3. Set up the Raia agent

1. **Instructions:** paste [`raia/agent_instructions.md`](raia/agent_instructions.md) into the agent's instructions (Train tab).
2. **Knowledge:** upload the four files in [`raia/knowledge/`](raia/knowledge). Fill in the placeholders in `03_company_context_and_tool_guide.md` (registrars, providers, critical domains, contacts) with your own details. The agent uses them to judge severity.
3. **Functions:** create one Function for each entry in [`raia/functions.json`](raia/functions.json): name, internal name, instructions, webhook URL, method `POST` and parameters. Replace `your-instance.app.n8n.cloud` with your host, and add the header `X-Domain-Guard-Key` with your secret as the value.
4. **Webhook skill:** add a webhook that triggers **when a dialog ends**, calling `https://<your n8n host>/webhook/domain-guard-thread-end` with the same `X-Domain-Guard-Key` header. This logs each conversation to the **Conversations** tab.

### 4. Check it works

- Ask the agent "Is example.com up right now?". It should call the HTTP tool and end with `STATUS: Resolved`.
- Ask it something off-topic. It should reply with a single line saying it only handles domain issues, with no `STATUS:` line.
- A request to a check tool without the header should get `403`:

  ```bash
  curl -s -o /dev/null -w "%{http_code}\n" -X POST -H 'Content-Type: application/json' -d '{"domain":"example.com"}' https://your-instance.app.n8n.cloud/webhook/domain-guard-check-http
  ```

The monitor and the sweep start with empty memory, so their first runs alert on every domain that has a problem at that moment.

## Webhooks

| Workflow | Path | Protected by |
|---|---|---|
| incident router | `POST /webhook/domain-guard-incident` | `X-Domain-Guard-Key` |
| incident router | `POST /webhook/domain-guard-thread-end` | `X-Domain-Guard-Key` |
| incident router | `GET /webhook/domain-guard-approval` | Per-request token in the link |
| incident router | `POST /webhook/domain-guard-approval-confirm` | Per-request token in the form |
| check_http | `POST /webhook/domain-guard-check-http` | `X-Domain-Guard-Key` |
| check_dns | `POST /webhook/domain-guard-check-dns` | `X-Domain-Guard-Key` |
| check_ssl | `POST /webhook/domain-guard-check-ssl` | `X-Domain-Guard-Key` |
| check_redirects | `POST /webhook/domain-guard-check-redirects` | `X-Domain-Guard-Key` |
| check_whois | `POST /webhook/domain-guard-check-whois` | `X-Domain-Guard-Key` |

The two approval webhooks are opened from a browser, so they can't send a header. Instead, each approval has its own random token. A confirmation is recorded only if the token matches and the approval is still `Pending`.

## Day-to-day use

- **Add or remove a domain:** edit the **Domains** tab. The next run picks it up.
- **Change the secret:** update the n8n **Header Auth account** credential, all five Raia Functions, and the Raia webhook skill. Anything left on the old value gets `403`.
- **Incident IDs** look like `INC-20261103-ONXG`. They appear in the Incident Log, the Approvals tab and email subjects. An incident closes when the agent replies `STATUS: Resolved`, which records `Resolved At` and `Minutes Open`. The next alert for that domain opens a new incident.

## Limitations

- There are no automated fixes. The agent diagnoses and proposes, and people make changes.
- `03_company_context_and_tool_guide.md` is a template. It names some tools that aren't built here (`get_domain_status`, `get_change_history` and the remediation tools). The agent works with the five check tools that are built.
- The monitor checks only the HTTPS home page of each domain.
- Schedules run in the n8n instance's timezone.
