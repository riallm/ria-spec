# SPEC-031: GGUF Metadata Schema

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Format

## 1. Overview

This specification defines the complete metadata schema for RIA GGUF files.

## 2. Metadata Organization

Metadata is stored as key-value pairs in the GGUF header before tensor data.

## 3. General Metadata

### 3.1 Required Fields

| Key | Type | Example | Description |
|-----|------|---------|-------------|
| `general.architecture` | STRING | "ria" | Model architecture family |
| `general.name` | STRING | "RIA-8B-v1.0-Q4_K_M" | Full model name with version |
| `general.description` | STRING | "RIA 8B for interactive coding" | Human-readable description |
| `general.author` | STRING | "riallm Team" | Model creator |
| `general.version` | STRING | "1.0" | Version string |
| `general.license` | STRING | "DOSL-IIE-1.0" | License identifier |
| `general.url` | STRING | "https://github.com/riallm/ria" | Model URL |
| `general.source_url` | STRING | "..." | Source repository URL |
| `general.base_model` | STRING | "..." | Base model if fine-tuned |
| `general.file_type` | UINT32 | 12 | File type (see below) |
| `general.quantization_version` | UINT32 | 2 | Quantization version |

### 3.2 File Type Enum

| Value | Name | Description |
|-------|------|-------------|
| 0 | ALL_F32 | All tensors F32 |
| 1 | MOSTLY_F16 | Mostly F16 |
| 2 | MOSTLY_Q4_0 | Mostly Q4_0 |
| 3 | MOSTLY_Q4_1 | Mostly Q4_1 |
| 6 | MOSTLY_Q5_0 | Mostly Q5_0 |
| 7 | MOSTLY_Q5_1 | Mostly Q5_1 |
| 8 | MOSTLY_Q8_0 | Mostly Q8_0 |
| 10 | MOSTLY_Q2_K | Mostly Q2_K |
| 11 | MOSTLY_Q3_K_S | Mostly Q3_K_S |
| 12 | MOSTLY_Q4_K_S | Mostly Q4_K_S |
| 13 | MOSTLY_Q4_K_M | Mostly Q4_K_M |
| 14 | MOSTLY_Q5_K_S | Mostly Q5_K_S |
| 15 | MOSTLY_Q5_K_M | Mostly Q5_K_M |
| 16 | MOSTLY_Q6_K | Mostly Q6_K |
| 17 | MOSTLY_Q8_K | Mostly Q8_K |

## 4. RIA Architecture Metadata

### 4.1 Required RIA Fields

| Key | Type | Example | Description |
|-----|------|---------|-------------|
| `ria.context_length` | UINT32 | 131072 | Maximum context length |
| `ria.embedding_length` | UINT32 | 4096 | Hidden dimension |
| `ria.block_count` | UINT32 | 36 | Number of transformer layers |
| `ria.feed_forward_length` | UINT32 | 14336 | FFN intermediate dimension |
| `ria.attention.head_count` | UINT32 | 32 | Number of query heads |
| `ria.attention.head_count_kv` | UINT32 | 8 | Number of KV heads (GQA) |
| `ria.attention.layer_norm_rms_epsilon` | FLOAT32 | 1e-5 | RMSNorm epsilon |
| `ria.rope.freq_base` | FLOAT32 | 10000.0 | RoPE theta |
| `ria.rope.dimension_count` | UINT32 | 128 | RoPE dimensions |
| `ria.vocab_size` | UINT32 | 106000 | Vocabulary size |

### 4.2 Optional RIA Fields

| Key | Type | Example | Description |
|-----|------|---------|-------------|
| `ria.expert_count` | UINT32 | - | For MoE variants |
| `ria.expert_used_count` | UINT32 | - | Active experts |
| `ria.file_type` | UINT32 | - | Override general.file_type |

## 5. Tokenizer Metadata

### 5.1 Tokenizer Schema

| Key | Type | Example | Description |
|-----|------|---------|-------------|
| `tokenizer.ggml.model` | STRING | "bpe" | Tokenizer type |
| `tokenizer.ggml.pre` | STRING | "default" | Pre-tokenizer |
| `tokenizer.ggml.tokens` | ARRAY | [...] | Token strings |
| `tokenizer.ggml.scores` | ARRAY | [...] | Token scores |
| `tokenizer.ggml.token_type` | ARRAY | [...] | Token types |
| `tokenizer.ggml.bos_token_id` | UINT32 | 2 | BOS token ID |
| `tokenizer.ggml.eos_token_id` | UINT32 | 1 | EOS token ID |
| `tokenizer.ggml.unknown_token_id` | UINT32 | 3 | Unknown token ID |
| `tokenizer.ggml.padding_token_id` | UINT32 | 0 | Padding token ID |
| `tokenizer.ggml.add_bos_token` | BOOL | true | Auto-add BOS |
| `tokenizer.ggml.add_eos_token` | BOOL | false | Auto-add EOS |

## 6. Training Metadata

### 6.1 Training Information

| Key | Type | Example | Description |
|-----|------|---------|-------------|
| `ria.training.tokens` | UINT64 | 328000000000 | Training tokens |
| `ria.training.seqlen` | UINT32 | 8192 | Training sequence length |
| `ria.training.batch_size` | UINT32 | 4194304 | Training batch size |

## 7. Quantization Metadata

### 7.1 Quantization Details

| Key | Type | Example | Description |
|-----|------|---------|-------------|
| `quantize.imatrix` | UINT8[] | [...] | Importance matrix |
| `quantize.samples` | UINT8[] | [...] | Calibration samples |
| `quantize.version` | UINT32 | 2 | Quantization version |

## 8. Complete Example

```json
{
  "general.architecture": "ria",
  "general.name": "RIA-8B-v1.0-Q4_K_M",
  "general.description": "RIA 8B parameter model for interactive coding assistance",
  "general.author": "riallm Team",
  "general.version": "1.0",
  "general.license": "DOSL-IIE-1.0",
  "general.url": "https://github.com/riallm/ria",
  "general.file_type": 13,
  "general.quantization_version": 2,
  
  "ria.context_length": 131072,
  "ria.embedding_length": 4096,
  "ria.block_count": 36,
  "ria.feed_forward_length": 14336,
  "ria.attention.head_count": 32,
  "ria.attention.head_count_kv": 8,
  "ria.attention.layer_norm_rms_epsilon": 0.00001,
  "ria.rope.freq_base": 10000.0,
  "ria.rope.dimension_count": 128,
  "ria.vocab_size": 106000,
  
  "tokenizer.ggml.model": "bpe",
  "tokenizer.ggml.tokens": ["<|pad|>", "<|eos|>", ...],
  "tokenizer.ggml.scores": [0.0, 0.0, ...],
  "tokenizer.ggml.token_type": [3, 3, ...],
  "tokenizer.ggml.bos_token_id": 2,
  "tokenizer.ggml.eos_token_id": 1,
  "tokenizer.ggml.unknown_token_id": 3,
  "tokenizer.ggml.padding_token_id": 0,
  "tokenizer.ggml.add_bos_token": true,
  "tokenizer.ggml.add_eos_token": false
}
```

## 9. Validation Rules

| Rule | Description |
|------|-------------|
| **Required fields** | All required fields must be present |
| **Type checking** | Values must match declared types |
| **Consistency** | Architecture params must match tensor shapes |
| **Vocab size** | Must match token array length |
| **Layer count** | Must match tensor count |

## 10. References

- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-032: GGUF Quantization](./SPEC-032-gguf-quantization.md)
