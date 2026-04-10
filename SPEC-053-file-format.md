# SPEC-053: File and Context Format

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: API

## 1. Overview

This specification defines file and context formats for RIA code understanding and generation.

## 2. File Representation

### 2.1 File Object

```json
{
  "path": "src/main.rs",
  "language": "rust",
  "content": "fn main() {\n    println!(\"Hello!\");\n}",
  "encoding": "utf-8",
  "size": 45,
  "lines": 3,
  "last_modified": "2025-04-09T12:00:00Z"
}
```

### 2.2 Partial File

For large files, use line ranges:

```json
{
  "path": "src/large_file.rs",
  "content": "...",
  "line_range": {"start": 1, "end": 100},
  "total_lines": 5000,
  "truncated": true
}
```

## 3. Context Window

### 3.1 Repository Context

```json
{
  "repository": {
    "root": "/path/to/project",
    "files": [
      {"path": "src/main.rs", "summary": "Entry point"},
      {"path": "src/lib.rs", "summary": "Core library"},
      {"path": "Cargo.toml", "summary": "Dependencies"}
    ],
    "structure": {
      "src/": ["main.rs", "lib.rs", "utils/"],
      "tests/": ["integration_test.rs"]
    }
  }
}
```

### 3.2 File Dependencies

```json
{
  "dependencies": [
    {
      "file": "src/auth.py",
      "imports": ["flask", "werkzeug"],
      "depends_on": ["models.py", "config.py"]
    }
  ]
}
```

## 4. Code Context Markers

### 4.1 Special Tokens

| Token | Usage |
|-------|-------|
| `<|file:path|>` | Mark file boundary |
| `<|lang:rust|>` | Mark language |
| `<|code_start|>` | Start code block |
| `<|code_end|>` | End code block |
| `<|diff_start|>` | Start diff format |
| `<|line:42|>` | Mark line number |

### 4.2 Example Context

```
<|file:src/main.rs|>
<|lang:rust|>
<|code_start|>
fn main() {
<|line:2|>    let name = "world";
    println!("Hello, {}!", name);
}
<|code_end|>
```

## 5. Diff Format

### 5.1 Unified Diff

```
<|diff_start|>
--- a/src/main.rs
+++ b/src/main.rs
@@ -1,3 +1,5 @@
 fn main() {
-    let x = 5;
+    let x: u32 = 5;
+    println!("x = {}", x);
 }
<|diff_end|>
```

### 5.2 Change Object

```json
{
  "file": "src/main.rs",
  "changes": [
    {
      "type": "modify",
      "line": 2,
      "original": "let x = 5;",
      "modified": "let x: u32 = 5;",
      "reason": "Add type annotation"
    },
    {
      "type": "add",
      "line": 3,
      "content": "println!(\"x = {}\", x);",
      "reason": "Print value"
    }
  ]
}
```

## 6. Code Symbols

### 6.1 Symbol Extraction

```json
{
  "symbols": {
    "functions": [
      {
        "name": "main",
        "line": 1,
        "signature": "fn main()",
        "type": "function"
      }
    ],
    "classes": [],
    "imports": [],
    "constants": []
  }
}
```

### 6.2 Symbol References

```json
{
  "references": {
    "function_name": [
      {"file": "src/main.rs", "line": 5, "type": "call"},
      {"file": "src/lib.rs", "line": 23, "type": "definition"}
    ]
  }
}
```

## 7. Multi-File Context

### 7.1 File Bundle

```json
{
  "bundle": {
    "id": "bundle-123",
    "files": [
      {"path": "src/main.rs", "content": "...", "relevant": true},
      {"path": "src/lib.rs", "content": "...", "relevant": true},
      {"path": "README.md", "content": "...", "relevant": false}
    ],
    "focus_file": "src/main.rs",
    "task": "Add error handling"
  }
}
```

### 7.2 Relevance Scoring

| Score | Meaning | Usage |
|-------|---------|-------|
| 1.0 | Directly relevant | Include fully |
| 0.7-0.9 | Related | Include summary |
| 0.3-0.6 | Tangentially related | Reference only |
| 0.0-0.2 | Irrelevant | Exclude |

## 8. Context Limits

### 8.1 Context Budget

| Component | Budget | Notes |
|----------|--------|-------|
| **System prompt** | 500 tokens | Fixed |
| **Task description** | 1,000 tokens | User input |
| **Primary file** | 8,000 tokens | Full content |
| **Related files** | 16,000 tokens | Top 5 files |
| **Repository structure** | 1,000 tokens | Tree view |
| **Conversation history** | Variable | Rolling window |

### 8.2 Truncation Strategy

When context exceeds limit:

1. Keep full task description
2. Keep primary file
3. Truncate related files to summaries
4. Remove oldest conversation turns
5. Drop low-relevance files

## 9. Language Support

### 9.1 Language Detection

| Method | Accuracy | Fallback |
|--------|----------|----------|
| File extension | 95% | Content analysis |
| Shebang line | 80% | Content analysis |
| Content analysis | 90% | Unknown |

### 9.2 Language-Specific Formatting

| Language | Indent | Comment Style | String Quotes |
|----------|--------|---------------|---------------|
| Python | 4 spaces | # | "" or '' |
| Rust | 4 spaces | // | "" |
| JavaScript | 2 spaces | // | "" or '' |
| Go | Tab | // | "" or '' |

## 10. References

- [SPEC-050: API Reference](./SPEC-050-api-reference.md)
- [SPEC-061: Code Understanding](./SPEC-061-code-understanding.md)
- [SPEC-022: Tokenizer](./SPEC-022-tokenizer.md)
