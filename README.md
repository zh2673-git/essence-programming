# 本质探索 (Essence Programming)

A systematic methodology for software design and implementation based on First Principles and Formal Logic.

English | [中文](#中文说明)

---

## The Philosophy: Logic-Driven Design

Essence Programming is not merely a development technique — it is a philosophical approach to software engineering rooted in formal logic and ontological analysis.

### The Logical Foundation

| Logical Operation | Software Concept | Question Addressed |
|---|---|---|
| **Connotation (内涵)** | Essential Characteristics | What is it in essence? |
| **Extension (外延)** | Key Properties | What does it consist of? |
| **Differentiation (区分)** | Distinguishing Properties | What is it NOT? |

### The Core Formula

```
Project = Class, Module = Property,
Interface = Contract, Route = Method Dispatch
```

**Key Principle**: All design must be derived from Essential Logic, eliminating arbitrariness.

### Gradient Understanding (坡度理解)

A progressive cognition method applied within each stage of the Triad Methodology:

| Stage | Action | Output | Time Target |
|---|---|---|---|
| **Stage 1** | Point out the concept | One-sentence definition + essence table | 30 seconds |
| **Stage 2** | Build connections | Property table with ✅❌ markers | 2 minutes |
| **Stage 3** | Detailed explanation | In-depth analysis + code/steps | As needed |

---

## The Triad Methodology

### Stage 1: WHAT — Defining Essence Through Logic

From the perspective of formal definition theory:
- **Essential Characteristics (Connotation)**: The necessary and sufficient conditions that make the system what it is
- **Key Properties (Extension)**: The complete set of attributes derived logically from the essential characteristics
- **Distinguishing Properties (Differentiation)**: The boundaries that separate this system from similar but distinct systems

### Stage 2: WHY — Causal Architecture

Deriving architectural decisions from the essence:
- **Existence**: Why does each property exist? (Causal necessity)
- **Technology**: Why this stack? (Derived from property requirements)
- **Structure**: Why this architecture? (Derived from relational logic)

### Stage 3: HOW — Implementation from Properties

```
Properties (Static) → Methods (Dynamic) → Routes (Dispatch)
       ↓                    ↓                  ↓
   Module Design      Method Orchestration   Scenario Handling
```

---

## Three Usage Scenarios

| Scenario | Direction | Starting Point | End Point | Core Goal |
|---|---|---|---|---|
| **A: Knowledge Exploration** | Forward derivation | Concept/Question | Structured knowledge | Build knowledge system from scratch |
| **B: New Project Development** | Forward design | Problem/Requirement | Runnable code | Build system from scratch |
| **C: Existing Project Analysis** | Reverse analysis | Existing code | Understanding docs | Understand existing system |

---

## Design Principles

### Principle 1: Single Source of Truth

> State is the single source of information; all nodes read and write the same state object.

- **Project-level**: Global state is the sole data source for cross-module collaboration
- **Module-level**: Each module has exactly one internal state source, exposed through a unified interface
- **Cross-level access prohibited**: External modules can only access module state through interface contracts

### Principle 2: Four-Layer Infrastructure

> Any project, any module, must first define four layers of infrastructure.

| Layer | Core Question | Typical Implementation |
|---|---|---|
| **Data Rules** | What types? What constraints? | Pydantic, Dataclass, TypeScript Interface |
| **Data Storage** | Where are values stored? | Database, Cache, File, In-memory |
| **Data Flow** | How does data transition between states? | Validation, Transformation, Transaction |
| **Interface Layer** | How do modules exchange data? | API, Function params, Events, Message Queue |

### Principle 3: Code Discipline

| Discipline | Core Requirement |
|---|---|
| **Think Before Write** | Don't assume, expose trade-offs |
| **Simplicity First** | Minimum code to solve the problem |
| **Precise Modification** | Only touch what must be touched |
| **Goal-Driven** | Define success criteria, iterate until verified |

---

## Quick Start

1. **Choose your scenario**: Knowledge exploration (A), New project (B), or Existing project analysis (C)
2. **Follow the Triad Methodology**: WHAT → WHY → HOW (or reverse for scenario C)
3. **Apply Gradient Understanding** at each stage: Concept → Connections → Details
4. **Use the four-layer infrastructure** for every module design
5. **Recursively deepen** on ✅-marked properties

---

## Project Structure

```
本质探索/
└── SKILL.md          # Complete methodology documentation
```

---

## License

MIT

---

## 中文说明

基于"三阶方法论"（是什么-为什么-怎么做）和"类-属性-方法-路由"模型，融合"坡度理解"渐进式认知方法的系统化软件设计与认知系统。

### 核心思想

> **项目 = 类，模块 = 属性，接口 = 契约，路由 = 方法分发**

软件开发的本质：**属性化的静态结构 + 方法化的动态行为**

### 三阶方法论

- **是什么（WHAT）**：从逻辑定义理论出发，确定本质特征、关键属性和区分属性
- **为什么（WHY）**：从本质推导架构决策——存在理由、技术选型、结构设计
- **怎么做（HOW）**：从属性推导实现——属性（静态）→ 方法（动态）→ 路由（分发）

### 坡度理解

每一阶内部都遵循渐进式认知方法：

1. **点出概念**（30秒抓住核心）：一句话定义 + 本质特征表格
2. **建立联系**（2分钟理解框架）：关键属性表格 + ✅❌标记
3. **详细解释**（按需深入）：展开说明 + 代码/步骤

### 三大使用场景

| 场景 | 方向 | 起点 | 终点 |
|---|---|---|---|
| **A：通用知识探索** | 正向推导 | 概念/问题 | 结构化知识 |
| **B：开发新项目** | 正向设计 | 问题/需求 | 可运行代码 |
| **C：解析现有项目** | 逆向分析 | 现有代码 | 理解文档 |

### 设计原则

1. **单一状态源**：项目级全局状态 + 模块级内部状态，通过接口契约隔离
2. **基础设施四层**：数据规矩 → 数据存储 → 数据流转 → 接口层
3. **编码纪律**：先思后写 + 简洁优先 + 精准修改 + 目标驱动
