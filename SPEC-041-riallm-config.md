# SPEC-041: riallm Configuration

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Integration

## 1. Overview

This specification defines all configuration options for riallm inference engine.

## 2. Configuration Hierarchy

```
Environment Variables
    ↓
Configuration File (YAML/TOML)
    ↓
Command Line Arguments
    ↓
Runtime Configuration
```

## 3. Model Configuration

### 3.1 ModelOptions Struct

```rust
pub struct ModelOptions {
    /// Compression type for model loading
    pub compression: CompressionType,
    
    /// Device for inference
    pub device: DeviceSpec,
    
    /// Maximum sequence length
    pub max_seq_len: Option<usize>,
    
    /// Enable profiling mode
    pub profiling_mode: bool,
    
    /// Enable async layer prefetch
    pub prefetch_layers: bool,
    
    /// Number of layers to prefetch
    pub prefetch_buffer_size: usize,
    
    /// Data type for computation
    pub dtype: String,
    
    /// Layer cache size in bytes
    pub cache_size_bytes: usize,
    
    /// Enable memory-mapped loading
    pub mmap: bool,
    
    /// Verbosity level (0-3)
    pub verbosity: u8,
}
```

### 3.2 CompressionType

```rust
pub enum CompressionType {
    None,        // No compression (F16)
    FourBit,     // Q4_K_M quantization
    EightBit,    // Q8_0 quantization
}
```

### 3.3 DeviceSpec

```rust
pub enum DeviceSpec {
    Auto,        // Auto-select best device
    Cpu,         // CPU only
    Cuda(usize), // CUDA device index
    Metal,       // Apple Metal
}
```

### 3.4 Default Values

| Parameter | Default | Range | Description |
|----------|---------|-------|-------------|
| compression | None | None/FourBit/EightBit | Quantization type |
| device | Auto | Auto/Cpu/Cuda/Metal | Inference device |
| max_seq_len | None | 1024-524288 | Context window |
| profiling_mode | false | true/false | Enable profiling |
| prefetch_layers | true | true/false | Enable prefetch |
| prefetch_buffer_size | 2 | 0-4 | Prefetch depth |
| dtype | "float16" | float16/float32 | Computation type |
| cache_size_bytes | 2GB | 1GB-16GB | Layer cache |
| mmap | true | true/false | Memory mapping |
| verbosity | 1 | 0-3 | Log verbosity |

## 4. Generation Configuration

### 4.1 GenerationConfig

```rust
pub struct GenerationConfig {
    /// Maximum new tokens to generate
    pub max_new_tokens: usize,
    
    /// Sampling temperature (0.0-2.0)
    pub temperature: f64,
    
    /// Top-p nucleus sampling (0.0-1.0)
    pub top_p: Option<f64>,
    
    /// Top-k sampling
    pub top_k: Option<usize>,
    
    /// Repeat penalty
    pub repeat_penalty: f32,
    
    /// Repeat last N tokens
    pub repeat_last_n: usize,
    
    /// Presence penalty
    pub presence_penalty: f32,
    
    /// Frequency penalty
    pub frequency_penalty: f32,
    
    /// Stop sequences
    pub stop_sequences: Vec<String>,
    
    /// Return token probabilities
    pub logprobs: bool,
    
    /// Echo prompt in output
    pub echo: bool,
    
    /// Seed for reproducibility
    pub seed: Option<u64>,
}
```

### 4.2 Default Generation Parameters

| Parameter | Default | Range | Description |
|----------|---------|-------|-------------|
| max_new_tokens | 256 | 1-8192 | Generation length |
| temperature | 0.7 | 0.0-2.0 | Randomness |
| top_p | 0.95 | 0.0-1.0 | Nucleus sampling |
| top_k | 50 | 0-1000 | Top-k sampling |
| repeat_penalty | 1.1 | 1.0-2.0 | Repetition control |
| repeat_last_n | 64 | 0-512 | Context for penalty |
| presence_penalty | 0.0 | -2.0-2.0 | New token bonus |
| frequency_penalty | 0.0 | -2.0-2.0 | Frequency penalty |
| logprobs | false | true/false | Return probabilities |

## 5. Cache Configuration

### 5.1 CacheConfig

```rust
pub struct CacheConfig {
    /// Enable KV cache
    pub enable_kv_cache: bool,
    
    /// KV cache type
    pub kv_cache_type: KVCacheType,
    
    /// Maximum cache size in bytes
    pub max_cache_size: usize,
    
    /// Cache eviction strategy
    pub eviction_strategy: EvictionStrategy,
}

pub enum KVCacheType {
    F16,    // Full precision
    Q8,     // 8-bit quantized
    Q4,     // 4-bit quantized
}

pub enum EvictionStrategy {
    OldestFirst,
    LeastUsed,
}
```

## 6. Configuration File Format

### 6.1 YAML Example

