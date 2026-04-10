# SPEC-003: Model Architecture

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Core

## 1. Overview

RIA models are decoder-only transformers with specialized components for agentic coding tasks. All four tiers (1B, 8B, 64B, 128B) share the same architecture with scaled dimensions.

## 2. Base Architecture

### 2.1 Core Components

```
RIA Model Architecture
├── Token Embedding
├── Position Embedding (RoPE)
├── Transformer Layers (N×)
│   ├── RMSNorm (pre-norm)
│   ├── Multi-Hop Code Attention (MHCA)
│   │   ├── Query projection
│   │   ├── Key projection (GQA)
│   │   ├── Value projection (GQA)
│   │   ├── File-aware attention bias
│   │   ├── Hierarchical attention windows
│   │   └── Output projection
│   ├── Tool Integration Router (TIR)
│   ├── Feed-Forward Network (SwiGLU)
│   │   ├── Planning path
│   │   └── Execution path
│   └── Residual connections
├── Final RMSNorm
└── LM Head (tied with embedding)
```

### 2.2 Mathematical Formulation

For each transformer layer `l`:

```
h_0 = embed(tokens) + pos_embed(positions)

For l = 1 to N:
    # Pre-norm
    a = RMSNorm(h_{l-1})
    
    # Multi-Hop Code Attention
    m = MHCA(a) + file_bias(a)
    h' = h_{l-1} + m
    
    # Tool Integration Router
    t = TIR(h')
    h'' = concatenate(h', t)
    
    # Dual-Path FFN
    f = RMSNorm(h'')
    planning = SwiGLU(f · W1_p) · W2_p
    execution = SwiGLU(f · W1_e) · W2_e
    gate = sigmoid(f · W_g)
    ff_output = gate · planning + (1 - gate) · execution
    
    # Residual
    h_l = h'' + ff_output

output = RMSNorm(h_N) · W_embed^T
```

## 3. Multi-Hop Code Attention (MHCA)

### 3.1 Overview

MHCA extends standard attention with file-awareness and hierarchical windows for code understanding.

### 3.2 File-Aware Attention Bias

Standard attention:
```
Attention(Q,K,V) = softmax(QK^T / sqrt(d_k)) · V
```

MHCA attention:
```
Attention(Q,K,V) = softmax((QK^T / sqrt(d_k)) + M_file + M_hier) · V
```

Where:
- `M_file`: File relationship bias
- `M_hier`: Hierarchical window bias

### 3.3 File Relationship Matrix

```
M_file[i,j] = 
    α_same      if tokens i,j in same file
    α_import    if files have import relationship
    α_module    if files in same module
    α_repo      if files in same repository
    0           otherwise
```

Learned parameters (per attention head):
```
α_same     ∈ [-2.0, 2.0]   # Same file boost
α_import   ∈ [-1.0, 1.5]   # Import relationship
α_module   ∈ [-0.5, 1.0]   # Same module
α_repo     ∈ [-0.2, 0.5]   # Same repository
```

### 3.4 Hierarchical Attention Windows

```
Window Level | Size    | Coverage
-------------|---------|---------------------------
Local        | 4,096   | Current file context
File         | 16,384  | Entire current file
Cross-file   | 65,536  | Related files (imports)
Repository   | Full    | Entire repository (sparse)
```

## 4. Tool Integration Router (TIR)

### 4.1 Overview

TIR enables the model to decide when and how to invoke external tools during inference.

### 4.2 Architecture

```python
class ToolIntegrationRouter:
    def __init__(self, hidden_dim, num_tools):
        self.tool_gate = Linear(hidden_dim, 1)
        self.tool_selector = Linear(hidden_dim, num_tools)
        self.tool_arg_generator = MLP(hidden_dim, hidden_dim * 2)
    
    def forward(self, hidden_state):
        # Decide if tool needed
        tool_prob = sigmoid(self.tool_gate(hidden_state))
        
        if tool_prob > threshold:
            # Select tool
            tool_logits = self.tool_selector(hidden_state)
            tool_id = argmax(tool_logits)
            
            # Generate arguments
            tool_args = self.tool_arg_generator(hidden_state)
            
            return ToolCall(tool_id, tool_args, tool_prob)
        else:
            return None
```

### 4.3 Tool Registry

RIA models support these tool categories:

