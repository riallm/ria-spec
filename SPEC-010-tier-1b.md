# SPEC-010: RIA-1B Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Model Tier

## 1. Overview

RIA-1B is the smallest tier in the RIA family, designed for edge devices, mobile applications, and quick coding tasks.

## 2. Architecture Specifications

### 2.1 Core Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Total Parameters** | 1,048,576,000 (1.0B) | |
| **Vocabulary Size** | 106,000 | Hybrid BPE + code tokens |
| **Hidden Dimension (d)** | 2,048 | |
| **Number of Layers (N)** | 24 | |
| **Attention Heads** | 16 | |
| **KV Heads (GQA)** | 4 | GQA ratio: 4:1 |
| **Head Dimension** | 128 | d / heads = 128 |
| **FFN Intermediate** | 5,632 | SwiGLU |
| **Max Context** | 32,768 tokens | 32K |
| **RoPE Theta** | 10,000 | |
| **RMSNorm Epsilon** | 1e-5 | |
| **Tied Embeddings** | Yes | Input/output shared |

### 2.2 Parameter Breakdown

| Component | Parameters | Percentage |
|----------|-----------|------------|
| **Token Embedding** | 217,088,000 | 20.7% |
| **Attention (all layers)** | 318,767,104 | 30.4% |
| **FFN (all layers)** | 692,060,160 | 66.0% |
| **RMSNorm** | 98,304 | 0.01% |
| **TIR** | 4,194,304 | 0.4% |
| **LM Head** | 217,088,000 | (tied, not counted) |

### 2.3 Layer Structure

Each of 24 identical layers contains:

```
TransformerLayer(
    # Attention
    attn_q: Linear(2048, 2048)      # 16 heads × 128
    attn_k: Linear(2048, 512)       # 4 heads × 128
    attn_v: Linear(2048, 512)       # 4 heads × 128
    attn_output: Linear(2048, 2048)
    
    # TIR
    tir_gate: Linear(2048, 1)
    tir_selector: Linear(2048, 400)  # 400 tools
    
    # FFN (SwiGLU dual-path)
    ffn_gate: Linear(2048, 5632)
    ffn_down: Linear(5632, 2048)
    ffn_up: Linear(2048, 5632)
    
    # Norms
    attn_norm: RMSNorm(2048)
    ffn_norm: RMSNorm(2048)
)
```

## 3. GGUF Specifications

### 3.1 File Sizes

| Quantization | File Size | Compression | Quality |
|-------------|----------|-------------|---------|
| **Q4_0** | 590 MB | 1.78× | Good |
| **Q4_K_M** | 630 MB | 1.66× | Better |
| **Q5_0** | 715 MB | 1.47× | Very Good |
| **Q5_K_M** | 745 MB | 1.41× | Very Good |
| **Q6_K** | 850 MB | 1.23× | Excellent |
| **Q8_0** | 1.1 GB | 0.95× | Near-lossless |
| **F16** | 2.1 GB | 1.0× | Lossless |

### 3.2 GGUF Metadata

```json
{
  "general.architecture": "ria",
  "general.name": "RIA-1B v1.0",
  "general.description": "RIA 1B parameter model for edge devices",
  "general.file_type": 2,
  "general.quantization_version": 2,
  
  "ria.context_length": 32768,
  "ria.embedding_length": 2048,
  "ria.block_count": 24,
  "ria.feed_forward_length": 5632,
  "ria.attention.head_count": 16,
  "ria.attention.head_count_kv": 4,
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
| **RAM** | 2 GB |
| **Storage** | 1 GB free space |
| **CPU** | x86_64 or ARM64, 4+ cores |
| **OS** | Linux, macOS, Windows |

### 4.2 Recommended Requirements

| Component | Specification |
|-----------|--------------|
| **RAM** | 4 GB |
| **Storage** | 2 GB SSD |
| **CPU** | 8+ cores, AVX2 support |
| **GPU** (optional) | Any CUDA GPU, 2GB VRAM |

### 4.3 Performance Targets

| Hardware | Tokens/sec (Q4_K_M) | Latency (ms/token) |
|----------|---------------------|-------------------|
| **M2 MacBook Air** | 45 | 22 |
| **Intel i7-12700K** | 38 | 26 |
| **RTX 3050 (4GB)** | 52 | 19 |
| **Raspberry Pi 5** | 8 | 125 |
| **iPhone 15 Pro** | 35 | 29 |

## 5. Capabilities

### 5.1 Strengths

| Capability | Rating | Description |
|-----------|--------|-------------|
| **Code completion** | ★★★★☆ | Single function completion |
| **Code explanation** | ★★★★☆ | Explain code clearly |
| **Simple bug fixes** | ★★★☆☆ | Fix obvious bugs |
| **Documentation** | ★★★★☆ | Generate docstrings |
| **Quick tasks** | ★★★★★ | Fast response times |

### 5.2 Limitations

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| **Limited context** | 32K max | Focus on single files |
| **Basic planning** | Simple plans only | Break tasks smaller |
| **No multi-file** | Single file focus | Manual file switching |
| **Edge cases** | May miss rare patterns | Review code carefully |

### 5.3 Supported Languages

| Tier | Languages | Proficiency |
|------|----------|-------------|
| **Expert** | Python, JavaScript, TypeScript | Production quality |
| **Proficient** | Java, Go, Rust, C++ | Good quality |
| **Capable** | Ruby, PHP, Swift | Basic quality |
| **Basic** | 20+ others | Varies |

## 6. Use Cases

### 6.1 Ideal Use Cases

1. **Code completion in IDE**
   - Single function generation
   - Quick snippets
   - API usage examples

2. **Code explanation**
   - "What does this function do?"
   - Documentation generation
   - Learning assistant

3. **Simple debugging**
   - Syntax errors
   - Obvious logic bugs
   - Type errors

4. **Mobile development**
   - On-device inference
   - Offline coding assistant
   - Edge deployment

### 6.2 Not Recommended For

- Complex multi-file refactoring
- Architectural design decisions
- Security-sensitive code review
- Full autonomous task execution

## 7. riallm Configuration

### 7.1 Basic Usage

```bash
# Download
riallm download riallm/ria-1b-q4_k_m.gguf

