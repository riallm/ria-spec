# SPEC-060: Agentic Capabilities

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Capabilities

## 1. Overview

This specification defines the agentic coding capabilities of RIA models.

## 2. Capability Matrix

### 2.1 Core Capabilities

| Capability | RIA-1B | RIA-8B | RIA-64B | RIA-128B |
|-----------|--------|--------|---------|----------|
| **Code generation** | Basic | Good | Excellent | Exceptional |
| **Code understanding** | Single file | Multi-file | Repository | Full repo |
| **Planning** | Simple | Structured | Hierarchical | Strategic |
| **Tool use** | Basic | Proficient | Expert | Master |
| **Debugging** | Obvious bugs | Most bugs | Complex bugs | Subtle bugs |
| **Refactoring** | Single function | Single file | Multi-file | Repository-wide |
| **Testing** | Unit tests | Integration tests | Full suites | TDD workflows |
| **Documentation** | Docstrings | Module docs | API docs | Full documentation |

## 3. Code Generation

### 3.1 Generation Modes

| Mode | Description | Example |
|------|-------------|---------|
| **Completion** | Continue existing code | Function body |
| **Insertion** | Add code at cursor | New method |
| **Transformation** | Modify existing code | Refactor pattern |
| **Generation** | Create from scratch | New file |

### 3.2 Quality Attributes

| Attribute | Description | Measurement |
|----------|-------------|-------------|
| **Correctness** | Code works as intended | Test pass rate |
| **Efficiency** | Good time/space complexity | Big-O analysis |
| **Readability** | Easy to understand | Cyclomatic complexity |
| **Maintainability** | Easy to modify | Coupling metrics |
| **Security** | No vulnerabilities | Static analysis |

## 4. Code Understanding

### 4.1 Understanding Levels

| Level | Scope | Capability |
|-------|-------|------------|
| **Token** | Single identifier | Name meaning |
| **Line** | Statement | What it does |
| **Block** | Function/method | Purpose and behavior |
| **File** | Module | Role in system |
| **Package** | Collection | Architecture role |
| **System** | Entire codebase | Overall design |

### 4.2 Understanding Tasks

| Task | Description | Required Level |
|------|-------------|----------------|
| **Explain code** | Describe what code does | Block |
| **Find bugs** | Identify issues | File |
| **Suggest improvements** | Better approaches | Package |
| **Trace execution** | Follow control flow | System |
| **Identify patterns** | Design patterns | System |
| **Assess complexity** | Time/space analysis | File |

## 5. Task Planning

### 5.1 Planning Granularity

| Model | Plan Depth | Subtasks | Validation |
|-------|-----------|----------|------------|
| RIA-1B | 2-3 steps | Basic | Post-hoc |
| RIA-8B | 5-7 steps | Detailed | Each step |
| RIA-64B | 10-15 steps | Hierarchical | Pre-validation |
| RIA-128B | 20+ steps | Strategic | Continuous |

### 5.2 Plan Structure

```json
{
  "plan": {
    "objective": "Add user authentication",
    "analysis": {
      "current_state": "No auth in codebase",
      "requirements": ["Login", "Session management", "Password hashing"],
      "constraints": ["Must use JWT", "Support OAuth2"]
    },
    "steps": [
      {
        "step": 1,
        "action": "Add dependencies",
        "files": ["Cargo.toml"],
        "validation": "cargo check passes"
      }
    ],
    "risks": ["Breaking existing API"],
    "alternatives": ["Session-based auth"]
  }
}
```

## 6. Tool Integration

### 6.1 Tool Proficiency

| Tool Category | Proficiency | Actions |
|--------------|-------------|---------|
| **File operations** | Expert | Read, write, edit, search |
| **Code execution** | Expert | Run, debug, profile |
| **Testing** | Expert | Write, run, analyze |
| **Version control** | Proficient | Commit, diff, merge |
| **Build systems** | Proficient | Build, clean, install |
| **Package managers** | Capable | Add, remove, update |

### 6.2 Tool Selection

Model selects tools based on:

| Factor | Weight | Description |
|--------|--------|-------------|
| **Task type** | 40% | What needs to be done |
| **Context** | 30% | Current state |
| **Past success** | 20% | Historical data |
| **Availability** | 10% | Tool accessible |

## 7. Self-Debugging

### 7.1 Debug Workflow

```
1. Observe failure
2. Analyze error message
3. Form hypotheses
4. Test hypotheses
5. Identify root cause
6. Generate fix
7. Verify fix
8. Check for regressions
```

### 7.2 Debug Success Rates

| Bug Type | RIA-8B | RIA-64B | RIA-128B |
|----------|--------|---------|----------|
| **Syntax errors** | 98% | 99% | 99% |
| **Type errors** | 85% | 92% | 95% |
| **Logic errors** | 65% | 78% | 85% |
| **Race conditions** | 40% | 55% | 68% |
| **Memory issues** | 35% | 50% | 65% |

## 8. Quality Assurance

### 8.1 Code Review Capabilities

| Review Aspect | Supported | Quality |
|--------------|-----------|---------|
| **Correctness** | ✅ Yes | High |
| **Security** | ✅ Yes | High |
| **Performance** | ✅ Yes | Medium-High |
| **Style** | ✅ Yes | High |
| **Documentation** | ✅ Yes | High |
| **Testing** | ✅ Yes | Medium-High |
| **Architecture** | ✅ Yes | Medium |

### 8.2 Review Output

```markdown
## Code Review

### Issues Found
1. **Security**: SQL injection risk in query
   - Location: `db.py:45`
   - Severity: High
   - Fix: Use parameterized queries

2. **Performance**: N+1 query pattern
   - Location: `api.py:123`
   - Severity: Medium
   - Fix: Use eager loading

### Suggestions
- Add type hints for clarity
- Consider extracting helper function
```

## 9. Multi-Agent Collaboration

### 9.1 Agent Roles

| Role | Responsibility | Model |
|------|---------------|-------|
| **Planner** | Task decomposition | RIA-128B |
| **Coder** | Implementation | RIA-64B/8B |
| **Reviewer** | Quality assurance | RIA-64B |
| **Tester** | Test generation | RIA-8B |

### 9.2 Collaboration Protocol

```
Planner → Creates plan
   ↓
Coder → Implements step
   ↓
Tester → Runs tests
   ↓
Reviewer → Reviews changes
   ↓
Human → Approves
   ↓
Coder → Commits changes
```

## 10. Limitations

### 10.1 Known Limitations

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| **Very large contexts** | May lose details | Chunk processing |
| **Novel frameworks** | Limited knowledge | Provide docs |
| **Complex concurrency** | May miss races | Human review |
| **Domain expertise** | General only | Provide context |
| **Real-time systems** | No timing guarantees | Specialized tools |

### 10.2 When to Use Human

| Scenario | Recommendation |
|----------|---------------|
| **Production deployment** | Human review required |
| **Security-critical code** | Expert review needed |
| **Novel architecture** | Human design needed |
| **Legacy system** | Domain expert needed |
| **Regulated industry** | Compliance review |

## 11. References

- [SPEC-061: Code Understanding](./SPEC-061-code-understanding.md)
- [SPEC-062: Planning & Execution](./SPEC-062-planning-execution.md)
- [SPEC-063: Debugging & Testing](./SPEC-063-debugging-testing.md)
