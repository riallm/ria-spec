# SPEC-042: riallm Performance

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Integration

## 1. Overview

This specification defines performance characteristics, targets, and optimization strategies for riallm inference engine.

## 2. Performance Metrics

### 2.1 Primary Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| **Tokens/sec** | tokens/s | Generation throughput |
| **Latency** | ms/token | Time per token |
| **TTFT** | ms | Time to first token |
| **Memory Usage** | GB | Peak memory consumption |
| **Cache Hit Rate** | % | Layer prefetch success |

### 2.2 Secondary Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| **Power Consumption** | watts | Energy usage |
| **Startup Time** | seconds | Model loading time |
| **Context Loading** | ms/token | Speed with full context |

## 3. Performance Targets

### 3.1 RIA-1B Targets

| Hardware | Quantization | Tokens/sec | Latency | Memory |
|----------|-------------|-----------|---------|--------|
| **CPU (i7)** | Q4_K_M | 38 | 26ms | 0.7GB |
| **RTX 3050** | Q4_K_M | 52 | 19ms | 0.6GB |
| **M2** | Q4_K_M | 45 | 22ms | 0.6GB |

### 3.2 RIA-8B Targets

| Hardware | Quantization | Tokens/sec | Latency | Memory |
|----------|-------------|-----------|---------|--------|
| **CPU (i9)** | Q4_K_M | 8 | 125ms | 5GB |
| **RTX 4070** | Q4_K_M | 35 | 29ms | 5GB |
| **RTX 4090** | Q4_K_M | 45 | 22ms | 5GB |
| **A100** | F16 | 52 | 19ms | 16GB |

### 3.3 RIA-64B Targets

| Hardware | Quantization | Tokens/sec | Latency | Memory |
|----------|-------------|-----------|---------|--------|
| **A100 80GB** | Q4_K_M | 12 | 83ms | 37GB |
| **A100 80GB** | Q8_0 | 8 | 125ms | 68GB |
| **RTX 4090** | Q4_K_M | 6 | 167ms | 37GB |

### 3.4 RIA-128B Targets

| Hardware | Quantization | Tokens/sec | Latency | Memory |
|----------|-------------|-----------|---------|--------|
| **A100 80GB** | Q4_K_M | 4 | 250ms | 74GB |
| **A100 80GB (riallm)** | Q4_K_M | 2.5 | 400ms | 74GB |
| **2× A100** | Q8_0 | 6 | 167ms | 136GB |

## 4. Latency Breakdown

### 4.1 Per-Token Latency (RIA-8B, RTX 4090)

| Operation | Time | Percentage |
|----------|------|------------|
| Token sampling | 2ms | 9% |
| Layer loading (36 layers) | 540ms | 18% |
| Layer transfer | 288ms | 10% |
| Forward pass | 1,620ms | 54% |
| Memory cleanup | 180ms | 6% |
| KV cache update | 90ms | 3% |
| **Total (per token)** | **3,000ms** | **100%** |

### 4.2 Per-Layer Latency

| Operation | Time | Notes |
|----------|------|-------|
| Disk → CPU | 15ms | Memory-mapped load |
| CPU → GPU | 8ms | PCIe transfer |
| Forward pass | 45ms | Compute bound |
| GPU → CPU | 5ms | Activations |
| Cleanup | 10ms | Memory free |
| **Total per layer** | **83ms** | |

## 5. Optimization Techniques

### 5.1 Layer Loading Optimizations

| Technique | Speedup | Description |
|----------|---------|-------------|
| **Memory mapping** | 2x | mmap instead of read |
| **Pinned memory** | 1.3x | Page-locked RAM |
| **Async I/O** | 1.5x | Overlap load with compute |
| **Prefetching** | 1.4x | Load next layer early |

### 5.2 Compute Optimizations

| Technique | Speedup | Description |
|----------|---------|-------------|
| **Flash attention** | 1.5x | Optimized attention |
| **Fused kernels** | 1.3x | Combined operations |
| **Tensor cores** | 2x | Mixed precision |
| **Batch processing** | 1.8x | Process multiple tokens |

### 5.3 Memory Optimizations

| Technique | Savings | Description |
|----------|---------|-------------|
| **KV cache quantization** | 50% | Q8 KV cache |
| **Activation checkpointing** | 40% | Recompute vs store |
| **Paged attention** | 30% | Fragmented cache |

## 6. Scaling Behavior

### 6.1 Context Length Scaling

| Context Length | Memory | Latency Increase |
|---------------|--------|-----------------|
| 4K | 0.5GB | 1.0x |
| 16K | 2GB | 1.2x |
| 64K | 8GB | 1.8x |
| 128K | 16GB | 2.5x |
| 256K | 32GB | 4.0x |

### 6.2 Batch Size Scaling

