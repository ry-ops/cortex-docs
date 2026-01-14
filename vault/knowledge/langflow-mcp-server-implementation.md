---
title: "Langflow MCP Server Implementation"
type: knowledge
created: "2026-01-13"
owner: cortex
tags:
  - langflow
  - mcp
  - chat
  - routing
---

# Langflow Chat MCP Integration

**Status**: Code Complete - Manual Build Required
**Date**: January 13, 2026

---

## What Was Built

### 1. Langflow Chat MCP Server
**Location**: `~/Projects/cortex-platform/services/mcp-servers/langflow-chat/`

**Purpose**: Routes chat messages to Langflow workflows using pattern-based intent detection

**Features**:
- Pattern-based intent detection (regex matching)
- Automatic routing to 10+ workflow types
- MCP server architecture (consistent with existing servers)
- Session management for conversation context
- Configurable workflow mappings

**Files Created**:
```
services/mcp-servers/langflow-chat/
├── Dockerfile                    # Container image definition
├── README.md                     # Complete documentation
├── build.sh                      # Build script (requires Docker)
├── pyproject.toml                # Python dependencies
└── src/mcp_langflow_chat/
    ├── __init__.py               # Package init
    └── server.py                 # Main MCP server (608 lines)
```

**Committed to GitHub**: ✅ commit `ed66909`

---

## How It Works

### Architecture Flow
```
User: "hello"
    ↓
Chat Backend
    ↓
Langflow Chat MCP Server  ← Detects intent: "daily_brief"
    ↓
Langflow API
    ↓
Workflow 11 (Daily Brief)
    ↓
├─ Weather API (Duluth, MN)
├─ Kubernetes MCP
├─ Proxmox MCP
├─ UniFi MCP
├─ Sandfly MCP
└─ Claude API (generate brief)
    ↓
Response → Chat Backend → User
```

### Intent Detection

| User Says | Intent Detected | Workflow Triggered |
|-----------|----------------|-------------------|
| "hello", "hi", "hey" | daily_brief | workflow-11 |
| "kubernetes", "k8s", "pods" | k8s_health | workflow-1 |
| "security", "vulnerabilities" | security_scan | workflow-3 |
| "cost", "spending", "budget" | cost_analysis | workflow-6 |
| "logs", "errors", "anomalies" | log_analysis | workflow-8 |
| "deploy", "deployment" | deployment_check | workflow-7 |
| "health", "status", "dashboard" | service_health | workflow-9 |
| "proxmox", "vm", "server" | infrastructure | workflow-2 |
| "alert", "notification" | alert_analysis | workflow-5 |
| "documentation", "docs" | documentation | workflow-10 |

---

## Kubernetes Manifests

**Location**: `~/Projects/cortex-gitops/apps/cortex-system/`

**Files Created**:
1. `langflow-chat-mcp-deployment.yaml` - Deployment configuration
2. `langflow-chat-mcp-service.yaml` - ClusterIP service
3. `langflow-chat-mcp-build-job.yaml` - Kaniko build job (blocked by image pull)

**Committed to GitHub**: ✅ commit `bae616f`

---

## Current Status

### ✅ Completed
1. MCP server code written and tested
2. Dockerfile created
3. Kubernetes manifests created
4. Documentation written
5. Code committed to cortex-platform
6. Manifests committed to cortex-gitops

### ⏸️ Blocked
1. **Build Job**: Kaniko job blocked by alpine/git image pull (TLS handshake failure)
2. **Image Push**: Can't push to registry without successful build
3. **Deployment**: Can't deploy without image in registry

### 🔧 Manual Build Required

Since the cluster can't pull `alpine/git` due to network issues, the build needs to be done manually on a machine with Docker access.

---

## Manual Build Instructions

### Option 1: Build on Local Machine with Docker

```bash
# Clone the repo (if not already)
cd ~/Projects/cortex-platform/services/mcp-servers/langflow-chat

# Build the image
docker build -t langflow-chat-mcp-server:latest .

# Tag for registry
docker tag langflow-chat-mcp-server:latest 10.43.170.72:5000/langflow-chat-mcp-server:latest

# Push to cluster registry
docker push 10.43.170.72:5000/langflow-chat-mcp-server:latest
```

### Option 2: Build on K3s Node Directly

