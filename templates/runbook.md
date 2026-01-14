---
title: "{{title}}"
runbook_id: "RB-{{id}}"
created: "{{date}}"
updated: "{{date}}"
severity: "{{low|medium|high|critical}}"
status: active
tags:
  - runbook
  - operations
---

# {{title}}

> One-line description of when to use this runbook.

## Trigger Conditions

When should this runbook be executed?

- Alert: `{{alert_name}}`
- Symptom: What the operator observes
- Threshold: Specific metrics or conditions

## Impact

What happens if this isn't addressed?

- **User Impact**:
- **System Impact**:
- **Data Impact**:

## Prerequisites

- [ ] Access to: {{system/tool}}
- [ ] Permissions: {{required_permissions}}
- [ ] Knowledge of: {{relevant_concepts}}

## Procedure

### 1. Assess

```bash
# Commands to understand the current state
kubectl get pods -n {{namespace}}
```

**Expected**: What you should see if this is the right runbook.

### 2. Mitigate

```bash
# Commands to stop the bleeding
```

**Verify**: How to confirm mitigation worked.

### 3. Resolve

```bash
# Commands to fully resolve the issue
```

**Verify**: How to confirm resolution.

### 4. Cleanup

```bash
# Any cleanup steps needed
```

## Rollback

If the procedure makes things worse:

```bash
# Rollback commands
```

## Escalation

If this runbook doesn't resolve the issue:

1. **First escalation**: {{who/what}}
2. **Second escalation**: {{who/what}}

## Post-Incident

- [ ] Update this runbook if procedures changed
- [ ] Create incident report if severity >= high
- [ ] Review for automation opportunities

## Related

- [[components/{{component}}|Component Doc]]
- [[operations/{{related_runbook}}|Related Runbook]]

---

*Last tested: {{date}}*
*Test result: {{pass|fail}}*