```yaml
# riallm configuration

model:
  path: "ria-8b-q4_k_m.gguf"
  compression: "four_bit"
  device: "cuda:0"
  max_seq_len: 131072
  dtype: "float16"
  
cache:
  enable_kv_cache: true
  kv_cache_type: "f16"
  max_cache_size_gb: 4
  eviction_strategy: "oldest_first"
  prefetch_layers: 2
  prefetch_buffer_size: 2

inference:
  max_new_tokens: 512
  temperature: 0.7
  top_p: 0.95
  top_k: 50
  repeat_penalty: 1.1
  repeat_last_n: 64
  seed: 42

logging:
  verbosity: 1
  profile: true
  output: "riallm.log"
  level: "info"
```

### 6.2 TOML Example

```toml
[model]
path = "ria-8b-q4_k_m.gguf"
compression = "four_bit"
device = "cuda:0"
max_seq_len = 131072

[cache]
enable_kv_cache = true
kv_cache_type = "f16"
max_cache_size_gb = 4
prefetch_layers = 2

[inference]
max_new_tokens = 512
temperature = 0.7
top_p = 0.95
```

## 7. Command Line Interface

### 7.1 Serve Command

```bash
riallm serve [OPTIONS]

Options:
  --model <PATH>             Model file path
  --device <DEVICE>          Device (cpu/cuda/metal)
  --port <PORT>              HTTP port (default: 8080)
  --host <HOST>              HTTP host (default: 0.0.0.0)
  --max-seq-len <LEN>        Maximum sequence length
  --compression <TYPE>       Quantization (none/4bit/8bit)
  --cache-size <GB>          Cache size in GB
  --prefetch <N>             Prefetch N layers
  --profile                  Enable profiling
  --verbose                  Increase verbosity
  --config <FILE>            Config file path
```

### 7.2 Generate Command

```bash
riallm generate [OPTIONS] <PROMPT>

Options:
  --model <PATH>             Model file
  --max-tokens <N>           Max tokens to generate
  --temperature <TEMP>       Sampling temperature
  --top-p <P>                Top-p sampling
  --top-k <K>                Top-k sampling
  --repeat-penalty <P>       Repeat penalty
  --seed <SEED>              Random seed
  --output <FILE>            Output file
```

## 8. Environment Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `RIALLM_MODEL` | string | - | Model path |
| `RIALLM_DEVICE` | string | "auto" | Device selection |
| `RIALLM_CACHE_SIZE` | string | "2GB" | Cache size |
| `RIALLM_PREFETCH` | int | "2" | Prefetch layers |
| `RIALLM_MAX_SEQ_LEN` | int | "131072" | Max sequence |
| `RIALLM_PROFILE` | bool | "false" | Enable profile |
| `RIALLM_VERBOSITY` | int | "1" | Verbosity level |
| `RIALLM_CUDA_MEMORY_FRACTION` | float | "0.9" | CUDA memory fraction |
| `RIALLM_LOG_LEVEL` | string | "info" | Log level |
| `RIALLM_LOG_FILE` | string | - | Log file path |

## 9. Configuration Validation

### 9.1 Validation Rules

| Rule | Check | Error |
|------|-------|-------|
| **Model path** | File exists | ModelNotFound |
| **Device** | Available | DeviceUnavailable |
| **Cache size** | < available RAM | CacheTooLarge |
| **Max seq len** | Power of 2 | InvalidSeqLen |
| **Temperature** | 0.0-2.0 | InvalidTemp |
| **Top-p** | 0.0-1.0 | InvalidTopP |

### 9.2 Validation Example

```rust
impl ModelOptions {
    pub fn validate(&self) -> Result<(), ConfigError> {
        if !Path::new(&self.model_path).exists() {
            return Err(ConfigError::ModelNotFound(self.model_path.clone()));
        }
        
        if self.temperature < 0.0 || self.temperature > 2.0 {
            return Err(ConfigError::InvalidTemp(self.temperature));
        }
        
        if self.cache_size_bytes > available_memory() {
            return Err(ConfigError::CacheTooLarge);
        }
        
        Ok(())
    }
}
```

## 10. Dynamic Reconfiguration

### 10.1 Runtime Changes

Some parameters can be changed at runtime:

| Parameter | Runtime Changeable | Notes |
|----------|-------------------|-------|
| temperature | ✅ Yes | Immediate effect |
| top_p | ✅ Yes | Immediate effect |
| max_new_tokens | ✅ Yes | Per-request |
| device | ❌ No | Requires reload |
| compression | ❌ No | Requires reload |
| cache_size | ⚠️ Partial | Can increase only |

### 10.2 Hot Reload

```rust
// Update generation config without reload
model.update_generation_config(GenerationConfig {
    temperature: 0.9,
    ..Default::default()
})?;
```

## 11. References

- [SPEC-040: riallm Integration](./SPEC-040-riallm-integration.md)
- [SPEC-042: riallm Performance](./SPEC-042-riallm-performance.md)
