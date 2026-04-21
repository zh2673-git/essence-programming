# OpenMythos 主模型 - 三阶解构

> 父文档：[[00-OpenMythos-三阶解构]]
> 模块类型：主模型/编排器

---

## 一、是什么 (What) - 模块本质

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**整体模型封装，组装三阶段架构**
- 因此该模块的本质是：**模型编排器 (Model Orchestrator)**

| 推导要素 | 逻辑分析 | 结论 |
|---------|---------|------|
| **核心问题** | 如何将Prelude、RecurrentBlock、Coda组装成完整模型？ | 需要一个统一的入口和协调者 |
| **模块定位** | 用户直接交互的接口 | 提供`forward`和`generate`方法 |
| **核心价值** | 隐藏内部复杂性，暴露简洁API | 用户只需关心input/output |

**本质特征**：
1. **组装器** - 将各个子模块组装成完整流水线
2. **协调者** - 控制数据流在各模块间的传递
3. **接口封装** - 对外提供统一的前向传播和生成接口
4. **配置管理** - 根据MythosConfig初始化所有子模块

### 1.2 关键属性

| 属性名 | 推导来源 | 说明 | 标记 |
|-------|---------|------|------|
| **embedding** | 组装器需要 | 词嵌入层 | ❌ 基础层 |
| **prelude** | 组装器需要 | 前奏编码层 | ❌ 见父文档 |
| **recurrent** | 组装器需要 | 循环推理块 | ❌ 见父文档 |
| **coda** | 组装器需要 | 尾声处理层 | ❌ 见父文档 |
| **lm_head** | 组装器需要 | 输出投影层 | ❌ 基础层 |
| **forward** | 协调者需要 | 前向传播方法 | ✅ 核心方法 |
| **generate** | 接口封装需要 | 自回归生成方法 | ✅ 核心方法 |

### 1.3 区分属性

| 对比项 | OpenMythos主类 | nn.Sequential | HuggingFace Model |
|-------|---------------|-------------|-------------------|
| **组装方式** | 显式组装，自定义数据流 | 线性堆叠 | 配置驱动 |
| **灵活性** | 高（可自定义循环次数） | 低 | 中 |
| **生成支持** | 内置generate方法 | 无 | 有 |
| **配置管理** | MythosConfig | 无 | PretrainedConfig |

---

## 二、为什么 (Why) - 设计理由

### 2.1 存在理由

```
因（问题）：
    1. 用户需要简单的API来训练和推理
    2. 三阶段架构需要协调数据流
    3. 循环次数需要在运行时动态调整
    4. 生成过程需要自回归解码

果（解决方案）：
    1. 提供forward和generate统一接口
    2. 在forward中显式编排数据流
    3. n_loops作为forward参数传入
    4. 内置自回归生成逻辑
```

### 2.2 方法设计理由

| 方法 | 存在理由 | 推导逻辑 |
|-----|---------|---------|
| `__init__` | 组装所有子模块 | 从配置推导需要哪些模块 |
| `forward` | 训练和推理的前向传播 | 需要编排三阶段数据流 |
| `generate` | 自回归文本生成 | 大语言模型的核心功能 |

---

## 三、怎么做 (How) - 实现细节

### 3.1 模块结构

```python
class OpenMythos(nn.Module):
    """OpenMythos主模型类"""
    
    # 属性定义
    config: MythosConfig
    embedding: nn.Embedding
    prelude: Prelude
    recurrent: RecurrentBlock
    coda: Coda
    lm_head: nn.Linear
    
    # 方法定义
    def __init__(self, config: MythosConfig) -> None
    def forward(self, input_ids: Tensor, n_loops: int = None) -> Tensor
    def generate(self, input_ids: Tensor, max_new_tokens: int, n_loops: int = 8) -> Tensor
```

### 3.2 核心方法

#### `__init__` 方法

```python
def __init__(self, config: MythosConfig):
    """初始化OpenMythos模型
    
    初始化流程：
    1. 保存配置引用
    2. 初始化词嵌入层
    3. 初始化Prelude（前奏层）
    4. 初始化RecurrentBlock（循环块）⭐
    5. 初始化Coda（尾声层）
    6. 初始化LM Head（输出投影）
    """
    super().__init__()
    self.config = config
    
    self.embedding = nn.Embedding(config.vocab_size, config.dim)
    self.prelude = Prelude(config)
    self.recurrent = RecurrentBlock(config)  # 核心
    self.coda = Coda(config)
    self.lm_head = nn.Linear(config.dim, config.vocab_size, bias=False)
```

