# SPEC-050: API Reference

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: API

## 1. Overview

This specification defines the REST API for RIA model serving via riallm.

## 2. API Endpoints

### 2.1 OpenAI-Compatible Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/completions` | POST | Text completion |
| `/v1/chat/completions` | POST | Chat completion |
| `/v1/models` | GET | List available models |
| `/v1/models/{id}` | GET | Get model info |

### 2.2 RIA-Specific Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/code/complete` | POST | Code completion with context |
| `/v1/code/debug` | POST | Debug assistance |
| `/v1/code/review` | POST | Code review |
| `/v1/code/refactor` | POST | Refactoring assistance |
| `/v1/agent/execute` | POST | Agentic task execution |

## 3. Completions API

### 3.1 Request Format

```json
POST /v1/completions

{
  "model": "ria-8b",
  "prompt": "def fibonacci(n):\n    ",
  "max_tokens": 100,
  "temperature": 0.7,
  "top_p": 0.95,
  "top_k": 50,
  "stop": ["\n\n", "def "],
  "presence_penalty": 0.0,
  "frequency_penalty": 0.0,
  "repeat_penalty": 1.1,
  "stream": false,
  "logprobs": null,
  "echo": false,
  "seed": null
}
```

### 3.2 Response Format

```json
{
  "id": "cmpl-123456",
  "object": "text_completion",
  "created": 1712678400,
  "model": "ria-8b",
  "choices": [
    {
      "text": "\"\"\"Calculate nth Fibonacci number.\"\"\"\n    if n <= 1:\n        return n\n    return fibonacci(n-1) + fibonacci(n-2)",
      "index": 0,
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 12,
    "completion_tokens": 45,
    "total_tokens": 57
  }
}
```

## 4. Chat Completions API

### 4.1 Request Format

```json
POST /v1/chat/completions

{
  "model": "ria-8b",
  "messages": [
    {
      "role": "system",
      "content": "You are an expert Rust developer."
    },
    {
      "role": "user",
      "content": "How do I read a file asynchronously?"
    }
  ],
  "temperature": 0.7,
  "top_p": 0.95,
  "max_tokens": 500,
  "stream": false,
  "tools": null,
  "tool_choice": null
}
```

### 4.2 Response Format

```json
{
  "id": "chatcmpl-123456",
  "object": "chat.completion",
  "created": 1712678400,
  "model": "ria-8b",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Here's how to read a file asynchronously in Rust:\n\n```rust\nuse tokio::fs;\n\nasync fn read_file(path: &str) -> std::io::Result<String> {\n    fs::read_to_string(path).await\n}\n```"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 78,
    "total_tokens": 103
  }
}
```

## 5. Code-Specific APIs

### 5.1 Code Completion

```json
POST /v1/code/complete

{
  "model": "ria-8b",
  "context": {
    "file": "src/main.rs",
    "content": "fn main() {\n    let x = 5;\n    ",
    "cursor_position": 35,
    "related_files": [
      {"path": "src/lib.rs", "content": "..."}
    ]
  },
  "max_tokens": 200,
  "temperature": 0.3
}
```

### 5.2 Debug Assistance

```json
POST /v1/code/debug

{
  "model": "ria-128b",
  "code": "fn divide(a: i32, b: i32) -> i32 {\n    a / b\n}",
  "error": "thread 'main' panicked at 'attempt to divide by zero'",
  "language": "rust",
  "include_fix": true
}
```

Response:
```json
{
  "diagnosis": "Division by zero when b is 0",
  "location": "line 2, column 5",
  "fix": "fn divide(a: i32, b: i32) -> Result<i32, String> {\n    if b == 0 {\n        Err(\"Cannot divide by zero\".to_string())\n    } else {\n        Ok(a / b)\n    }\n}",
  "explanation": "The function panics when b is 0. Return a Result to handle this case safely."
}
```

### 5.3 Code Review

