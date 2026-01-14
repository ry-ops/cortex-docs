# Cortex Documentation

The living knowledge base for Cortex - self-managing, self-curating, and human-friendly.

## Philosophy

This repository serves as the **single source of truth** for all Cortex documentation. It's designed to be:

- **Authored** in [Obsidian](https://obsidian.md) for a rich, graph-based writing experience
- **Stored** in GitHub for version control, collaboration, and GitOps alignment
- **Published** via static site generation for public-facing documentation

## Structure

```
vault/
├── _index.md          # Map of Content - the living table of contents
├── _inbox/            # Unsorted intake - Cortex and humans dump here
├── architecture/      # System design, ADRs, diagrams
├── components/        # Per-service documentation
├── operations/        # Runbooks, SOPs, incident response
├── knowledge/         # Extracted insights and learnings
├── projects/          # Bounded efforts with defined scope
└── meta/              # Documentation about documentation
```

## Getting Started

### For Humans

1. Clone this repo
2. Open the `vault/` folder in Obsidian
3. Start writing - use templates from `templates/` for consistency

### For Cortex

1. Drop documents into `vault/_inbox/`
2. Follow templates in `templates/` for structured output
3. Read curation rules in `vault/meta/` before any automated actions

## Curation Workflow

Documentation curation is (will be) managed by Cortex itself. See `vault/meta/curation-workflow.md` for the design and rules.

## Publishing

> **Future State**: GitHub Actions will build and deploy a static site on push.

Options under consideration:
- [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)
- [Docusaurus](https://docusaurus.io/)
- [Quartz](https://quartz.jzhao.xyz/) (Obsidian-native)

---

*This documentation is managed by Cortex. For questions, talk to the machine.*
