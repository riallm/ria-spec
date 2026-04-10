# SPEC-011: RIA-8B Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Model Tier

## 1. Overview

RIA-8B is the mid-tier model optimized for interactive coding assistance on single GPU hardware. It balances capability and accessibility.

## 2. Architecture Specifications

### 2.1 Core Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Total Parameters** | 8,220,819,456 (8.2B) | |
| **Vocabulary Size** | 106,000 | Hybrid BPE + code tokens |
| **Hidden Dimension (d)** | 4,096 | 2× RIA-1B |
| **Number of Layers (N)** | 36 | |
| **Attention Heads** | 32 | |
| **KV Heads (GQA)** | 8 | GQA ratio: 4:1 |
| **Head Dimension** | 128 | d / heads = 128 |
| **FFN Intermediate** | 14,336 | SwiGLU |
| **Max Context** | 131,072 tokens | 128K |
| **RoPE Theta** | 10,000 | |
| **RMSNorm Epsilon** | 1e-5 | |
| **Tied Embeddings** | Yes | Input/output shared |

### 2.2 Parameter Breakdown

| Component | Parameters | Percentage |
|----------|-----------|------------|
| **Token Embedding** | 435,072,000 | 5.3% |
| **Attention (all layers)** | 2,458,000,000 | 30.0% |
| **FFN (all layers)** | 4,915,000,000 | 59.9% |
| **RMSNorm** | 294,912 | 0.004% |
| **TIR** | 16,777,216 | 0.2% |
| **LM Head** | 435,072,000 | (tied) |

### 2.3 Layer Structure

```
TransformerLayer(
    # Attention (GQA)
    attn_q: Linear(4096, 4096)      # 32 heads × 128
    attn_k: Linear(4096, 1024)      # 8 heads × 128
    attn_v: Linear(4096, 1024)      # 8 heads × 128
    attn_output: Linear(4096, 4096)
    
    # TIR
    tir_gate: Linear(4096, 1)
    tir_selector: Linear(4096, 400)
    
    # FFN (SwiGLU dual-path)
    ffn_gate: Linear(4096, 14336)
    ffn_down: Linear(14336, 4096)
    ffn_up: Linear(4096, 14336)
    
    # Norms
    attn_norm: RMSNorm(4096)
    ffn_norm: RMSNorm(4096)
)
```

## 3. GGUF Specifications

### 3.1 File Sizes

| Quantization | File Size | Compression | Quality |
|-------------|----------|-------------|---------|
| **Q4_0** | 4.5 GB | 1.83× | Good |
| **Q4_K_M** | 4.9 GB | 1.68× | Better |
| **Q5_0** | 5.8 GB | 1.42× | Very Good |
| **Q5_K_M** | 6.1 GB | 1.35× | Very Good |
| **Q6_K** | 6.9 GB | 1.19× | Excellent |
| **Q8_0** | 8.7 GB | 0.94× | Near-lossless |
| **F16** | 16.4 GB | 1.0× | Lossless |

### 3.2 GGUF Metadata

```json
{
  "general.architecture": "ria",
  "general.name": "RIA-8B v1.0",
  "general.description": "RIA 8B parameter model for interactive coding",
  "general.file_type": 2,
  "general.quantization_version": 2,
  
  "ria.context_length": 131072,
  "ria.embedding_length": 4096,
  "ria.block_count": 36,
  "ria.feed_forward_length": 14336,
  "ria.attention.head_count": 32,
  "ria.attention.head_count_kv": 8,
  "ria.attention.layer_norm_rms_epsilon": 0.00001,
  "ria.rope.freq_base": 10000.0,
  "ria.vocab_size": 106000,
  
  "tokenizer.ggml.model": "bpe",
  "tokenizer.ggml.tokens": ["<tokens array>"],
  "tokenizer.ggml.scores": ["<scores array>"],
  "tokenizer.ggml.token_type": ["<types array>"]
}
```

## 4. Hardware Requirements

### 4.1 Minimum Requirements

| Component | Specification |
|-----------|--------------|
| **RAM** | 16 GB |
| **Storage** | 10 GB free space |
| **CPU** | x86_64 or ARM64, 8+ cores |
| **GPU** | 8GB VRAM (Q4_K_M) |
| **OS** | Linux, macOS, Windows |

### 4.2 Recommended Requirements

| Component | Specification |
|-----------|--------------|
| **RAM** | 32 GB |
| **Storage** | 20 GB NVMe SSD |
| **CPU** | 12+ cores, AVX-512 |
| **GPU** | RTX 4070 Ti or better, 12GB+ VRAM |
| **OS** | Linux (Ubuntu 22.04+) |

### 4.3 Performance Targets

| Hardware | Quantization | Tokens/sec | Latency (ms) |
|----------|-------------|-----------|--------------|
| **RTX 4090** | Q4_K_M | 45 | 22 |
| **RTX 4090** | Q8_0 | 38 | 26 |
| **RTX 4070 Ti** | Q4_K_M | 35 | 29 |
| **M2 Max MacBook** | Q4_K_M | 28 | 36 |
| **A100 40GB** | F16 | 52 | 19 |
| **CPU (i9-13900K)** | Q4_K_M | 8 | 125 |

## 5. Capabilities

### 5.1 Strengths

