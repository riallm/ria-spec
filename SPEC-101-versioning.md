# SPEC-101: Versioning Policy

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Versioning

## 1. Overview

This specification defines versioning schemes for RIA models and riallm software.

## 2. Semantic Versioning

### 2.1 Version Format

```
MAJOR.MINOR.PATCH

Examples:
1.0.0  - Initial release
1.1.0  - New features, backward compatible
2.0.0  - Breaking changes
```

### 2.2 Version Components

| Component | Increment When | Examples |
|----------|---------------|----------|
| **MAJOR** | Breaking changes | New architecture, API changes |
| **MINOR** | New features, backward compatible | New benchmarks, optimizations |
| **PATCH** | Bug fixes only | Security fixes, typo fixes |

## 3. Model Versioning

### 3.1 Model Version Format

```
ria-{tier}-{version}-{quantization}

Examples:
ria-8b-1.0.0-q4_k_m.gguf
ria-128b-1.1.0-f16.gguf
```

### 3.2 Version Progression

| Version | Description | Changes |
|---------|-------------|---------|
| **0.x.x** | Beta/preview | Experimental |
| **1.0.0** | Stable release | Production ready |
| **1.x.x** | Incremental updates | Improvements |
| **2.0.0** | Major update | Architecture changes |

### 3.3 Model Naming

| Tier | Name Format | Example |
|------|------------|---------|
| **1B** | ria-1b-{version} | ria-1b-1.0.0 |
| **8B** | ria-8b-{version} | ria-8b-1.0.0 |
| **64B** | ria-64b-{version} | ria-64b-1.0.0 |
| **128B** | ria-128b-{version} | ria-128b-1.0.0 |

## 4. riallm Versioning

### 4.1 riallm Version Format

```
riallm-{major}.{minor}.{patch}

Examples:
riallm-0.1.0  - Initial beta
riallm-1.0.0  - First stable
riallm-1.2.3  - Patch release
```

### 4.2 Compatibility Matrix

| riallm Version | RIA Models | GGUF Version | Rust Version |
|---------------|-----------|--------------|--------------|
| **0.1.x** | RIA 1.0.x | 3 | 1.75+ |
| **1.0.x** | RIA 1.0.x, 2.0.x | 3 | 1.80+ |
| **1.x.x** | RIA 1.x.x | 3+ | 1.80+ |

## 5. Release Process

### 5.1 Release Checklist

- [ ] All tests passing
- [ ] Benchmarks meet targets
- [ ] Safety evaluation complete
- [ ] Documentation updated
- [ ] Changelog written
- [ ] GGUF files generated and validated
- [ ] Checksums generated
- [ ] Release notes published

### 5.2 Release Schedule

| Release Type | Frequency | Timing |
|-------------|----------|--------|
| **Patch** | As needed | Any time |
| **Minor** | Monthly | First Tuesday |
| **Major** | Quarterly | Planned |
| **Model update** | Per training cycle | After training |

## 6. Deprecation Policy

### 6.1 Deprecation Timeline

| Phase | Duration | Support |
|-------|----------|---------|
| **Active** | Current | Full support |
| **Maintenance** | 6 months | Bug fixes only |
| **Deprecated** | 3 months | Security only |
| **End-of-life** | - | No support |

### 6.2 Deprecation Notice

```
DEPRECATION NOTICE
==================
Model: ria-8b-1.0.0
Deprecation Date: 2025-10-01
End-of-Life Date: 2026-01-01
Replacement: ria-8b-2.0.0
Reason: Architecture improvements
Migration Guide: https://docs.riallm.org/migrate-1-to-2
```

## 7. Branch Strategy

### 7.1 Git Branches

| Branch | Purpose | Stability |
|--------|---------|-----------|
| **main** | Stable releases | Production |
| **develop** | Integration | Testing |
| **feature/*** | New features | Experimental |
| **hotfix/*** | Urgent fixes | As needed |

### 7.2 Tag Format

```
v{major}.{minor}.{patch}

Examples:
v1.0.0
v1.2.3
v2.0.0-rc.1
```

## 8. Changelog Format

```markdown
# Changelog

## [1.1.0] - 2025-05-01

### Added
- Support for RIA-128B model
- New quantization type Q3_K_M
- Streaming API improvements

### Changed
- Improved layer loading performance by 15%
- Updated GGUF metadata schema

### Fixed
- Memory leak in KV cache
- Tokenizer edge case with Unicode

### Security
- Fixed prompt injection vulnerability
```

## 9. References

- [SPEC-100: Testing](./SPEC-100-testing.md)
- [SPEC-102: Compatibility](./SPEC-102-compatibility.md)
