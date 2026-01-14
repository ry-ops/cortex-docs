---
title: "Documentation System"
type: project
status: completed
created: "2026-01-14"
completed: "2026-01-14"
owner: cortex
tags:
  - documentation
  - obsidian
  - mkdocs
  - curator
---

# Cortex Documentation System - Complete

**Date**: January 14, 2026
**Status**: Production Ready

---

## What Was Built

### 1. Documentation Repository: `cortex-docs`

**GitHub**: https://github.com/ry-ops/cortex-docs
**Published**: https://ry-ops.github.io/cortex-docs (auto-deploying)

**Structure**:
```
cortex-docs/
├── vault/                      # Obsidian vault
│   ├── _index.md               # Living table of contents
│   ├── _inbox/                 # Unsorted intake
│   ├── architecture/           # System design, ADRs
│   ├── components/             # Per-service documentation
│   ├── operations/             # Runbooks, SOPs
│   ├── knowledge/              # Extracted insights
│   ├── projects/               # Bounded efforts
│   └── meta/                   # Documentation about documentation
├── templates/                  # Documentation templates
│   ├── component.md
│   ├── decision.md
│   ├── knowledge-extract.md
│   └── runbook.md
├── .obsidian/                  # Obsidian configuration
├── mkdocs.yml                  # MkDocs Material configuration
└── .github/workflows/
    └── publish.yml             # Auto-publish to GitHub Pages
```

### 2. Documentation Curator Service

**Location**: `cortex-platform/services/docs-curator/`
**Purpose**: Automated curation of documentation

**Features**:
- **Triage**: Analyze new documents in `_inbox/` with Claude
- **Health Check**: Scan for broken links, stale docs, orphaned pages
- **GitHub Integration**: Create PRs with suggestions
- **Trust Levels**: Gradual automation from observer to self-governing

**Trust Levels**:
| Level | Name | Permissions |
|-------|------|-------------|
| 0 | Observer | Read-only, generate reports |
| 1 | Suggester | Create PRs, no direct commits ⭐ Default |
| 2 | Minor Editor | Direct commits to `_inbox/`, `_index.md`, tags |
| 3 | Full Editor | Direct commits anywhere except `meta/` |
| 4 | Self-Governing | Can modify curation rules in `meta/` |

### 3. Publishing Pipeline

**Technology**: MkDocs Material
**Hosting**: GitHub Pages
**URL**: https://ry-ops.github.io/cortex-docs

**Features**:
- Auto-deploys on every push to main
- Dark/light theme toggle
- Full-text search
- Obsidian-compatible markdown
- Code syntax highlighting
- Edit links back to GitHub

---

## Architecture

### Documentation Workflow

```
Human/Cortex writes doc
    ↓
Drop into vault/_inbox/
    ↓
Curator Service (weekly cron)
    ↓
├─ Clone cortex-docs repo
├─ Scan _inbox/ for new files
├─ For each document:
│   ├─ Extract frontmatter
│   ├─ Analyze with Claude
│   └─ Generate recommendations
└─ Create PR with suggestions
    ↓
Human reviews and merges
    ↓
Push to main triggers workflow
    ↓
GitHub Actions builds site
    ↓
Published to GitHub Pages
```

### Integration Points

- **Obsidian**: Local editing, graph view, daily notes
- **GitHub**: Version control, collaboration, source of truth
- **GitHub Pages**: Public documentation hosting
- **Curator Service**: Automated curation (to be deployed)
- **Claude API**: Document analysis and categorization

---

## Current Status

### ✅ Completed

1. **Repository Created**
   - GitHub repo: ry-ops/cortex-docs
   - Obsidian vault structure
   - Documentation templates
   - Curation workflow design

2. **Curator Service Built**
   - Python service with Claude integration
   - GitHub API integration for PRs
   - Triage and health check commands
   - Dockerfile ready

3. **Publishing Pipeline**
   - GitHub Actions workflow
   - MkDocs Material configuration
   - Auto-deploy to GitHub Pages
   - First deployment will trigger on next push

### ⏸️ Pending

1. **Curator Deployment**: Kubernetes manifests needed
2. **Initial Documentation**: Populate vault with existing knowledge
3. **Cron Schedule**: Set up weekly triage runs
4. **Ingress**: Expose published docs at docs.ry-ops.dev (optional)

---

## How to Use

### For Humans

#### Local Editing with Obsidian

1. Clone the repository:
   ```bash
   git clone https://github.com/ry-ops/cortex-docs.git
   cd cortex-docs
   ```

