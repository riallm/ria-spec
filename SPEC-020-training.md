# SPEC-020: Training Methodology

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Training

## 1. Overview

This specification defines the four-phase training pipeline for all RIA model tiers.

## 2. Training Phases

```
Phase 1: Pretraining → Phase 2: Code Specialization → Phase 3: ACR Training → Phase 4: Alignment
   (General LM)           (Code Understanding)         (Agentic Tasks)          (Safety + Quality)
```

### 2.1 Phase Summary

| Phase | Purpose | Data | Duration |
|-------|---------|------|----------|
| **Phase 1: Pretraining** | General language and code understanding | 2T tokens | 2-12 weeks |
| **Phase 2: Code Specialization** | Programming expertise | 500B code tokens | 1-4 weeks |
| **Phase 3: ACR Training** | Agentic reasoning | 100B agentic tokens | 1-3 weeks |
| **Phase 4: Alignment** | Safety and quality | 50B curated tokens | 1-2 weeks |

## 3. Phase 1: Pretraining

### 3.1 Objectives

- Learn general language patterns
- Acquire world knowledge
- Develop basic code understanding
- Build foundation for specialization

### 3.2 Data Composition

| Source | Percentage | Tokens (RIA-128B) | Purpose |
|--------|-----------|-------------------|---------|
| Common Crawl | 40% | 2.05T | General knowledge |
| Wikipedia + Books | 15% | 770B | Factual knowledge |
| Academic Papers | 10% | 513B | Technical knowledge |
| GitHub Code | 25% | 1.28T | Code patterns |
| Technical Docs | 10% | 513B | API knowledge |
| **Total** | **100%** | **5.13T** | |

### 3.3 Training Objectives

#### 3.3.1 Causal Language Modeling

Primary objective: predict next token

```
L_clm = -Σ log P(x_t | x_<t)
```

#### 3.3.2 Span Corruption

15% of tokens replaced with sentinel tokens

```
Input:  "def hello(name): return f'Hello, <extra_id_0>'"
Target: "def hello(name): return f'Hello, {name}'"
```

#### 3.3.3 Document Infilling

Remove random sentences/paragraphs, model reconstructs

### 3.4 Hyperparameters by Tier

| Parameter | RIA-1B | RIA-8B | RIA-64B | RIA-128B |
|----------|--------|--------|---------|----------|
| **Learning rate** | 3e-4 | 3e-4 | 1.5e-4 | 1e-4 |
| **Warmup steps** | 2,000 | 2,000 | 5,000 | 5,000 |
| **LR schedule** | Cosine | Cosine | Cosine | Cosine |
| **Weight decay** | 0.1 | 0.1 | 0.1 | 0.1 |
| **Adam β1** | 0.9 | 0.9 | 0.9 | 0.9 |
| **Adam β2** | 0.95 | 0.95 | 0.95 | 0.95 |
| **Adam ε** | 1e-8 | 1e-8 | 1e-8 | 1e-8 |
| **Gradient clip** | 1.0 | 1.0 | 1.0 | 1.0 |
| **Batch size** | 2M tok | 4M tok | 8M tok | 16M tok |
| **Seq length** | 4,096 | 8,192 | 16,384 | 32,768 |

### 3.5 Infrastructure

| Tier | GPUs | GPU Type | Time | Total GPU-hours |
|------|------|----------|------|----------------|
| **RIA-1B** | 64 | A100 40GB | 2 weeks | 21,504 |
| **RIA-8B** | 256 | A100 80GB | 4 weeks | 430,080 |
| **RIA-64B** | 1,024 | A100 80GB | 8 weeks | 3,440,640 |
| **RIA-128B** | 2,048 | A100 80GB | 12 weeks | 10,321,920 |

## 4. Phase 2: Code Specialization

### 4.1 Objectives

- Deep programming language expertise
- Multi-language proficiency
- Code pattern recognition
- API knowledge

### 4.2 Data: CodeNet-Pro

| Source | Size | Filtering |
|--------|------|-----------|
| GitHub repositories | 50M files | Stars >10, license, activity |
| Stack Overflow | 25M posts | Accepted answers only |
| Programming tutorials | 500K | Step-by-step format |
| Code reviews | 10M PRs | Merged, reviewed PRs |
| Bug fixes | 5M commits | Linked issues, tests |
| Documentation | 100M docs | API docs, READMEs |

