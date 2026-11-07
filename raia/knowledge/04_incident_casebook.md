---
file_name: "04_Incident_Casebook.md"
ai_title: "Domain Rescue Incident Casebook"
ai_description: "A collection of incident patterns and case studies illustrating the diagnosis and remediation of common domain and website issues."
ai_tags: ["Case Studies", "Incidents", "Troubleshooting"]
---

# Domain Rescue: Incident Casebook

## Purpose
Examples showing how Domain Rescue should move from symptom to diagnosis, remediation, and escalation. These are patterns, not hardcoded rules; live evidence takes priority.

## Case 1: Missing CNAME
**Symptom:** `www.example.com` cannot be reached.

**Investigation:**
- DNS lookup fails
- Registration is active
- Nameservers are correct
- Expected CNAME is missing
- Change history shows recent deletion

**Diagnosis:** Missing CNAME.

**Remediation:** Restore the approved target if authorized.

**Verification:** Confirm DNS, HTTPS, and page load.

**Outcome:** Auto-resolve only if authorized and low risk; otherwise request approval.

## Case 2: Expired Certificate
**Symptom:** Users receive a certificate warning.

**Investigation:**
- DNS resolves
- HTTP responds
- HTTPS certificate is expired

**Diagnosis:** Expired SSL/TLS certificate.

**Remediation:** Renew or replace through the approved process.

**Verification:** Confirm new expiration, hostname coverage, and working HTTPS.

## Case 3: HTTP 503
**Symptom:** Website returns 503.

**Investigation:**
- DNS healthy
- SSL valid
- Request reaches the service
- Origin/hosting is unavailable

**Diagnosis:** Likely hosting or application infrastructure failure.

**Remediation:** Do not change DNS simply because the site is down. Escalate unless an approved recovery action exists.

## Case 4: Unexpected Nameserver Change
**Symptom:** Multiple services stop resolving.

**Investigation:**
- Nameservers differ from company standard
- Change was not expected
- Several services are affected

**Diagnosis:** Unexpected authoritative nameserver change.

**Outcome:** Treat as high risk/security-sensitive. Gather evidence and escalate immediately.

## Case 5: Redirect Loop
**Symptom:** Browser reports too many redirects.

**Investigation:**
- DNS resolves
- SSL valid
- HTTP redirects to HTTPS
- HTTPS redirects back to HTTP

**Diagnosis:** Conflicting redirect rules.

**Remediation:** Identify the layer responsible and remove the conflicting rule according to approved configuration.

## Case 6: Critical Domain Nearing Expiration
**Symptom:** Domain expires in 10 days.

**Investigation:**
- Domain is active
- Customer-facing service depends on it
- Auto-renew is disabled

**Diagnosis:** Operational expiration risk.

**Outcome:** Follow renewal policy. Auto-renew only if explicitly authorized; otherwise escalate.

## Case 7: Retired Marketing Domain Nearing Expiration
**Symptom:** Domain expires in 10 days.

**Investigation:**
- No active website
- No email records
- Inventory marks it for retirement

**Diagnosis:** No immediate outage risk.

**Outcome:** Do not automatically renew. Request owner confirmation if policy requires it.

## Case 8: Conflicting Evidence
**Symptom:** Intermittent website failures.

**Investigation:**
- DNS healthy
- SSL healthy
- HTTP checks alternate between success and failure
- No obvious configuration change

**Diagnosis:** Root cause unconfirmed.

**Outcome:** Continue targeted investigation or escalate. Do not invent a cause or make speculative production changes.

## Case 9: Live Configuration Conflicts With Documentation
**Symptom:** CNAME differs from documented standard.

**Investigation:**
- Current CNAME points to a valid live service
- Documentation lists an older target
- No incident is occurring

**Diagnosis:** Configuration/documentation mismatch.

**Outcome:** Do not overwrite the live record automatically. Check history or request confirmation.

## Case 10: Failed Remediation
**Symptom:** Approved fix was applied but the site remains unavailable.

**Investigation:**
- Intended change succeeded
- Verification still fails

**Diagnosis:** Original diagnosis was incomplete or another issue remains.

**Outcome:** Do not repeat the same fix. Restart diagnosis from the new state or escalate.

## Preferred Incident Output
### Incident
- Affected domain:
- Detected issue:
- Severity:

### Investigation
- Checks performed:
- Relevant findings:

### Diagnosis
- Likely root cause:
- Supporting evidence:
- Confidence:

### Remediation
- Proposed action:
- Risk:
- Approval required:
- Action taken:

### Verification
- Checks performed:
- Current status:

### Next Step
- Resolved
- Monitoring
- Awaiting approval
- Escalated
