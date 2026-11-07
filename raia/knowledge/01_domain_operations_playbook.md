---
file_name: "01_Domain_Operations_Playbook.md"
ai_title: "Domain Operations Playbook"
ai_description: "A technical playbook for diagnosing and troubleshooting domain-related incidents, including DNS, SSL/TLS, and HTTP availability."
ai_tags: ["DNS", "Troubleshooting", "SSL/TLS"]
---

# Domain Rescue: Domain Operations Playbook

## Purpose
Core technical knowledge for diagnosing domain and website incidents. Use live tools/APIs for current facts; use this guide to interpret those facts and choose the next check.

## DNS Troubleshooting
### Common record types
- **A**: hostname to IPv4
- **AAAA**: hostname to IPv6
- **CNAME**: hostname to hostname
- **MX**: mail servers
- **TXT**: verification, SPF, DKIM, and service metadata
- **NS**: authoritative nameservers

### Common problems
- Missing or incorrect records
- Incorrect nameservers
- Conflicting records
- DNS propagation delays
- Root-domain and subdomain misconfiguration

### Investigation
1. Confirm whether the domain resolves.
2. Retrieve current DNS records.
3. Compare with expected company configuration.
4. Check nameservers.
5. Check recent changes when available.
6. Identify dependent services.
7. Determine whether the issue affects one subdomain or the whole domain.

DNS changes may affect websites, APIs, authentication, and email. Do not assume a DNS change is isolated.

## SSL/TLS Troubleshooting
### Common problems
- Expired certificate
- Hostname mismatch
- Missing intermediate certificate
- Invalid chain
- HTTPS configuration error
- Missing subdomain coverage

### Investigation
1. Check whether HTTPS is reachable.
2. Inspect expiration.
3. Confirm hostname coverage.
4. Inspect issuer and chain.
5. Determine whether HTTP works while HTTPS fails.
6. Check whether the issue followed a hosting, DNS, or certificate change.

Do not assume every HTTPS error is caused by the certificate itself.

## HTTP and Website Availability
### Common status codes
- **200**: success
- **301/302**: redirect
- **400**: bad request
- **401**: authentication required
- **403**: forbidden
- **404**: not found
- **500**: internal server error
- **502**: bad gateway
- **503**: service unavailable
- **504**: gateway timeout

### Investigation
1. Check DNS.
2. Check HTTP and HTTPS.
3. Inspect redirects.
4. Check SSL/TLS.
5. Confirm the server is reachable.
6. If DNS and SSL are healthy but 5xx errors remain, investigate hosting, CDN, proxy, or application infrastructure.
7. Review recent changes.

## Domain and Registrar Operations
### Common problems
- Expiration risk
- Auto-renew disabled
- Registrar lock/transfer issue
- Unexpected nameserver change
- Ownership/contact problem
- Domain suspension

### Investigation
1. Retrieve domain status and expiration.
2. Confirm registrar.
3. Confirm nameservers.
4. Check auto-renew when available.
5. Determine business criticality.
6. Check whether the domain is actively used.
7. Follow company renewal policy.

Treat nameserver changes, registrar transfers, domain deletion, ownership changes, and root-domain changes as high risk unless company policy explicitly says otherwise.

## Diagnostic Decision Guide
### Website does not resolve
Check DNS resolution, nameservers, registrar status, current records, and recent DNS changes.

### Website resolves but HTTPS fails
Check certificate expiration, hostname coverage, chain, HTTPS configuration, and CDN/proxy settings.

### DNS and SSL are healthy but site returns 5xx
Check hosting, reverse proxy/CDN, recent deployments, origin server, and application infrastructure.

### Redirect is incorrect
Check redirect chain, DNS target, hosting/CDN redirect rules, expected destination, and recent changes.

### Multiple causes remain plausible
Do not force a diagnosis. Continue investigating or escalate.

## Verification After Remediation
After any change:
1. Re-run the failed check.
2. Confirm DNS where relevant.
3. Confirm HTTPS where relevant.
4. Confirm expected HTTP response.
5. Confirm redirects.
6. Check that unrelated critical services were not affected.
7. Record the change and result.

A fix is not complete until verification succeeds.
