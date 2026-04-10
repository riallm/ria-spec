# SPEC-005: Terminology and Glossary

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Core

## 1. Overview

This specification defines all terms used in RIA documentation and specifications.

## 2. Core Terms

### RIA Family

| Term | Definition |
|------|-----------|
| **RIA** | Rust Intelligence for Agentic Coding—the model family name |
| **RIA-1B** | RIA model with ~1 billion parameters |
| **RIA-8B** | RIA model with ~8.2 billion parameters |
| **RIA-64B** | RIA model with ~64.5 billion parameters |
| **RIA-128B** | RIA model with ~128.3 billion parameters |
| **ria** | Lowercase abbreviation for informal use |

### Inference

| Term | Definition |
|------|-----------|
| **riallm** | Rust-based inference engine for RIA models |
| **GGUF** | GPT-Generated Unified Format—binary model format |
| **GGML** | Tensor library underlying GGUF, C-based |
| **Inference** | Running a trained model to generate predictions |
| **Forward pass** | Single computation through the model |
| **Token generation** | Producing next token in sequence |
| **KV cache** | Key-Value cache for efficient generation |
| **Context window** | Maximum input sequence length |
| **Temperature** | Sampling randomness control (0.0-2.0) |
| **Top-p** | Nucleus sampling threshold (0.0-1.0) |
| **Top-k** | Limit sampling to k most likely tokens |

### Architecture

| Term | Definition |
|------|-----------|
| **Transformer** | Neural network architecture based on attention |
| **Decoder-only** | Architecture using only decoder blocks |
| **Layer** | Single transformer block in the stack |
| **Hidden dimension** | Size of internal representations (d) |
| **Attention head** | Single attention computation stream |
| **GQA** | Grouped Query Attention—share KV heads |
| **RoPE** | Rotary Position Embedding |
| **RMSNorm** | Root Mean Square Layer Normalization |
| **SwiGLU** | Swish-Gated Linear Unit activation |
| **FFN** | Feed-Forward Network |
| **Embedding** | Token to vector transformation |
| **LM head** | Final layer projecting to vocabulary |

### Agentic Coding

| Term | Definition |
|------|-----------|
| **Agentic** | Autonomous task execution capability |
| **Planning** | Generating multi-step approach before execution |
| **Tool use** | Invoking external programs (linters, tests) |
| **Self-debugging** | Identifying and fixing own code errors |
| **Code review** | Analyzing code for issues and improvements |
| **Multi-hop reasoning** | Connecting information across files |
| **Iterative refinement** | Improving output through cycles |
| **TIR** | Tool Integration Router |
| **MHCA** | Multi-Hop Code Attention |
| **ACR** | Agentic Code Reasoning training |

### Quantization

| Term | Definition |
|------|-----------|
| **Quantization** | Reducing precision to save memory |
| **Q4_0** | 4-bit quantization (block 32) |
| **Q5_0** | 5-bit quantization (block 32) |
| **Q8_0** | 8-bit quantization (block 32) |
| **Q4_K_M** | 4-bit K-quants medium quality |
| **Q5_K_M** | 5-bit K-quants medium quality |
| **Q6_K** | 6-bit K-quants |
| **F16** | 16-bit floating point (half precision) |
| **F32** | 32-bit floating point (single precision) |
| **NF4** | Normal Float 4-bit |
| **Block size** | Number of weights sharing scale |

### Hardware

| Term | Definition |
|------|-----------|
| **VRAM** | Video RAM—GPU memory |
| **RAM** | System memory (CPU) |
| **CUDA** | NVIDIA GPU compute platform |
| **Metal** | Apple GPU compute framework |
| **CPU inference** | Running model on processor |
| **GPU inference** | Running model on graphics card |
| **Memory-mapped** | File mapped to memory space |
| **Layer-by-layer** | Loading one layer at a time |

### Training

| Term | Definition |
|------|-----------|
| **Pretraining** | Initial training on general data |
| **Fine-tuning** | Additional training on specific data |
| **SFT** | Supervised Fine-Tuning |
| **RLHF** | Reinforcement Learning from Human Feedback |
| **RLCF** | Reinforcement Learning from Code Feedback |
| **Epoch** | One pass through entire dataset |
| **Batch size** | Samples processed before update |
| **Learning rate** | Step size for parameter updates |
| **Gradient** | Direction of steepest loss increase |
| **Checkpoint** | Saved model state during training |
| **Token** | Single unit from vocabulary |

