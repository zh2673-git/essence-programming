# MoE 混合专家 - 三阶解构

> 父文档：[[00-OpenMythos-三阶解构]]
> 模块类型：前馈网络（FFN替代）

---

## 一、是什么 (What) - 模块本质

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**通过专家路由实现稀疏激活，提高参数效率**
- 因此该模块的本质是：**条件计算路由器 (Conditional Computation Router)**

MoE（Mixture of Experts）是现代大模型的关键组件。

| 推导要素 | 逻辑分析 | 结论 |
|---------|---------|------|
| **核心问题** | 如何在不增加计算量的情况下扩展模型容量？ | 每次只激活部分参数 |
| **解决思路** | 多个专家网络，路由选择Top-K | 类似集成学习 |
| **模块定位** | FFN层的替代实现 | 替代标准前馈网络 |
| **核心价值** | 参数高效扩展 | 总参数量大，但激活参数少 |

**本质特征**：
1. **专家分工** - 多个专家网络，每个学习不同模式
2. **稀疏路由** - 每个token只路由到Top-K个专家
3. **门控机制** - 学习的路由网络决定专家选择
4. **共享专家** - 所有token都经过的通用专家
5. **负载均衡** - 确保专家利用率均衡（可选）

### 1.2 关键属性

| 属性名 | 推导来源 | 说明 | 标记 |
|-------|---------|------|------|
| **router** | 稀疏路由需要 | 路由网络 | ❌ 线性层 |
| **experts** | 专家分工需要 | 专家网络列表 | ❌ 模块列表 |
| **shared_experts** | 共享专家需要 | 共享专家列表 | ❌ 模块列表 |
| **forward** | 门控机制需要 | 路由+计算 | ✅ 核心方法 |

### 1.3 区分属性

| 对比项 | MoE | 标准FFN | 集成学习 |
|-------|-----|---------|---------|
| **参数使用** | 稀疏（Top-K） | 密集（全部） | 全部 |
| **专家数量** | 多（8-512） | 1 | 少（2-10） |
| **选择机制** | 可学习路由 | 无 | 固定/投票 |
| **计算效率** | 高（稀疏） | 中 | 低（全部） |

---

## 二、为什么 (Why) - 设计理由

### 2.1 存在理由

```
因（问题）：
    1. 模型容量越大能力越强，但计算成本也越高
    2. 标准FFN所有参数都激活，计算密集
    3. 不同token需要不同的处理能力

果（解决方案）：
    1. 使用多个专家网络扩展容量
    2. 每个token只激活Top-K个专家（稀疏）
    3. 学习的路由网络自动分配token到合适的专家
    4. 共享专家处理通用模式
```

### 2.2 核心公式

**标准FFN**：
```
output = FFN(x)  # 所有参数都参与计算
计算量 = O(dim × hidden_dim)
```

**MoE**：
```
# 1. 路由
router_logits = Router(x)  # [batch, seq, n_experts]
weights, selected = TopK(Softmax(router_logits), k)

# 2. 稀疏计算
output = Σ(i in selected) weights[i] × Expert_i(x)
         + Σ(shared_expert) SharedExpert(x)

计算量 = O(k × dim × expert_dim)  # k << n_experts
```

**效率提升**：
```
假设：n_experts=64, n_experts_per_tok=2

总参数量 = 64 × expert_params = 64×
实际计算量 = 2 × expert_params = 2×
计算效率提升 = 64 / 2 = 32×
```

---

## 三、怎么做 (How) - 实现细节

### 3.1 模块结构

```python
class MoE(nn.Module):
    """Mixture of Experts"""
    
    # 属性定义
    config: MythosConfig
    router: nn.Linear
    n_experts: int
    n_experts_per_tok: int
    experts: nn.ModuleList
    n_shared_experts: int
    shared_experts: nn.ModuleList
    
    # 方法定义
    def __init__(self, config: MythosConfig) -> None
    def forward(self, x: Tensor) -> Tensor
    def compute_aux_loss(self, router_probs: Tensor, expert_indices: Tensor) -> Tensor
```

### 3.2 核心方法

#### `__init__` 方法

```python
def __init__(self, config: MythosConfig):
    """初始化MoE层
    
    初始化流程：
    1. 初始化路由网络
    2. 初始化n_experts个专家网络
    3. 初始化n_shared_experts个共享专家
    """
    super().__init__()
    self.config = config
    
    # 路由网络
    self.router = nn.Linear(config.dim, config.n_experts, bias=False)
    
    self.n_experts = config.n_experts
    self.n_experts_per_tok = config.n_experts_per_tok
    
    # 专家网络
    self.experts = nn.ModuleList([
        nn.Sequential(
            nn.Linear(config.dim, config.expert_dim),
            nn.GELU(),
            nn.Linear(config.expert_dim, config.dim)
        )
        for _ in range(config.n_experts)
    ])
    
    self.n_shared_experts = config.n_shared_experts
    
    # 共享专家
    self.shared_experts = nn.ModuleList([
        nn.Sequential(
            nn.Linear(config.dim, config.expert_dim),
            nn.GELU(),
            nn.Linear(config.expert_dim, config.dim)
        )
        for _ in range(config.n_shared_experts)
    ])
```

#### `forward` 方法

