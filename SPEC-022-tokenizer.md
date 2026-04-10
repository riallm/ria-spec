# SPEC-022: Tokenizer Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Training

## 1. Overview

RIA uses a hybrid tokenizer combining Byte-Pair Encoding (BPE) with code-specific tokens for optimal code and text processing.

## 2. Tokenizer Architecture

### 2.1 Type

**Hybrid BPE + Custom Tokens**

- Base: BPE trained on mixed code and text
- Custom: Added programming language tokens
- Special: Agentic workflow control tokens

### 2.2 Vocabulary Composition

| Token Type | Count | Percentage | Examples |
|-----------|-------|-----------|----------|
| **BPE subword** | 98,000 | 92.5% | `def`, `tion`, `ing` |
| **Identifier** | 5,000 | 4.7% | `main`, `user`, `config` |
| **Syntax** | 2,000 | 1.9% | `()`, `{}`, `;` |
| **Tool** | 500 | 0.5% | `<|tool_call|>` |
| **Agentic** | 500 | 0.5% | `<|plan_start|>` |
| **Total** | **106,000** | **100%** | |

## 3. Special Tokens

### 3.1 Control Tokens

| Token | ID Range | Purpose |
|-------|----------|---------|
| `<|pad|>` | 0 | Padding |
| `<|eos|>` | 1 | End of sequence |
| `<|bos|>` | 2 | Beginning of sequence |
| `<|unk|>` | 3 | Unknown token |

### 3.2 Agentic Workflow Tokens

| Token | ID | Usage |
|-------|----|-------|
| `<|think|>` | 100 | Start reasoning |
| `<|/think|>` | 101 | End reasoning |
| `<|plan_start|>` | 102 | Start planning |
| `<|plan_end|>` | 103 | End planning |
| `<|code_start|>` | 104 | Start code block |
| `<|code_end|>` | 105 | End code block |
| `<|test_start|>` | 106 | Start test block |
| `<|test_end|>` | 107 | End test block |
| `<|debug_start|>` | 108 | Start debugging |
| `<|debug_end|>` | 109 | End debugging |
| `<|tool_call|>` | 110 | Invoke tool |
| `<|tool_result|>` | 111 | Tool output |
| `<|verify|>` | 112 | Verification step |
| `<|verify_result|>` | 113 | Verification output |
| `<|file:|>` | 114-150 | File markers |
| `<|error:|>` | 151-160 | Error annotations |
| `<|success|>` | 161 | Task success |
| `<|failure|>` | 162 | Task failure |

### 3.3 Language Markers

| Token | Purpose |
|-------|---------|
| `<|lang:python|>` | Python code |
| `<|lang:rust|>` | Rust code |
| `<|lang:javascript|>` | JavaScript code |
| `<|lang:...|>` | Other languages (50+) |

## 4. Training Procedure

### 4.1 Training Data

| Source | Tokens | Purpose |
|--------|--------|---------|
| Code (all languages) | 500B | Code patterns |
| Technical text | 200B | Text patterns |
| Mixed content | 100B | Boundary cases |

### 4.2 BPE Training

```python
from tokenizers import Tokenizer, models, trainers, pre_tokenizers

# Initialize BPE tokenizer
tokenizer = Tokenizer(models.BPE())
tokenizer.pre_tokenizer = pre_tokenizers.ByteLevel(add_prefix_space=False)

# Train on mixed corpus
trainer = trainers.BpeTrainer(
    vocab_size=106000,
    min_frequency=2,
    special_tokens=["<|pad|>", "<|eos|>", "<|bos|>", "<|unk|>", ...]
)

tokenizer.train(files, trainer=trainer)
```

### 4.3 Custom Token Addition

After BPE training:

1. **Identify common identifiers** from code corpus
2. **Add syntax tokens** for all programming symbols
3. **Add special tokens** for agentic workflows
4. **Reassign IDs** maintaining special token order

## 5. Encoding Rules

### 5.1 Priority Order

1. Special tokens (highest priority)
2. Custom identifier tokens
3. Syntax tokens
4. BPE subword tokens
5. Byte fallback (lowest priority)

