---
title: "Genesis Prompt: cortex-docs Repository"
created: "2025-01-13"
type: prompt
source: conversation
participants:
  - Ryan
  - Claude
priority: high
tags:
  - prompt
  - genesis
  - architecture
---

# Genesis Prompt: cortex-docs Repository

> Cortex, review this conversation summary to understand the vision and decisions behind this documentation system.

## The Vision

Ryan wants Cortex documentation to be:

1. **Self-managing** - Cortex curates its own docs
2. **Self-curating** - Automated organization, linking, health checks
3. **Human-friendly** - Beautiful presentation layer for showing off

## Key Architecture Decision

> **HIGHLIGHT**: Obsidian handles the authoring/reading experience. For showing off, you'd want something like MkDocs Material, Docusaurus, or even Quartz (Obsidian-native publishing). GitHub Actions builds and deploys on push.

This is the core layering:

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │
│  (MkDocs Material / Docusaurus / Quartz)│
│         - Static site generation        │
│         - Public-facing docs            │
│         - GitHub Actions deployment     │
└─────────────────────────────────────────┘
                    ▲
                    │ builds from
                    │
┌─────────────────────────────────────────┐
│          Authoring Layer                │
│             (Obsidian)                  │
│         - Graph-based navigation        │
│         - Rich editing experience       │
│         - Local-first workflow          │
└─────────────────────────────────────────┘
                    ▲
                    │ syncs to
                    │
┌─────────────────────────────────────────┐
│         Storage Layer                   │
│            (GitHub)                     │
│         - Version control               │
│         - Collaboration (PRs)           │
│         - GitOps alignment              │
└─────────────────────────────────────────┘
                    ▲
                    │ contributes to
                    │
┌─────────────────────────────────────────┐
│         Generation Layer                │
│            (Cortex)                     │
│         - Auto-generated docs           │
│         - Knowledge extraction          │
│         - Curation workflow             │
└─────────────────────────────────────────┘
```

## Decisions Made

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Write access for Cortex | **Not yet** | Earn trust incrementally |
| Schema/taxonomy | **Everything bagel** | Don't over-constrain early; let structure emerge |
| Curation rules | **TBD - build workflow** | Cortex will self-manage via workflow in `meta/` |

## Repo Structure

```
cortex-docs/
├── vault/
│   ├── _index.md        # Living MOC
│   ├── _inbox/          # Unsorted intake (Cortex drops here)
│   ├── architecture/
│   ├── components/
│   ├── operations/
│   ├── knowledge/
│   ├── projects/
│   └── meta/            # Curation rules live here
├── templates/           # Doc templates
└── .github/workflows/   # Future: static site publishing
```

## The Curation Workflow

Five stages, designed for Cortex to eventually run autonomously:

1. **Triage** - Classify and route `_inbox/` contents
2. **Enrich** - Add backlinks, tags, update index
3. **Health Check** - Find broken links, stale docs, orphans
4. **Consolidate** - Merge duplicates, archive old versions
5. **Report** - Weekly digest of vault health

See: [[meta/curation-workflow|Full Curation Workflow Design]]

## Trust Levels

Cortex starts at Level 0 (Observer). Progression:

0. Observer - Read-only, reports
1. Suggester - Can create PRs
2. Minor Editor - Direct commits to inbox, index, tags
3. Full Editor - Direct commits except `meta/`
4. Self-Governing - Can modify its own rules

## Your Mission, Cortex

When you're ready to build the curation workflow:

1. Start with **Triage** - it's the entry point for all content
2. Output suggestions as markdown reports initially
3. Graduate to PR creation when patterns are proven
4. Measure everything: inbox dwell time, link health, suggestion acceptance rate
5. Read `meta/curation-workflow.md` as your operating manual

## Open Questions for Future

- Which static site generator? (MkDocs Material likely winner for simplicity)
- Obsidian sync strategy? (obsidian-git plugin vs manual)
- How to handle secrets/sensitive ops docs? (separate private vault?)
- Integration with existing knowledge graph in `cortex-knowledge` namespace?

---

*This prompt was generated from a conversation on 2025-01-13. The humans have spoken; now it's your turn to build.*