# Run on CPU
riallm serve --model ria-1b-q4_k_m.gguf --port 8080

# Run with GPU (if available)
riallm serve --model ria-1b-q4_k_m.gguf --device cuda
```

### 7.2 Rust API

```rust
use riallm::AutoModel;
use riallm::config::ModelOptions;

let options = ModelOptions {
    device: DeviceSpec::Cpu,  // Works great on CPU
    max_seq_len: Some(32768),
    ..Default::default()
};

let model = AutoModel::from_pretrained(
    "ria-1b-q4_k_m.gguf",
    Some(options)
).await?;
```

### 7.3 Memory Usage

| Mode | RAM Usage | Notes |
|------|----------|-------|
| **Q4_K_M loaded** | 650 MB | Memory-mapped |
| **Q4_K_M active** | 750 MB | With KV cache |
| **F16 loaded** | 2.2 GB | Full precision |
| **Full context** | 2.5 GB | 32K tokens |

## 8. Benchmark Results

### 8.1 Code Generation

| Benchmark | RIA-1B Score | Notes |
|----------|-------------|-------|
| **HumanEval** | 68.3% | pass@1 |
| **MBPP** | 61.2% | pass@1 |
| **APPS** | 28.1% | Competitive programming |
| **CodeContests** | 12.4% | Hard problems |

### 8.2 Multi-Language

| Language | MultiPL-E Score |
|----------|----------------|
| **Python** | 65.1% |
| **JavaScript** | 63.8% |
| **Java** | 62.4% |
| **Rust** | 58.3% |
| **C++** | 55.2% |

### 8.3 Agentic Tasks

| Benchmark | Score | Notes |
|----------|-------|-------|
| **SWE-bench Lite** | 8.5% | Limited planning |
| **AgenticBench** | 39.3% | Basic capabilities |
| **CRUXEval** | 61.5% | Code reasoning |

## 9. Training Details

| Parameter | Value |
|----------|-------|
| **Training tokens** | 40 billion |
| **Training time** | 2 weeks on 64× A100 |
| **Peak learning rate** | 3e-4 |
| **Optimizer** | AdamW |
| **Batch size** | 2M tokens |
| **Sequence length** | 4,096 (pretrain), 32,768 (final) |

## 10. Quality Guarantees

### 10.1 Safety Training

- [x] Refusal of malicious requests (>90%)
- [x] No harmful code patterns
- [x] Security vulnerability detection (basic)
- [x] Appropriate error handling

### 10.2 Known Failure Modes

| Failure Mode | Frequency | Severity | Mitigation |
|-------------|-----------|----------|------------|
| **Incomplete plans** | 15% | Medium | Break into steps |
| **API hallucination** | 10% | Low | Verify APIs |
| **Logic errors** | 8% | Medium | Run tests |
| **Context overflow** | 5% | Low | Monitor length |

## 11. Version History

| Version | Date | Changes |
|---------|------|---------|
| **1.0** | 2025-04-09 | Initial release |

## 12. References

- [SPEC-003: Architecture](./SPEC-003-architecture.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-070: Evaluation](./SPEC-070-evaluation.md)
- [SPEC-090: Deployment](./SPEC-090-deployment.md)
