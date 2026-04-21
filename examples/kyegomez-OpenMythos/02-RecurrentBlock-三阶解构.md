# RecurrentBlock 循环块 - 三阶解构

> 父文档：[[00-OpenMythos-三阶解构]]
> 模块类型：核心创新模块 ⭐

---

## 一、是什么 (What) - 模块本质

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**执行T次循环迭代，每次更新隐状态**
- 因此该模块的本质是：**循环推理引擎 (Recurrent Inference Engine)**

这是OpenMythos的**核心创新**，是整个架构的灵魂。

| 推导要素 | 逻辑分析 | 结论 |
|---------|---------|------|
| **核心问题** | 如何在固定参数下实现可变深度推理？ | 通过循环复用同一组层 |
| **模块定位** | 三阶段架构的核心 | 连接Prelude和Coda |
| **核心价值** | 参数复用 + 深度可变 | 用循环代替堆叠 |

**本质特征**：
1. **权重复用** - 同一组Transformer块重复执行T次
2. **状态传递** - 每次循环的隐状态传递给下一次
3. **输入注入** - 每次循环注入原始输入，防止漂移
4. **深度可变** - T可在运行时调整，参数量不变
5. **稳定性约束** - 谱半径ρ(A) < 1保证不发散

### 1.2 关键属性

| 属性名 | 推导来源 | 说明 | 标记 |
|-------|---------|------|------|
| **injection** | 输入注入需要 | InputInjection实例 | ❌ 见父文档 |
| **blocks** | 权重复用需要 | Transformer块列表 | ❌ 基础组件 |
| **forward** | 状态传递需要 | 循环前向传播 | ✅ 核心方法 |
| **get_A** | 稳定性约束需要 | 获取注入矩阵A | ❌ 简单方法 |

### 1.3 区分属性

| 对比项 | RecurrentBlock | RNN | Transformer |
|-------|---------------|-----|-------------|
| **时间维度** | 循环深度（隐式推理） | 序列时间步 | 无 |
| **权重共享** | 跨循环共享 | 跨时间步共享 | 无共享 |
| **状态更新** | h_{t+1} = f(h_t, e) | h_t = f(h_{t-1}, x_t) | 无状态 |
| **输入处理** | 每次循环注入原始输入e | 每步新输入x_t | 一次性处理 |

---

## 二、为什么 (Why) - 设计理由

### 2.1 存在理由

```
因（问题）：
    1. 传统Transformer每层参数独立，无法复用
    2. 不同任务需要不同推理深度
    3. 单纯循环会导致信息漂移（忘记原始输入）
    4. 循环需要稳定性保证

果（解决方案）：
    1. 使用ModuleList存储可复用的Transformer块
    2. n_loops作为参数传入，动态调整
    3. InputInjection每次循环注入原始输入e
    4. 学习A矩阵，约束谱半径ρ(A) < 1
```

### 2.2 核心公式

**循环更新公式**：
```
h_{t+1} = A·h_t + B·e + Transformer(h_t, e)

其中：
- h_t: 第t次循环的隐状态
- e: 来自Prelude的编码输入
- A, B: 学习的注入参数矩阵
- Transformer: 注意力 + MoE前馈
```

**稳定性约束**：
```
谱半径ρ(A) < 1

确保当t→∞时，A^t → 0，系统不发散
```

---

## 三、怎么做 (How) - 实现细节

### 3.1 模块结构

```python
class RecurrentBlock(nn.Module):
    """循环块 - OpenMythos核心创新"""
    
    # 属性定义
    config: MythosConfig
    injection: InputInjection
    blocks: nn.ModuleList
    
    # 方法定义
    def __init__(self, config: MythosConfig) -> None
    def forward(self, x: Tensor, n_loops: int = None) -> Tensor
    def get_A(self) -> Tensor
```

### 3.2 核心方法

#### `__init__` 方法

