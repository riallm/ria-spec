# SPEC-033: GGUF Conversion Procedure

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Format

## 1. Overview

This specification defines the procedure for converting RIA PyTorch checkpoints to GGUF format.

## 2. Conversion Pipeline

```
PyTorch Checkpoint → Weight Extraction → Transformation → GGUF Writing → Validation
```

## 3. Input Format

### 3.1 PyTorch Checkpoint Structure

```
ria-8b-pt/
├── config.json
├── model.safetensors
├── tokenizer.json
├── tokenizer.model
└── special_tokens_map.json
```

### 3.2 Required Files

| File | Purpose | Required |
|------|---------|----------|
| `config.json` | Model hyperparameters | Yes |
| `model.safetensors` | Model weights | Yes |
| `tokenizer.json` | Tokenizer configuration | Yes |
| `pytorch_model.bin` | Alternative weight format | No |

## 4. Tensor Name Mapping

### 4.1 PyTorch to GGUF Names

| PyTorch Name | GGUF Name | Notes |
|-------------|----------|-------|
| `model.embed_tokens.weight` | `token_embd.weight` | |
| `model.layers.{i}.self_attn.q_proj.weight` | `blk.{i}.attn_q.weight` | |
| `model.layers.{i}.self_attn.k_proj.weight` | `blk.{i}.attn_k.weight` | |
| `model.layers.{i}.self_attn.v_proj.weight` | `blk.{i}.attn_v.weight` | |
| `model.layers.{i}.self_attn.o_proj.weight` | `blk.{i}.attn_output.weight` | |
| `model.layers.{i}.mlp.gate_proj.weight` | `blk.{i}.ffn_gate.weight` | SwiGLU |
| `model.layers.{i}.mlp.down_proj.weight` | `blk.{i}.ffn_down.weight` | |
| `model.layers.{i}.mlp.up_proj.weight` | `blk.{i}.ffn_up.weight` | |
| `model.layers.{i}.input_layernorm.weight` | `blk.{i}.attn_norm.weight` | |
| `model.layers.{i}.post_attention_layernorm.weight` | `blk.{i}.ffn_norm.weight` | |
| `model.norm.weight` | `output_norm.weight` | |
| `lm_head.weight` | `output.weight` | Tied with embed |

### 4.2 RIA-Specific Tensors

| PyTorch Name | GGUF Name | Notes |
|-------------|----------|-------|
| `model.layers.{i}.tool_router.gate.weight` | `blk.{i}.tir_gate.weight` | Tool router |
| `model.layers.{i}.tool_router.selector.weight` | `blk.{i}.tir_selector.weight` | Tool selector |
| `model.layers.{i}.ffn_plan_gate.weight` | `blk.{i}.ffn_plan_gate.weight` | Planning gate |

## 5. Weight Transformation

### 5.1 Transposition Rules

Some weights need transposition for GGML compatibility:

| Tensor | Transpose | Reason |
|--------|-----------|--------|
| `*.attn_q.weight` | Yes | Row-major storage |
| `*.attn_k.weight` | Yes | Row-major storage |
| `*.attn_v.weight` | Yes | Row-major storage |
| `*.attn_output.weight` | Yes | Row-major storage |
| `*.ffn_gate.weight` | Yes | Row-major storage |
| `*.ffn_down.weight` | Yes | Row-major storage |
| `*.ffn_up.weight` | Yes | Row-major storage |

### 5.2 Conversion Code

```python
def transform_weight(name: str, tensor: torch.Tensor) -> torch.Tensor:
    """Transform PyTorch weight for GGUF."""
    needs_transpose = any(x in name for x in [
        'attn_q', 'attn_k', 'attn_v', 'attn_output',
        'ffn_gate', 'ffn_down', 'ffn_up'
    ])
    
    if needs_transpose:
        tensor = tensor.T.contiguous()
    
    return tensor
```

## 6. Conversion Procedure

### 6.1 Step-by-Step Process

```python
def convert_to_gguf(
    model_dir: str,
    output_path: str,
    outtype: str = "f16",
    quant_type: str = None
):
    """Convert PyTorch checkpoint to GGUF."""
    
    # Step 1: Load configuration
    config = load_config(f"{model_dir}/config.json")
    
    # Step 2: Load tokenizer
    tokenizer = load_tokenizer(model_dir)
    
    # Step 3: Load model weights
    state_dict = load_safetensors(f"{model_dir}/model.safetensors")
    
    # Step 4: Map tensor names
    mapped_tensors = map_tensor_names(state_dict, config)
    
    # Step 5: Transform weights
    transformed_tensors = {
        name: transform_weight(name, tensor)
        for name, tensor in mapped_tensors.items()
    }
    
    # Step 6: Write GGUF header
    write_gguf_header(output_path, config, tokenizer, len(transformed_tensors))
    
    # Step 7: Write tensor metadata
    write_tensor_info(output_path, transformed_tensors)
    
    # Step 8: Write tensor data
    write_tensor_data(output_path, transformed_tensors, outtype)
    
    # Step 9: Quantize if requested
    if quant_type:
        quantize_gguf(output_path, quant_type)
    
    # Step 10: Validate output
    validate_gguf(output_path)
```