#### `forward` 方法

```python
def forward(self, input_ids: Tensor, n_loops: int = None) -> Tensor:
    """前向传播
    
    数据流编排：
    input_ids → embedding → prelude → recurrent → coda → lm_head → logits
    
    Args:
        input_ids: [batch, seq_len]
        n_loops: 循环次数（动态调整）
    
    Returns:
        logits: [batch, seq_len, vocab_size]
    """
    # 1. 词嵌入
    x = self.embedding(input_ids)
    
    # 2. Prelude编码
    x = self.prelude(x)
    
    # 3. 循环推理（核心）
    x = self.recurrent(x, n_loops=n_loops)
    
    # 4. Coda处理
    x = self.coda(x)
    
    # 5. 输出投影
    logits = self.lm_head(x)
    
    return logits
```

#### `generate` 方法

```python
def generate(self, input_ids: Tensor, max_new_tokens: int, 
             n_loops: int = 8, temperature: float = 1.0,
             top_k: int = 50, top_p: float = 0.95) -> Tensor:
    """自回归生成
    
    生成流程：
    1. 循环max_new_tokens次
    2. 每次调用forward获取logits
    3. 应用采样策略
    4. 拼接新token
    """
    self.eval()
    
    for _ in range(max_new_tokens):
        logits = self.forward(input_ids, n_loops=n_loops)
        next_token_logits = logits[:, -1, :] / temperature
        
        # Top-K过滤
        if top_k > 0:
            indices = next_token_logits < torch.topk(next_token_logits, top_k)[0][..., -1, None]
            next_token_logits[indices] = float("-inf")
        
        # Top-P过滤
        if top_p < 1.0:
            sorted_logits, sorted_indices = torch.sort(next_token_logits, descending=True)
            cum_probs = torch.cumsum(F.softmax(sorted_logits, dim=-1), dim=-1)
            sorted_indices_to_remove = cum_probs > top_p
            sorted_indices_to_remove[..., 1:] = sorted_indices_to_remove[..., :-1].clone()
            sorted_indices_to_remove[..., 0] = 0
            indices_to_remove = sorted_indices_to_remove.scatter(1, sorted_indices, sorted_indices_to_remove)
            next_token_logits[indices_to_remove] = float("-inf")
        
        # 采样
        probs = F.softmax(next_token_logits, dim=-1)
        next_token = torch.multinomial(probs, num_samples=1)
        input_ids = torch.cat([input_ids, next_token], dim=-1)
    
    return input_ids
```

### 3.3 数据流图

```
┌─────────────────────────────────────────────────────────────────┐
│                        OpenMythos Forward                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Input: input_ids [batch, seq_len]                               │
│      ↓                                                           │
│  [Embedding] → x [batch, seq_len, dim]                          │
│      ↓                                                           │
│  [Prelude] → x [batch, seq_len, dim]                            │
│      ↓                                                           │
│  [RecurrentBlock] → x [batch, seq_len, dim] ⭐                   │
│      ↓                                                           │
│  [Coda] → x [batch, seq_len, dim]                               │
│      ↓                                                           │
│  [LM Head] → logits [batch, seq_len, vocab_size]                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 接口契约

```python
class OpenMythosInterface(Protocol):
    def __init__(self, config: MythosConfig) -> None: ...
    def forward(self, input_ids: Tensor, n_loops: Optional[int] = None) -> Tensor: ...
    def generate(self, input_ids: Tensor, max_new_tokens: int, 
                 n_loops: int = 8, **sampling_kwargs) -> Tensor: ...
```

---

## 四、关键设计决策

| 决策 | 选择 | 理由 |
|-----|------|------|
| **n_loops参数位置** | forward参数 | 支持动态调整，训练和推理可用不同值 |
| **权重共享** | 可选 | embedding和lm_head共享可减少参数 |
| **生成策略** | 内置 | 提供完整的推理能力，无需外部实现 |
| **采样方法** | temperature + top_k + top_p | 覆盖主流采样策略，灵活可控 |

---

## 参考链接

- [[00-OpenMythos-三阶解构]] - 项目级分析
- [[02-RecurrentBlock-三阶解构]] - 循环块深入
- [[03-MLAAttention-三阶解构]] - MLA注意力深入
- [[04-MoE-三阶解构]] - MoE深入
- [[99-架构图谱]] - 架构汇总