| Category | Tools | Token IDs |
|----------|-------|-----------|
| **Execution** | Python REPL, shell | 100,000-100,099 |
| **Testing** | pytest, unittest | 100,100-100,149 |
| **Analysis** | linter, type checker | 100,150-100,199 |
| **Version Control** | git commands | 100,200-100,249 |
| **Build** | cargo, make, cmake | 100,250-100,299 |
| **Search** | grep, find | 100,300-100,349 |
| **Package Mgmt** | pip, npm, cargo | 100,350-100,399 |

## 5. Dual-Path Feed-Forward Network

### 5.1 Overview

The FFN uses separate planning and execution paths to distinguish between "thinking about what to do" and "actually doing it."

### 5.2 Structure

```
Input → SwiGLU → Planning Path  → \
                                    → Gated Merge → Output
Input → SwiGLU → Execution Path → /
```

### 5.3 Parameters

For hidden dimension `d` and intermediate dimension `m`:

```
Planning Path:
    W1_p: [d, m]
    W2_p: [m, d]

Execution Path:
    W1_e: [d, m]
    W2_e: [m, d]

Gate:
    W_g: [d, 1]

Total FFN parameters: 4*d*m + d
```

## 6. Shared Architecture Across Tiers

All RIA models share:

| Component | Specification |
|----------|---------------|
| **Attention Type** | Grouped Query Attention (GQA) |
| **KV Heads** | 8 (all tiers) |
| **Activation** | SwiGLU |
| **Position Encoding** | RoPE (θ=10,000) |
| **Normalization** | RMSNorm (ε=1e-5) |
| **Embedding** | Tied input/output |
| **Attention Bias** | None (file-aware bias separate) |
| **Vocabulary** | 106,000 tokens |
| **Head Dimension** | 128 |

## 7. Scaling Rules

Parameters scale predictably:

```
Parameters ≈ 12 * N * d^2 + V * d

Where:
    N = number of layers
    d = hidden dimension
    V = vocabulary size
```

Each tier doubles or quadruples capacity:

| Tier | d | N | Parameters |
|------|---|---|------------|
| 1B | 2,048 | 24 | 1.0B |
| 8B | 4,096 | 36 | 8.2B |
| 64B | 8,192 | 64 | 64.5B |
| 128B | 12,288 | 80 | 128.3B |

## 8. GGUF Compatibility

### 8.1 Architecture Constraints for GGUF

To ensure GGUF compatibility:

1. **Layer uniformity**: All transformer layers identical size
2. **Standard operations**: Only GGUF-supported operations
3. **No custom kernels**: Standard GGML operations only
4. **Metadata embedding**: Full architecture in GGUF header

### 8.2 Tensor Naming Convention

GGUF tensor names follow llamacpp convention:

```
token_embd.weight                    # Embedding
blk.{i}.attn_q.weight                # Query projection
blk.{i}.attn_k.weight                # Key projection
blk.{i}.attn_v.weight                # Value projection
blk.{i}.attn_output.weight           # Attention output
blk.{i}.ffn_gate.weight              # FFN gate (SwiGLU)
blk.{i}.ffn_down.weight              # FFN down projection
blk.{i}.ffn_up.weight                # FFN up projection
blk.{i}.attn_norm.weight             # Attention RMSNorm
blk.{i}.ffn_norm.weight              # FFN RMSNorm
output_norm.weight                   # Final RMSNorm
output.weight                        # LM head (tied)
```

## 9. Memory Footprint Analysis

### 9.1 Parameter Count by Component (RIA-8B)

| Component | Parameters | Percentage |
|----------|-----------|------------|
| **Embedding** | 435M | 5.3% |
| **Attention** | 2,458M | 30.0% |
| **FFN** | 4,915M | 59.9% |
| **TIR** | 33M | 0.4% |
| **Norms** | 59M | 0.7% |
| **LM Head** | 435M | 5.3% (tied) |
| **Total** | 8,192M | 100% |

### 9.2 Activation Memory (per token, FP16)

| Component | Memory (KB) | Notes |
|----------|-------------|-------|
| **Hidden states** | 8 | 4096 × 2 bytes |
| **Attention** | 32 | Q, K, V projections |
| **FFN** | 64 | Planning + execution paths |
| **TIR** | 2 | Tool router |
| **Total per token** | 106 | |
| **For 128K context** | 13.6 GB | Full activation checkpointing |

## 10. References

- [SPEC-010: RIA-1B Specification](./SPEC-010-tier-1b.md)
- [SPEC-011: RIA-8B Specification](./SPEC-011-tier-8b.md)
- [SPEC-012: RIA-64B Specification](./SPEC-012-tier-64b.md)
- [SPEC-013: RIA-128B Specification](./SPEC-013-tier-128b.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
