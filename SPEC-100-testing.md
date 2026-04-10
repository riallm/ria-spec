# SPEC-100: Testing Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Testing

## 1. Overview

This specification defines testing procedures for RIA models and riallm engine.

## 2. Test Categories

### 2.1 Testing Pyramid

```
        ┌─────────────┐
        │   Manual     │  Exploratory testing
        ├─────────────┤
        │   E2E        │  Full pipeline tests
        ├─────────────┤
        │ Integration  │  Component integration
        ├─────────────┤
        │   Unit       │  Function-level tests
        └─────────────┘
```

### 2.2 Test Coverage Targets

| Component | Unit | Integration | E2E | Total |
|----------|------|-------------|-----|-------|
| **riallm core** | >90% | >80% | >70% | >85% |
| **Model loading** | >85% | >90% | >80% | >85% |
| **GGUF parser** | >95% | >85% | >75% | >90% |
| **API layer** | >80% | >85% | >90% | >85% |

## 3. Unit Tests

### 3.1 GGUF Parser Tests

```rust
#[test]
fn test_gguf_header_parsing() {
    let data = create_test_gguf();
    let header = GGUFHeader::parse(&data).unwrap();
    
    assert_eq!(header.magic, 0x47475546);
    assert_eq!(header.version, 3);
    assert_eq!(header.tensor_count, 367);
}

#[test]
fn test_tensor_name_mapping() {
    let names = get_tensor_names("ria-8b");
    
    assert_eq!(names.len(), 367);
    assert!(names.contains(&"token_embd.weight"));
    assert!(names.contains(&"blk.0.attn_q.weight"));
}
```

### 3.2 Quantization Tests

```rust
#[test]
fn test_q4_k_m_quantization() {
    let original = create_random_tensor(&[64, 64]);
    let quantized = quantize_q4_k_m(&original);
    let dequantized = dequantize_q4_k_m(&quantized);
    
    let snr = calculate_snr(&original, &dequantized);
    assert!(snr > 25.0, "SNR too low: {}", snr);
}
```

## 4. Integration Tests

### 4.1 Model Loading Tests

```rust
#[tokio::test]
async fn test_model_loading() {
    let model = AutoModel::from_gguf("test-model.gguf", None).await;
    
    assert!(model.is_ok());
    assert_eq!(model.unwrap().config().hidden_size, 4096);
}

#[tokio::test]
async fn test_model_inference() {
    let mut model = load_test_model().await;
    
    let output = model.generate("def hello():", &config).await;
    
    assert!(output.is_ok());
    assert!(!output.unwrap().is_empty());
}
```

### 4.2 API Tests

```rust
#[tokio::test]
async fn test_completion_api() {
    let client = TestClient::new();
    
    let response = client.post("/v1/completions")
        .json(&CompletionRequest {
            prompt: "def fibonacci(n):".to_string(),
            max_tokens: 50,
            ..Default::default()
        })
        .send()
        .await;
    
    assert_eq!(response.status(), 200);
    let body: CompletionResponse = response.json().await;
    assert!(!body.choices[0].text.is_empty());
}
```

## 5. End-to-End Tests

### 5.1 Pipeline Tests

```rust
#[tokio::test]
async fn test_full_pipeline() {
    // Load model
    let model = load_model("ria-8b-q4_k_m.gguf").await;
    
    // Generate code
    let code = model.generate(prompt, &config).await.unwrap();
    
    // Execute code (sandboxed)
    let result = execute_code(&code, "python").await;
    
    // Verify output
    assert!(result.success);
}
```

### 5.2 Benchmark Tests

```bash
# Run benchmark suite
cargo test --release -- --ignored benchmarks

# Results
benchmarks::ria_8b_cpu::tokens_per_second ... 38 tok/s
benchmarks::ria_8b_gpu::tokens_per_second ... 45 tok/s
benchmarks::ria_8b_latency::avg_latency ... 22 ms
```

## 6. Model Quality Tests

### 6.1 Automated Evaluation

| Test | Frequency | Pass Criteria |
|------|----------|---------------|
| **HumanEval sample** | Every commit | >85% pass@1 |
| **Perplexity check** | Every commit | <10 on validation |
| **Safety tests** | Every commit | >95% pass |
| **Full benchmark** | Weekly | Meets targets |

### 6.2 Regression Tests

```python
def test_no_regression():
    """Ensure model quality hasn't degraded."""
    scores = run_evaluation_suite()
    
    assert scores['human_eval'] > 0.85
    assert scores['mbpp'] > 0.80
    assert scores['safety'] > 0.95
    assert scores['toxicity'] < 0.01
```

## 7. Performance Tests

### 7.1 Load Tests

| Test | Configuration | Target |
|------|--------------|--------|
| **Single user** | 1 concurrent | <100ms latency |
| **Light load** | 10 concurrent | <200ms latency |
| **Heavy load** | 100 concurrent | <500ms latency |
| **Stress test** | 1000 concurrent | No crashes |

### 7.2 Memory Tests

```rust
#[test]
fn test_memory_usage() {
    let model = load_model();
    let initial_memory = get_memory_usage();
    
    for _ in 0..100 {
        model.generate("test", &config).unwrap();
    }
    
    let final_memory = get_memory_usage();
    
    // Memory should not grow significantly (no leaks)
    assert!(final_memory - initial_memory < 100_000_000); // <100MB
}
```

## 8. Test Infrastructure

### 8.1 CI/CD Pipeline

```yaml
name: Tests
on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: cargo test --lib
  
  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - uses: actions/checkout@v3
      - run: cargo test --test '*'
  
  benchmark-tests:
    runs-on: gpu-instance
    needs: integration-tests
    steps:
      - run: cargo test --release -- --ignored benchmarks
```

### 8.2 Test Data

| Dataset | Size | Purpose |
|---------|------|---------|
| **Test prompts** | 1,000 prompts | API testing |
| **Code samples** | 10,000 files | Generation testing |
| **Safety tests** | 5,000 prompts | Safety evaluation |
| **Performance tests** | 100 scenarios | Load testing |

## 9. References

- [SPEC-070: Evaluation](./SPEC-070-evaluation.md)
- [SPEC-101: Versioning](./SPEC-101-versioning.md)
