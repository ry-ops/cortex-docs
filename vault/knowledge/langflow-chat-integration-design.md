---
title: "Langflow Chat Integration Design"
type: knowledge
created: "2026-01-13"
owner: cortex
tags:
  - langflow
  - chat
  - integration
  - architecture
---

# Langflow Chat Integration Plan

**Goal**: Make all 12 Langflow workflows accessible through Cortex Chat

**Current State**: Chat → Backend → Orchestrator → Claude API (17 tools)
**Target State**: Chat → Backend → Langflow Workflows → Response

---

## Problem Analysis

### Current Flow
1. User types message in chat frontend
2. `cortex-chat-backend-simple` receives message
3. Backend forwards to `cortex-orchestrator` (port 8000)
4. Orchestrator calls Claude API directly with 17 MCP tools
5. Response returns through the chain

### What's Missing
1. Workflows are not imported into Langflow yet
2. Chat backend doesn't route to Langflow
3. No intent detection/routing logic
4. Workflows need webhook endpoints or API triggers

---

## Solution: Three-Phase Integration

### Phase 1: Import Workflows into Langflow (Manual - User Task)

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

**Steps**:
1. Open https://langflow.ry-ops.dev
2. Click "New Project" or "Import"
3. Upload each JSON file
4. Save and enable each workflow
5. Note the flow IDs for each workflow

### Phase 2: Create Chat-to-Langflow Router Workflow

Create a new "Master Router" workflow in Langflow:

**Input**: User message from chat
**Logic**:
- Analyze user intent using Claude
- Route to appropriate workflow based on keywords/intent
- Return workflow response to chat

**Pattern Mapping**:
| User Pattern | Workflow | Trigger |
|--------------|----------|---------|
| "hello", "hi", "hey" | workflow-11-cortex-daily-brief | Daily status brief |
| "kubernetes", "k8s", "pods", "cluster" | workflow-1-k8s-health | K8s health check |
| "security", "vulnerabilities", "github" | workflow-3-github-security | Security scan |
| "cost", "spending", "budget" | workflow-6-cost-tracker | Cost analysis |
| "health", "status", "dashboard" | workflow-9-service-dashboard | Service health |
| "logs", "errors", "anomalies" | workflow-8-log-analyzer | Log analysis |
| "deploy", "deployment", "validate" | workflow-7-deployment-validator | Pre-deploy check |
| "proxmox", "vm", "infrastructure" | workflow-2-mcp-symphony | Multi-server query |
| "document", "docs", "documentation" | workflow-10-auto-docs | Generate docs |
| "alert", "alerts", "notifications" | workflow-5-alert-analyzer | Alert categorization |

**Fallback**: If no pattern matches, use workflow-4-chat-router for intelligent routing

### Phase 3: Update Chat Backend to Route to Langflow

**Two Options**:

#### Option A: Modify Chat Backend (Requires Code Changes)

Update `cortex-chat-backend-simple` to:
1. Detect user intent
2. Call Langflow webhook instead of orchestrator
3. Return Langflow response

**File to Modify**: Chat backend source code (needs to be located)

**Environment Variables to Add**:
```yaml
- name: LANGFLOW_URL
  value: http://langflow.cortex-system.svc.cluster.local:7860
- name: LANGFLOW_API_KEY
  valueFrom:
    secretKeyRef:
      name: langflow-global-vars
      key: LANGFLOW_STORE_API_KEY
- name: ENABLE_LANGFLOW_ROUTING
  value: "true"
```

**Routing Logic** (pseudo-code):
```javascript
async function handleChat(message, session) {
  if (process.env.ENABLE_LANGFLOW_ROUTING === 'true') {
    // Route to Langflow
    const intent = detectIntent(message);
    const workflowId = getWorkflowForIntent(intent);

    const response = await fetch(
      `${process.env.LANGFLOW_URL}/api/v1/run/${workflowId}`,
      {
        method: 'POST',
        headers: {
          'x-api-key': process.env.LANGFLOW_API_KEY,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ input: message, session: session })
      }
    );

    return response.json();
  } else {
    // Fallback to orchestrator
    return forwardToOrchestrator(message);
  }
}
```

#### Option B: Use Langflow Master Router (No Code Changes)

Create a single webhook endpoint in Langflow that handles all routing:

**Workflow**: `workflow-13-master-chat-router.json`

**Architecture**:
```
User Message
    ↓
Chat Backend (unchanged)
    ↓
Update CORTEX_URL to point to Langflow webhook
    ↓
Langflow Master Router
    ↓
Intent Detection (Claude)
    ↓
Route to Specific Workflow
    ↓
Execute Workflow
    ↓
Return Response
    ↓
Chat Backend
    ↓
User
```

**Change Required**:
```yaml
# In cortex-chat-backend-simple-deployment.yaml
env:
- name: CORTEX_URL
  value: http://langflow.cortex-system.svc.cluster.local:7860/api/v1/webhook/master-router
  # Instead of: http://cortex-orchestrator.cortex.svc.cluster.local:8000
```

---

## Recommended Approach: Option B (Master Router)

**Why**:
1. No code changes required
2. All routing logic in Langflow (easier to modify)
3. Visual workflow editing
4. Can test and iterate quickly
5. Maintains existing chat backend

**Implementation Steps**:

### Step 1: Create Master Router Workflow

**File**: `~/Desktop/LANGFLOW-WORKFLOWS/workflow-13-master-chat-router.json`

