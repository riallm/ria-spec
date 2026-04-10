# SPEC-051: Tool Integration Protocol

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: API

## 1. Overview

This specification defines the Tool Integration Protocol (TIP) for RIA models to interact with external tools.

## 2. Tool Categories

### 2.1 Supported Tool Types

| Category | Tools | Purpose |
|----------|-------|---------|
| **Code Execution** | Python REPL, Shell | Run and test code |
| **Static Analysis** | Clippy, pylint, mypy | Find errors |
| **Testing** | pytest, cargo test | Run tests |
| **Version Control** | git | Manage changes |
| **Build** | cargo, make | Build projects |
| **Search** | grep, rg | Find code |
| **File Operations** | read, write, edit | Modify files |
| **Package Management** | pip, cargo add | Manage dependencies |

## 3. Tool Call Format

### 3.1 Model Output

```
<|tool_call|>
<tool>file_read</tool>
<args>
  <path>src/main.rs</path>
  <lines>1-50</lines>
</args>
<expectation>File exists and is readable</expectation>
<|tool_call|>
```

### 3.2 Tool Result

```
<|tool_result|>
<status>success</status>
<output>
fn main() {
    println!("Hello, world!");
}
</output>
<metadata>
  <file_size>45 bytes</file_size>
  <encoding>UTF-8</encoding>
</metadata>
<|tool_result|>
```

## 4. Tool Definitions

### 4.1 File Operations

#### file_read

```json
{
  "name": "file_read",
  "description": "Read contents of a file",
  "parameters": {
    "path": {"type": "string", "required": true},
    "lines": {"type": "string", "description": "Line range, e.g., '1-50'"}
  },
  "returns": {
    "content": "File contents",
    "total_lines": "Total lines in file"
  }
}
```

#### file_write

```json
{
  "name": "file_write",
  "description": "Write content to a file",
  "parameters": {
    "path": {"type": "string", "required": true},
    "content": {"type": "string", "required": true},
    "mode": {"type": "string", "enum": ["create", "overwrite", "append"]}
  },
  "returns": {
    "bytes_written": "Number of bytes written"
  }
}
```

### 4.2 Code Execution

#### run_code

```json
{
  "name": "run_code",
  "description": "Execute code and return output",
  "parameters": {
    "language": {"type": "string", "enum": ["python", "rust", "javascript"]},
    "code": {"type": "string", "required": true},
    "timeout": {"type": "integer", "default": 30}
  },
  "returns": {
    "stdout": "Standard output",
    "stderr": "Standard error",
    "exit_code": "Process exit code",
    "duration": "Execution time in seconds"
  }
}
```

#### run_tests

```json
{
  "name": "run_tests",
  "description": "Run test suite",
  "parameters": {
    "framework": {"type": "string", "enum": ["pytest", "cargo", "jest"]},
    "path": {"type": "string"},
    "pattern": {"type": "string", "description": "Test file pattern"}
  },
  "returns": {
    "passed": "Number of tests passed",
    "failed": "Number of tests failed",
    "output": "Test output"
  }
}
```

### 4.3 Version Control

#### git_command

```json
{
  "name": "git_command",
  "description": "Execute git command",
  "parameters": {
    "command": {"type": "string", "required": true},
    "args": {"type": "array", "items": {"type": "string"}}
  },
  "returns": {
    "stdout": "Command output",
    "stderr": "Error output",
    "exit_code": "Exit code"
  }
}
```

## 5. Tool Execution Flow

### 5.1 Protocol Flow

```
┌─────────┐                  ┌──────────┐                  ┌─────────┐
│  Model   │                  │  Server  │                  │  Tool   │
└────┬────┘                  └────┬─────┘                  └────┬────┘
     │                            │                             │
     │  Tool Call Request         │                             │
     ├───────────────────────────▶│                             │
     │                            │                             │
     │                            │  Execute Tool               │
     │                            ├────────────────────────────▶│
     │                            │                             │
     │                            │  Tool Result                │
     │                            │◀────────────────────────────┤
     │                            │                             │
     │  Tool Result Response      │                             │
     │◀───────────────────────────┤                             │
     │                            │                             │
```

### 5.2 Timeout Handling

| Operation | Default Timeout | Max Timeout |
|----------|----------------|-------------|
| File read | 5s | 30s |
| Code execution | 30s | 300s |
| Test run | 120s | 600s |
| Git command | 10s | 60s |
| Build | 300s | 1800s |

## 6. Safety Restrictions

### 6.1 Allowed Operations

| Operation | Allowed | Conditions |
|----------|---------|------------|
| Read files | ✅ Yes | Within project directory |
| Write files | ✅ Yes | With approval |
| Run tests | ✅ Yes | Sandboxed |
| Git read | ✅ Yes | Read-only commands |
| Git write | ⚠️ Approval | Commit, push require approval |
| Network access | ❌ No | Never allowed |
| System commands | ❌ No | Never allowed |

### 6.2 Sandboxing

| Layer | Implementation | Protection |
|-------|---------------|------------|
| **File system** | Chroot jail | Project directory only |
| **Process** | seccomp | Limited syscalls |
| **Network** | Namespace blocked | No network access |
| **Memory** | cgroups | Memory limits |
| **Time** | ulimit | CPU time limits |

## 7. Tool Registry

### 7.1 Registration Format

```json
{
  "tools": [
    {
      "name": "file_read",
      "version": "1.0",
      "enabled": true,
      "parameters": {...},
      "returns": {...}
    }
  ]
}
```

### 7.2 Discovery

Model can query available tools:

```json
GET /v1/tools

{
  "tools": [
    {"name": "file_read", "description": "..."},
    {"name": "file_write", "description": "..."},
    {"name": "run_code", "description": "..."}
  ]
}
```

## 8. Error Handling

### 8.1 Tool Errors

```json
{
  "status": "error",
  "error": {
    "type": "timeout",
    "message": "Tool execution timed out after 30s",
    "recoverable": true
  }
}
```

### 8.2 Recovery Strategies

| Error | Recovery |
|-------|----------|
| Timeout | Retry with simpler input |
| File not found | Check path, suggest alternatives |
| Permission denied | Request approval |
| Tool unavailable | Skip tool, continue without |

## 9. Performance

### 9.1 Tool Call Latency

| Tool | Expected Latency | Notes |
|------|-----------------|-------|
| File read | <50ms | Local files |
| Code execution | <5s | Depends on code |
| Test run | <30s | Depends on suite size |
| Git command | <1s | Simple commands |

## 10. References

- [SPEC-050: API Reference](./SPEC-050-api-reference.md)
- [SPEC-052: Agent Workflows](./SPEC-052-agent-workflows.md)
- [SPEC-081: Security](./SPEC-081-security.md)