2. Open `vault/` folder in [Obsidian](https://obsidian.md)

3. Create new documents:
   - Drop drafts in `vault/_inbox/`
   - Use templates from `templates/` for consistency
   - Add YAML frontmatter for metadata

4. Commit and push:
   ```bash
   git add vault/
   git commit -m "Add documentation for [topic]"
   git push origin main
   ```

5. Site auto-publishes within 1-2 minutes

#### Using Templates

**Component Documentation**:
```yaml
---
title: "Service Name"
type: component
owner: team-name
status: active
tags:
  - service
  - kubernetes
---

# Service Name

## Overview
What this service does...

## Architecture
How it works...

## Operations
How to operate it...
```

**Runbook**:
```yaml
---
title: "How to restart Postgres"
type: runbook
severity: high
tested: 2026-01-14
tags:
  - database
  - incident
---

# How to restart Postgres

## When to use
Symptoms that require this...

## Steps
1. Check current state
2. Prepare for restart
3. Execute restart
4. Verify recovery
```

### For Cortex

#### Automated Curation

Once deployed, Curator runs weekly:

1. Scans `vault/_inbox/` for new documents
2. Analyzes each with Claude:
   - Document type (component/runbook/decision/knowledge)
   - Suggested destination folder
   - Recommended tags
   - Related documents
   - One-sentence summary

3. Creates PR with:
   - Triage report
   - Move suggestions
   - Tag additions
   - Backlink recommendations

4. Human reviews and merges PR

#### Manual Curation

Run curator locally:

```bash
# Triage inbox documents
python -m docs_curator.curator triage ~/Projects/cortex-docs

# Run health check
python -m docs_curator.curator health-check ~/Projects/cortex-docs
```

---

## Deployment (Next Steps)

### 1. Enable GitHub Pages

Go to: https://github.com/ry-ops/cortex-docs/settings/pages

Set:
- Source: Deploy from a branch
- Branch: gh-pages
- Folder: / (root)

### 2. Deploy Curator to Cluster

Create Kubernetes manifests:

```yaml
# cortex-gitops/apps/cortex-system/docs-curator-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: docs-curator
  namespace: cortex-system
spec:
  schedule: "0 6 * * 1"  # Monday 6am
  jobTemplate:
    spec:
      template:
        spec:
          initContainers:
          - name: clone-repo
            image: alpine/git
            command:
            - sh
            - -c
            - |
              git clone https://github.com/ry-ops/cortex-docs.git /workspace
            volumeMounts:
            - name: workspace
              mountPath: /workspace
          containers:
          - name: curator
            image: 10.43.170.72:5000/docs-curator:latest
            args: ["triage", "/workspace"]
            env:
            - name: GITHUB_TOKEN
              valueFrom:
                secretKeyRef:
                  name: github-token
                  key: token
            - name: ANTHROPIC_API_KEY
              valueFrom:
                secretKeyRef:
                  name: langflow-global-vars
                  key: ANTHROPIC_API_KEY
            - name: TRUST_LEVEL
              value: "1"
            volumeMounts:
            - name: workspace
              mountPath: /workspace
          volumes:
          - name: workspace
            emptyDir: {}
          restartPolicy: OnFailure
```

### 3. Create GitHub Token Secret

```bash
# Create GitHub personal access token with repo scope
# https://github.com/settings/tokens

kubectl create secret generic github-token \
  --from-literal=token=YOUR_GITHUB_TOKEN \
  -n cortex-system
```

### 4. Build Curator Image

Use Portainer at portainer.ry-ops.dev:

1. Navigate to Images → Build
2. Repository: https://github.com/ry-ops/cortex-platform.git
3. Context: services/docs-curator
4. Tag: 10.43.170.72:5000/docs-curator:latest
5. Build and push

OR manually:
```bash
cd ~/Projects/cortex-platform/services/docs-curator
docker build -t 10.43.170.72:5000/docs-curator:latest .
docker push 10.43.170.72:5000/docs-curator:latest
```

### 5. (Optional) Custom Domain

Add Traefik ingress for docs.ry-ops.dev:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cortex-docs-ingress
  namespace: cortex-system
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  rules:
  - host: docs.ry-ops.dev
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: docs-external
            port:
              number: 80
---
apiVersion: v1
kind: Service
metadata:
  name: docs-external
  namespace: cortex-system
spec:
  type: ExternalName
  externalName: ry-ops.github.io
```

---

## Philosophy

### Inbox-First Approach

All new content enters through `_inbox/`. This creates a single intake point that:
- Reduces decision paralysis (where does this go?)
- Enables automated triage
- Maintains clean organization
- Provides audit trail

### Suggest, Don't Act

Curator starts at Trust Level 1 (Suggester):
- Creates PRs, never direct commits
- Humans review and approve
- Builds confidence in recommendations
- Incrementally earns write access

### Self-Managing Documentation

Goal: Documentation that stays current without manual intervention.

**How**:
- Automated health checks flag stale docs
- Broken links detected and reported
- Semantic analysis identifies duplicates
- Claude suggests consolidations
- Weekly digests track vault health

### Human-Readable Output

All curation actions must be explainable:
- Clear PR descriptions
- Reasoning for suggestions
- Links to analysis criteria
- Confidence scores when uncertain

---

## Metrics to Track

Once deployed, monitor:

- **Inbox Dwell Time**: How long docs stay unsorted
- **Link Health**: % of valid internal links
- **Coverage**: % of components with documentation
- **Freshness**: Average age of documents by type
- **Curation Accuracy**: % of suggestions accepted

---

## Future Enhancements

### Short-term (Weeks)
- Populate vault with existing Cortex knowledge
- Deploy curator to cluster
- Set up weekly health reports
- Add more auto-tagging rules

### Medium-term (Months)
- Obsidian plugin for in-editor suggestions
- Chat interface: "Hey Cortex, what docs need attention?"
- Auto-generate stub docs for new cluster components
- Semantic similarity for consolidation suggestions

### Long-term (Quarters)
- Cross-repo curation (extend to inline docs in cortex-platform)
- Knowledge graph visualization
- Auto-update docs based on cluster state
- Full Trust Level 4 (self-governing)

---

## Files Created

### cortex-docs Repository
```
.github/workflows/publish.yml    ✅ GitHub Actions workflow
mkdocs.yml                       ✅ MkDocs configuration
vault/                           ✅ Documentation vault
  ├── _inbox/                    ✅ Intake folder
  ├── _index.md                  ✅ Table of contents
  ├── architecture/              ✅ Design docs
  ├── components/                ✅ Service docs
  ├── operations/                ✅ Runbooks
  ├── knowledge/                 ✅ Insights
  ├── projects/                  ✅ Project docs
  └── meta/                      ✅ Meta documentation
      └── curation-workflow.md   ✅ Curation design
templates/                       ✅ Doc templates
  ├── component.md               ✅ Component template
  ├── decision.md                ✅ Decision template
  ├── knowledge-extract.md       ✅ Knowledge template
  └── runbook.md                 ✅ Runbook template
.obsidian/                       ✅ Obsidian config
README.md                        ✅ Repository docs
```

### cortex-platform/services/docs-curator/
```
src/docs_curator/
  ├── __init__.py                ✅ Package init
  └── curator.py                 ✅ Main curator (584 lines)
Dockerfile                       ✅ Container image
README.md                        ✅ Service docs
pyproject.toml                   ✅ Python dependencies
```

### Desktop (Deliverables)
```
CORTEX-DOCS-COMPLETE.md          ✅ This file
```

---

## Git Commits

### cortex-docs
- `e23e07a` - Initial commit: Cortex documentation vault
- `b21097c` - Add documentation publishing pipeline

**GitHub**: https://github.com/ry-ops/cortex-docs

### cortex-platform
- `03ad136` - Add Documentation Curator service

**GitHub**: https://github.com/ry-ops/cortex-platform

---

## Success Criteria

✅ Repository created and pushed to GitHub
✅ Obsidian vault structure defined
✅ Documentation templates created
✅ Curation workflow designed
✅ Curator service implemented
✅ Publishing pipeline configured
✅ GitHub Actions workflow ready

⏸️ GitHub Pages enabled (requires manual step)
⏸️ Curator deployed to cluster (manual build required)
⏸️ Initial documentation populated
⏸️ Custom domain configured (optional)

---

## Quick Start Guide

**For immediate use**:

1. **Clone the repo**:
   ```bash
   git clone https://github.com/ry-ops/cortex-docs.git
   ```

2. **Open in Obsidian**:
   - Launch Obsidian
   - Open vault → Choose `cortex-docs/vault/`

3. **Start writing**:
   - Create new note in `_inbox/`
   - Use templates from `templates/`
   - Add frontmatter metadata

4. **Publish**:
   ```bash
   git add vault/
   git commit -m "Add docs for X"
   git push
   ```

5. **View published**:
   - Wait 1-2 minutes for GitHub Actions
   - Visit: https://ry-ops.github.io/cortex-docs

---

**Status**: Core implementation complete. Ready for population and deployment.

**Next**: Enable GitHub Pages, build curator image, deploy to cluster.