### 6.2 Command Line Tool

```bash
# Convert to F16 GGUF
python convert-ria-to-gguf.py \
  --input ria-8b-hf/ \
  --output ria-8b-f16.gguf \
  --outtype f16

# Convert and quantize
python convert-ria-to-gguf.py \
  --input ria-8b-hf/ \
  --output ria-8b-q4_k_m.gguf \
  --outtype q4_k_m

# With custom vocabulary
python convert-ria-to-gguf.py \
  --input ria-8b-hf/ \
  --output ria-8b-q4_k_m.gguf \
  --outtype q4_k_m \
  --vocab vocab.json
```

## 7. Validation

### 7.1 Post-Conversion Checks

| Check | Method | Pass Criteria |
|-------|--------|---------------|
| **File structure** | Parse GGUF | Valid header |
| **Tensor count** | Compare with config | Match |
| **Tensor shapes** | Verify dimensions | Match source |
| **Checksums** | SHA-256 | Match manifest |
| **Inference test** | Load in riallm | Successful load |
| **Output comparison** | Compare logits | <1e-5 difference |

### 7.2 Validation Script

```bash
#!/bin/bash
# validate_gguf.sh

GGUF_FILE=$1

echo "Validating GGUF file: $GGUF_FILE"

# Check file exists
if [ ! -f "$GGUF_FILE" ]; then
    echo "ERROR: File not found"
    exit 1
fi

# Check magic number
MAGIC=$(xxd -p -l 4 "$GGUF_FILE")
if [ "$MAGIC" != "47475546" ]; then
    echo "ERROR: Invalid GGUF magic number"
    exit 1
fi

# Check version
VERSION=$(xxd -p -s 4 -l 4 "$GGUF_FILE")
if [ "$VERSION" != "03000000" ]; then
    echo "ERROR: Unsupported GGUF version"
    exit 1
fi

echo "✓ GGUF header valid"

# Check tensor count
python3 -c "
import gguf
reader = gguf.GGUFReader('$GGUF_FILE', 'r')
print(f'✓ Tensor count: {len(reader.tensors)}')
print(f'✓ Metadata keys: {len(reader.fields)}')
"

echo "✓ Validation complete"
```

## 8. Output Files

### 8.1 Generated Files

```
output/
├── ria-8b-f16.gguf          # Full precision
├── ria-8b-q4_k_m.gguf       # Default quantization
├── ria-8b-q5_k_m.gguf       # Higher quality
├── ria-8b-q8_0.gguf         # Near-lossless
├── conversion_report.json   # Conversion metadata
└── checksums.sha256         # File checksums
```

### 8.2 Conversion Report

```json
{
  "source": "ria-8b-hf",
  "output": "ria-8b-q4_k_m.gguf",
  "date": "2025-04-09T12:00:00Z",
  "tool_version": "1.0.0",
  "tensor_count": 367,
  "quantization": "Q4_K_M",
  "file_size": 5261332480,
  "checksums": {
    "sha256": "abc123..."
  },
  "validation": {
    "tensor_count_match": true,
    "shapes_match": true,
    "logits_match": true,
    "max_diff": 1.2e-6
  }
}
```

## 9. Performance Benchmarks

### 9.1 Conversion Time

| Model | Conversion Time | Output Size |
|-------|----------------|-------------|
| RIA-1B | 2 minutes | 2.1 GB |
| RIA-8B | 15 minutes | 16.4 GB |
| RIA-64B | 2 hours | 129 GB |
| RIA-128B | 4 hours | 257 GB |

### 9.2 Memory Requirements

| Model | RAM Required | Notes |
|-------|-------------|-------|
| RIA-1B | 4 GB | |
| RIA-8B | 32 GB | |
| RIA-64B | 256 GB | |
| RIA-128B | 512 GB | |

## 10. Troubleshooting

### 10.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **Invalid magic** | Corrupted file | Re-download source |
| **Tensor mismatch** | Wrong mapping | Update converter |
| **OOM error** | Insufficient RAM | Use smaller batch |
| **Shape mismatch** | Architecture change | Update config |
| **Quantization fails** | Invalid weights | Check source model |

## 11. References

- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-031: GGUF Metadata](./SPEC-031-gguf-metadata.md)
- [SPEC-032: GGUF Quantization](./SPEC-032-gguf-quantization.md)
