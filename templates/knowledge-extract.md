---
title: "{{title}}"
extracted: "{{date}}"
source: "{{source_type}}"
source_ref: "{{url_or_reference}}"
confidence: "{{high|medium|low}}"
verified: false
tags:
  - knowledge
  - "{{domain}}"
---

# {{title}}

> One-line summary of the extracted knowledge.

## Source

- **Type**: {{incident|conversation|documentation|observation|external}}
- **Reference**: {{link_or_description}}
- **Date**: {{when_this_occurred}}
- **Extracted by**: {{human|cortex}}

## Context

What situation or problem led to this knowledge being captured?

## Key Insight

The core learning or knowledge being preserved:

> **TL;DR**: State the insight in one clear sentence.

Expanded explanation with relevant details.

## Evidence

What supports this knowledge?

```
# Logs, metrics, or other evidence
```

## Application

When and how should this knowledge be applied?

- **Scenario**: When you see X, consider Y
- **Caveats**: This may not apply when Z

## Related Knowledge

- [[knowledge/{{related}}|Related Insight]]
- [[components/{{component}}|Relevant Component]]

## Verification

- [ ] Verified by human review
- [ ] Tested in practice
- [ ] Cross-referenced with other sources

---

*Confidence: {{confidence}}*
*Last verified: {{date}}*

{{#if cortex_generated}}
---
**Cortex Metadata**
- Extraction job: {{job_id}}
- Model: {{model_version}}
- Processing time: {{duration}}
{{/if}}