```python
def __init__(self, config: MythosConfig):
    """初始化RecurrentBlock
    
    初始化流程：
    1. 保存配置引用
    2. 初始化InputInjection层
    3. 初始化recurrent_layers个Transformer块
    """
    super().__init__()
    self.config = config
    
    # 输入注入层（关键创新）
    self.injection = InputInjection(config)
    
    # Transformer块列表（权重复用的核心）
    self.blocks = nn.ModuleList([
        TransformerBlock(config)
        for _ in range(config.recurrent_layers)
    ])
```

#### `forward` 方法

```python
def forward(self, x: Tensor, n_loops: int = None) -> Tensor:
    """循环前向传播
    
    核心逻辑：
    h_0 = x
    for t in 1 to T:
        h_t = injection(h_{t-1}, x, t)  # 注入原始输入
        for block in blocks:
            h_t = block(h_t)            # Transformer处理
    
    Args:
        x: 来自Prelude的编码输入e [batch, seq_len, dim]
        n_loops: 循环次数T
    
    Returns:
        h_T: 经过T次循环后的隐状态
    """
    n_loops = n_loops or self.config.max_loop_iters
    
    # 初始化隐状态
    h = x
    
    # 循环迭代（核心逻辑）
    for t in range(n_loops):
        # 1. 输入注入：h = A·h + B·x
        h = self.injection(h, x, t)
        
        # 2. Transformer块处理（复用！）
        for block in self.blocks:
            h = block(h)
    
    return h
```

### 3.3 InputInjection 详细设计

```python
class InputInjection(nn.Module):
    """输入注入层"""
    
    def __init__(self, config: MythosConfig):
        super().__init__()
        # 可学习的注入参数
        self.A = nn.Parameter(torch.randn(config.dim, config.dim) * 0.01)
        self.B = nn.Parameter(torch.randn(config.dim, config.dim) * 0.01)
    
    def forward(self, h: Tensor, e: Tensor, loop_idx: int) -> Tensor:
        """输入注入前向传播
        
        核心公式：h' = A·h + B·e
        """
        return torch.einsum("bsd,dD->bsD", h, self.A) + \
               torch.einsum("bsd,dD->bsD", e, self.B)
    
    def get_A(self) -> Tensor:
        """获取A矩阵（用于检查谱半径）"""
        return self.A
    
    def spectral_radius(self) -> float:
        """计算谱半径ρ(A)"""
        eigenvalues = torch.linalg.eigvals(self.A)
        return torch.max(torch.abs(eigenvalues)).item()
```

### 3.4 数据流图

```
输入 x: [batch, seq_len, dim] (来自Prelude的编码输入e)
    │
初始化: h = x
    │
For t = 0 to n_loops-1:
│
│   ┌─────────────────────────────────────────┐
│   │ [InputInjection]                        │
│   │   h = A·h + B·e  (注入原始输入)          │
│   └─────────────────────────────────────────┘
│       ↓
│   ┌─────────────────────────────────────────┐
│   │ [Transformer Block 1]                   │
│   │   h = block_1(h)                        │
│   │   ├── [Attention] (MLA/GQA)             │
│   │   └── [MoE/FFN]                         │
│   └─────────────────────────────────────────┘
│       ↓
│   ┌─────────────────────────────────────────┐
│   │ [Transformer Block 2]                   │
│   │   h = block_2(h)                        │
│   └─────────────────────────────────────────┘
│       ↓
│      ...
│       ↓
│   ┌─────────────────────────────────────────┐
│   │ [Transformer Block N]                   │
│   │   h = block_N(h)                        │
│   └─────────────────────────────────────────┘
│
# 注意：这些blocks在每次循环中复用！

Return: h [batch, seq_len, dim]
```

---

## 四、关键设计决策

| 决策 | 选择 | 理由 |
|-----|------|------|
| **A/B初始化** | 小随机值（0.01） | 确保初始稳定性 |
| **h初始化** | h = x | 从输入开始，而非从零开始 |
| **blocks数量** | config.recurrent_layers | 通常2-4层，平衡能力和效率 |
| **谱半径检查** | 提供get_A接口 | 让用户可以监控稳定性 |

---

## 参考链接

- [[00-OpenMythos-三阶解构]] - 项目级分析
- [[01-OpenMythos-三阶解构]] - 主模型深入
- [[99-架构图谱]] - 架构汇总