# SPEC-102: Compatibility Matrix

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Compatibility

## 1. Overview

This specification defines compatibility between RIA models, riallm versions, and dependencies.

## 2. Model Compatibility

### 2.1 riallm Version Compatibility

| riallm Version | RIA-1B | RIA-8B | RIA-64B | RIA-128B | GGUF Version |
|---------------|--------|--------|---------|----------|--------------|
| **0.1.x** | ✅ | ❌ | ❌ | ❌ | 3 |
| **1.0.x** | ✅ | ✅ | ✅ | ❌ | 3 |
| **1.1.x** | ✅ | ✅ | ✅ | ✅ | 3 |
| **2.0.x** | ✅ | ✅ | ✅ | ✅ | 4 |

### 2.2 Rust Version Compatibility

| riallm Version | Min Rust | Max Rust | Edition |
|---------------|----------|----------|---------|
| **0.1.x** | 1.75 | 1.85 | 2021 |
| **1.0.x** | 1.80 | 1.90 | 2021 |
| **1.1.x** | 1.80 | 1.95 | 2021 |
| **2.0.x** | 1.85 | - | 2021 |

## 3. Platform Compatibility

### 3.1 Operating Systems

| OS | CPU | CUDA | Metal | Status |
|----|-----|------|-------|--------|
| **Ubuntu 20.04+** | ✅ | ✅ | ❌ | Primary |
| **macOS 13+** | ✅ | ❌ | ✅ | Primary |
| **Windows 11** | ✅ | ✅ | ❌ | Supported |
| **Debian 12** | ✅ | ✅ | ❌ | Supported |
| **Fedora 38+** | ✅ | ✅ | ❌ | Community |
| **Arch Linux** | ✅ | ✅ | ❌ | Community |

### 3.2 Hardware Compatibility

| Hardware | CPU Arch | Min RAM | Status |
|----------|---------|---------|--------|
| **Intel x86_64** | x86_64 | 4 GB | ✅ Primary |
| **AMD x86_64** | x86_64 | 4 GB | ✅ Primary |
| **Apple Silicon** | ARM64 | 8 GB | ✅ Primary |
| **ARM Server** | ARM64 | 8 GB | ✅ Supported |
| **RISC-V** | RV64 | 8 GB | 🧪 Experimental |

## 4. GPU Compatibility

### 4.1 CUDA GPUs

| GPU | Architecture | VRAM | Support Level |
|-----|-------------|------|---------------|
| **RTX 4090** | Ada | 24 GB | ✅ Primary |
| **RTX 4080** | Ada | 16 GB | ✅ Primary |
| **RTX 4070** | Ada | 12 GB | ✅ Supported |
| **A100** | Ampere | 80 GB | ✅ Primary |
| **H100** | Hopper | 80 GB | ✅ Supported |
| **RTX 3090** | Ampere | 24 GB | ✅ Supported |
| **V100** | Volta | 16 GB | ⚠️ Legacy |

### 4.2 CUDA Versions

| CUDA Version | Min Driver | Status |
|-------------|-----------|--------|
| **12.4** | 550.x | ✅ Recommended |
| **12.0** | 525.x | ✅ Supported |
| **11.8** | 520.x | ⚠️ Legacy |
| **11.7** | 515.x | ❌ Deprecated |

## 5. API Compatibility

### 5.1 OpenAI API Compatibility

| Endpoint | Compatibility | Notes |
|----------|--------------|-------|
| **/v1/completions** | ✅ Full | Drop-in replacement |
| **/v1/chat/completions** | ✅ Full | Drop-in replacement |
| **/v1/models** | ✅ Full | Drop-in replacement |
| **/v1/embeddings** | ❌ Not supported | Future |

### 5.2 Client Library Compatibility

| Library | Language | Version | Status |
|---------|---------|---------|--------|
| **openai-python** | Python | 1.0+ | ✅ Compatible |
| **openai-node** | JavaScript | 4.0+ | ✅ Compatible |
| **openai-rust** | Rust | 0.1+ | ✅ Compatible |

## 6. GGUF Compatibility

### 6.1 GGUF Version Support

| GGUF Version | riallm Support | Features |
|-------------|---------------|----------|
| **V1** | ❌ | Deprecated |
| **V2** | ✅ | Legacy |
| **V3** | ✅ | Current |
| **V4** | 📋 | Planned |

### 6.2 Quantization Compatibility

| Quant Type | Read | Write | Notes |
|-----------|------|-------|-------|
| **F16** | ✅ | ✅ | Full precision |
| **Q4_0** | ✅ | ✅ | Legacy |
| **Q4_K_M** | ✅ | ✅ | Recommended |
| **Q5_K_M** | ✅ | ✅ | High quality |
| **Q8_0** | ✅ | ✅ | Near-lossless |
| **IQ4_XS** | 🧪 | ❌ | Experimental |

## 7. Breaking Changes

### 7.1 Version History

| Version | Breaking Changes | Migration |
|---------|-----------------|-----------|
| **0.1 → 1.0** | API stabilization | Update client |
| **1.0 → 1.1** | None | None needed |
| **1.1 → 2.0** | Config format change | Run migration |

### 7.2 Deprecation Schedule

| Feature | Deprecated | EOL | Replacement |
|---------|-----------|-----|-------------|
| **GGUF V2** | 2025-Q3 | 2026-Q1 | GGUF V3 |
| **CUDA 11.8** | 2025-Q4 | 2026-Q2 | CUDA 12.x |
| **Q4_0** | 2025-Q4 | 2026-Q1 | Q4_K_M |

## 8. Testing Matrix

### 8.1 CI/CD Coverage

| Platform | CUDA | Rust | Frequency |
|----------|------|------|----------|
| Ubuntu 22.04 | 12.4 | 1.80 | Every commit |
| Ubuntu 22.04 | 12.0 | 1.80 | Daily |
| macOS 14 | - | 1.80 | Daily |
| Windows 11 | 12.4 | 1.80 | Weekly |

## 9. References

- [SPEC-101: Versioning](./SPEC-101-versioning.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
- [SPEC-040: riallm Integration](./SPEC-040-riallm-integration.md)
