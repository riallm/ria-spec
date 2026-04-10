# SPEC-111: Contributing Guidelines

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Community

## 1. Overview

This specification defines how to contribute to the RIA project.

## 2. Getting Started

### 2.1 Prerequisites

| Requirement | Version | Purpose |
|------------|---------|---------|
| **Rust** | 1.80+ | riallm development |
| **Python** | 3.10+ | Training scripts |
| **Git** | 2.30+ | Version control |
| **CUDA** | 12.0+ (optional) | GPU support |

### 2.2 Setup

```bash
# Clone repository
git clone https://github.com/riallm/ria.git
cd ria

# Install Rust dependencies
cargo build

# Install Python dependencies
pip install -r requirements.txt

# Run tests
cargo test
python -m pytest
```

## 3. Contribution Types

### 3.1 Code Contributions

| Type | Description | Process |
|------|-------------|---------|
| **Bug fixes** | Fix issues | PR with tests |
| **Features** | New functionality | RFC + PR |
| **Optimizations** | Performance improvements | Benchmarks required |
| **Documentation** | Docs, examples | Direct PR |

### 3.2 Non-Code Contributions

| Type | Description | How |
|------|-------------|-----|
| **Bug reports** | Issue reports | GitHub Issues |
| **Feature requests** | Enhancement ideas | GitHub Discussions |
| **Documentation** | Guides, tutorials | PR to docs |
| **Translations** | Localization | PR with i18n |
| **Community** | Help others | Discussions, Discord |

## 4. Development Workflow

### 4.1 Branch Strategy

```
main (stable)
  ↑
  ├── release/v1.0 (release prep)
  │
develop (integration)
  ↑
  ├── feature/my-feature
  ├── bugfix/fix-issue
  └── hotfix/urgent-fix
```

### 4.2 Pull Request Process

```
1. Fork repository
2. Create feature branch
3. Make changes
4. Write tests
5. Update documentation
6. Submit PR
7. Address review feedback
8. Merge after approval
```

### 4.3 PR Requirements

| Requirement | Description |
|------------|-------------|
| **Description** | What and why |
| **Tests** | New/updated tests |
| **Documentation** | Updated docs if needed |
| **Changelog** | Entry in CHANGELOG.md |
| **Breaking changes** | Migration guide if needed |

## 5. Code Style

### 5.1 Rust Style

```bash
# Format code
cargo fmt

# Check clippy
cargo clippy -- -D warnings

# Run tests
cargo test
```

#### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| **Types** | PascalCase | `ModelConfig` |
| **Functions** | snake_case | `load_model()` |
| **Constants** | SCREAMING_SNAKE_CASE | `MAX_TOKENS` |
| **Modules** | snake_case | `model_loading` |

### 5.2 Python Style

```bash
# Format code
black .

# Type checking
mypy .

# Linting
ruff check .
```

## 6. Testing Requirements

### 6.1 Test Coverage

| Component | Minimum Coverage |
|----------|-----------------|
| **Core logic** | 90% |
| **API layer** | 85% |
| **Utilities** | 80% |
| **Examples** | N/A (manual) |

### 6.2 Running Tests

```bash
# All tests
cargo test

# Specific module
cargo test -p riallm-core

# Integration tests
cargo test --test '*'

# Benchmarks
cargo bench
```

## 7. Review Process

### 7.1 Review Checklist

| Check | Description |
|-------|-------------|
| **Correctness** | Code works as intended |
| **Tests** | Adequate test coverage |
| **Performance** | No regressions |
| **Security** | No vulnerabilities |
| **Documentation** | Docs updated |
| **Style** | Follows conventions |

### 7.2 Approval Requirements

| PR Type | Approvals Required |
|---------|-------------------|
| **Bug fix** | 1 reviewer |
| **Feature** | 2 reviewers |
| **Breaking change** | 2 reviewers + maintainer |
| **Hotfix** | 1 reviewer (expedited) |

## 8. Commit Guidelines

### 8.1 Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### 8.2 Commit Types

| Type | Description | Example |
|------|-------------|---------|
| **feat** | New feature | feat(api): add streaming |
| **fix** | Bug fix | fix(gguf): parse error |
| **docs** | Documentation | docs: update README |
| **style** | Formatting | style: cargo fmt |
| **refactor** | Code change | refactor: simplify |
| **test** | Tests | test: add gguf tests |
| **perf** | Performance | perf: optimize loading |

### 8.3 Example Commit

```
feat(quantization): add Q3_K_M support

Add Q3_K_M quantization type for better quality/size trade-off.

Closes #123
```

## 9. Release Process

### 9.1 Release Checklist

- [ ] All PRs merged
- [ ] Tests passing
- [ ] Benchmarks meet targets
- [ ] Changelog updated
- [ ] Version bumped
- [ ] Release notes written
- [ ] Tags created
- [ ] Artifacts published

### 9.2 Release Schedule

| Release | Frequency | Day |
|---------|----------|-----|
| **Patch** | As needed | Any |
| **Minor** | Monthly | First Tuesday |
| **Major** | Quarterly | Planned |

## 10. Community

### 10.1 Communication Channels

| Channel | Purpose | Link |
|---------|---------|------|
| **GitHub Issues** | Bug reports | github.com/riallm/ria/issues |
| **GitHub Discussions** | Questions, ideas | github.com/riallm/ria/discussions |
| **Discord** | Real-time chat | discord.gg/riallm |
| **Email** | Formal contact | hello@riallm.org |

### 10.2 Code of Conduct

| Principle | Description |
|----------|-------------|
| **Respect** | Be respectful to all |
| **Inclusion** | Welcome everyone |
| **Collaboration** | Work together |
| **Focus** | Stay on topic |
| **Growth** | Help others learn |

## 11. Recognition

### 11.1 Contributor Levels

| Level | Contributions | Recognition |
|-------|--------------|-------------|
| **First-time** | 1 PR | Welcome message |
| **Contributor** | 5 PRs | README mention |
| **Core** | 20 PRs | Team access |
| **Maintainer** | Ongoing | Full access |

### 11.2 Hall of Fame

Contributors are recognized in:
- CONTRIBUTORS.md file
- Release notes
- Annual report
- Conference talks

## 12. Legal

### 12.1 License

All contributions are licensed under Apache 2.0.

### 12.2 DCO

We use Developer Certificate of Origin:

```
Signed-off-by: Random J Developer <random@developer.example.org>
```

## 13. References

- [SPEC-101: Versioning](./SPEC-101-versioning.md)
- [SPEC-110: Roadmap](./SPEC-110-roadmap.md)
