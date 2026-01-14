---
title: "Documentation Curation Workflow"
created: "2025-01-13"
updated: "2025-01-13"
status: design
owner: cortex
tags:
  - meta
  - curation
  - workflow
---

# Documentation Curation Workflow

> How Cortex manages its own documentation.

## Overview

This document defines the workflow by which Cortex curates, organizes, and maintains the documentation in this repository. The goal is **self-managing documentation** that stays current, connected, and useful.

## Design Principles

1. **Inbox-First**: All new content enters through `_inbox/`. No exceptions.
2. **Suggest, Don't Act** (Phase 1): Cortex proposes changes; humans approve.
3. **Earn Trust**: Write access granted incrementally as the workflow proves reliable.
4. **Preserve History**: Never delete; archive and link to replacements.
5. **Human-Readable Output**: All curation actions must be explainable.

## Workflow Stages

### Stage 1: Triage

**Trigger**: New document appears in `_inbox/`

**Actions**:
1. Parse document content and metadata
2. Classify document type (component, runbook, decision, knowledge, other)
3. Extract key entities (services, namespaces, people, concepts)
4. Suggest destination folder
5. Suggest tags
6. Identify potential duplicate or related docs

**Output**: Triage report with recommendations

### Stage 2: Enrich

**Trigger**: Document moved out of `_inbox/` (manually or auto-approved)

**Actions**:
1. Add backlinks to related documents
2. Update `_index.md` with new entry
3. Suggest additional tags based on content analysis
4. Link to relevant external resources (dashboards, repos, etc.)
5. Validate template compliance; suggest missing fields

**Output**: Enrichment PR or commit

### Stage 3: Health Check

**Trigger**: Scheduled (weekly) or on-demand

**Actions**:
1. Scan for broken internal links
2. Identify orphaned documents (no incoming links)
3. Flag stale documents (no updates in configurable period)
4. Check for outdated references (deprecated APIs, removed services)
5. Validate frontmatter schema

**Output**: Health report with issues and suggested fixes

### Stage 4: Consolidate

**Trigger**: Health check identifies candidates, or manual request

**Actions**:
1. Identify documents with high semantic similarity
2. Propose merge candidates
3. Generate consolidated document draft
4. Update all incoming links to point to consolidated doc
5. Archive originals with redirect notice

**Output**: Consolidation proposal for review

### Stage 5: Report

**Trigger**: Scheduled (weekly) or on-demand

**Actions**:
1. Generate vault statistics (doc count, by type, by age)
2. Summarize recent activity (new docs, updates, archives)
3. Highlight attention-needed items from health check
4. Track curation workflow effectiveness over time

**Output**: Weekly digest (could be posted to Slack, dashboard, etc.)

## Implementation

### Component Design

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docs-curator
  namespace: cortex-system
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: curator
          image: cortex/docs-curator:latest
          env:
            - name: GITHUB_REPO
              value: "your-org/cortex-docs"
            - name: VAULT_PATH
              value: "vault/"
            - name: CURATION_MODE
              value: "suggest"  # suggest | auto-minor | auto-full
            - name: SCHEDULE
              value: "0 6 * * 1"  # Weekly Monday 6am
```

### Integration Points

- **GitHub**: PR creation for suggestions, direct commits when trusted
- **Knowledge Graph**: Query for entity relationships, semantic similarity
- **Slack/Notifications**: Post reports and action items
- **Obsidian**: Respect `.obsidian/` config, use compatible markdown

### Trust Levels

| Level | Name | Permissions |
|-------|------|-------------|
| 0 | Observer | Read-only, generate reports |
| 1 | Suggester | Create PRs, no direct commits |
| 2 | Minor Editor | Direct commits to `_inbox/`, `_index.md`, tags |
| 3 | Full Editor | Direct commits anywhere except `meta/` |
| 4 | Self-Governing | Can modify curation rules in `meta/` |

**Current Level**: 0 (Observer)

## Curation Rules

### Staleness Thresholds

| Document Type | Stale After | Action |
|---------------|-------------|--------|
| Component | 90 days | Flag for review |
| Runbook | 60 days | Flag + suggest test |
| Decision | 180 days | Flag if status != deprecated |
| Knowledge | 120 days | Flag for verification |

### Auto-Tagging Rules

```yaml
rules:
  - pattern: "kubernetes|k8s|kubectl|pod|deployment"
    add_tag: "kubernetes"
  - pattern: "incident|outage|alert|pagerduty"
    add_tag: "incident"
  - pattern: "TODO|FIXME|WIP"
    add_tag: "draft"
```

### Consolidation Criteria

- Semantic similarity > 0.85
- Same primary topic/entity
- Neither document updated in 30+ days
- Human approval required

## Metrics

Track effectiveness over time:

- **Inbox dwell time**: How long docs stay unsorted
- **Link health**: % of valid internal links
- **Coverage**: % of components with documentation
- **Freshness**: Average age of documents by type
- **Curation accuracy**: % of suggestions accepted by humans

## Future Considerations

- **Obsidian Plugin**: Surface curation suggestions directly in the editor
- **Chat Interface**: "Hey Cortex, what docs need attention this week?"
- **Auto-Generation**: Create stub docs for new components detected in cluster
- **Cross-Repo**: Extend curation to inline docs in other cortex-* repos

---

*This workflow is itself subject to curation. Meta, right?*