**Structure**:
```
WebhookInput (path: master-router)
    ↓
ParseJSON (extract message)
    ↓
Claude Intent Detector
    ↓
Router Component (based on intent)
    ↓
├─> workflow-1 (k8s intent)
├─> workflow-3 (security intent)
├─> workflow-6 (cost intent)
├─> workflow-8 (logs intent)
├─> workflow-11 (hello intent)
├─> workflow-12 (webhook hello)
└─> Default: workflow-4 (general chat)
    ↓
MergeResponses
    ↓
ChatOutput
```

### Step 2: Update Chat Backend Environment

```yaml
# cortex-chat-backend-simple-deployment.yaml
env:
- name: CORTEX_URL
  value: http://langflow.cortex-system.svc.cluster.local:7860/api/v1/webhook/master-router
- name: LANGFLOW_ENABLED
  value: "true"
```

### Step 3: Test Each Workflow

**Test Matrix**:
| Input | Expected Workflow | Expected Response |
|-------|-------------------|-------------------|
| "hello" | workflow-11 | Daily brief with weather, status |
| "show k8s pods" | workflow-1 | Kubernetes health report |
| "check security" | workflow-3 | GitHub security scan results |
| "how much are we spending" | workflow-6 | Cost analysis |
| "analyze logs" | workflow-8 | Log pattern analysis |
| "can you help me deploy" | workflow-7 | Deployment validation |
| "what's the infrastructure status" | workflow-9 | Service health dashboard |

---

## Phase 4: Gradual Rollout

### Week 1: Soft Launch
- Enable Langflow routing
- Keep orchestrator as fallback
- Monitor response quality
- Gather user feedback

### Week 2: Optimization
- Adjust intent detection thresholds
- Add more patterns
- Improve workflow responses
- Add error handling

### Week 3: Full Rollout
- Set Langflow as primary
- Deprecate orchestrator for chat
- Keep orchestrator for API-only usage
- Document final architecture

---

## Monitoring & Metrics

### Key Metrics to Track
1. **Routing Accuracy**: Did the right workflow trigger?
2. **Response Time**: Langflow vs Orchestrator latency
3. **Error Rate**: Failed workflow executions
4. **User Satisfaction**: Response quality feedback
5. **Workflow Usage**: Which workflows are most popular?

### Logging Requirements
```
[LangflowRouter] Received: "hello" → Intent: greeting → Workflow: workflow-11
[LangflowRouter] Workflow workflow-11 executed in 1.2s
[LangflowRouter] Response length: 543 chars
```

---

## Fallback & Error Handling

### If Langflow is Down
```yaml
# Add to chat backend
- name: FALLBACK_TO_ORCHESTRATOR
  value: "true"
- name: ORCHESTRATOR_URL
  value: http://cortex-orchestrator.cortex.svc.cluster.local:8000
```

### If Workflow Fails
- Return friendly error message
- Log failure for debugging
- Fall back to general chat workflow
- Alert on repeated failures

---

## Current Status: What's Blocking

**User said**: "currently the hello workflow only shows the following: Hello! I'm Cortex..."

**Root Cause**:
1. Workflows not imported into Langflow yet (manual step required)
2. Chat backend still pointing to orchestrator (not Langflow)
3. No routing logic to trigger workflows based on user input

**Immediate Action Required**:
1. Import workflows into Langflow
2. Create master-router workflow (workflow-13)
3. Update chat backend CORTEX_URL to point to Langflow
4. Test end-to-end flow

---

## Next Steps

**For Control Plane (Me)**:
1. Create workflow-13-master-chat-router.json
2. Update cortex-chat-backend-simple-deployment.yaml
3. Commit and push to cortex-gitops
4. Wait for ArgoCD sync

**For User (Manual)**:
1. Import all 13 workflows into Langflow UI
2. Enable each workflow
3. Test webhook endpoints
4. Verify chat integration

**Timeline**: 1-2 hours for full integration

---

## Architecture Diagram

### Before (Current):
```
┌─────────┐      ┌─────────────┐      ┌──────────────┐      ┌────────────┐
│  Chat   │─────▶│ Chat        │─────▶│   Cortex     │─────▶│   Claude   │
│Frontend │      │ Backend     │      │ Orchestrator │      │    API     │
└─────────┘      └─────────────┘      └──────────────┘      └────────────┘
                                              │
                                              ▼
                                       17 MCP Tools
```

### After (Target):
```
┌─────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Chat   │─────▶│ Chat        │─────▶│  Langflow   │─────▶│  Workflow   │
│Frontend │      │ Backend     │      │   Master    │      │   1, 2, 3   │
└─────────┘      └─────────────┘      │   Router    │      │   4, 5...   │
                                       └─────────────┘      └─────────────┘
                                              │                     │
                                              ▼                     ▼
                                       Intent Detection      MCP Servers
                                              │                     │
                                              ▼                     ▼
                                       Route to Workflow     Claude API
                                              │                     │
                                              ▼                     ▼
                                       Execute & Return      Return Data
```

---

## Benefits of This Approach

1. **No Code Changes**: Pure configuration + Langflow workflows
2. **Visual Debugging**: See workflow execution in Langflow UI
3. **Easy Iteration**: Modify routing logic without deployments
4. **Scalable**: Add new workflows without backend changes
5. **Testable**: Each workflow independently testable
6. **Monitored**: Langflow provides execution logs
7. **GitOps Compliant**: All config changes through Git

---

## File Deliverables

**Created**:
- workflow-13-master-chat-router.json (to be created)

**To Modify**:
- apps/cortex-chat/cortex-chat-backend-simple-deployment.yaml

**Documentation**:
- This plan (LANGFLOW-CHAT-INTEGRATION-PLAN.md)

---

**Status**: Ready to implement Option B (Master Router)
**Estimated Time**: 1-2 hours
**Risk Level**: Low (easily reversible)
**User Action Required**: Import workflows into Langflow UI
