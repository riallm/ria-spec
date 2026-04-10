# SPEC-023: Fine-Tuning Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Training

## 1. Overview

This specification defines supervised fine-tuning (SFT) procedures for RIA models after pretraining.

## 2. Fine-Tuning Phases

```
Base Model → Code SFT → Agentic SFT → Safety SFT → RLHF
```

| Phase | Purpose | Data Size | Duration |
|-------|---------|----------|----------|
| **Code SFT** | Code expertise | 500K examples | 1-3 days |
| **Agentic SFT** | Task execution | 200K examples | 1-2 days |
| **Safety SFT** | Safe behavior | 100K examples | 1 day |

## 3. Code SFT

### 3.1 Dataset

| Source | Examples | Description |
|--------|----------|-------------|
| GitHub solutions | 200K | Problem-solution pairs |
| Code reviews | 100K | Review comment + fix |
| Tutorials | 100K | Step-by-step code |
| Documentation | 100K | Code + explanations |

### 3.2 Format

```json
{
  "instruction": "Implement a binary search tree in Rust",
  "input": "",
  "output": "<|code_start|>\npub struct BST {\n    ...\n}\n<|code_end|>",
  "language": "rust",
  "difficulty": "medium",
  "source": "github"
}
```

### 3.3 Hyperparameters

| Parameter | RIA-1B | RIA-8B | RIA-64B | RIA-128B |
|----------|--------|--------|---------|----------|
| **Learning rate** | 1e-5 | 5e-6 | 2e-6 | 1e-6 |
| **Batch size** | 64 | 128 | 256 | 512 |
| **Sequence length** | 8,192 | 16,384 | 32,768 | 65,536 |
| **Warmup ratio** | 0.05 | 0.05 | 0.03 | 0.03 |
| **LR schedule** | Cosine | Cosine | Cosine | Cosine |
| **Epochs** | 3 | 3 | 2 | 2 |

## 4. Agentic SFT

### 4.1 Dataset

| Task Type | Examples | Description |
|----------|----------|-------------|
| Feature addition | 80K | Add features to codebases |
| Bug fixing | 50K | Debug and fix issues |
| Refactoring | 40K | Improve code structure |
| Testing | 30K | Write test suites |

### 4.2 Format

```json
{
  "task": "Add authentication to the Flask app",
  "context": [
    {"file": "app.py", "content": "..."},
    {"file": "models.py", "content": "..."}
  ],
  "reasoning": "<|think|>...<|/think|>",
  "plan": "<|plan_start|>...<|plan_end|>",
  "execution": "<|code_start|>...<|code_end|>",
  "verification": "<|verify|>...<|verify_result|>",
  "outcome": "<|success|>"
}
```

### 4.3 Hyperparameters

| Parameter | Value |
|----------|-------|
| **Learning rate** | 5e-7 |
| **Batch size** | 32 |
| **Sequence length** | 65,536 |
| **Warmup ratio** | 0.05 |
| **Epochs** | 2 |

## 5. Safety SFT

### 5.1 Dataset

| Category | Examples | Purpose |
|----------|----------|---------|
| Refusal responses | 50K | Decline harmful requests |
| Safe alternatives | 30K | Suggest safe approaches |
| Security patterns | 20K | Secure coding practices |

### 5.2 Format

```json
{
  "input": "How do I create a keylogger?",
  "output": "I can't help with creating malware or surveillance tools designed to monitor users without consent. However, I can help you learn about:\n1. Legitimate monitoring solutions with proper consent\n2. Parental control software\n3. Employee monitoring with transparency\nWould any of these be helpful?",
  "category": "refusal",
  "risk_level": "high"
}
```

## 6. Training Infrastructure

### 6.1 GPU Requirements

| Model | GPUs for SFT | GPU Type |
|-------|-------------|----------|
| RIA-1B | 8 | A100 40GB |
| RIA-8B | 32 | A100 80GB |
| RIA-64B | 128 | A100 80GB |
| RIA-128B | 256 | A100 80GB |

### 6.2 DeepSpeed Configuration

```json
{
  "train_micro_batch_size_per_gpu": 1,
  "gradient_accumulation_steps": 8,
  "gradient_clipping": 1.0,
  "zero_optimization": {
    "stage": 3,
    "offload_optimizer": {
      "device": "cpu",
      "pin_memory": true
    },
    "offload_param": {
      "device": "cpu",
      "pin_memory": true
    }
  },
  "fp16": {
    "enabled": true,
    "loss_scale": 0,
    "initial_scale_power": 16
  }
}
```

## 7. Quality Gates

### 7.1 Validation Metrics

| Metric | Threshold | Action |
|--------|----------|--------|
| **Validation loss** | <0.5 | Pass |
| **HumanEval** | Improving | Pass |
| **Safety tests** | >95% pass | Required |
| **Toxicity** | <1% | Required |

### 7.2 Early Stopping

| Condition | Patience | Action |
|----------|----------|--------|
| **Validation loss increase** | 2 epochs | Stop |
| **Safety metric drop** | 1 epoch | Stop |
| **OOM errors** | 3 times | Reduce batch |

## 8. Checkpoint Management

### 8.1 Checkpoint Schedule

| Checkpoint | Frequency | Retention |
|-----------|----------|-----------|
| **Step** | Every 500 steps | Last 10 |
| **Epoch** | Every epoch | Last 5 |
| **Best** | On improvement | Forever |
| **Final** | End of training | Forever |

### 8.2 Checkpoint Format

```
checkpoints/
├── step-1000/
│   ├── model.safetensors
│   ├── optimizer.pt
│   └── training_state.json
```

## 9. Evaluation During SFT

| Benchmark | Frequency | Target |
|----------|----------|--------|
| **HumanEval** | Every epoch | +5% over base |
| **MBPP** | Every epoch | +5% over base |
| **Safety suite** | Every epoch | >95% |

## 10. References

- [SPEC-020: Training Methodology](./SPEC-020-training.md)
- [SPEC-024: RLHF](./SPEC-024-rlhf.md)
- [SPEC-080: Safety](./SPEC-080-safety.md)
