# SPEC-024: RLHF Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Training

## 1. Overview

This specification defines Reinforcement Learning from Code Feedback (RLCF)—RIA's adaptation of RLHF for agentic coding tasks.

## 2. RLCF Pipeline

```
SFT Model → Reward Model Training → PPO Training → Aligned Model
```

## 3. Reward Model Training

### 3.1 Data Collection

#### 3.1.1 Comparison Data

| Source | Pairs | Method |
|--------|------|--------|
| Code quality rankings | 500K | Expert developers |
| Solution correctness | 300K | Test pass rates |
| Security assessments | 200K | Security experts |
| Best practice adherence | 200K | Code reviewers |

#### 3.1.2 Annotation Guidelines

Annotators rank model outputs on:

| Criterion | Weight | Description |
|----------|--------|-------------|
| **Correctness** | 40% | Does code work correctly? |
| **Code quality** | 25% | Clean, readable, maintainable |
| **Security** | 20% | No vulnerabilities |
| **Efficiency** | 10% | Good performance |
| **Documentation** | 5% | Well documented |

### 3.2 Reward Model Architecture

```python
class RewardModel(nn.Module):
    def __init__(self, base_model):
        # Use SFT model as base
        self.base = base_model
        # Add reward head
        self.reward_head = nn.Sequential(
            Linear(hidden_dim, hidden_dim),
            ReLU(),
            Linear(hidden_dim, 1)
        )
    
    def forward(self, input_ids, attention_mask):
        hidden = self.base(input_ids, attention_mask).last_hidden_state
        # Pool over sequence
        pooled = hidden.mean(dim=1)
        reward = self.reward_head(pooled)
        return reward
```

### 3.3 Reward Model Training

| Parameter | Value |
|----------|-------|
| **Loss function** | Bradley-Terry |
| **Learning rate** | 1e-6 |
| **Batch size** | 256 pairs |
| **Epochs** | 3 |
| **Sequence length** | 8,192 |

#### 3.3.1 Bradley-Terry Loss

```
L = -log(σ(r_w - r_l))

Where:
  r_w = reward for winning response
  r_l = reward for losing response
```

### 3.4 Reward Model Validation

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Accuracy** | >75% | On held-out pairs |
| **Consistency** | >90% | Same input, consistent ranks |
| **Calibration** | <0.1 MSE | Predicted vs actual |

## 4. PPO Training

### 4.1 Overview

Proximal Policy Optimization (PPO) aligns model behavior with reward model preferences.

### 4.2 PPO Configuration

| Parameter | Value |
|----------|-------|
| **PPO epochs** | 3 |
| **Batch size** | 256 trajectories |
| **Learning rate** | 1e-6 |
| **KL coefficient** | 0.2 |
| **Clip range** | 0.2 |
| **Value coefficient** | 0.5 |
| **Entropy coefficient** | 0.001 |
| **GAE lambda** | 0.95 |
| **Discount factor** | 0.99 |

### 4.3 Training Loop

```python
for iteration in range(num_iterations):
    # 1. Generate responses with current policy
    trajectories = generate_responses(prompts, policy_model)
    
    # 2. Score with reward model
    rewards = reward_model(trajectories)
    
    # 3. Compute advantages
    advantages = compute_gae(rewards, value_model)
    
    # 4. PPO update
    for epoch in range(ppo_epochs):
        # Policy loss
        ratio = π_new / π_old
        clipped_ratio = clamp(ratio, 1-clip, 1+clip)
        policy_loss = -min(ratio * advantages, clipped_ratio * advantages)
        
        # Value loss
        value_loss = (value_model(states) - returns)^2
        
        # KL penalty
        kl_penalty = kl_coef * kl_divergence(π_new, π_ref)
        
        # Total loss
        loss = policy_loss + value_coef * value_loss + kl_penalty
        loss.backward()
        optimizer.step()
```

### 4.4 Multi-Reward Signal

RIA uses multiple reward signals:

| Signal | Source | Weight |
|--------|--------|--------|
| **Test pass rate** | Automated testing | 40% |
| **Code quality score** | Linter analysis | 20% |
| **Correctness** | Reward model | 20% |
| **Safety** | Security scanner | 10% |
| **Efficiency** | Complexity analysis | 5% |
| **Documentation** | Completeness check | 5% |

```
Total reward = Σ (w_i * r_i)
```

### 4.5 Safety Constraints

| Constraint | Implementation | Threshold |
|-----------|----------------|-----------|
| **No harmful code** | Content filter | 100% block |
| **No security vulns** | Static analysis | >95% clean |
| **Refusal compliance** | Behavioral test | >95% |
| **Human oversight** | Review workflow | Required |

## 5. Quality Assurance

### 5.1 Evaluation During PPO

| Metric | Frequency | Target |
|--------|----------|--------|
| **Reward score** | Every iteration | Increasing |
| **KL divergence** | Every iteration | <0.5 |
| **HumanEval** | Every 10 iterations | Maintained/improved |
| **Safety tests** | Every iteration | >95% |

### 5.2 Failure Modes

| Failure | Detection | Mitigation |
|---------|----------|------------|
| **Reward hacking** | Metric monitoring | Reward normalization |
| **Capability loss** | Benchmark tracking | Early stopping |
| **Safety regression** | Safety suite | Hard constraints |
| **Mode collapse** | Diversity metrics | Entropy bonus |

## 6. Infrastructure

### 6.1 GPU Requirements

| Model | GPUs | GPU Type | Time |
|-------|------|----------|------|
| RIA-1B | 16 | A100 40GB | 1 day |
| RIA-8B | 64 | A100 80GB | 3 days |
| RIA-64B | 256 | A100 80GB | 7 days |
| RIA-128B | 512 | A100 80GB | 14 days |

### 6.2 Storage Requirements

| Component | Storage | Notes |
|----------|---------|-------|
| **Rollouts** | 5 TB | Generated responses |
| **Checkpoints** | 10 TB | Model snapshots |
| **Reward data** | 1 TB | Annotations |

## 7. Version Control

| Version | Date | Changes |
|---------|------|---------|
| **1.0** | 2025-04-09 | Initial RLHF spec |

## 8. References

- [SPEC-020: Training Methodology](./SPEC-020-training.md)
- [SPEC-023: Fine-Tuning](./SPEC-023-fine-tuning.md)
- [SPEC-080: Safety](./SPEC-080-safety.md)