### File Formats

| Term | Definition |
|------|-----------|
| **GGUF** | Binary model weight format |
| **Safetensors** | Tensor serialization format |
| **JSON** | Configuration file format |
| **Tokenizer** | Text-to-token conversion files |
| **Config** | Model hyperparameters file |
| **Metadata** | Additional model information |

## 3. Acronyms

| Acronym | Expansion |
|---------|----------|
| **RIA** | Rust Intelligence for Agentic Coding |
| **LLM** | Large Language Model |
| **GGUF** | GPT-Generated Unified Format |
| **GGML** | GPT-Generated Machine Learning |
| **MHCA** | Multi-Hop Code Attention |
| **TIR** | Tool Integration Router |
| **ACR** | Agentic Code Reasoning |
| **FFN** | Feed-Forward Network |
| **GQA** | Grouped Query Attention |
| **RoPE** | Rotary Position Embedding |
| **RMSNorm** | Root Mean Square Normalization |
| **SFT** | Supervised Fine-Tuning |
| **RLHF** | Reinforcement Learning from Human Feedback |
| **RLCF** | Reinforcement Learning from Code Feedback |
| **API** | Application Programming Interface |
| **CLI** | Command Line Interface |
| **IDE** | Integrated Development Environment |
| **CI/CD** | Continuous Integration/Deployment |
| **OOM** | Out Of Memory |
| **VRAM** | Video Random Access Memory |
| **RAM** | Random Access Memory |
| **CPU** | Central Processing Unit |
| **GPU** | Graphics Processing Unit |
| **CUDA** | Compute Unified Device Architecture |
| **CVE** | Common Vulnerabilities and Exposures |
| **SWE** | Software Engineering |

## 4. Symbols and Notation

### Mathematical

| Symbol | Meaning |
|--------|---------|
| **d** | Hidden dimension |
| **N** | Number of layers |
| **h** | Number of attention heads |
| **d_k** | Key dimension per head |
| **V** | Vocabulary size |
| **θ** | RoPE theta parameter |
| **ε** | Normalization epsilon |
| **α** | Attention bias parameter |

### Code Notation

| Notation | Meaning |
|----------|---------|
| `weight` | Model parameter tensor |
| `bias` | Optional bias vector |
| `input_ids` | Token ID sequence |
| `attention_mask` | Which tokens to attend to |
| `position_ids` | Token positions |
| `hidden_states` | Layer activations |

## 5. Naming Conventions

### Model Files

```
ria-{size}-{quantization}.gguf

Examples:
ria-8b-q4_k_m.gguf
ria-128b-q5_0.gguf
ria-1b-f16.gguf
```

### GGUF Tensors

```
{layer}.{component}.weight

Examples:
token_embd.weight
blk.0.attn_q.weight
blk.0.ffn_gate.weight
output_norm.weight
```

### Configuration

```yaml
# Standard config file: config.json
{
  "model_type": "ria",
  "hidden_size": 4096,
  "num_hidden_layers": 36,
  ...
}
```

## 6. Version Terminology

| Term | Definition |
|------|-----------|
| **v1.0** | Major version—breaking changes possible |
| **v1.1** | Minor version—new features, compatible |
| **v1.0.1** | Patch version—bug fixes only |
| **RC** | Release Candidate—testing phase |
| **Beta** | Feature-complete, testing needed |
| **Alpha** | Incomplete, experimental |

## 7. Domain-Specific Terms

### Software Engineering

| Term | Definition in RIA Context |
|------|--------------------------|
| **Refactoring** | Restructuring code without changing behavior |
| **Regression** | New bug introduced by change |
| **Coverage** | Percentage of code tested |
| **Linting** | Static code analysis for issues |
| **Type checking** | Verifying type correctness |
| **Dependency** | External code library required |
| **Module** | Collection of related files |
| **Repository** | Complete project with version history |

### Machine Learning

| Term | Definition in RIA Context |
|------|--------------------------|
| **Perplexity** | Language model quality metric |
| **Loss** | Training objective value |
| **Gradient checkpointing** | Memory-saving training technique |
| **Mixed precision** | Using FP16 and FP32 together |
| **Distributed training** | Training across multiple GPUs |
| **Sharding** | Splitting model across devices |

## 8. References

- [SPEC-001: Project Overview](./SPEC-001-overview.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-003: Architecture](./SPEC-003-architecture.md)
