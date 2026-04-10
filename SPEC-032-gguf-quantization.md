# SPEC-032: GGUF Quantization Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Format

## 1. Overview

This specification defines quantization methods supported for RIA GGUF models.

## 2. Quantization Types

### 2.1 Legacy Quantization

| Type | Bits | Block Size | Description |
|------|------|-----------|-------------|
| **Q4_0** | 4.0 | 32 | Simple 4-bit |
| **Q4_1** | 4.5 | 32 | 4-bit with scale |
| **Q5_0** | 5.0 | 32 | Simple 5-bit |
| **Q5_1** | 5.5 | 32 | 5-bit with scale |
| **Q8_0** | 8.0 | 32 | Simple 8-bit |
| **Q8_1** | 8.5 | 32 | 8-bit with scale |

### 2.2 K-Quants (Recommended)

| Type | Bits | Block Size | Description |
|------|------|-----------|-------------|
| **Q2_K** | 2.56 | 256 | Super extra small |
| **Q3_K_S** | 3.0 | 256 | 3-bit small |
| **Q3_K_M** | 3.375 | 256 | 3-bit medium |
| **Q3_K_L** | 3.5 | 256 | 3-bit large |
| **Q4_K_S** | 4.0 | 256 | 4-bit small |
| **Q4_K_M** | 4.5 | 256 | 4-bit medium (default) |
| **Q5_K_S** | 5.0 | 256 | 5-bit small |
| **Q5_K_M** | 5.5 | 256 | 5-bit medium |
| **Q6_K** | 6.0 | 256 | 6-bit |
| **Q8_K** | 8.0 | 256 | 8-bit K-quant |

## 3. RIA Recommended Quantizations

### 3.1 Default Recommendations

| Tier | Recommended | Alternative | Notes |
|------|------------|-------------|-------|
| **RIA-1B** | Q5_K_M | Q4_K_M | Small models benefit from quality |
| **RIA-8B** | Q4_K_M | Q5_K_M or Q8_0 | Balance quality/size |
| **RIA-64B** | Q4_K_M | Q5_K_M | Default for large models |
| **RIA-128B** | Q4_K_M | Q5_K_M or Q6_K | Size constraints |

### 3.2 Quality vs Size Trade-off

| Quant | Size (RIA-8B) | Perplexity Delta | Recommendation |
|-------|--------------|-----------------|----------------|
| **F16** | 16.4 GB | 0.0 | Baseline |
| **Q8_0** | 8.7 GB | +0.01 | Near-lossless |
| **Q6_K** | 6.9 GB | +0.02 | Excellent |
| **Q5_K_M** | 6.1 GB | +0.05 | Very Good |
| **Q4_K_M** | 4.9 GB | +0.10 | Good (default) |
| **Q3_K_M** | 3.8 GB | +0.25 | Acceptable |
| **Q2_K** | 3.1 GB | +0.50 | Minimal use |

## 4. Quantization Algorithms

### 4.1 Q4_0 Quantization

```python
def quantize_q4_0(weights: np.ndarray) -> bytes:
    """Quantize to 4-bit with single scale factor per block."""
    # Reshape into blocks of 32
    blocks = weights.reshape(-1, 32)
    result = b''
    
    for block in blocks:
        # Find max absolute value
        max_val = np.max(np.abs(block))
        if max_val == 0:
            max_val = 1e-6
        
        # Compute scale
        scale = max_val / 8.0  # 4-bit range: -8 to 7
        
        # Quantize
        quantized = np.round(block / scale + 8).astype(np.uint8)
        quantized = np.clip(quantized, 0, 15)
        
        # Pack: scale (2 bytes) + 16 bytes for 32 values
        result += struct.pack('e', scale)  # Half precision scale
        for i in range(0, 32, 2):
            byte = (quantized[i] << 4) | quantized[i+1]
            result += bytes([byte])
    
    return result
```

### 4.2 Q4_K_M Quantization

```python
def quantize_q4_k_m(weights: np.ndarray) -> bytes:
    """Quantize to 4-bit K-quants with super-blocks."""
    # Super-block size: 256
    # Sub-block size: 32
    # 8 sub-blocks per super-block
    
    super_blocks = weights.reshape(-1, 256)
    result = b''
    
    for super_block in super_blocks:
        # Compute super-block scale
        max_val = np.max(np.abs(super_block))
        scale_super = max_val / 8.0
        
        # Divide into 8 sub-blocks
        sub_blocks = super_block.reshape(8, 32)
        
        # Quantize each sub-block
        sub_scales = []
        sub_quants = []
        
        for sub_block in sub_blocks:
            sub_max = np.max(np.abs(sub_block))
            sub_scale = sub_max / 8.0
            sub_scales.append(sub_scale)
            
            quant = np.round(sub_block / sub_scale + 8).astype(np.uint8)
            quant = np.clip(quant, 0, 15)
            sub_quants.append(quant)
        
        # Pack super-block
        result += struct.pack('e', scale_super)
        result += struct.pack('8e', *sub_scales)
        
        for quant in sub_quants:
            for i in range(0, 32, 2):
                byte = (quant[i] << 4) | quant[i+1]
                result += bytes([byte])
    
    return result
```