### 4.3 Training Objectives

| Objective | Weight | Description |
|----------|--------|-------------|
| Code completion | 40% | Next line/block prediction |
| Code translation | 15% | Cross-language conversion |
| Code summarization | 15% | Generate docstrings |
| Code repair | 15% | Fix buggy code |
| Code retrieval | 10% | Find code from description |
| Cross-file QA | 5% | Multi-file understanding |

## 5. Phase 3: Agentic Code Reasoning (ACR)

### 5.1 Objectives

- Multi-step task planning
- Tool integration proficiency
- Self-debugging capabilities
- Iterative refinement loops

### 5.2 ACR-500B Dataset

| Task Type | Examples | Tokens |
|----------|----------|--------|
| Feature addition | 50M | 200B |
| Bug fixing | 30M | 120B |
| Refactoring | 20M | 80B |
| Migration | 15M | 60B |
| Testing | 25M | 100B |
| Documentation | 20M | 40B |
| **Total** | **160M** | **600B** |

### 5.3 Training Format

Each example includes complete reasoning trace:

```
<|think|>Analysis and planning<|/think|>
<|file:filename|>
<|code_start|>
code changes
<|code_end|>
<|verify|>Verification steps<|verify_result|>
<|tool_call|>Tool invocation<|tool_result|>
<|success|> or <|failure|>
```

### 5.4 Loss Functions

| Loss | Weight | Purpose |
|------|--------|---------|
| Planning loss | 25% | Correct plan generation |
| Execution loss | 30% | Correct code generation |
| Verification loss | 15% | Predict test outcomes |
| Debugging loss | 15% | Identify and fix bugs |
| Tool selection loss | 10% | Choose appropriate tools |
| Consistency loss | 5% | Multi-turn coherence |

## 6. Phase 4: Alignment

### 6.1 Supervised Fine-Tuning (SFT)

| Data | Size | Source |
|------|------|--------|
| Expert demonstrations | 100K tasks | Professional developers |
| Multi-turn interactions | 50K sessions | Real coding sessions |
| Best practices | 200K examples | Code review datasets |
| Security patterns | 50K examples | Security experts |

### 6.2 Reinforcement Learning (RLCF)

#### 6.2.1 Reward Model Training

| Signal | Weight | Description |
|--------|--------|-------------|
| Test pass rate | 40% | Generated tests pass |
| Code quality | 20% | Linter scores, complexity |
| Correctness | 20% | Solves the problem |
| Safety | 10% | No vulnerabilities |
| Efficiency | 5% | Good complexity |
| Documentation | 5% | Complete docs |

#### 6.2.2 PPO Training

| Parameter | Value |
|----------|-------|
| **PPO epochs** | 3 |
| **Batch size** | 256 trajectories |
| **Learning rate** | 1e-6 |
| **KL coefficient** | 0.2 |
| **Clip range** | 0.2 |
| **Value coefficient** | 0.5 |

## 7. Quality Control

### 7.1 Checkpoint Management

| Checkpoint | Frequency | Retention |
|-----------|-----------|-----------|
| **Step checkpoint** | Every 1000 steps | Last 10 |
| **Epoch checkpoint** | Every epoch | Last 5 |
| **Best checkpoint** | On metric improvement | Forever |
| **Final checkpoint** | End of training | Forever |

### 7.2 Validation Metrics

| Metric | Frequency | Target |
|--------|----------|--------|
| **Validation loss** | Every 100 steps | Decreasing |
| **Perplexity** | Every epoch | <15 (code) |
| **HumanEval** | Every 2 epochs | Increasing |
| **Safety tests** | Every epoch | >90% pass |

## 8. Reproducibility

### 8.1 Seeds

| Component | Seed |
|----------|------|
| Data shuffling | 42 |
| Model initialization | 123 |
| Dropout | 456 |
| Sampling | 789 |

### 8.2 Deterministic Training

- All operations use deterministic algorithms where available
- Data loading order fixed by seed
- Checkpoint verification with checksums

## 9. References

- [SPEC-021: Data Pipeline](./SPEC-021-data-pipeline.md)
- [SPEC-023: Fine-Tuning](./SPEC-023-fine-tuning.md)
- [SPEC-024: RLHF](./SPEC-024-rlhf.md)