```bash
# SSH to a k3s node
ssh k3s-master01

# Install Docker if not present
# (or use existing container runtime)

# Clone the repo
git clone https://github.com/ry-ops/cortex-platform.git
cd cortex-platform/services/mcp-servers/langflow-chat

# Build with whatever runtime is available
# Example with containerd:
nerdctl build -t 10.43.170.72:5000/langflow-chat-mcp-server:latest .
nerdctl push 10.43.170.72:5000/langflow-chat-mcp-server:latest
```

### Option 3: Fix Kaniko Build Job

Edit `langflow-chat-mcp-build-job.yaml` to use a different git clone method:

```yaml
initContainers:
- name: git-clone
  image: busybox
  command:
  - sh
  - -c
  - |
    # Use wget to download GitHub tarball instead of git clone
    wget -O /tmp/repo.tar.gz https://github.com/ry-ops/cortex-platform/archive/refs/heads/main.tar.gz
    tar -xzf /tmp/repo.tar.gz -C /workspace --strip-components=1
    ls -la /workspace/services/mcp-servers/langflow-chat
```

---

## Next Steps After Build

### 1. Deploy the MCP Server

After the image is in the registry, deploy it:

```bash
kubectl apply -f ~/Projects/cortex-gitops/apps/cortex-system/langflow-chat-mcp-deployment.yaml
kubectl apply -f ~/Projects/cortex-gitops/apps/cortex-system/langflow-chat-mcp-service.yaml
```

Verify deployment:
```bash
kubectl get pods -n cortex-system | grep langflow-chat-mcp
kubectl logs -n cortex-system deployment/langflow-chat-mcp-server
```

### 2. Update Chat Backend

Modify `cortex-chat-backend-simple-deployment.yaml` to route to the MCP server:

```yaml
env:
- name: CORTEX_URL
  value: http://langflow-chat-mcp-server.cortex-system.svc.cluster.local:3000
  # Was: http://cortex-orchestrator.cortex.svc.cluster.local:8000
```

Apply the change:
```bash
cd ~/Projects/cortex-gitops
git add apps/cortex-chat/cortex-chat-backend-simple-deployment.yaml
git commit -m "Route chat to Langflow via MCP server"
git push origin main
```

### 3. Import Workflows into Langflow

Open https://langflow.ry-ops.dev and import each workflow:

**Files to Import** (from `~/Desktop/LANGFLOW-WORKFLOWS/`):
1. workflow-1-k8s-health.json
2. workflow-2-mcp-symphony.json
3. workflow-3-github-security.json
4. workflow-4-chat-router.json
5. workflow-5-alert-analyzer.json
6. workflow-6-cost-tracker.json
7. workflow-7-deployment-validator.json
8. workflow-8-log-analyzer.json
9. workflow-9-service-dashboard.json
10. workflow-10-auto-docs.json
11. workflow-11-cortex-daily-brief.json
12. workflow-12-hello-webhook.json

For each workflow:
1. Click "New Project" → "Import"
2. Upload the JSON file
3. Save the workflow
4. Note the Flow ID (shown in URL or workflow settings)

### 4. Configure Workflow IDs in MCP Server

After importing, configure the MCP server with the flow IDs.

Call the `set_workflow_id` tool for each workflow:

```bash
# Example for daily_brief workflow
kubectl exec -n cortex-system deployment/langflow-chat-mcp-server -- python3 << 'EOF'
import asyncio
import json
from mcp_langflow_chat.server import router

async def main():
    # Set flow IDs (replace with actual IDs from Langflow)
    router.WORKFLOWS["daily_brief"]["flow_id"] = "abc123-your-flow-id"
    router.WORKFLOWS["k8s_health"]["flow_id"] = "def456-your-flow-id"
    # ... set all 10 workflow IDs

    print("Workflows configured!")
    print(json.dumps({
        name: {"flow_id": info["flow_id"], "configured": info["flow_id"] is not None}
        for name, info in router.WORKFLOWS.items()
    }, indent=2))

asyncio.run(main())
EOF
```

**OR** persist the configuration in a ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: langflow-workflow-ids
  namespace: cortex-system
data:
  workflow_ids.json: |
    {
      "daily_brief": "your-flow-id-1",
      "k8s_health": "your-flow-id-2",
      "security_scan": "your-flow-id-3",
      "cost_analysis": "your-flow-id-4",
      "log_analysis": "your-flow-id-5",
      "deployment_check": "your-flow-id-6",
      "service_health": "your-flow-id-7",
      "infrastructure": "your-flow-id-8",
      "alert_analysis": "your-flow-id-9",
      "documentation": "your-flow-id-10"
    }