```python
def forward(self, x: Tensor) -> Tensor:
    """MoE前向传播
    
    计算流程：
    1. 计算路由分数
    2. 选择Top-K专家
    3. 并行计算被选中的专家（稀疏）
    4. 计算共享专家（密集）
    5. 加权聚合输出
    """
    batch_size, seq_len, dim = x.shape
    
    # ========== 1. 路由计算 ==========
    router_logits = self.router(x)  # [batch, seq_len, n_experts]
    router_probs = F.softmax(router_logits, dim=-1)
    
    # Top-K选择
    weights, selected_experts = torch.topk(
        router_probs,
        k=self.n_experts_per_tok,
        dim=-1,
        sorted=False
    )
    
    # 归一化权重
    weights = weights / weights.sum(dim=-1, keepdim=True)
    
    # ========== 2. 专家计算（稀疏） ==========
    expert_output = torch.zeros_like(x)
    
    for expert_idx, expert in enumerate(self.experts):
        # 找到选择这个专家的token
        mask = (selected_experts == expert_idx).any(dim=-1)
        
        if mask.any():
            expert_input = x[mask]  # [n_selected, dim]
            expert_out = expert(expert_input)  # [n_selected, dim]
            
            # 获取对应的权重
            expert_mask = (selected_experts[mask] == expert_idx)
            expert_weights = weights[mask][expert_mask]
            
            # 加权累加
            expert_output[mask] += expert_weights.unsqueeze(-1) * expert_out
    
    # ========== 3. 共享专家计算（密集） ==========
    shared_output = torch.zeros_like(x)
    
    for shared_expert in self.shared_experts:
        shared_output += shared_expert(x)
    
    # ========== 4. 合并输出 ==========
    output = expert_output + shared_output
    
    return output
```

#### `compute_aux_loss` 方法

```python
def compute_aux_loss(self, router_probs: Tensor, expert_indices: Tensor) -> Tensor:
    """计算负载均衡辅助损失
    
    目的：确保每个专家被均匀使用，防止某些专家过载
    
    公式：aux_loss = α × Σ(f_i × P_i)
    
    其中：
    - f_i: 专家i的token比例（实际使用频率）
    - P_i: 专家i的平均路由概率（期望使用频率）
    """
    # 计算每个专家的实际token数
    expert_mask = F.one_hot(expert_indices, num_classes=self.n_experts).sum(dim=-2)
    
    # 每个专家处理的token比例
    f = expert_mask.float().mean(dim=0)
    
    # 每个专家的平均路由概率
    P = router_probs.mean(dim=[0, 1])
    
    # 负载均衡损失
    aux_loss = torch.sum(f * P) * self.n_experts
    
    return aux_loss
```

### 3.3 数据流图

```
输入 x: [batch, seq_len, dim]
    │
    ├──→ router ──→ [batch, seq_len, n_experts]
    │       │
    │       └──→ softmax ──→ router_probs: [batch, seq_len, n_experts]
    │               │
    │               └──→ topk(k=n_experts_per_tok)
    │                       │
    │                       ├──→ weights: [batch, seq_len, k]
    │                       └──→ selected_experts: [batch, seq_len, k]
    │
    ├──→ 稀疏专家计算
    │       │
    │       For expert_i in experts:
    │       │
    │       ├──→ mask = (selected_experts == i).any(dim=-1)
    │       │       │
    │       │       └──→ [batch, seq_len] - 布尔掩码
    │       │
    │       ├──→ if mask.any():
    │       │       │
    │       │       ├──→ expert_input = x[mask]  # [n_selected, dim]
    │       │       │
    │       │       ├──→ expert_out = expert_i(expert_input)
    │       │       │       │
    │       │       │       └──→ [n_selected, dim]
    │       │       │
    │       │       ├──→ expert_weights = weights[mask][selected_experts[mask] == i]
    │       │       │       │
    │       │       │       └──→ [n_selected]
    │       │       │
    │       │       └──→ output[mask] += expert_weights.unsqueeze(-1) * expert_out
    │       │
    │       └──→ End if
    │       
    │       End For
    │
    ├──→ expert_output: [batch, seq_len, dim]
    │
    ├──→ 共享专家计算
    │       │
    │       For shared_expert in shared_experts:
    │       │
    │       ├──→ shared_out = shared_expert(x)  # [batch, seq_len, dim]
    │       │
    │       └──→ shared_output += shared_out
    │       
    │       End For
    │
    ├──→ shared_output: [batch, seq_len, dim]
    │
    └──→ output = expert_output + shared_output
            │
            └──→ [batch, seq_len, dim]
```

### 3.4 专家选择示例

```
假设：n_experts=8, n_experts_per_tok=2

Token 1 (关于数学):  → 专家3 (数学) + 专家7 (逻辑)
Token 2 (关于代码):  → 专家1 (代码) + 专家5 (语法)
Token 3 (关于诗歌):  → 专家2 (文学) + 专家4 (创意)
Token 4 (关于历史):  → 专家6 (历史) + 专家0 (事实)

共享专家: 所有token都经过专家8 (通用知识)

最终输出 = 选中专家输出 × 权重 + 共享专家输出
```

---

## 四、关键设计决策

| 决策 | 选择 | 理由 |
|-----|------|------|
| **Top-K选择** | K=2 | 平衡效率和质量 |
| **权重归一化** | 是 | Top-K权重和为1 |
| **共享专家** | 1-2个 | 处理通用模式 |
| **专家结构** | 标准FFN | 简单高效 |
| **激活函数** | GELU/SwiGLU | 现代LLM标准 |
| **负载均衡** | 可选aux_loss | 训练稳定性 |

---

## 参考链接

- [[00-OpenMythos-三阶解构]] - 项目级分析
- [[99-架构图谱]] - 架构汇总