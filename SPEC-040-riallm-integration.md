# SPEC-040: riallm Integration

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Integration

## 1. Overview

This specification defines how RIA models integrate with the riallm Rust inference engine.

## 2. Integration Architecture

```
┌────────────────────────────────────────────────────────┐
│                    Application                          │
├────────────────────────────────────────────────────────┤
│                    riallm API                           │
├────────────────────────────────────────────────────────┤
│                  riallm Core                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  GGUF Loader │→ │  Layer Cache │→ │   Inference  │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
├────────────────────────────────────────────────────────┤
│                  Candle Backend                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │     CPU      │  │    CUDA      │  │    Metal     │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└────────────────────────────────────────────────────────┘
```

## 3. GGUF Loading

### 3.1 Loading Process

```rust
use riallm::AutoModel;
use riallm::config::ModelOptions;

// Load model
let model = AutoModel::from_gguf("ria-8b-q4_k_m.gguf", options).await?;
```

### 3.2 Memory-Mapped Loading

| Step | Description | Time |
|------|-------------|------|
| **Parse header** | Read metadata | <1ms |
| **Map file** | Memory map GGUF | <10ms |
| **Load tensors** | Lazy load on demand | 0ms (lazy) |
| **Initialize cache** | Setup layer cache | <100ms |

### 3.3 Layer Cache Configuration

```rust
let options = ModelOptions {
    // Cache size for layer loading
    cache_size_bytes: 2 * 1024 * 1024 * 1024,  // 2GB
    
    // Prefetch next N layers
    prefetch_layers: 2,
    
    // Device for inference
    device: DeviceSpec::Cuda(0),
    
    // Maximum sequence length
    max_seq_len: Some(131072),
};
```

## 4. Layer-by-Layer Loading

### 4.1 Algorithm

```
For each token generation step:
  1. Load current layer from disk to CPU
  2. Transfer layer to GPU
  3. Run forward pass for layer
  4. Transfer activations to next layer
  5. Unload layer from GPU
  6. Repeat for all layers
```

### 4.2 Performance Characteristics

| Operation | Time (RIA-8B) | Percentage |
|----------|---------------|------------|
| Disk → CPU load | 15ms | 18% |
| CPU → GPU transfer | 8ms | 10% |
| GPU forward pass | 45ms | 54% |
| GPU → CPU transfer | 5ms | 6% |
| Memory cleanup | 10ms | 12% |
| **Total per layer** | **83ms** | **100%** |
| **Total per token (36 layers)** | **3.0s** | |

### 4.3 Optimization Techniques

| Technique | Speedup | Memory Impact |
|----------|---------|---------------|
| **Async prefetch** | 1.5x | +2GB |
| **Pinned memory** | 1.3x | No change |
| **CUDA streams** | 1.2x | No change |
| **Layer fusion** | 1.4x | +1GB |

## 5. Device Support

### 5.1 Supported Devices

| Device | Backend | Status | Notes |
|--------|---------|--------|-------|
| **CPU** | Candle CPU | ✅ Stable | All features |
| **CUDA** | Candle CUDA | ✅ Stable | Best performance |
| **Metal** | Candle Metal | ✅ Stable | Apple Silicon |
| **Vulkan** | Candle Vulkan | 🧪 Beta | AMD GPUs |

### 5.2 Device Selection

```rust
// Automatic device selection
let device = DeviceSpec::Auto;  // Prefer CUDA, fallback to CPU

// Explicit device selection
let device = DeviceSpec::Cuda(0);  // First CUDA device
let device = DeviceSpec::Cpu;       // CPU only
let device = DeviceSpec::Metal;     // Apple Metal
```

## 6. API Integration

### 6.1 Rust API

```rust
use riallm::{AutoModel, GenerationConfig};
use riallm::config::{ModelOptions, DeviceSpec, CompressionType};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Configure model
    let options = ModelOptions {
        compression: CompressionType::FourBit,
        device: DeviceSpec::Cuda(0),
        max_seq_len: Some(131072),
        profiling_mode: true,
        ..Default::default()
    };
    
    // Load model
    let mut model = AutoModel::from_gguf(
        "ria-8b-q4_k_m.gguf",
        Some(options)
    ).await?;
    
    // Generate text
    let prompt = "Write a function to calculate factorial in Rust:";
    let config = GenerationConfig {
        max_new_tokens: 200,
        temperature: 0.7,
        top_p: 0.95,
        ..Default::default()
    };
    
    let output = model.generate(prompt, &config).await?;
    println!("{}", output);
    
    Ok(())
}
```

### 6.2 HTTP API

```bash
# Start server
riallm serve --model ria-8b-q4_k_m.gguf --port 8080

# Generate completion
curl http://localhost:8080/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "def fibonacci(n):",
    "max_tokens": 100,
    "temperature": 0.7
  }'
```

## 7. Configuration File

### 7.1 YAML Configuration

```yaml
model:
  path: "ria-8b-q4_k_m.gguf"
  device: "cuda:0"
  max_seq_len: 131072
  
inference:
  temperature: 0.7
  top_p: 0.95
  top_k: 50
  repeat_penalty: 1.1
  
cache:
  prefetch_layers: 2
  cache_size_gb: 2
  async_loading: true
  
logging:
  level: "info"
  profile: true
  output: "riallm.log"
```

### 7.2 Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `RIALLM_MODEL` | - | Model path |
| `RIALLM_DEVICE` | "auto" | Device selection |
| `RIALLM_CACHE_SIZE` | "2GB" | Layer cache size |
| `RIALLM_PREFETCH` | "2" | Layers to prefetch |
| `RIALLM_PROFILE` | "false" | Enable profiling |

## 8. Performance Monitoring

### 8.1 Built-in Metrics

| Metric | Description | Access |
|--------|-------------|--------|
| `tokens_per_second` | Generation speed | API |
| `memory_usage` | Current memory usage | API |
| `layer_load_time` | Time per layer load | Profile |
| `kv_cache_size` | KV cache memory | Profile |

### 8.2 Profiling Output

```
rial lm Profiling Report
========================
Model: RIA-8B (Q4_K_M)
Device: CUDA (RTX 4090)

Performance:
  Tokens/sec: 12.5
  Avg latency: 80ms/token
  Peak memory: 6.2 GB

Layer Statistics:
  Avg load time: 15.2ms
  Avg forward time: 45.3ms
  Prefetch hit rate: 87%

Token Generation:
  Total tokens: 1,234
  Total time: 98.7s
  Prompt tokens: 128
  Generated tokens: 1,106
```

## 9. Error Handling

### 9.1 Error Types

| Error | Cause | Recovery |
|-------|-------|----------|
| **ModelNotFound** | Invalid GGUF path | Check path |
| **OutOfMemory** | Insufficient VRAM | Reduce cache |
| **InvalidGGUF** | Corrupted file | Re-download |
| **DeviceError** | GPU unavailable | Fallback CPU |
| **ContextOverflow** | Prompt too long | Truncate prompt |

### 9.2 Error Handling Example

```rust
match model.generate(prompt, &config).await {
    Ok(output) => println!("{}", output),
    Err(RiallmError::OutOfMemory { needed, available }) => {
        eprintln!("OOM: Need {}GB, have {}GB", needed, available);
        // Retry with smaller cache
    },
    Err(e) => eprintln!("Error: {}", e),
}
```

## 10. References

- [SPEC-041: riallm Configuration](./SPEC-041-riallm-config.md)
- [SPEC-042: riallm Performance](./SPEC-042-riallm-performance.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