## 5. Dequantization

### 5.1 Q4_0 Dequantization

```python
def dequantize_q4_0(data: bytes, shape: tuple) -> np.ndarray:
    """Dequantize Q4_0 back to float32."""
    num_elements = np.prod(shape)
    num_blocks = num_elements // 32
    
    result = np.zeros(num_elements, dtype=np.float32)
    offset = 0
    
    for i in range(num_blocks):
        # Read scale
        scale = struct.unpack('e', data[offset:offset+2])[0]
        offset += 2
        
        # Read and unpack 4-bit values
        block = np.zeros(32, dtype=np.float32)
        for j in range(0, 32, 2):
            byte = data[offset]
            offset += 1
            block[j] = ((byte >> 4) & 0xF) - 8
            block[j+1] = (byte & 0xF) - 8
        
        result[i*32:(i+1)*32] = block * scale
    
    return result.reshape(shape)
```

## 6. Quantization Quality Metrics

### 6.1 SNR Measurement

```
SNR = 10 * log10(P_signal / P_noise)

Where:
  P_signal = ||W||^2
  P_noise = ||W - W_quantized||^2
```

### 6.2 Target SNR Values

| Quantization | Target SNR | Quality |
|-------------|-----------|---------|
| Q8_0 | >40 dB | Excellent |
| Q6_K | >35 dB | Very Good |
| Q5_K_M | >30 dB | Good |
| Q4_K_M | >25 dB | Acceptable |
| Q3_K_M | >20 dB | Limited |

## 7. Calibration for K-Quants

### 7.1 Importance Matrix

K-quants use importance matrix to allocate bits:

```
1. Run model on calibration dataset
2. Compute activation statistics
3. Identify important weights (high activation)
4. Assign more bits to important weights
5. Less important weights get fewer bits
```

### 7.2 Calibration Dataset

| Source | Samples | Purpose |
|--------|---------|---------|
| Code corpus | 512 samples | Code patterns |
| Text corpus | 256 samples | General language |
| Edge cases | 128 samples | Difficult patterns |

## 8. Quantization Procedure

### 8.1 Standard Quantization

```bash
# Convert to GGUF F16 first
python convert-hf-to-gguf.py \
  --model ria-8b-hf/ \
  --outtype f16 \
  --outfile ria-8b-f16.gguf

# Quantize to Q4_K_M
./quantize \
  ria-8b-f16.gguf \
  ria-8b-q4_k_m.gguf \
  Q4_K_M
```

### 8.2 With Importance Matrix

```bash
# Generate importance matrix
./imatrix \
  --model ria-8b-f16.gguf \
  --file calibration.txt \
  --output ria-8b-imatrix.dat

# Quantize with importance matrix
./quantize \
  --imatrix ria-8b-imatrix.dat \
  ria-8b-f16.gguf \
  ria-8b-q4_k_m.gguf \
  Q4_K_M
```

## 9. Validation

### 9.1 Post-Quantization Checks

| Check | Method | Threshold |
|-------|--------|-----------|
| **SNR** | Signal-to-noise | >25 dB (Q4_K_M) |
| **Perplexity** | On validation set | <5% increase |
| **HumanEval** | pass@1 | <5% decrease |
| **File integrity** | SHA-256 checksum | Match |

### 9.2 Quantization Report

Generate report after quantization:

```
Quantization Report
==================
Model: RIA-8B
Source: ria-8b-f16.gguf
Target: ria-8b-q4_k_m.gguf
Type: Q4_K_M

Size: 16.4 GB → 4.9 GB (3.35x compression)
SNR: 28.5 dB
Perplexity: 5.23 → 5.31 (+1.5%)
HumanEval: 89.6% → 88.9% (-0.8%)

Status: PASSED ✓
```

## 10. References

- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-031: GGUF Metadata](./SPEC-031-gguf-metadata.md)
- [SPEC-033: GGUF Conversion](./SPEC-033-gguf-conversion.md)