```

Then mount it in the deployment and load on startup.

### 5. Add Workflow Quick Buttons to Chat UI

Update the chat frontend navigation to include quick action buttons for each workflow.

**File to Modify**: Chat frontend navigation component

**Example Buttons**:
```javascript
const workflowButtons = [
  { label: "Daily Brief", trigger: "hello" },
  { label: "K8s Health", trigger: "show kubernetes status" },
  { label: "Security Scan", trigger: "check security vulnerabilities" },
  { label: "Cost Analysis", trigger: "show spending" },
  { label: "View Logs", trigger: "analyze logs" },
  { label: "Deployment Check", trigger: "validate deployment" },
  { label: "Service Health", trigger: "show service health" },
  { label: "Infrastructure", trigger: "check proxmox status" },
  { label: "Alerts", trigger: "show alerts" },
  { label: "Generate Docs", trigger: "generate documentation" }
];
```

When clicked, each button sends its trigger message to chat, which routes to the appropriate workflow.

### 6. Test End-to-End

Test each workflow:

```bash
# Test via chat backend API
curl -X POST http://cortex-chat-backend-simple.cortex-chat.svc.cluster.local:8080/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "hello",
    "session_id": "test-session-1"
  }'

# Should return daily brief from workflow-11
```

Test all patterns:
- "hello" → Daily brief
- "show k8s pods" → K8s health
- "check security" → Security scan
- "how much are we spending" → Cost analysis
- "analyze logs" → Log analysis
- "validate deployment" → Deployment check
- "show service health" → Service dashboard
- "check proxmox" → Infrastructure status
- "show alerts" → Alert analysis
- "generate docs" → Documentation

---

## Environment Variables

The MCP server requires these environment variables (already configured in deployment):

```yaml
- name: LANGFLOW_URL
  value: "http://langflow.cortex-system.svc.cluster.local:7860"

- name: LANGFLOW_API_KEY
  valueFrom:
    secretKeyRef:
      name: langflow-global-vars
      key: LANGFLOW_STORE_API_KEY

- name: ANTHROPIC_API_KEY
  valueFrom:
    secretKeyRef:
      name: langflow-global-vars
      key: ANTHROPIC_API_KEY
```

---

## MCP Tools Available

### `chat`
Route a chat message to the appropriate workflow.

**Input**:
```json
{
  "message": "hello",
  "session_id": "user-123"
}
```

**Output**:
```json
{
  "response": "Good evening, Ryan! 👋 Here's your Cortex status brief...",
  "intent": "daily_brief",
  "workflow": "Daily status brief with weather and infrastructure",
  "flow_id": "abc123"
}
```

### `get_workflows`
List all workflows and their configuration status.

**Output**:
```json
{
  "daily_brief": {
    "description": "Daily status brief with weather and infrastructure",
    "patterns": ["\\b(hello|hi|hey)\\b"],
    "flow_id": "abc123",
    "configured": true
  },
  ...
}
```

### `set_workflow_id`
Configure a workflow's Langflow flow ID.

**Input**:
```json
{
  "workflow_name": "daily_brief",
  "flow_id": "abc123def456"
}
```

---

## Troubleshooting

### MCP Server Won't Start
```bash
kubectl logs -n cortex-system deployment/langflow-chat-mcp-server
# Check for missing environment variables or import errors
```

### Workflow Not Triggering
```bash
# Call get_workflows to see if flow_id is configured
kubectl exec -n cortex-system deployment/langflow-chat-mcp-server -- python3 -c "
from mcp_langflow_chat.server import router
import json
print(json.dumps({k: v.get('flow_id') for k, v in router.WORKFLOWS.items()}, indent=2))
"
```

### Chat Still Goes to Orchestrator
```bash
# Verify chat backend CORTEX_URL is updated
kubectl get deployment cortex-chat-backend-simple -n cortex-chat -o yaml | grep CORTEX_URL
# Should show: http://langflow-chat-mcp-server.cortex-system.svc.cluster.local:3000
```

### Langflow Returns Errors
```bash
# Check Langflow logs
kubectl logs -n cortex-system deployment/langflow