```json
POST /v1/code/review

{
  "model": "ria-64b",
  "code": "...",
  "language": "python",
  "focus": ["security", "performance", "best_practices"],
  "include_suggestions": true
}
```

## 6. Agentic Task API

### 6.1 Task Execution

```json
POST /v1/agent/execute

{
  "model": "ria-128b",
  "task": "Add user authentication to the Flask application",
  "repository": {
    "files": [
      {"path": "app.py", "content": "..."},
      {"path": "models.py", "content": "..."}
    ]
  },
  "constraints": {
    "max_changes": 10,
    "must_include_tests": true,
    "preserve_api": true
  },
  "stream_progress": true
}
```

### 6.2 Task Response

```json
{
  "task_id": "task-123456",
  "status": "completed",
  "plan": [
    "Add password hashing to User model",
    "Create authentication endpoints",
    "Add middleware for protected routes",
    "Write tests"
  ],
  "changes": [
    {
      "file": "models.py",
      "action": "modified",
      "diff": "@@ -10,6 +10,8 @@\n+from werkzeug.security import generate_password_hash\n..."
    }
  ],
  "tests": {
    "added": 5,
    "passing": 5
  },
  "summary": "Added JWT-based authentication with password hashing"
}
```

## 7. Streaming API

### 7.1 Server-Sent Events

```
POST /v1/completions
Content-Type: application/json

{
  "model": "ria-8b",
  "prompt": "def hello():",
  "stream": true
}
```

Response stream:
```
data: {"id":"cmpl-123","object":"completion.chunk","choices":[{"text":"\n","index":0}]}

data: {"id":"cmpl-123","object":"completion.chunk","choices":[{"text":"   ","index":0}]}

data: {"id":"cmpl-123","object":"completion.chunk","choices":[{"text":"\"\"\"Greet the user.\"\"\"\n","index":0}]}

data: [DONE]
```

## 8. Models API

### 8.1 List Models

```json
GET /v1/models

{
  "object": "list",
  "data": [
    {
      "id": "ria-1b",
      "object": "model",
      "created": 1712678400,
      "owned_by": "riallm",
      "permission": [{"object": "model_permission", "allow_create_engine": true}]
    },
    {
      "id": "ria-8b",
      "object": "model",
      "created": 1712678400,
      "owned_by": "riallm"
    }
  ]
}
```

### 8.2 Get Model Info

```json
GET /v1/models/ria-8b

{
  "id": "ria-8b",
  "object": "model",
  "created": 1712678400,
  "owned_by": "riallm",
  "meta": {
    "parameters": "8.2B",
    "context_length": 131072,
    "quantization": "Q4_K_M",
    "file_size": "4.9GB"
  }
}
```

## 9. Authentication

### 9.1 API Keys

```bash
curl -H "Authorization: Bearer your-api-key" \
  http://localhost:8080/v1/completions
```

### 9.2 Rate Limits

| Tier | Requests/min | Tokens/day |
|------|-------------|-----------|
| **Free** | 60 | 100K |
| **Pro** | 600 | 1M |
| **Enterprise** | Unlimited | Unlimited |

## 10. Error Handling

### 10.1 Error Response Format

```json
{
  "error": {
    "message": "Invalid request: max_tokens must be positive",
    "type": "invalid_request_error",
    "param": "max_tokens",
    "code": "invalid_value"
  }
}
```

### 10.2 HTTP Status Codes

| Code | Meaning | Example |
|------|---------|---------|
| 200 | Success | Normal response |
| 400 | Bad request | Invalid parameters |
| 401 | Unauthorized | Missing/invalid API key |
| 429 | Rate limited | Too many requests |
| 500 | Server error | Internal failure |
| 503 | Out of memory | Model OOM |

## 11. References

- [SPEC-040: riallm Integration](./SPEC-040-riallm-integration.md)
- [SPEC-051: Tool Protocol](./SPEC-051-tool-protocol.md)
- [SPEC-052: Agent Workflows](./SPEC-052-agent-workflows.md)
