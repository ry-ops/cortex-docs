---
title: "{{title}}"
decision_id: "ADR-{{id}}"
created: "{{date}}"
updated: "{{date}}"
status: "{{proposed|accepted|deprecated|superseded}}"
deciders:
  - "{{name}}"
tags:
  - decision
  - architecture
---

# ADR-{{id}}: {{title}}

## Status

**{{status}}**

{{#if superseded_by}}
Superseded by: [[architecture/{{superseded_by}}|ADR-XXX]]
{{/if}}

## Context

What is the issue that we're seeing that is motivating this decision or change?

Describe the forces at play:
- Technical constraints
- Business requirements
- Team capabilities
- Timeline pressures

## Decision

What is the change that we're proposing and/or doing?

Be specific and actionable.

## Alternatives Considered

### Option A: {{name}}

**Pros**:
-

**Cons**:
-

### Option B: {{name}}

**Pros**:
-

**Cons**:
-

### Option C: Do Nothing

**Pros**:
- No effort required

**Cons**:
- {{existing_problem_persists}}

## Consequences

What becomes easier or more difficult to do because of this change?

### Positive
-

### Negative
-

### Neutral
-

## Implementation

High-level implementation plan:

1.
2.
3.

## References

- [[{{related_doc}}|Related Documentation]]
- External Link: {{url}}

---

*Decision made: {{date}}*
*Review date: {{date + 6 months}}*