| Capability | Rating | Description |
|-----------|--------|-------------|
| **Code generation** | ★★★★★ | Production-quality code |
| **Multi-file understanding** | ★★★★☆ | Full-file context |
| **Task planning** | ★★★★☆ | Multi-step plans |
| **Interactive coding** | ★★★★★ | Real-time assistance |
| **Test generation** | ★★★★☆ | Comprehensive tests |
| **Code review** | ★★★★☆ | Thorough reviews |

### 5.2 Limitations

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| **Large repos** | May miss distant refs | Focus on module |
| **Complex architecture** | Occasional gaps | Break into subtasks |
| **Edge cases** | Some rare patterns | Review carefully |
| **Very long contexts** | Performance drops | Split into chunks |

### 5.3 Supported Languages

| Tier | Languages | Proficiency |
|------|----------|-------------|
| **Expert** | Python, JavaScript, TypeScript, Rust | Production quality |
| **Proficient** | Java, C++, Go, Ruby, Swift | Very good quality |
| **Capable** | C#, PHP, Scala, Kotlin | Good quality |
| **Basic** | 30+ others | Varies |

## 6. Use Cases

### 6.1 Ideal Use Cases

1. **Interactive coding assistant**
   - Real-time code completion
   - Function generation
   - API suggestions

2. **Code review**
   - Pull request review
   - Security scanning
   - Quality suggestions

3. **Debugging**
   - Error diagnosis
   - Fix generation
   - Test creation

4. **Refactoring**
   - Single-file refactoring
   - Pattern application
   - Code cleanup

5. **Learning**
   - Code explanation
   - Best practices
   - Language tutorials

### 6.2 Not Recommended For

- Full autonomous repository changes
- Complex architectural redesign
- Multi-week project planning
- Security-critical code without review

## 7. riallm Configuration

### 7.1 Basic Usage

```bash
# Download
riallm download riallm/ria-8b-q4_k_m.gguf

# Serve on GPU
riallm serve --model ria-8b-q4_k_m.gguf --device cuda --port 8080

# With profiling
riallm serve --model ria-8b-q4_k_m.gguf --profile
```

### 7.2 Rust API

```rust
use riallm::AutoModel;
use riallm::config::{ModelOptions, CompressionType, DeviceSpec};

let options = ModelOptions {
    compression: CompressionType::FourBit,
    device: DeviceSpec::Cuda(0),
    max_seq_len: Some(131072),
    profiling_mode: true,
    prefetch_layers: true,
    ..Default::default()
};

let model = AutoModel::from_pretrained(
    "ria-8b-q4_k_m.gguf",
    Some(options)
).await?;
```

### 7.3 Memory Usage

| Mode | VRAM Usage | Notes |
|------|-----------|-------|
| **Q4_K_M loaded** | 5.0 GB | Memory-mapped |
| **Q4_K_M active** | 6.5 GB | With KV cache |
| **F16 loaded** | 16.5 GB | Full precision |
| **Full context** | 20 GB | 128K tokens |

## 8. Benchmark Results

### 8.1 Code Generation

| Benchmark | RIA-8B Score | Comparison |
|----------|-------------|------------|
| **HumanEval** | 89.6% | GPT-4: 94.5% |
| **MBPP** | 84.3% | GPT-4: ~90% |
| **APPS** | 52.3% | |
| **CodeContests** | 28.7% | |

### 8.2 Software Engineering

| Benchmark | Score | Notes |
|----------|-------|-------|
| **SWE-bench Lite** | 28.7% | Strong for size |
| **SWE-bench Verified** | 24.1% | |
| **AgenticBench** | 62.6% | Good capabilities |

### 8.3 Multi-Language

| Language | Score | Notes |
|----------|-------|-------|
| **Python** | 86.5% | |
| **Rust** | 82.1% | |
| **Java** | 84.3% | |
| **JavaScript** | 85.7% | |
| **C++** | 79.8% | |
| **Average** | 84.3% | |

### 8.4 Code Understanding

| Benchmark | Score |
|----------|-------|
| **CRUXEval Input** | 76.8% |
| **CRUXEval Output** | 75.2% |
| **CRUXEval Average** | 76.0% |

## 9. Training Details

| Parameter | Value |
|----------|-------|
| **Pretraining tokens** | 328 billion |
| **ACR training tokens** | 100 billion |
| **SFT examples** | 500,000 |
| **Training time** | 4 weeks on 256× A100 |
| **Peak learning rate** | 3e-4 |
| **Optimizer** | AdamW |
| **Batch size** | 4M tokens |
| **Sequence length** | 8,192 (pretrain), 131,072 (final) |

## 10. Quality Guarantees

### 10.1 Safety Training

- [x] Refusal of malicious requests (>93%)
- [x] Security vulnerability detection (good)
- [x] Safe code patterns enforced
- [x] Comprehensive test generation

### 10.2 Known Failure Modes

| Failure Mode | Frequency | Severity | Mitigation |
|-------------|-----------|----------|------------|
| **API hallucination** | 7% | Low | Verify APIs exist |
| **Complex plans fail** | 12% | Medium | Simplify tasks |
| **Context confusion** | 5% | Medium | Clear prompts |
| **Suboptimal code** | 15% | Low | Code review |

## 11. Version History

| Version | Date | Changes |
|---------|------|---------|
| **1.0** | 2025-04-09 | Initial release |

## 12. References

- [SPEC-003: Architecture](./SPEC-003-architecture.md)
- [SPEC-010: RIA-1B](./SPEC-010-tier-1b.md)
- [SPEC-012: RIA-64B](./SPEC-012-tier-64b.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-070: Evaluation](./SPEC-070-evaluation.md)
