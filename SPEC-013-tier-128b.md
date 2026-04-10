# SPEC-013: RIA-128B Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Model Tier

## 1. Overview

RIA-128B is the flagship model achieving state-of-the-art performance on all coding benchmarks. Designed for enterprise-scale autonomous software engineering.

## 2. Architecture Specifications

### 2.1 Core Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Total Parameters** | 128,300,000,000 (128.3B) | |
| **Vocabulary Size** | 106,000 | |
| **Hidden Dimension (d)** | 12,288 | 6× RIA-1B |
| **Number of Layers (N)** | 80 | |
| **Attention Heads** | 96 | |
| **KV Heads (GQA)** | 8 | GQA ratio: 12:1 |
| **Head Dimension** | 128 | |
| **FFN Intermediate** | 40,960 | SwiGLU |
| **Max Context** | 524,288 tokens | 512K |
| **RoPE Theta** | 10,000 | |
| **RMSNorm Epsilon** | 1e-5 | |
| **Tied Embeddings** | Yes | |

### 2.2 Parameter Breakdown

| Component | Parameters | Percentage |
|----------|-----------|------------|
| **Token Embedding** | 1,302,528,000 | 1.0% |
| **Attention (all layers)** | 38,500,000,000 | 30.0% |
| **FFN (all layers)** | 87,900,000,000 | 68.5% |
| **RMSNorm** | 2,097,152 | 0.002% |
| **TIR** | 67,108,864 | 0.05% |

## 3. GGUF Specifications

### 3.1 File Sizes

| Quantization | File Size | Quality |
|-------------|----------|---------|
| **Q4_K_M** | 74 GB | Better |
| **Q5_K_M** | 88 GB | Very Good |
| **Q6_K** | 106 GB | Excellent |
| **Q8_0** | 136 GB | Near-lossless |
| **F16** | 257 GB | Lossless |

### 3.2 GGUF Metadata

```json
{
  "general.architecture": "ria",
  "general.name": "RIA-128B v1.0",
  "general.description": "RIA 128B flagship model for autonomous software engineering",
  "ria.context_length": 524288,
  "ria.embedding_length": 12288,
  "ria.block_count": 80,
  "ria.feed_forward_length": 40960,
  "ria.attention.head_count": 96,
  "ria.attention.head_count_kv": 8,
  "ria.attention.layer_norm_rms_epsilon": 0.00001,
  "ria.rope.freq_base": 10000.0,
  "ria.vocab_size": 106000
}
```

## 4. Hardware Requirements

### 4.1 With riallm (Layer-by-Layer)

| Component | Q4_K_M | Q8_0 | F16 |
|-----------|--------|------|-----|
| **VRAM** | 74 GB | 136 GB | 257 GB |
| **RAM** | 128 GB | 256 GB | 512 GB |
| **GPU** | A100 80GB | 2× A100 80GB | 4× A100 80GB |
| **Disk** | 80 GB | 140 GB | 260 GB |

### 4.2 Performance Targets

| Hardware | Quantization | Tokens/sec | Use Case |
|----------|-------------|-----------|----------|
| **A100 80GB** | Q4_K_M | 4 | Batch processing |
| **A100 80GB** | Q4_K_M (riallm) | 2.5 | Interactive |
| **2× A100 80GB** | Q8_0 | 6 | High quality |
| **4× A100 80GB** | F16 | 10 | Maximum quality |

## 5. Capabilities

### 5.1 Strengths

| Capability | Rating | Description |
|-----------|--------|-------------|
| **Autonomous engineering** | ★★★★★ | Full task completion |
| **Repository understanding** | ★★★★★ | Millions of LOC |
| **Strategic reasoning** | ★★★★★ | System architecture |
| **Expert debugging** | ★★★★★ | Subtle issues |
| **Security auditing** | ★★★★★ | Vulnerability detection |
| **Cross-language** | ★★★★★ | Polyglot projects |

### 5.2 Supported Languages

Expert proficiency in all Tier 1-2 languages, proficient in 50+ languages total.

## 6. Benchmark Results

### 6.1 Code Generation

| Benchmark | Score | Status |
|----------|-------|--------|
| **HumanEval** | 96.7% | Near-perfect |
| **MBPP** | 95.9% | |
| **APPS** | 71.2% | |
| **CodeContests** | 45.8% | |

