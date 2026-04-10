# RIA Specification Documents

Official technical specifications for the **RIA** (Rust Intelligence for Agentic Coding) family of large language models.

## 📚 Documentation

| Document | Description | Status |
|----------|-------------|--------|
| [RIA White Paper](./ria-white-paper.md) | Comprehensive technical specification, architecture, training methodology, evaluation, and deployment guidelines | v1.0 ✅ |
| [Model Cards](./model-cards.md) | Detailed specifications for each model tier (1B, 8B, 64B, 128B) | 🚧 Coming Soon |
| [API Reference](./api-reference.md) | REST API and SDK documentation | 🚧 Coming Soon |
| [Integration Guide](./integration-guide.md) | IDE plugins, CI/CD integration, tool development | 🚧 Coming Soon |
| [Safety & Alignment](./safety-alignment.md) | Safety measures, red teaming, evaluation protocols | 🚧 Coming Soon |
| [Benchmark Methodology](./benchmark-methodology.md) | Evaluation procedures, dataset details, scoring | 🚧 Coming Soon |

## 🎯 Quick Start

### Understanding RIA

The **RIA White Paper** is your starting point. It covers:

- **What is RIA**: A family of LLMs specialized for autonomous software development
- **Four model tiers**: 1B, 8B, 64B, 128B parameters
- **Key innovations**: Agentic Code Reasoning (ACR), Multi-Hop Code Attention, Tool Integration
- **riallm integration**: Memory-optimized deployment on consumer hardware
- **Benchmarks**: State-of-the-art performance on SWE-bench, HumanEval, MultiPL-E

### For Developers

1. **Read the [White Paper](./ria-white-paper.md)** → Understand the architecture and capabilities
2. **Check riallm docs** → Learn how to deploy RIA models
3. **Choose your model tier** → Based on hardware and use case
4. **Start coding** → Integrate RIA into your workflow

### For Researchers

1. **Read Sections 2-3** → Architecture and training methodology
2. **Review Section 7** → Evaluation results and ablation studies
3. **Check Appendix** → Detailed hyperparameters and configurations
4. **Cite our work** → See citation format below

## 📊 Model Overview

| Model | Parameters | Context | riallm VRAM | Best For |
|-------|-----------|---------|-------------|----------|
| **RIA-1B** | 1.0B | 32K | 1 GB | Edge devices, quick tasks |
| **RIA-8B** | 8.2B | 128K | 4-8 GB | Interactive coding assistant |
| **RIA-64B** | 64.5B | 256K | 16-32 GB | Complex software engineering |
| **RIA-128B** | 128.3B | 512K | 32-64 GB | Enterprise-scale autonomous coding |

All models are fully compatible with [riallm](../riallm/) for memory-optimized inference.

## 🔗 Related Resources

- **[riallm](../riallm/)** - Rust inference engine for running RIA models
- **[Examples](../examples/)** - Usage examples and tutorials
- **[Benchmarks](../benchmarks/)** - Performance comparison and evaluation scripts

## 📋 Citation

If you use RIA models or reference these specifications in your research, please cite:

```bibtex
@article{ria2025,
  title={RIA: Rust Intelligence for Agentic Coding},
  author={riallm Research Team},
  journal={arXiv preprint},
  year={2025},
  url={https://github.com/riallm/ria}
}
```

## 📝 Contributing

We welcome contributions to the specification documents:

- **Bug reports**: Errors or inconsistencies in the documentation
- **Clarifications**: Sections that need more detail or better explanations
- **Extensions**: New use cases, integration patterns, or benchmark results
- **Translations**: Community translations of the specification

Please open an issue or submit a pull request.

## 📄 License

The RIA specification documents are licensed under the [DOSL-IIE-1.0 License](./LICENSE).

## 📞 Contact

- **Email**: spec@riallm.org
- **GitHub Issues**: [Report documentation issues](https://github.com/riallm/ria-spec/issues)
- **Discussions**: [Join the conversation](https://github.com/riallm/ria-spec/discussions)

---

**Version**: 1.0  
**Last Updated**: April 2025  
**Maintained by**: riallm Research Team

*These specifications describe research in progress and may be updated as development continues.*
