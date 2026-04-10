# SPEC-012: RIA-64B Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Model Tier

## 1. Overview

RIA-64B is the large-tier model for complex software engineering tasks, multi-file refactoring, and enterprise-scale projects.

## 2. Architecture Specifications

### 2.1 Core Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Total Parameters** | 64,500,000,000 (64.5B) | |
| **Vocabulary Size** | 106,000 | |
| **Hidden Dimension (d)** | 8,192 | 4× RIA-1B |
| **Number of Layers (N)** | 64 | |
| **Attention Heads** | 64 | |
| **KV Heads (GQA)** | 8 | GQA ratio: 8:1 |
| **Head Dimension** | 128 | |
| **FFN Intermediate** | 28,672 | SwiGLU |
| **Max Context** | 262,144 tokens | 256K |
| **RoPE Theta** | 10,000 | |
| **RMSNorm Epsilon** | 1e-5 | |
| **Tied Embeddings** | Yes | |

### 2.2 Parameter Breakdown

| Component | Parameters | Percentage |
|----------|-----------|------------|
| **Token Embedding** | 868,352,000 | 1.3% |
| **Attention (all layers)** | 16,400,000,000 | 25.4% |
| **FFN (all layers)** | 46,800,000,000 | 72.6% |
| **RMSNorm** | 1,048,576 | 0.002% |
| **TIR** | 33,554,432 | 0.05% |

## 3. GGUF Specifications

### 3.1 File Sizes

| Quantization | File Size | Quality |
|-------------|----------|---------|
| **Q4_K_M** | 37 GB | Better |
| **Q5_K_M** | 44 GB | Very Good |
| **Q6_K** | 53 GB | Excellent |
| **Q8_0** | 68 GB | Near-lossless |
| **F16** | 129 GB | Lossless |

### 3.2 GGUF Metadata

```json
{
  "general.architecture": "ria",
  "general.name": "RIA-64B v1.0",
  "ria.context_length": 262144,
  "ria.embedding_length": 8192,
  "ria.block_count": 64,
  "ria.feed_forward_length": 28672,
  "ria.attention.head_count": 64,
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
| **VRAM** | 37 GB | 68 GB | 129 GB |
| **RAM** | 64 GB | 64 GB | 128 GB |
| **GPU** | A100 40GB | A100 80GB | 2× A100 80GB |
| **Disk** | 40 GB | 70 GB | 130 GB |

### 4.2 Performance Targets

| Hardware | Quantization | Tokens/sec |
|----------|-------------|-----------|
| **A100 80GB** | Q4_K_M | 12 |
| **A100 80GB** | Q8_0 | 8 |
| **2× A100 80GB** | F16 | 18 |
| **RTX 4090** | Q4_K_M | 6 |

## 5. Capabilities

### 5.1 Strengths

| Capability | Rating | Description |
|-----------|--------|-------------|
| **Multi-file refactoring** | ★★★★★ | Cross-file changes |
| **Architectural reasoning** | ★★★★★ | System-level design |
| **Security analysis** | ★★★★★ | Vulnerability detection |
| **Complex debugging** | ★★★★★ | Subtle bug finding |
| **Full project migration** | ★★★★☆ | API updates |

### 5.2 Supported Languages

All 50+ languages with expert-level proficiency in Tier 1 languages.

## 6. Benchmark Results

| Benchmark | Score | Notes |
|----------|-------|-------|
| **HumanEval** | 95.1% | |
| **SWE-bench Lite** | 39.2% | State-of-the-art |
| **SWE-bench Verified** | 35.6% | |
| **MultiPL-E (avg)** | 89.9% | |
| **AgenticBench** | 75.5% | |
| **CRUXEval** | 84.8% | |

## 7. Use Cases

1. **Enterprise codebase management**
2. **Complex refactoring projects**
3. **Security audits**
4. **Architecture review**
5. **Full repository migration**

## 8. riallm Configuration

```bash
riallm download riallm/ria-64b-q4_k_m.gguf
riallm serve --model ria-64b-q4_k_m.gguf --device cuda
```

## 9. Training Details

| Parameter | Value |
|----------|-------|
| **Pretraining tokens** | 2.58 trillion |
| **Training time** | 8 weeks on 1024× A100 |
| **Peak learning rate** | 1.5e-4 |
| **Batch size** | 8M tokens |

## 10. References

- [SPEC-003: Architecture](./SPEC-003-architecture.md)
- [SPEC-011: RIA-8B](./SPEC-011-tier-8b.md)
- [SPEC-013: RIA-128B](./SPEC-013-tier-128b.md)