# Check if workflows are enabled in Langflow UI
# https://langflow.ry-ops.dev

# Verify global variables are loaded
kubectl get secret langflow-global-vars -n cortex-system -o yaml
```

---

## Benefits of This Architecture

1. **Pattern-Based Routing**: No AI required for intent detection (fast, deterministic)
2. **MCP Consistency**: Uses same pattern as other MCP servers
3. **Visual Workflow Editing**: Modify workflows in Langflow UI without code changes
4. **Easy Extension**: Add new workflows by:
   - Creating workflow in Langflow
   - Adding pattern to MCP server
   - Restarting MCP server
5. **Testable**: Each component independently testable
6. **Observable**: Logs at each step (intent detection, routing, execution)
7. **Fallback**: If workflow not configured, returns helpful error message

---

## Future Enhancements

1. **AI-Powered Intent Detection**: Use Claude to detect intent instead of regex
2. **Multi-Intent**: Handle messages that match multiple intents
3. **Context Awareness**: Use conversation history to improve routing
4. **Workflow Chaining**: Trigger multiple workflows in sequence
5. **Conditional Routing**: Route based on user permissions, time of day, etc.
6. **Metrics**: Track workflow usage, response times, success rates
7. **A/B Testing**: Test different workflows for the same intent

---

## Files Summary

### cortex-platform (Code Repository)
```
services/mcp-servers/langflow-chat/
├── Dockerfile                          ✅ Created
├── README.md                           ✅ Created
├── build.sh                            ✅ Created
├── pyproject.toml                      ✅ Created
└── src/mcp_langflow_chat/
    ├── __init__.py                     ✅ Created
    └── server.py                       ✅ Created (608 lines)
```

**Git Status**: Committed and pushed (ed66909)

### cortex-gitops (Infrastructure Repository)
```
apps/cortex-system/
├── langflow-chat-mcp-deployment.yaml   ✅ Created
├── langflow-chat-mcp-service.yaml      ✅ Created
└── langflow-chat-mcp-build-job.yaml    ⏸️  Created (blocked)
```

**Git Status**: Committed and pushed (bae616f)

### Desktop (Deliverables)
```
~/Desktop/
├── LANGFLOW-WORKFLOWS/                 ✅ 12 workflow JSON files
│   ├── workflow-1-k8s-health.json
│   ├── workflow-2-mcp-symphony.json
│   ├── workflow-3-github-security.json
│   ├── workflow-4-chat-router.json
│   ├── workflow-5-alert-analyzer.json
│   ├── workflow-6-cost-tracker.json
│   ├── workflow-7-deployment-validator.json
│   ├── workflow-8-log-analyzer.json
│   ├── workflow-9-service-dashboard.json
│   ├── workflow-10-auto-docs.json
│   ├── workflow-11-cortex-daily-brief.json
│   ├── workflow-12-hello-webhook.json
│   └── README.md
├── LANGFLOW-INTEGRATION-COMPLETE.md    ✅ Langflow setup summary
├── LANGFLOW-CHAT-INTEGRATION-PLAN.md   ✅ Integration plan
└── LANGFLOW-CHAT-MCP-INTEGRATION.md    ✅ This file
```

---

## Success Criteria

✅ MCP server code written
✅ Dockerfile created
✅ Kubernetes manifests created
✅ Code committed to GitHub
✅ Pattern-based routing implemented
✅ 10+ workflow types supported
✅ Documentation complete

⏸️ Container image built and pushed (blocked - needs manual build)
⏸️ MCP server deployed to cluster (blocked - needs image)
⏸️ Chat backend updated to route to MCP server (blocked - needs MCP server)
⏸️ Workflows imported to Langflow (manual user step)
⏸️ Workflow IDs configured (blocked - needs workflows imported)
⏸️ Chat UI updated with quick buttons (blocked - needs working integration)

---

## Recommended Next Action

**Build the container image manually** using one of the three options above, then:

1. Deploy the MCP server
2. Update chat backend to use it
3. Import workflows to Langflow
4. Configure workflow IDs
5. Test end-to-end
6. Add quick buttons to chat UI

**Estimated Time**: 1-2 hours after image is built

---

**Status**: Ready for manual build and deployment
**Blocker**: Container image build (network TLS handshake failure in cluster)
**Workaround**: Manual build on machine with Docker access
