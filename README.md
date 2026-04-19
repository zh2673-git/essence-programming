# Essence Programming

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> A systematic methodology for software design and implementation based on **First Principles**.

## Core Philosophy

**Project = Class, Module = Property, Interface = Contract, Route = Method Dispatch**

The essence of software development: **Property-based static development + Method-based dynamic development**

- **Property**: Static description, state, configuration (modules/data/config)
- **Method**: Dynamic behavior, execution logic (orchestration and utilization of properties)
- **Route**: Multi-scenario dispatch mechanism for methods (same scenario, different strategies)
- **Interface**: Cross-module contracts (each manages its own affairs, collaborating through contracts)

**Key Principle**: All design must be derived from **Essential Logic**, eliminating arbitrariness.

---

## When to Use This Skill

- **Starting a new project**: Design system architecture from scratch, clarifying core problems
- **Refactoring existing systems**: Reorganize design thinking, optimize architecture
- **Designing complex modules**: Deep analysis of module essence, avoiding over-engineering
- **Code review**: Verify if implementation aligns with design essence

---

## The Triad Methodology: Logical Derivation Framework

```
┌─────────────────────────────────────────────────────────┐
│                    Triad Logic Derivation                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   Stage 1: WHAT (What is it?)                           │
│   ├── Essential characteristics (connotation)           │
│   ├── Key properties (extension)                        │
│   └── Distinguishing properties (boundaries)            │
│              ↓                                          │
│   Stage 2: WHY (Why is it needed?)                      │
│   ├── Reason for existence                              │
│   ├── Technology selection                              │
│   └── Architecture design                               │
│              ↓                                          │
│   Stage 3: HOW (How to implement?)                      │
│   ├── Module division                                   │
│   ├── Method design                                     │
│   └── Interface definition                              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Core Principle**: Each stage's derivation must be based on the previous stage's conclusions, forming a rigorous logical chain.

---

## Core Formula

```
┌─────────────────────────────────────────────────────────┐
│                    Essence Programming Formula          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   Triad Derivation:                                     │
│   WHAT (Essence) → WHY (Architecture) → HOW (Impl)      │
│                                                         │
│   Development Mode:                                     │
│   Project Dev = Property-based Static + Method-based    │
│                                                         │
│   Method Extension:                                     │
│   Multi-scenario = Same core method with route branches │
│                                                         │
│   Interface Essence:                                    │
│   Interface = Cross-module collaboration contract       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Start

### For AI Assistants (Claude/Trae/Cursor)

1. Copy [SKILL.md](./SKILL.md) to your project's `.trae/skills/` or equivalent directory
2. Reference the skill when starting new projects or designing modules
3. Follow the Triad Methodology for systematic derivation

### For Developers

1. Read the [SKILL.md](./SKILL.md) to understand the methodology
2. Check [examples/](./examples/) for real-world case studies
3. Apply the framework to your own projects

---

## Project Structure

```
essence-programming/
├── README.md              # This file
├── README.zh-CN.md        # Chinese version
├── SKILL.md               # Core methodology documentation
├── examples/              # Real-world examples
│   └── datadone-analysis/ # Data analysis system example
│       ├── 00-datadone-三阶解构.md
│       ├── 01-datasource-三阶解构.md
│       ├── 02-analysis-三阶解构.md
│       ├── 03-export-三阶解构.md
│       ├── 04-utils-三阶解构.md
│       └── 99-架构图谱.md
└── docs/                  # Additional documentation
    ├── what-why-how.md    # Triad methodology deep dive
    └── case-studies/      # More case studies
```

---

## Example: Data Analysis System

The [examples/datadone-analysis/](./examples/datadone-analysis/) directory contains a complete real-world example of applying Essence Programming to a data analysis system.

This example demonstrates:
- How to decompose a complex system using the Triad Methodology
- Property-based module design
- Method-based dynamic logic
- Route-based scenario dispatch

---

## Documentation

- [SKILL.md](./SKILL.md) - Complete methodology documentation
- [docs/what-why-how.md](./docs/what-why-how.md) - Triad methodology deep dive
- [examples/](./examples/) - Real-world case studies

---

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- Inspired by First Principles thinking
- Built for systematic software design
- Made for developers who care about essence over form
