# Domain Guard: setup and reference

Domain Guard watches a list of domains and, when one breaks, has an AI agent work out why. It checks every site each hour, and every registration, DNS setup and certificate each day, comparing them with what you expect. When something changes, it opens an incident, has the agent diagnose it with live checks, and either logs the result, emails you an escalation, or emails you Approve and Reject links for a proposed fix. When the fix is a single DNS record change, approving it applies the change in Cloudflare and has the agent check the result. A weekly email summarizes everything.

It is built from [n8n](https://n8n.io) workflows (scheduling, checks, logging, email) and a [Raia](https://raia2.com) agent (diagnosis). A Google Sheet holds the domain list and all the logs.

## How it works

- **Scheduled monitor:** requests `https://<domain>` for every domain in the sheet. It alerts only when a site goes down (unreachable or HTTP 400+) or comes back up, so a site that stays down isn't reported every hour.
- **Expiry sweep:** looks up each domain's registration (RDAP) and alerts when its problems change: not registered, expired, expiring within 30 days, on hold, and so on.
- **Config sweep:** once a day, for every domain, compares DNS records (A, AAAA, CNAME, NS, MX, TXT) with the previous day's snapshot and with the expected values in the **Domains** tab, and checks certificate expiry. It alerts on `NS_MISMATCH`, `HOST_MISMATCH`, `DNS_CHANGED` (with the list of changes), `CERT_EXPIRING` (14 days or less), `MULTIPLE_SPF`, and `CONFIG_RESTORED` when earlier findings clear. Address records that rotate a few at a time behind a CDN are ignored, and only a complete replacement counts.
- **Incident router:** keeps one incident per domain in the **Incident Log** tab, updating it on repeat alerts. It sends the alert, the incident history and the domain's profile from the **Domains** tab to the Raia agent, and acts on the `STATUS:` line that ends the agent's reply.
- **Check tools:** five small webhook workflows the agent calls for live evidence. They require a shared secret header. The DNS tool also reports SPF, DMARC and the DMARC policy, with warnings for duplicate records and mail domains missing SPF or DMARC.
- **Approvals:** each link opens a confirmation page, so email scanners that pre-open links can't approve anything. Decisions are recorded in the **Approvals** tab. When the agent's proposal includes a `FIX:` line, the approval email shows the exact change, and confirming **Approve** applies it (see *Fixes* below). Any other approved change is carried out by a person.
- **Apply fix:** a sub-workflow the router calls after an approval. It checks the fix against strict rules, looks up the current record in Cloudflare, creates or updates the record, logs the before and after values and how to undo the change in the **Changes** tab, and returns the outcome to the confirmation page. After a successful change, the router sends a `FIX_APPLIED` alert so the agent re-checks the domain live.
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
| **Domains** | One domain per row, without `https://`, under these headers: `Domain`, `Purpose`, `Criticality`, `Expected Nameservers`, `Expected Host`, `Notes`. Only `Domain` is required. Separate multiple nameservers with commas. |
| **Incident Log** | Empty. The first incident writes its header row. |
| **Approvals** | Empty |
| **Active Incidents** | Empty (per-event history) |
| **Incidents** | Empty (per-event history) |
| **Conversations** | Empty (one row per finished agent conversation) |
| **Changes** | Headers `Timestamp`, `Approval ID`, `Incident ID`, `Zone`, `Action`, `Type`, `Name`, `Before`, `After`, `Record ID`, `Result`, `Message`, `Undo` |

Copy the spreadsheet ID from its URL (the long string between `/d/` and `/edit`).

### 2. Set up n8n

1. **Install the Raia node:** community package `n8n-nodes-raia`.
2. **Create five credentials:**
   - **Google Sheets account** (Google sign-in)
   - **Gmail account** (Google sign-in)
   - **Raia account** (your Raia API key)
   - **Header Auth account:** name `X-Domain-Guard-Key`, value = a long random secret you make up. Keep a copy, because Raia needs it too.
   - **Cloudflare account** (Cloudflare API): an API token with **Zone · DNS · Edit** and **Zone · Zone · Read** for your zones. If you don't want automatic fixes yet, enter any placeholder value such as `NOT-SET`. Every approved fix then fails safely with a "not applied" message, and you can paste a real token later.
3. **Import the workflows.** For each file in `workflows/`, create a workflow, open the **⋯** menu, choose **Import from file**, and select the credential on any node that asks for one. Import and publish `apply_fix.json` before the incident router, then open the router's **Apply fix** node and select **Domain Guard - apply fix** (workflow IDs change on import).
4. **Replace the placeholders:**

   | Placeholder | Replace with | Where |
   |---|---|---|
   | `YOUR_GOOGLE_SHEET_ID` | Your spreadsheet ID | Every Google Sheets node (or pick the sheet from the node's list) |
   | `you@example.com` | The address that should get alerts, approvals and reports | Email nodes in the incident router, weekly report and failure alerts |
   | `your-instance.app.n8n.cloud` | Your n8n host | `Send to incident router` in the monitor, the expiry sweep and the config sweep; `Send approval request` and `Build confirm page` in the incident router |

5. **Set the error workflow.** Workflow IDs change on import, so open each workflow's **Settings** and set **Error workflow** to **Domain Guard - failure alerts**.
6. **Publish** every workflow.

### 3. Set up the Raia agent

1. **Instructions:** paste [`raia/agent_instructions.md`](../raia/agent_instructions.md) into the agent's instructions (Train tab).
2. **Knowledge:** upload the four files in [`raia/knowledge/`](../raia/knowledge). Fill in the placeholders in `03_company_context_and_tool_guide.md` (registrars, providers, critical domains, contacts) with your own details. The agent uses them to judge severity.
3. **Functions:** create one Function for each entry in [`raia/functions.json`](../raia/functions.json): name, internal name, instructions, webhook URL, method `POST` and parameters. Replace `your-instance.app.n8n.cloud` with your host, and add the header `X-Domain-Guard-Key` with your secret as the value.
4. **Webhook skill:** add a webhook that triggers **when a dialog ends**, calling `https://<your n8n host>/webhook/domain-guard-thread-end` with the same `X-Domain-Guard-Key` header. This logs each conversation to the **Conversations** tab.

### 4. Check it works

- Ask the agent "Is example.com up right now?". It should call the HTTP tool and end with `STATUS: Resolved`.
- Ask it something off-topic. It should reply with a single line saying it only handles domain issues, with no `STATUS:` line.
- A request to a check tool without the header should get `403`:

  ```bash
  curl -s -o /dev/null -w "%{http_code}\n" -X POST -H 'Content-Type: application/json' -d '{"domain":"example.com"}' https://your-instance.app.n8n.cloud/webhook/domain-guard-check-http
  ```

The monitor and the sweeps start with empty memory. Their first runs alert on every domain that has a problem at that moment, and the config sweep's first run records the DNS snapshot it compares against from then on.

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

## Fixes

When the agent is confident about one DNS record change, it ends its proposal with a line like this, directly before `STATUS: Awaiting Approval`:

```
FIX: {"action":"create","type":"TXT","name":"_dmarc.example.com","content":"v=DMARC1; p=none"}
```

The approval email then says exactly what **Approve** will change. After you confirm, the apply-fix workflow enforces these rules regardless of what the agent wrote:

- Only `create` and `update`, never delete.
- Only A, AAAA, CNAME, MX and TXT records, never NS, registrar or transfer changes.
- The record must be the incident's domain or a name under it.
- An update must match exactly one existing record (the agent adds `old_content` when several records share a name).
- If the record already has the new value, nothing is written.

Every attempt is logged in the **Changes** tab with the result and, for applied changes, the old value and how to undo it. The confirmation page shows the same outcome, and the **Approvals** row gets a `Fix Result`. The agent is told never to invent IP addresses, hosts or mail servers, and to leave the `FIX:` line out when it isn't sure of the exact value.

## Day-to-day use

- **Add or remove a domain:** edit the **Domains** tab. The next run picks it up.
- **Describe a domain:** fill in `Purpose` and `Criticality` so the agent can judge business impact, and keep `Expected Nameservers` and `Expected Host` current. After a planned DNS move, update them, or the config sweep reports a mismatch.
- **Connect Cloudflare:** open the **Cloudflare account** credential in n8n and paste a real token. One token covers every zone the Cloudflare login can access. Zones in another login need their own token, which this version doesn't support yet.
- **Undo a fix:** follow the `Undo` column for that change in the **Changes** tab.
- **Change the secret:** update the n8n **Header Auth account** credential, all five Raia Functions, and the Raia webhook skill. Anything left on the old value gets `403`.
- **Incident IDs** look like `INC-20261103-ONXG`. They appear in the Incident Log, the Approvals tab and email subjects. An incident closes when the agent replies `STATUS: Resolved`, which records `Resolved At` and `Minutes Open`. The next alert for that domain opens a new incident.

## Limitations

- Automatic fixes cover single DNS record creates and updates in one Cloudflare login. Everything else (deletions, nameservers, registrar, hosting, certificates) is done by a person.
- Nothing is applied without your approval. There is no auto-approve for low-risk changes yet.
- `03_company_context_and_tool_guide.md` is a template. It names some tools that aren't built here (`get_domain_status`, `get_change_history` and the remediation tools). The agent instructions list the five tools that are built and tell the agent the others don't exist.
- Change history is one day deep: the config sweep compares with the previous day's snapshot, so it can say what changed but not who changed it.
- The monitor checks only the HTTPS home page of each domain. To watch a subdomain, add it as its own row in the **Domains** tab.
- Schedules run in the n8n instance's timezone.