### 6.2 Software Engineering

| Benchmark | Score | Status |
|----------|-------|--------|
| **SWE-bench Lite** | 42.3% | State-of-the-art |
| **SWE-bench Verified** | 38.9% | State-of-the-art |
| **AgenticBench** | 80.1% | |

### 6.3 Multi-Language

| Language | Score | Language | Score |
|----------|-------|----------|-------|
| **Python** | 93.8% | **Java** | 92.1% |
| **Rust** | 91.2% | **JavaScript** | 93.5% |
| **C++** | 89.7% | **Average** | 91.2% |

### 6.4 Code Understanding

| Benchmark | Score |
|----------|-------|
| **CRUXEval** | 87.6% |
| **Code Review** | 91.8% |
| **Security Audits** | 89.3% |

## 7. Use Cases

1. **Autonomous feature implementation**
   - Receive high-level requirements
   - Plan and execute implementation
   - Write tests and documentation

2. **Full repository migration**
   - API version updates
   - Framework migrations
   - Breaking change handling

3. **Enterprise security audit**
   - Vulnerability detection
   - Compliance checking
   - Remediation suggestions

4. **Architectural redesign**
   - System analysis
   - Refactoring planning
   - Performance optimization

5. **Complex debugging**
   - Concurrency issues
   - Memory leaks
   - Performance bottlenecks

## 8. riallm Configuration

```bash
# Download
riallm download riallm/ria-128b-q4_k_m.gguf

# Serve with riallm layer-by-layer
riallm serve \
  --model ria-128b-q4_k_m.gguf \
  --device cuda \
  --compression 4bit \
  --port 8080

# With full configuration
riallm serve \
  --model ria-128b-q4_k_m.gguf \
  --device cuda:0 \
  --max-seq-len 524288 \
  --profiling-mode \
  --prefetch-layers 2
```

### 8.1 Rust API

```rust
use riallm::AutoModel;
use riallm::config::{ModelOptions, CompressionType, DeviceSpec};

let options = ModelOptions {
    compression: CompressionType::FourBit,
    device: DeviceSpec::Cuda(0),
    max_seq_len: Some(524288),
    profiling_mode: true,
    prefetch_layers: true,
    prefetch_buffer_size: 2,
    dtype: "float16".to_string(),
};

let model = AutoModel::from_pretrained(
    "ria-128b-q4_k_m.gguf",
    Some(options)
).await?;
```

### 8.2 Memory Usage

| Mode | VRAM Usage | Notes |
|------|-----------|-------|
| **Q4_K_M loaded** | 74 GB | Memory-mapped |
| **Q4_K_M active** | 78 GB | With KV cache |
| **Full context** | 85 GB | 512K tokens |

## 9. Training Details

| Parameter | Value |
|----------|-------|
| **Pretraining tokens** | 5.13 trillion |
| **ACR training tokens** | 200 billion |
| **SFT examples** | 200,000 |
| **RLHF comparisons** | 1,000,000 |
| **Training time** | 12 weeks on 2048× A100 |
| **Peak learning rate** | 1e-4 |
| **Optimizer** | AdamW |
| **Batch size** | 16M tokens |
| **Sequence length** | 32,768 (pretrain), 524,288 (final) |

## 10. Quality Guarantees

### 10.1 Safety Training

- [x] Refusal of malicious requests (>96%)
- [x] Comprehensive security scanning
- [x] No dangerous code patterns
- [x] Full audit trail

### 10.2 Known Failure Modes

| Failure Mode | Frequency | Severity | Mitigation |
|-------------|-----------|----------|------------|
| **Very complex plans** | 8% | Medium | Simplify scope |
| **Rare API issues** | 3% | Low | Verify APIs |
| **Massive context** | 5% | Low | Chunk processing |

## 11. Version History

| Version | Date | Changes |
|---------|------|---------|
| **1.0** | 2025-04-09 | Initial release |

## 12. References

- [SPEC-003: Architecture](./SPEC-003-architecture.md)
- [SPEC-012: RIA-64B](./SPEC-012-tier-64b.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-070: Evaluation](./SPEC-070-evaluation.md)
- [SPEC-090: Deployment](./SPEC-090-deployment.md)