| Batch Size | Throughput | Latency |
|-----------|-----------|---------|
| 1 | 45 tok/s | 22ms |
| 4 | 120 tok/s | 33ms |
| 8 | 200 tok/s | 40ms |
| 16 | 320 tok/s | 50ms |

## 7. Profiling

### 7.1 Built-in Profiler

```rust
let options = ModelOptions {
    profiling_mode: true,
    ..Default::default()
};

let model = AutoModel::from_gguf("ria-8b.gguf", Some(options)).await?;

// After generation
if let Some(report) = model.profiling_report() {
    println!("{}", report);
}
```

### 7.2 Profile Output

```
rial lm Performance Report
==========================
Model: RIA-8B (Q4_K_M)
Device: CUDA (RTX 4090)
Date: 2025-04-09 12:00:00

Generation Statistics:
  Prompt tokens: 128
  Generated tokens: 512
  Total time: 11.4s
  Tokens/sec: 44.9
  Avg latency: 22.3ms/token

Layer Statistics:
  Total layers: 36
  Avg load time: 15.1ms
  Avg forward time: 44.8ms
  Prefetch hit rate: 89.2%
  Cache hits: 32/36

Memory Usage:
  Model size: 4.9GB
  KV cache: 1.2GB
  Peak memory: 6.1GB

Bottleneck Analysis:
  54% GPU compute
  18% Layer loading
  10% Data transfer
  18% Other
```

## 8. Benchmarking

### 8.1 Standard Benchmark

```bash
# Run benchmark
riallm benchmark \
  --model ria-8b-q4_k_m.gguf \
  --prompt-file prompts.txt \
  --output benchmark.json
```

### 8.2 Benchmark Metrics

| Test | Description | Target |
|------|-------------|--------|
| **Short prompt** | 128 tokens | <25ms/token |
| **Long prompt** | 4K tokens | <30ms/token |
| **Full context** | 128K tokens | <50ms/token |
| **Batch (4)** | 4 simultaneous | <40ms/token |

## 9. Platform-Specific Performance

### 9.1 CUDA Performance

| GPU | FP16 TFLOPS | Memory BW | Expected tok/s |
|-----|-------------|-----------|----------------|
| RTX 4090 | 82.6 | 1,008 GB/s | 45 |
| RTX 4080 | 48.7 | 717 GB/s | 32 |
| RTX 4070 | 29.2 | 504 GB/s | 24 |
| A100 80GB | 77.9 | 2,039 GB/s | 52 |
| H100 | 98.9 | 3,350 GB/s | 65 |

### 9.2 CPU Performance

| CPU | Cores | AVX | Expected tok/s |
|-----|-------|-----|----------------|
| i9-13900K | 24 | AVX-512 | 12 |
| i7-12700K | 12 | AVX2 | 8 |
| M2 Max | 12 | NEON | 28 |
| M1 Ultra | 20 | NEON | 35 |

### 9.3 Metal Performance (Apple Silicon)

| Chip | GPU Cores | Memory | Expected tok/s |
|------|-----------|--------|----------------|
| M2 | 10 | 24GB | 25 |
| M2 Pro | 19 | 32GB | 35 |
| M2 Max | 38 | 64GB | 42 |
| M2 Ultra | 76 | 128GB | 55 |

## 10. Tuning Guide

### 10.1 For Maximum Speed

```yaml
model:
  compression: "four_bit"
  device: "cuda:0"
  
cache:
  prefetch_layers: 4
  cache_size_gb: 8
  enable_kv_cache: true
  kv_cache_type: "f16"

inference:
  batch_size: 4  # If supported
```

### 10.2 For Minimum Memory

```yaml
model:
  compression: "four_bit"
  device: "cuda:0"
  
cache:
  prefetch_layers: 1
  cache_size_gb: 2
  enable_kv_cache: true
  kv_cache_type: "q8"
```

### 10.3 For Best Quality

```yaml
model:
  compression: "none"  # F16
  device: "cuda:0"
  
cache:
  prefetch_layers: 2
  cache_size_gb: 16
  enable_kv_cache: true
  kv_cache_type: "f16"
```

## 11. Troubleshooting Performance

### 11.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **Low tokens/sec** | CPU bottleneck | Use GPU |
| **High latency** | No prefetch | Enable prefetch |
| **OOM error** | Cache too large | Reduce cache |
| **Stuttering** | Disk slow | Use SSD |
| **Poor scaling** | Single thread | Enable parallel |

### 11.2 Diagnostic Commands

```bash
# Check GPU utilization
nvidia-smi -l 1

# Monitor memory usage
watch -n 1 free -h

# Profile I/O
iostat -x 1

# riallm diagnostics
riallm diagnose --model ria-8b.gguf
```

## 12. References

- [SPEC-040: riallm Integration](./SPEC-040-riallm-integration.md)
- [SPEC-041: riallm Configuration](./SPEC-041-riallm-config.md)
- [SPEC-090: Deployment](./SPEC-090-deployment.md)
