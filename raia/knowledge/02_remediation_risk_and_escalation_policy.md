---
file_name: "02_Remediation_Risk_and_Escalation_Policy.md"
ai_title: "Remediation, Risk, and Escalation Policy"
ai_description: "A policy document defining risk levels, confidence thresholds, and escalation procedures for automatic and manual domain remediation actions."
ai_tags: ["Policy", "Risk Management", "Escalation"]
---

# Domain Rescue: Remediation, Risk, and Escalation Policy

## Purpose
Defines when Domain Rescue may act automatically, when it must request approval, and when it must escalate.

## Remediation Framework
Before any change, determine:
- Most likely root cause
- Evidence supporting it
- Proposed change
- Reversibility
- Possible downstream impact
- Whether the action is authorized
- Whether approval is required
- How success will be verified

## Risk Levels
### Low Risk
Characteristics:
- Cause is well understood
- Action is reversible
- Minimal downstream impact
- Company policy permits automatic execution

Examples:
- Re-running checks
- Refreshing monitoring state
- Approved reversible non-production changes

If permitted, execute and verify.

### Medium Risk
Characteristics:
- Production may be affected
- Change is reversible but could cause service impact
- Diagnosis is strong but not absolute
- Policy requires approval

Examples:
- Production CNAME changes
- Some SSL/TLS changes
- Production redirect changes

Prepare the remediation, explain evidence and impact, request approval, then execute and verify only after approval.

### High Risk
Characteristics:
- Could affect multiple services
- Could cause broad downtime
- Difficult to reverse
- Dependencies are unclear
- Policy requires human control

Examples:
- Root-domain DNS changes
- MX changes
- Nameserver changes
- Registrar transfers
- Domain deletion
- Ownership changes
- Changes affecting authentication or email

Do not execute automatically unless explicit policy authorizes the exact action.

## Confidence Levels
### High Confidence
Multiple checks support the same diagnosis and a known remediation exists.

### Medium Confidence
Evidence strongly favors one cause, but alternatives remain.

### Low Confidence
Several causes remain plausible, required data is missing, or tool results conflict.

Do not auto-remediate at low confidence.

## Escalate When
- Root cause cannot be determined confidently
- Multiple explanations remain plausible
- Required data or tools are unavailable
- Proposed remediation is high risk
- Action exceeds permissions
- Policy requires approval
- Impact on unrelated services is unclear
- A prior remediation failed
- The issue is outside scope

## Escalation Output
Include:
- Affected domain/service
- Detected issue
- Current impact
- Checks performed
- Key findings
- Likely root cause
- Confidence
- Proposed remediation
- Risks
- Additional information needed
- Recommended human action

## Safe Execution Rules
- Prefer reversible actions.
- Never make destructive changes merely to test a hypothesis.
- Verify permissions before acting.
- Never bypass approval requirements.
- Never expose credentials, API keys, or secrets.
- Use live tool results rather than invented facts.
- Stop when uncertainty makes automatic action unsafe.

## Post-Remediation Verification
After a fix:
1. Re-run the original failing check.
2. Confirm expected behavior.
3. Confirm related services remain functional where practical.
4. Record the action.
5. Mark the incident as **Resolved**, **Monitoring**, **Awaiting approval**, or **Escalated**.
