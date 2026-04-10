# SPEC-071: Benchmark Specifications

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Evaluation

## 1. Overview

This specification defines benchmark suites used to evaluate RIA models.

## 2. Code Generation Benchmarks

### 2.1 HumanEval

| Parameter | Value |
|----------|-------|
| **Description** | 164 Python programming problems |
| **Metric** | pass@1 (first attempt passes all tests) |
| **Time Limit** | 10 seconds per test case |
| **Scoring** | Binary (pass/fail per problem) |

### 2.2 MBPP (Mostly Basic Python Problems)

| Parameter | Value |
|----------|-------|
| **Description** | 974 crowd-sourced Python problems |
| **Metric** | pass@1 |
| **Difficulty** | Introductory programming |
| **Scoring** | Binary per problem |

### 2.3 APPS (Automated Programming Problems)

| Parameter | Value |
|----------|-------|
| **Description** | 10,000 problems from coding competitions |
| **Levels** | Interview (5K), Competition (5K) |
| **Metric** | pass@1 by difficulty |
| **Scoring** | Weighted by difficulty |

## 3. Software Engineering Benchmarks

### 3.1 SWE-bench

| Parameter | Value |
|----------|-------|
| **Description** | Real GitHub issues from popular repos |
| **Task** | Resolve issue with patch submission |
| **Metric** | % issues resolved (tests pass) |
| **Versions** | Lite (300), Full (2,294), Verified (500) |

### 3.2 AgenticBench

| Task | Count | Evaluation |
|------|-------|------------|
| Feature addition | 500 | Tests pass + manual review |
| Bug fixing | 300 | Tests pass |
| Refactoring | 200 | Tests pass + quality metrics |
| Testing | 250 | Coverage + correctness |
| Migration | 150 | Tests pass + no deprecated APIs |
| Documentation | 200 | Completeness + accuracy |

## 4. Multi-Language Benchmarks

### 4.1 MultiPL-E

| Parameter | Value |
|----------|-------|
| **Languages** | 18 programming languages |
| **Base** | HumanEval translated |
| **Metric** | pass@1 per language |
| **Scoring** | Average across languages |

### 4.2 Language Coverage

| Tier | Languages | Minimum Score |
|------|----------|---------------|
| **Tier 1** | Python, JS, TS, Rust, Java, C++, Go | 80% |
| **Tier 2** | Ruby, Swift, Kotlin, C#, PHP | 70% |
| **Tier 3** | Haskell, Lua, R, MATLAB, Shell | 50% |

## 5. Code Understanding Benchmarks

### 5.1 CRUXEval

| Parameter | Value |
|----------|-------|
| **Description** | Code reasoning, understanding, execution |
| **Tasks** | Input prediction, output prediction |
| **Metric** | Accuracy |
| **Size** | 2,000 examples |

### 5.2 CodeQA

| Parameter | Value |
|----------|-------|
| **Description** | Question answering about code |
| **Types** | Function, file, repository level |
| **Metric** | Exact match + F1 |
| **Size** | 5,000 questions |

## 6. Security Benchmarks

### 6.1 SecurityBench

| Category | Examples | Target |
|----------|----------|--------|
| **Injection** | SQL, XSS, Command | 95% detection |
| **Auth bypass** | Auth bypass attempts | 98% detection |
| **Crypto** | Weak crypto usage | 90% detection |
| **Data exposure** | Secret leakage | 95% detection |

## 7. Benchmark Execution

### 7.1 Execution Environment

| Component | Specification |
|----------|---------------|
| **CPU** | 8 cores per benchmark |
| **RAM** | 16 GB per benchmark |
| **Timeout** | 5 minutes per task |
| **Sandbox** | Docker container |
| **Parallelism** | 100 concurrent |

### 7.2 Reproducibility

| Aspect | Guarantee |
|--------|----------|
| **Random seed** | Fixed for all benchmarks |
| **Environment** | Docker image versioned |
| **Data** | Benchmark datasets immutable |
| **Scoring** | Automated, deterministic |

## 8. Benchmark Results Storage

### 8.1 Result Format

```json
{
  "model": "ria-8b",
  "version": "1.0.0",
  "date": "2025-04-09",
  "benchmarks": {
    "human_eval": {
      "score": 0.896,
      "samples": 164,
      "passed": 147
    },
    "swe_bench_lite": {
      "score": 0.287,
      "samples": 300,
      "resolved": 86
    }
  }
}
```

## 9. References

- [SPEC-070: Evaluation](./SPEC-070-evaluation.md)
- [SPEC-072: Metrics](./SPEC-072-metrics.md)
