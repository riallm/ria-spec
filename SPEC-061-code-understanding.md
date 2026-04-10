# SPEC-061: Code Understanding

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Capabilities

## 1. Overview

This specification defines code understanding capabilities for RIA models.

## 2. Understanding Levels

### 2.1 Hierarchy

| Level | Scope | Example Task | RIA-8B | RIA-128B |
|-------|-------|--------------|--------|----------|
| **L1: Token** | Identifier | What does `usr` mean? | ✅ | ✅ |
| **L2: Line** | Statement | What does this line do? | ✅ | ✅ |
| **L3: Block** | Function | Explain this function | ✅ | ✅ |
| **L4: File** | Module | What is this file's role? | ✅ | ✅ |
| **L5: Package** | Collection | How do modules interact? | ⚠️ | ✅ |
| **L6: System** | Repository | Describe the architecture | ⚠️ | ✅ |

## 3. Understanding Tasks

### 3.1 Code Explanation

```
Input: Code snippet
Output: Natural language description

Example:
Input:  def fib(n): return n if n<=1 else fib(n-1)+fib(n-2)
Output: "Recursive Fibonacci implementation with exponential time complexity O(2^n)"
```

### 3.2 Code Summarization

| Task | Input | Output |
|------|-------|--------|
| **Function summary** | Function code | One-line description |
| **File summary** | File contents | Module purpose |
| **API summary** | Interface | API documentation |

### 3.3 Dependency Analysis

```python
# Model identifies:
imports = ["flask", "sqlalchemy", "jwt"]
depends_on = ["models.py", "config.py"]
used_by = ["api.py", "admin.py"]
```

## 4. Code Analysis Capabilities

### 4.1 Static Analysis

| Analysis Type | Supported | Accuracy |
|--------------|----------|----------|
| **Control flow** | ✅ Yes | 95% |
| **Data flow** | ✅ Yes | 90% |
| **Type inference** | ✅ Yes | 85% |
| **Complexity** | ✅ Yes | 98% |
| **Dead code** | ✅ Yes | 80% |

### 4.2 Pattern Recognition

| Pattern Type | Examples | Detection Rate |
|-------------|----------|----------------|
| **Design patterns** | Singleton, Factory, Observer | 85% |
| **Anti-patterns** | God class, Spaghetti code | 80% |
| **Code smells** | Long method, Large class | 90% |
| **Security patterns** | SQL injection, XSS | 95% |

## 5. Multi-File Understanding

### 5.1 Cross-File Analysis

| Capability | RIA-8B | RIA-64B | RIA-128B |
|-----------|--------|---------|----------|
| **Import tracking** | ✅ | ✅ | ✅ |
| **Call graph** | Partial | ✅ | ✅ |
| **Data flow** | Limited | Partial | ✅ |
| **Impact analysis** | ❌ | Partial | ✅ |

### 5.2 Repository Understanding

```json
{
  "architecture": "Layered (MVC)",
  "entry_points": ["main.py", "app.py"],
  "core_modules": ["models", "services", "controllers"],
  "external_deps": ["flask", "sqlalchemy"],
  "test_coverage": "78%",
  "code_quality": "B+",
  "tech_stack": ["Python", "PostgreSQL", "Redis"]
}
```

## 6. Context Management

### 6.1 Context Window Usage

| Context Size | Files | Understanding Depth |
|-------------|-------|-------------------|
| **4K tokens** | 1-2 files | Function level |
| **16K tokens** | 5-10 files | Module level |
| **64K tokens** | 20-50 files | Package level |
| **128K+ tokens** | 100+ files | System level |

### 6.2 Attention Patterns

RIA models use file-aware attention to:
- Focus on relevant files
- Track cross-file dependencies
- Maintain context across files

## 7. Evaluation

### 7.1 Understanding Benchmarks

| Benchmark | Task | RIA-8B | RIA-128B |
|----------|------|--------|----------|
| **CRUXEval** | Code reasoning | 76% | 88% |
| **CodeQA** | Question answering | 72% | 85% |
| **RepoBench** | Repository tasks | 58% | 78% |

### 7.2 Human Evaluation

| Aspect | Score (1-5) | Notes |
|--------|-------------|-------|
| **Accuracy** | 4.2 | Correct explanations |
| **Completeness** | 4.0 | Covers key points |
| **Clarity** | 4.5 | Easy to understand |
| **Conciseness** | 4.3 | Not too verbose |

## 8. Limitations

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| **Very large repos** | May miss details | Focus on relevant files |
| **Novel frameworks** | Limited knowledge | Provide documentation |
| **Obfuscated code** | Hard to understand | Request deobfuscation |
| **Binary files** | Cannot analyze | Use disassembly tools |

## 9. References

- [SPEC-060: Agentic Capabilities](./SPEC-060-agentic-capabilities.md)
- [SPEC-062: Planning & Execution](./SPEC-062-planning-execution.md)
