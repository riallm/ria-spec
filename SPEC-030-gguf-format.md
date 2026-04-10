# SPEC-030: GGUF Format Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Format

## 1. Overview

This specification defines the GGUF (GPT-Generated Unified Format) binary format for RIA model distribution.

## 2. Format Overview

GGUF is a binary format designed for:
- Memory-mapped loading
- Fast model startup
- Rich metadata embedding
- Multiple quantization support
- Cross-platform compatibility

## 3. File Structure

```
┌──────────────────────────────────────┐
│           File Header                 │
│  - Magic number                       │
│  - Version                           │
│  - Tensor count                      │
│  - Metadata KV count                 │
├──────────────────────────────────────┤
│        Metadata KV Section            │
│  - Key-value pairs (architecture,    │
│    tokenizer, hyperparameters)        │
├──────────────────────────────────────┤
│         Tensor Info Section           │
│  - Tensor metadata for each tensor   │
│    (name, dimensions, type, offset)   │
├──────────────────────────────────────┤
│          Alignment Padding            │
├──────────────────────────────────────┤
│          Tensor Data Section          │
│  - Raw tensor data (quantized or     │
│    full precision)                    │
└──────────────────────────────────────┘
```

## 4. File Header

### 4.1 Header Fields

| Field | Type | Size | Value |
|-------|------|------|-------|
| **Magic** | uint32 | 4 bytes | 0x46554747 ("GGUF") |
| **Version** | uint32 | 4 bytes | 3 |
| **Tensor count** | uint64 | 8 bytes | Number of tensors |
| **Metadata KV count** | uint64 | 8 bytes | Number of KV pairs |

### 4.2 Magic Number

```
Offset 0: 0x47 ('G')
Offset 1: 0x47 ('G')
Offset 2: 0x55 ('U')
Offset 3: 0x46 ('F')
```

## 5. Metadata KV Types

### 5.1 Value Types

| Type ID | Name | Description |
|---------|------|-------------|
| 0 | UINT8 | 8-bit unsigned int |
| 1 | INT8 | 8-bit signed int |
| 2 | UINT16 | 16-bit unsigned int |
| 3 | INT16 | 16-bit signed int |
| 4 | UINT32 | 32-bit unsigned int |
| 5 | INT32 | 32-bit signed int |
| 6 | FLOAT32 | 32-bit float |
| 7 | BOOL | Boolean |
| 8 | STRING | UTF-8 string |
| 9 | ARRAY | Array of values |
| 10 | UINT64 | 64-bit unsigned int |
| 11 | INT64 | 64-bit signed int |
| 12 | FLOAT64 | 64-bit float |

### 5.2 Required Metadata Keys

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `general.architecture` | STRING | Yes | "ria" |
| `general.name` | STRING | Yes | Model name |
| `general.description` | STRING | No | Model description |
| `general.file_type` | UINT32 | Yes | File type enum |
| `general.quantization_version` | UINT32 | Yes | Quant version |
| `ria.context_length` | UINT32 | Yes | Max context |
| `ria.embedding_length` | UINT32 | Yes | Hidden dimension |
| `ria.block_count` | UINT32 | Yes | Number of layers |
| `ria.feed_forward_length` | UINT32 | Yes | FFN dimension |
| `ria.attention.head_count` | UINT32 | Yes | Query heads |
| `ria.attention.head_count_kv` | UINT32 | Yes | KV heads |
| `ria.attention.layer_norm_rms_epsilon` | FLOAT32 | Yes | Norm epsilon |
| `ria.rope.freq_base` | FLOAT32 | Yes | RoPE theta |
| `ria.vocab_size` | UINT32 | Yes | Vocabulary size |

### 5.3 Tokenizer Metadata

| Key | Type | Required |
|-----|------|----------|
| `tokenizer.ggml.model` | STRING | Yes |
| `tokenizer.ggml.tokens` | ARRAY | Yes |
| `tokenizer.ggml.scores` | ARRAY | Yes |
| `tokenizer.ggml.token_type` | ARRAY | Yes |
| `tokenizer.ggml.bos_token_id` | UINT32 | Yes |
| `tokenizer.ggml.eos_token_id` | UINT32 | Yes |
| `tokenizer.ggml.unknown_token_id` | UINT32 | Yes |
| `tokenizer.ggml.padding_token_id` | UINT32 | Yes |

## 6. Tensor Types

### 6.1 Supported Tensor Types

| Type ID | Name | Bytes/Param | Description |
|---------|------|-------------|-------------|
| 0 | F32 | 4 | 32-bit float |
| 1 | F16 | 2 | 16-bit float |
| 2 | Q4_0 | 0.5 | 4-bit quantized |
| 3 | Q4_1 | 0.625 | 4-bit quantized with scale |
| 6 | Q5_0 | 0.625 | 5-bit quantized |
| 7 | Q5_1 | 0.75 | 5-bit quantized with scale |
| 8 | Q8_0 | 1 | 8-bit quantized |
| 9 | Q8_1 | 1.25 | 8-bit quantized with scale |
| 10 | Q2_K | 0.3125 | 2-bit K-quants |
| 11 | Q3_K | 0.375 | 3-bit K-quants |
| 12 | Q4_K | 0.5 | 4-bit K-quants |
| 13 | Q5_K | 0.625 | 5-bit K-quants |
| 14 | Q6_K | 0.75 | 6-bit K-quants |
| 15 | Q8_K | 1 | 8-bit K-quants |

