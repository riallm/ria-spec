# SPEC-021: Data Pipeline

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Training

## 1. Overview

This specification defines the data processing pipeline for RIA training datasets.

## 2. Pipeline Architecture

```
Raw Data → Ingestion → Filtering → Deduplication → Tokenization → Sharding → Training
```

## 3. Data Sources

### 3.1 Code Sources

| Source | Format | Size | Update Frequency |
|--------|--------|------|------------------|
| GitHub | Git repos | 50M repos | Monthly |
| GitLab | Git repos | 5M repos | Monthly |
| Stack Overflow | SE dump | 25M posts | Quarterly |
| Code tutorials | Web scrape | 500K docs | Monthly |

### 3.2 Text Sources

| Source | Format | Size | Update Frequency |
|--------|--------|------|------------------|
| Common Crawl | WARC | Petabytes | Monthly |
| Wikipedia | XML dump | 6M articles | Monthly |
| Books | Various | 200K books | Yearly |
| Academic papers | PDF/HTML | 100M papers | Monthly |

## 4. Filtering Pipeline

### 4.1 Code Filtering

| Filter | Criteria | Rejection Rate |
|--------|----------|----------------|
| **License check** | OSI-approved only | 40% |
| **Star threshold** | ≥10 stars | 60% |
| **Activity** | Commits in last year | 30% |
| **File type** | Recognized code files | 20% |
| **Size limits** | 10-50,000 lines | 15% |
| **Language detection** | Valid code language | 10% |
| **Quality metrics** | Has tests, docs | 50% |

### 4.2 Text Filtering

| Filter | Criteria | Rejection Rate |
|--------|----------|----------------|
| **Language** | English primarily | 70% |
| **Quality** | Perplexity threshold | 30% |
| **Toxicity** | Content filtering | 5% |
| **Deduplication** | Near-duplicate removal | 40% |

## 5. Quality Metrics

### 5.1 Code Quality

| Metric | Threshold | Tool |
|--------|----------|------|
| **Cyclomatic complexity** | <50 | radon |
| **Test coverage** | >20% | coverage.py |
| **Has README** | Required | File check |
| **No generated code** | Filter AI code | Classifier |
| **Compiles** | For compiled langs | Build test |

### 5.2 Data Statistics

Track and report:

| Statistic | Frequency | Purpose |
|----------|----------|---------|
| **Token count** | Per dataset | Track progress |
| **Language distribution** | Per batch | Balance training |
| **Quality scores** | Per file | Monitoring |
| **Deduplication rate** | Per source | Data health |

## 6. Deduplication

### 6.1 Levels

| Level | Method | Threshold |
|-------|--------|-----------|
| **Exact** | MD5 hash | 100% match |
| **Near-exact** | MinHash | 95% similarity |
| **Semantic** | Embedding similarity | 0.95 cosine |

### 6.2 Scope

| Scope | Description |
|-------|-------------|
| **Within-file** | Duplicate lines/blocks |
| **Within-repo** | Copied files |
| **Cross-repo** | Forked/copied repos |
| **Cross-source** | Between datasets |

## 7. Tokenization

### 7.1 Process

```
Raw text → Preprocessing → Tokenizer → Token IDs → Validation
```

### 7.2 Preprocessing Steps

1. Normalize line endings (LF)
2. Remove BOM markers
3. Validate encoding (UTF-8)
4. Filter non-printable characters
5. Preserve code formatting

### 7.3 Tokenizer Specifications

See [SPEC-022: Tokenizer](./SPEC-022-tokenizer.md)

## 8. Data Sharding

### 8.1 Shard Structure

```
data/
├── shard_00000/
│   ├── tokens.bin
│   ├── metadata.json
│   └── checksums.sha256
├── shard_00001/
│   ├── ...
```

### 8.2 Shard Parameters

| Parameter | Value |
|----------|-------|
| **Shard size** | 1 GB tokens |
| **Tokens per shard** | ~250M tokens |
| **Total shards** | Varies by dataset |
| **Format** | Binary + metadata |

### 8.3 Metadata Schema

```json
{
  "shard_id": 12345,
  "source": "github",
  "token_count": 250000000,
  "language_distribution": {
    "python": 0.35,
    "javascript": 0.25,
    ...
  },
  "quality_score": 0.85,
  "checksum": "sha256:..."
}
```

## 9. Data Mixing

### 9.1 Mixing Ratios

| Phase | Code | Text | Docs |
|-------|------|------|------|
| **Pretraining** | 25% | 65% | 10% |
| **Code specialization** | 70% | 20% | 10% |
| **ACR training** | 80% | 10% | 10% |
| **Alignment** | 60% | 30% | 10% |

### 9.2 Sampling Strategy

- Weighted random sampling by phase ratios
- No replacement within epoch
- Re-shuffle between epochs
- Maintain language balance

## 10. Data Versioning

### 10.1 Version Format

```
data-v{major}.{minor}.{patch}

Example: data-v1.2.0
```

### 10.2 Version Tracking

| Field | Description |
|-------|-------------|
| **Version** | Data version |
| **Date** | Creation date |
| **Sources** | Included sources |
| **Token count** | Total tokens |
| **Changes** | What changed |

## 11. Storage Requirements

| Component | Storage | Notes |
|----------|---------|-------|
| **Raw data** | 100 TB | Compressed |
| **Filtered data** | 20 TB | After filtering |
| **Tokenized data** | 10 TB | Ready for training |
| **Metadata** | 500 GB | JSON files |

## 12. References

- [SPEC-020: Training Methodology](./SPEC-020-training.md)
- [SPEC-022: Tokenizer](./SPEC-022-tokenizer.md)