### 5.2 Example Encoding

Input:
```python
def hello(name):
    return f"Hello, {name}!"
```

Tokens:
```
['def', 'Ġhello', '(', 'name', '):', 'Ċ', 'ĠĠ', 'return', 'Ġf', '"', 'Hello', ',Ġ', '{', 'name', '}', '!"']
```

IDs:
```
[5001, 24532, 302, 12843, 412, 198, 220, 6821, 1234, 304, 8762, 5643, 201, 12843, 202, 432]
```

## 6. File Format

### 6.1 Tokenizer Files

```
tokenizer/
├── tokenizer.json          # Main tokenizer file
├── tokenizer.model         # SentencePiece model (if applicable)
├── tokenizer.vocab         # Vocabulary list
├── tokenizer.merges        # BPE merges
├── tokenizer.special       # Special tokens mapping
└── tokenizer.config.json   # Configuration
```

### 6.2 JSON Schema

```json
{
  "version": "1.0",
  "truncation": null,
  "padding": null,
  "added_tokens": [
    {
      "id": 0,
      "content": "<|pad|>",
      "single_word": false,
      "lstrip": false,
      "rstrip": false,
      "normalized": false,
      "special": true
    }
  ],
  "normalizer": {
    "type": "Sequence",
    "normalizers": []
  },
  "pre_tokenizer": {
    "type": "ByteLevel",
    "add_prefix_space": false
  },
  "post_processor": {
    "type": "TemplateProcessing",
    "single": [
      {"SpecialToken": {"id": "<|bos|>", "type_id": 0}},
      {"Sequence": {"id": "A", "type_id": 0}},
      {"SpecialToken": {"id": "<|eos|>", "type_id": 0}}
    ]
  },
  "decoder": {
    "type": "ByteLevel",
    "add_prefix_space": false
  },
  "model": {
    "type": "BPE",
    "dropout": 0.0,
    "unk_token": "<|unk|>",
    "fuse_unk": false
  }
}
```

## 7. GGUF Integration

### 7.1 GGUF Tokenizer Fields

```json
{
  "tokenizer.ggml.model": "bpe",
  "tokenizer.ggml.tokens": ["<token1>", "<token2>", ...],
  "tokenizer.ggml.scores": [0.123, 0.456, ...],
  "tokenizer.ggml.token_type": [0, 1, ...],
  "tokenizer.ggml.bos_token_id": 2,
  "tokenizer.ggml.eos_token_id": 1,
  "tokenizer.ggml.unknown_token_id": 3,
  "tokenizer.ggml.padding_token_id": 0
}
```

### 7.2 Token Type Values

| Value | Type |
|-------|------|
| 1 | Normal token |
| 2 | Unknown token |
| 3 | Control token |
| 4 | User-defined token |
| 5 | Unused token |

## 8. Performance Requirements

| Metric | Target | Notes |
|--------|--------|-------|
| **Encoding speed** | >1 MB/sec | Single thread |
| **Decoding speed** | >2 MB/sec | Single thread |
| **Vocabulary size** | 106,000 | Fixed |
| **Max token length** | 100 bytes | BPE merges |
| **Compression ratio** | 3-5 bytes/token | Average |

## 9. Validation

### 9.1 Tests

- [ ] Round-trip encoding/decoding
- [ ] Special token handling
- [ ] Unicode support
- [ ] Long sequence handling
- [ ] Empty input handling
- [ ] Batch encoding

### 9.2 Quality Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Unknown rate** | <0.1% | On validation set |
| **Avg token length** | 4 bytes | On code corpus |
| **Compression** | >3 bytes/token | On test corpus |

## 10. Versioning

| Version | Date | Changes |
|---------|------|---------|
| **1.0** | 2025-04-09 | Initial tokenizer |

## 11. References

- [SPEC-020: Training Methodology](./SPEC-020-training.md)
- [SPEC-021: Data Pipeline](./SPEC-021-data-pipeline.md)
- [SPEC-030: GGUF Format](./SPEC-030-gguf-format.md)
