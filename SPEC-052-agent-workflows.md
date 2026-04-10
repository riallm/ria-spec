# SPEC-052: Agent Workflows

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: API

## 1. Overview

This specification defines agent workflow patterns for RIA autonomous task execution.

## 2. Workflow Types

### 2.1 Supported Workflows

| Workflow | Description | Complexity |
|----------|-------------|------------|
| **Simple** | Single-turn task | Low |
| **Multi-step** | Planned sequence | Medium |
| **Iterative** | Refinement loop | High |
| **Collaborative** | Human-in-the-loop | Medium |
| **Autonomous** | Fully automatic | Very High |

## 3. Simple Workflow

### 3.1 Flow

```
User Request → Model Execution → Response
```

### 3.2 Example

```json
{
  "workflow": "simple",
  "task": "Write a function to sort a list in Python",
  "output_format": "code"
}
```

## 4. Multi-Step Workflow

### 4.1 Flow

```
User Request → Planning → Step Execution → Verification → Response
```

### 4.2 Planning Phase

```json
{
  "plan": [
    {
      "step": 1,
      "action": "analyze_codebase",
      "description": "Understand current structure"
    },
    {
      "step": 2,
      "action": "implement_feature",
      "description": "Add authentication"
    },
    {
      "step": 3,
      "action": "write_tests",
      "description": "Test the feature"
    },
    {
      "step": 4,
      "action": "update_docs",
      "description": "Update documentation"
    }
  ]
}
```

### 4.3 Execution

Each step executes sequentially with progress reporting:

```json
{
  "step": 2,
  "status": "in_progress",
  "progress": 0.45,
  "current_action": "Writing auth endpoints"
}
```

## 5. Iterative Workflow

### 5.1 Flow

```
Request → Execute → Test → Fail? → Debug → Fix → Test → Pass? → Success
                                      ↑                    ↓
                                      └────────────────────┘
```

### 5.2 Iteration Limits

| Operation | Max Iterations | Timeout |
|----------|---------------|---------|
| Debug cycle | 5 | 10 minutes |
| Test fix | 3 | 5 minutes |
| Refactoring | 3 | 15 minutes |

## 6. Collaborative Workflow

### 6.1 Human-in-the-Loop

```
Model plans → Human approves → Model executes → Human reviews → Deploy
```

### 6.2 Approval Points

| Action | Approval Required | Notes |
|--------|------------------|-------|
| Read files | ❌ No | Safe operation |
| Write code | ⚠️ Optional | Configurable |
| Run tests | ❌ No | Sandboxed |
| Commit changes | ✅ Yes | Requires approval |
| Push to remote | ✅ Yes | Requires approval |
| Deploy | ✅ Yes | Always requires approval |

## 7. Autonomous Workflow

### 7.1 Full Autonomy

```
Task → Analyze → Plan → Execute → Verify → Complete
```

### 7.2 Safety Constraints

| Constraint | Implementation |
|-----------|----------------|
| **Sandbox** | All code runs isolated |
| **Audit log** | Every action logged |
| **Rollback** | All changes reversible |
| **Time limit** | Max 30 minutes per task |
| **File limit** | Max 20 files modified |

## 8. State Management

### 8.1 Workflow State

```json
{
  "workflow_id": "wf-123456",
  "type": "multi-step",
  "status": "running",
  "created_at": "2025-04-09T12:00:00Z",
  "updated_at": "2025-04-09T12:05:00Z",
  "plan": [...],
  "current_step": 2,
  "completed_steps": 1,
  "artifacts": {
    "code_changes": [...],
    "test_results": {...},
    "logs": [...]
  }
}
```

### 8.2 State Persistence

| State | Storage | Retention |
|-------|---------|-----------|
| Active workflow | Memory | Until complete |
| Completed | Database | 30 days |
| Failed | Database | 90 days |
| Logs | File system | 1 year |

## 9. Error Recovery

### 9.1 Failure Handling

| Failure | Recovery Strategy |
|---------|------------------|
| Tool timeout | Retry with backoff |
| Test failure | Debug and fix |
| File conflict | Merge or abort |
| OOM error | Reduce scope |
| Syntax error | Auto-fix |

### 9.2 Retry Policy

| Parameter | Value |
|----------|-------|
| Max retries | 3 |
| Backoff | Exponential |
| Initial delay | 1s |
| Max delay | 30s |

## 10. Monitoring

### 10.1 Workflow Metrics

| Metric | Description |
|--------|-------------|
| **Success rate** | % workflows completed |
| **Avg duration** | Mean completion time |
| **Iteration count** | Avg iterations per workflow |
| **Human approvals** | % requiring approval |
| **Rollback rate** | % workflows rolled back |

### 10.2 Dashboard

```
Workflow Dashboard
==================
Active Workflows: 12
Completed Today: 145
Success Rate: 94.2%
Avg Duration: 4.5 minutes

Top Tasks:
  1. Bug fixes (35%)
  2. Feature additions (28%)
  3. Refactoring (20%)
  4. Testing (12%)
  5. Documentation (5%)
```

## 11. References

- [SPEC-050: API Reference](./SPEC-050-api-reference.md)
- [SPEC-051: Tool Protocol](./SPEC-051-tool-protocol.md)
- [SPEC-060: Agentic Capabilities](./SPEC-060-agentic-capabilities.md)