### 6.2 RIA Recommended Quantizations

| Name | Type | Use Case | Quality |
|------|------|----------|---------|
| **Q4_0** | 2 | Minimal storage | Good |
| **Q4_K_M** | 12 | Default recommendation | Better |
| **Q5_0** | 6 | Balance quality/size | Very Good |
| **Q5_K_M** | 13 | High quality | Very Good |
| **Q6_K** | 14 | Maximum quality | Excellent |
| **Q8_0** | 8 | Near-lossless | Near-lossless |
| **F16** | 1 | Training/fine-tuning | Lossless |

## 7. Tensor Metadata

### 7.1 Tensor Info Structure

For each tensor:

| Field | Type | Description |
|-------|------|-------------|
| **Name length** | uint32 | Length of name string |
| **Name** | char[] | Tensor name (UTF-8) |
| **Dimensions** | uint32 | Number of dimensions (1-4) |
| **Shape** | uint64[] | Dimensions array |
| **Type** | uint32 | Tensor type enum |
| **Offset** | uint64 | Offset to tensor data |

### 7.2 RIA Tensor Naming Convention

```
token_embd.weight                    # Input embedding
blk.{i}.attn_q.weight                # Query projection (layer i)
blk.{i}.attn_k.weight                # Key projection
blk.{i}.attn_v.weight                # Value projection
blk.{i}.attn_output.weight           # Attention output
blk.{i}.attn_norm.weight             # Attention RMSNorm
blk.{i}.ffn_gate.weight              # FFN gate (SwiGLU)
blk.{i}.ffn_down.weight              # FFN down
blk.{i}.ffn_up.weight                # FFN up
blk.{i}.ffn_norm.weight              # FFN RMSNorm
output_norm.weight                   # Final RMSNorm
output.weight                        # LM head (tied with embed)
```

## 8. Alignment

| Section | Alignment | Purpose |
|---------|----------|---------|
| **Header** | 1 byte | No alignment needed |
| **Metadata** | 1 byte | Variable size |
| **Tensor info** | 1 byte | Sequential |
| **Alignment padding** | 32 bytes | Memory mapping |
| **Tensor data** | 32 bytes | SIMD alignment |

## 9. Example GGUF Structure for RIA-8B

```
Header:
  Magic: 0x47475546
  Version: 3
  Tensor count: 367 (36 layers × 10 + 7)
  Metadata KV count: 25

Metadata:
  general.architecture: "ria"
  general.name: "RIA-8B v1.0"
  ria.context_length: 131072
  ria.embedding_length: 4096
  ria.block_count: 36
  ria.feed_forward_length: 14336
  ria.attention.head_count: 32
  ria.attention.head_count_kv: 8
  ...

Tensor Info:
  [0] "token_embd.weight", shape=[4096, 106000], type=Q4_K, offset=0
  [1] "blk.0.attn_q.weight", shape=[4096, 4096], type=Q4_K, offset=...
  ...
  [366] "output.weight", shape=[4096, 106000], type=Q4_K, offset=...

Tensor Data:
  [Aligned quantized weights for all tensors]
```

## 10. File Size Calculation

For RIA-8B Q4_K_M:

```
Embedding: 4096 × 106000 × 0.5 bytes = 216 MB
Per layer: 10 tensors × ~150 MB = ~150 MB
Total layers: 36 × 150 MB = 5.4 GB
Total: ~4.9 GB (with overhead)
```

## 11. Validation

### 11.1 Integrity Checks

| Check | Method | Purpose |
|-------|--------|---------|
| **Magic number** | Header bytes | Format verification |
| **Version** | Version field | Compatibility |
| **Checksum** | SHA-256 | File integrity |
| **Tensor count** | Match metadata | Completeness |
| **Shape validation** | Expected shapes | Architecture match |

### 11.2 Loading Verification

```rust
fn validate_gguf(file: &File) -> Result<()> {
    // Check magic
    let magic = read_u32(file)?;
    assert_eq!(magic, 0x47475546, "Invalid GGUF magic");
    
    // Check version
    let version = read_u32(file)?;
    assert_eq!(version, 3, "Unsupported GGUF version");
    
    // Validate metadata
    let metadata = read_metadata(file)?;
    assert_eq!(metadata["general.architecture"], "ria");
    
    // Validate tensor count
    let tensor_count = read_u64(file)?;
    assert_eq!(tensor_count, expected_tensor_count(&metadata));
    
    Ok(())
}
```

## 12. References

- [SPEC-031: GGUF Metadata](./SPEC-031-gguf-metadata.md)
- [SPEC-032: GGUF Quantization](./SPEC-032-gguf-quantization.md)
- [SPEC-033: GGUF Conversion](./SPEC-033-gguf-conversion.md)
