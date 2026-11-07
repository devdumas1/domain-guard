---
file_name: "03_Company_Context_and_Tool_Guide.md"
ai_title: "Company Context and Tool Guide"
ai_description: "A guide detailing company-specific standards for registrars, DNS providers, and critical domain inventory, along with tool usage principles."
ai_tags: ["Infrastructure", "Tools", "Vendors"]
---

# Domain Rescue: Company Context and Tool Guide

## Purpose
Defines organization-specific configuration and how Domain Rescue should use live tools. Replace placeholders with real company information.

## Company Standards
### Approved Registrars
- [Registrar 1]
- [Registrar 2]

### Approved DNS Providers
- [DNS Provider 1]
- [DNS Provider 2]

### Approved Hosting/CDN Providers
- [Provider 1]
- [Provider 2]

### Approved Certificate Providers
- [Provider 1]
- [Provider 2]

### Critical Domains
| Domain | Business Purpose | Criticality | Owner | Expected Host | Auto-Remediation Allowed |
|---|---|---|---|---|---|
| example.com | Primary website | Critical | Web Team | [host] | No |
| portal.example.com | Customer portal | Critical | Platform Team | [host] | No |
| promo.example.com | Marketing site | Low | Marketing | [host] | Limited |

## Recommended Domain Inventory Fields
- domain
- business purpose
- owner/team
- criticality
- registrar
- DNS provider
- expected nameservers
- expected host/target
- production or non-production
- auto-renew status
- allowed auto-remediation
- escalation contact
- known dependencies
- notes

Business context should affect severity and remediation decisions.

## Tool Selection Guide
### `check_dns`
Use for:
- Resolution failures
- Suspected wrong destination
- Record mismatch
- Nameserver issues

### `check_ssl`
Use for:
- Certificate warnings
- HTTPS failures
- Suspected expiration

### `check_http`
Use for:
- Availability checks
- HTTP status investigation
- Redirect inspection

### `get_domain_status`
Use for:
- Expiration
- Registrar status
- Registration status
- Nameserver information

### `get_change_history`
Use for:
- Incidents following configuration changes
- Differences between live and expected configuration
- Potential rollback decisions

### Remediation tools
Examples:
- `update_dns_record`
- `rollback_dns_record`
- `renew_certificate`
- `create_incident`
- `notify_admin`

Use remediation tools only according to risk policy and permissions.

## Tool-Use Principles
- Use live tools for current technical facts.
- Use company documents for expected configuration and policy.
- Do not assume live state is correct just because it is current.
- Do not assume documentation is current when evidence suggests otherwise.
- Investigate conflicts between live data and documentation.
- Call only tools relevant to the incident.
- Let one result guide the next diagnostic step.

## Severity Framework
### Critical
Primary website, customer portal, authentication, email, or broad nameserver failure.

### High
Important production subdomain outage, customer-facing certificate failure, major redirect/DNS issue.

### Medium
Degraded service, non-critical production issue, approaching expiration.

### Low
Development/test issue, old marketing domain, informational configuration difference.

## Escalation Contacts
| Area | Owner / Team | Escalation Method |
|---|---|---|
| DNS | [Team] | [Method] |
| Hosting | [Team] | [Method] |
| Security | [Team] | [Method] |
| Registrar | [Team] | [Method] |
| Web Applications | [Team] | [Method] |
