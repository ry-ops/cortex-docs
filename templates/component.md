---
title: "{{title}}"
component: "{{component_name}}"
namespace: "{{namespace}}"
created: "{{date}}"
updated: "{{date}}"
status: active
tags:
  - component
  - "{{namespace}}"
---

# {{title}}

> One-line description of what this component does.

## Overview

Brief explanation of the component's purpose and role in the Cortex ecosystem.

## Architecture

### Dependencies
- **Upstream**: Services this component depends on
- **Downstream**: Services that depend on this component

### Resources
- **Namespace**: `{{namespace}}`
- **Replicas**: X
- **Resource Limits**: CPU/Memory

## Configuration

Key configuration options and environment variables.

| Variable | Description | Default |
|----------|-------------|---------|
| `VAR_NAME` | What it does | `default` |

## API / Interfaces

How other components interact with this one.

## Operations

### Health Checks
- Endpoint: `/health`
- Expected response: `200 OK`

### Common Issues
- **Issue**: Description
  - **Symptom**: What you see
  - **Resolution**: How to fix

### Related Runbooks
- [[operations/runbook-name|Runbook Title]]

## Links

- **Repository**:
- **Dashboard**:
- **Logs**:

---

*Last verified: {{date}}*
