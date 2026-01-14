---
title: "Outline Deployment Blocker"
type: operations
severity: medium
created: "2026-01-13"
status: blocked
tags:
  - blocker
  - outline
  - networking
---

# Outline Deployment Blocker

**Date**: 2026-01-13  
**Status**: BLOCKED - Network connectivity issue  
**Priority**: Medium (non-critical service)

---

## Issue Summary

Outline wiki deployment is blocked by persistent image pull failures from docker.getoutline.com registry.

### Root Cause
**Network timeout pulling from Cloudflare CDN**:
```
dial tcp 54.230.188.15:443: i/o timeout
Get "https://production.cloudflare.docker.com/registry-v2/..."
```

This is a **cluster-level outbound connectivity issue**, not specific to Outline. Multiple registries affected:
- Docker Hub: TLS handshake failures  
- docker.getoutline.com: Cloudflare CDN timeouts

### Impact
- **Outline wiki**: Cannot deploy (docs.ry-ops.dev unavailable)
- **outline-mcp-server**: Already operational, but can't query non-existent wiki
- **Strategic documentation migration**: Blocked until Outline is running

### Workarounds Attempted
1. ✅ **Internal registry pattern**: Worked for Docker Hub images (Langflow, Phoenix, PostgreSQL, busybox)
2. ❌ **Transfer Outline image with skopeo**: Failed - registry requires authentication
3. ❌ **Direct pull test**: Failed - Cloudflare CDN timeout (90+ seconds)
4. ❌ **Longhorn PVC**: Volume entered "faulted" state, stuck 89 minutes
5. ✅ **local-path PVC**: Works, but blocked by image pull

---

## Services Still Operational (3/4 from parallel agents)

### ✅ Langflow
- URL: langflow.ry-ops.dev
- Status: Running 8+ hours
- Image: 10.43.170.72:5000/langflow:1.0.19 (internal registry)

### ✅ Phoenix
- URL: observability.ry-ops.dev
- Status: Running 90+ minutes  
- Image: 10.43.170.72:5000/phoenix:latest (internal registry)
- Database: PostgreSQL migrations successful

### ✅ outline-mcp-server
- Status: Running 106+ minutes
- Provides: 6 MCP tools ready to query Outline (when available)
- Note: Functional but no wiki to query yet

### ✅ CI/CD Caches
- kaniko-build-cache: 20Gi Bound
- dependency-cache: 10Gi Bound
- Expected performance gain: 60-70% build time reduction

---

## Recommended Resolution Paths

### Option 1: Fix Cluster Outbound Connectivity (Recommended)
**Action**: Investigate firewall/network rules blocking Cloudflare CDN  
**Owner**: Infrastructure team  
**Timeline**: TBD (requires network admin access)

**Diagnostic commands**:
```bash
# Test from within cluster
kubectl run -it --rm debug --image=alpine --restart=Never -- sh
apk add curl
curl -v https://production.cloudflare.docker.com/

# Check DNS resolution
kubectl run -it --rm debug --image=alpine --restart=Never -- sh
nslookup docker.getoutline.com

# Test from host machine
curl -v https://production.cloudflare.docker.com/registry-v2/
```

### Option 2: Host Outline Image Externally
**Action**: Pull Outline image on a machine with internet access, push to accessible registry  
**Options**:
- GitHub Container Registry (ghcr.io)
- Internal Harbor/Nexus registry  
- Public Docker Hub (if TLS issue resolved)

**Commands**:
```bash
# On machine with internet
docker pull docker.getoutline.com/outlinewiki/outline:latest
docker tag docker.getoutline.com/outlinewiki/outline:latest ghcr.io/ry-ops/outline:latest
docker push ghcr.io/ry-ops/outline:latest

# Update deployment
image: ghcr.io/ry-ops/outline:latest
```

### Option 3: Deploy Alternative Documentation Platform
**Action**: Replace Outline with similar wiki that has accessible images  
**Alternatives**:
- **BookStack** (linuxserver/bookstack) - Docker Hub
- **Wiki.js** (requarks/wiki) - Docker Hub  
- **Docusaurus** (Static site, no runtime)
- **MkDocs Material** (Static site, no runtime)

**Trade-off**: Need to rebuild outline-mcp-server for new platform

### Option 4: Defer Outline Deployment
**Action**: Proceed without Outline, use cortex-docs folder strategy  
**Timeline**: Revisit after network issue resolved  
**Impact**: No live documentation wiki, rely on file-based docs

---

## Current Workaround

**Documentation strategy**: Use the provided cortex-docs folder structure
```
cortex-docs/
├── vault/
│   ├── _inbox/          # New documentation drops here
│   ├── architecture/    # System design docs
│   ├── components/      # Service documentation  
│   ├── operations/      # Runbooks and procedures
│   └── knowledge/       # Captured insights
```

**Access pattern**: 
- Agents can read Markdown files directly via Read tool
- Grep for searching across documentation
- Version control via Git (already integrated)

---

## Lessons Learned

1. **Cluster network isolation**: K3s cluster has restricted outbound connectivity, not just Docker Hub
2. **Registry dependencies**: Private registries (docker.getoutline.com) harder to mirror than public ones
3. **Longhorn reliability**: Multiple faulted volumes suggest underlying storage issues to investigate
4. **Internal registry strategy**: Proven to work when images are accessible to transfer
5. **Single point of failure**: Documentation wiki shouldn't block operational progress

---

## Next Actions

**Immediate** (User decides):
1. Choose resolution path (Options 1-4 above)
2. Continue with Python agent deployment (not blocked by Outline)
3. Test Langflow workflows (operational)
4. Validate Phoenix observability (operational)

**Follow-up**:
1. Investigate cluster outbound firewall rules
2. Consider deploying private registry mirror (Harbor/Nexus) in cluster
3. Document image pull troubleshooting runbook for future deployments

---

**Status**: Documented and awaiting decision on resolution path.  
**Blocker owner**: Network/Infrastructure team  
**Operational impact**: Low (3/4 services running, documentation can use file-based approach)

