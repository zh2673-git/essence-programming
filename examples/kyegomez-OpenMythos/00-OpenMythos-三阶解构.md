# OpenMythos - 三阶解构

> GitHub: https://github.com/kyegomez/OpenMythos
> 分析日期：2026-04-21

## 元数据
- **项目名称**：OpenMythos
- **所属组织**：kyegomez
- **GitHub地址**：https://github.com/kyegomez/OpenMythos
- **主要语言**：Python
- **核心定位**：循环深度Transformer (RDT) 的理论实现
- **分析日期**：2026-04-21

---

## 一、是什么 (What) - 定义本质

### 1.1 本质特征（内涵）

从项目核心问题出发，推导本质：

| 推导要素 | 逻辑分析 | 结论 |
|---------|---------|------|
| **核心问题** | 传统Transformer需要堆叠大量层数，参数量巨大，推理成本高 | 如何用更少的参数实现更深的推理能力？ |
| **目标用户** | AI研究者、模型开发者、对高效推理架构感兴趣的人 | 需要探索新型架构的研究社区 |
| **核心价值** | 通过循环深度机制，在固定参数量下实现可变深度推理 | 参数高效 + 推理深度自适应 |
| **一句话定义** | "一个循环深度Transformer，用循环代替堆叠，让模型思考更深而不增加参数" | Recurrent-Depth Transformer (RDT) |

**本质特征推导**：

1. **循环深度机制** - 核心创新：用循环迭代代替层堆叠，同一组层重复执行多次
2. **参数固定，深度可变** - 推理时可以动态调整循环次数，参数量不变
3. **隐状态注入** - 每次循环注入原始输入信号，防止信息漂移
4. **三阶段架构** - Prelude(前奏) → Recurrent Block(循环块) → Coda(尾声)
5. **注意力可切换** - 支持MLA(Multi-head Latent Attention)和GQA(Grouped Query Attention)

### 1.2 关键属性（外延的具体表现）

从本质特征**逻辑推导**出系统必须具备的属性：

```
本质特征 "循环深度机制"
    → 需要循环块管理迭代
    → 属性：RecurrentBlock模块

本质特征 "参数固定，深度可变"
    → 需要配置循环次数
    → 属性：MythosConfig配置模块

本质特征 "隐状态注入"
    → 需要注入层处理状态
    → 属性：InputInjection模块

本质特征 "三阶段架构"
    → 需要Prelude层、Coda层
    → 属性：Prelude模块、Coda模块

本质特征 "注意力可切换"
    → 需要MLA和GQA实现
    → 属性：MLAAttention模块、GQAAttention模块

本质特征 "MoE前馈网络"
    → 需要专家路由机制
    → 属性：MoE模块、Router模块
```

**关键属性表**：

| 属性名 | 推导来源 | 说明 | 标记 |
|-------|---------|------|------|
| **MythosConfig** | 深度可变需要配置 | 模型配置中心 | ❌ 见下方 |
| **OpenMythos** | 整体模型封装 | 主模型类 | ✅ 需深入 → [[01-OpenMythos-三阶解构]] |
| **RecurrentBlock** | 循环深度核心 | 循环迭代块 | ✅ 需深入 → [[02-RecurrentBlock-三阶解构]] |
| **InputInjection** | 隐状态注入 | 状态注入层 | ❌ 见下方 |
| **Prelude** | 三阶段架构 | 前奏编码层 | ❌ 见下方 |
| **Coda** | 三阶段架构 | 尾声输出层 | ❌ 见下方 |
| **MLAAttention** | 注意力可切换 | 多头潜在注意力 | ✅ 需深入 → [[03-MLAAttention-三阶解构]] |
| **GQAAttention** | 注意力可切换 | 分组查询注意力 | ❌ 见下方 |
| **MoE** | MoE前馈 | 混合专家网络 | ✅ 需深入 → [[04-MoE-三阶解构]] |
| **MythosTokenizer** | 文本处理 | 分词器 | ❌ 见下方 |

### 1.3 区分属性（边界定义）

从本质特征推导**不是什么**：

| 对比项 | OpenMythos | 传统Transformer | Chain-of-Thought |
|-------|------------|-----------------|------------------|
| **核心机制** | 循环深度（隐式） | 层堆叠（显式） | 显式生成中间token |
| **推理过程** | 在latent空间循环 | 单向传播 | 生成文本序列 |
| **参数效率** | 高（共享权重） | 低（每层独立） | 中等 |
| **可解释性** | 低（黑盒循环） | 中等 | 高（可阅读） |
| **动态深度** | ✅ 支持 | ❌ 不支持 | ✅ 支持 |

**边界结论**：
- ✅ 做：循环深度推理、参数高效、注意力切换、MoE前馈
- ❌ 不做：显式思维链、层间独立参数、传统堆叠架构

### 1.4 设计哲学（从本质推导）

```
本质1：循环深度机制
    → 设计哲学：深度优于广度，复用优于堆叠
    
本质2：隐状态注入
    → 设计哲学：保持信号清晰，防止信息漂移
    
本质3：三阶段架构
    → 设计哲学：明确分工，预处理→循环推理→后处理
    
本质4：注意力可切换
    → 设计哲学：灵活适配，不同场景不同策略
    
本质5：MoE前馈
    → 设计哲学：稀疏激活，专家分工
```

**核心设计哲学**：**循环复用，深度思考，稀疏激活，灵活适配**

---

## 二、为什么 (Why) - 确定架构

### 2.1 存在理由（因果逻辑）

```
因（问题）：
    1. 大模型参数量爆炸，训练和推理成本极高
    2. 传统堆叠架构每层参数独立，无法复用
    3. 不同任务需要不同推理深度，固定层数浪费资源
    4. 单纯循环会导致信息漂移，忘记原始输入

果（解决方案）：
    1. 循环深度架构：同一组层重复执行，参数复用
    2. 可配置循环次数：根据任务难度动态调整
    3. 输入注入机制：每次循环注入原始信号，保持上下文
    4. MoE稀疏激活：每次只激活部分专家，提高效率
```

### 2.2 技术栈决策（从本质推导）

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **深度学习框架** | PyTorch | 动态图支持好，适合循环架构实现 |
| **注意力机制** | MLA/GQA可选 | MLA降低KV缓存，GQA平衡效率和质量 |
| **前馈网络** | MoE (Mixture of Experts) | 稀疏激活，提高参数效率 |
| **位置编码** | RoPE (Rotary Position Embedding) | 支持长上下文，相对位置编码 |
| **归一化** | RMSNorm | 比LayerNorm更稳定，计算更高效 |
| **精度** | bfloat16/float16 | 大模型训练标准，节省显存 |

### 2.3 参考项目分析

| 项目 | 可借鉴点 | 需避免 |
|-----|---------|-------|
| DeepSeek-V2/V3 | MLA注意力、MoE架构 | 过于复杂的负载均衡 |
| LLaMA | RMSNorm、RoPE、SwiGLU | 传统堆叠架构 |
| GPT | 生成式架构 | 纯堆叠，无循环机制 |
| Claude Mythos (推测) | 循环深度思想 | 闭源，无法直接参考 |

---

## 三、怎么做 (How) - 实现交付

### 3.1 模块划分（从属性推导）

```
OpenMythos/
├── open_mythos/
│   ├── __init__.py           # 模块导出
│   ├── main.py               # OpenMythos主类 ⭐
│   ├── config.py             # MythosConfig配置
│   ├── attention/
│   │   ├── __init__.py
│   │   ├── mla.py            # MLAAttention ⭐
│   │   └── gqa.py            # GQAAttention
│   ├── core/
│   │   ├── __init__.py
│   │   ├── prelude.py        # Prelude前奏层
│   │   ├── recurrent.py      # RecurrentBlock循环块 ⭐
│   │   ├── coda.py           # Coda尾声层
│   │   └── injection.py      # InputInjection注入层
│   ├── moe/
│   │   ├── __init__.py
│   │   └── moe.py            # MoE混合专家 ⭐
│   └── tokenizer.py          # MythosTokenizer
├── training/
│   └── 3b_fine_web_edu.py    # 训练脚本
└── docs/                     # 文档
```

### 3.2 核心模块详细设计

#### 3.2.1 MythosConfig（配置中心）

```python
@dataclass
class MythosConfig:
    """Mythos配置类
    
    推导来源：深度可变需要配置、注意力可切换需要配置
    
    职责：集中管理所有模型超参数
    """
    # 基础维度
    vocab_size: int
    dim: int                    # 模型维度
    n_heads: int                # 注意力头数
    
    # 架构配置
    max_seq_len: int
    max_loop_iters: int         # 最大循环次数 ⭐
    prelude_layers: int         # 前奏层数
    coda_layers: int            # 尾声层数
    
    # MoE配置
    n_experts: int              # 专家总数
    n_shared_experts: int       # 共享专家数
    n_experts_per_tok: int      # 每token激活专家数
    expert_dim: int             # 专家维度
    
    # 注意力配置
    attn_type: str              # "mla" 或 "gqa"
    n_kv_heads: int             # GQA: KV头数
    kv_lora_rank: int           # MLA: KV低秩
    q_lora_rank: int            # MLA: Q低秩
    qk_rope_head_dim: int       # MLA: RoPE维度
    qk_nope_head_dim: int       # MLA: 非RoPE维度
    v_head_dim: int             # MLA: V头维度
    
    # LoRA配置
    lora_rank: int              # 低秩适配秩
```

#### 3.2.2 OpenMythos（主模型）

```python
class OpenMythos(nn.Module):
    """OpenMythos主模型
    
    推导来源：整体模型封装
    
    职责：组装三阶段架构，提供前向传播和生成接口
    
    利用的属性：
    - MythosConfig: 获取配置参数
    - Prelude: 输入编码
    - RecurrentBlock: 循环推理
    - Coda: 输出生成
    - MythosTokenizer: 文本编解码
    """
    
    def __init__(self, config: MythosConfig):
        self.config = config
        self.embedding = nn.Embedding(config.vocab_size, config.dim)
        self.prelude = Prelude(config)
        self.recurrent = RecurrentBlock(config)  # ⭐ 核心
        self.coda = Coda(config)
        self.lm_head = nn.Linear(config.dim, config.vocab_size)
    
    def forward(self, input_ids: Tensor, n_loops: int = None) -> Tensor:
        """前向传播
        
        利用的属性：
        - embedding: 词嵌入
        - prelude: 前奏编码
        - recurrent: 循环推理（核心）
        - coda: 尾声处理
        - lm_head: 输出投影
        
        逻辑流程：
        1. 词嵌入
        2. Prelude编码
        3. RecurrentBlock循环推理（n_loops次）
        4. Coda处理
        5. 输出logits
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
    
    def generate(self, input_ids: Tensor, max_new_tokens: int, n_loops: int = 8) -> Tensor:
        """自回归生成
        
        利用的属性：
        - forward: 前向传播
        - 自回归解码逻辑
        
        Args:
            input_ids: 输入token IDs
            max_new_tokens: 最大生成token数
            n_loops: 每次前向传播的循环次数
        """
        # 自回归生成逻辑
        for _ in range(max_new_tokens):
            logits = self.forward(input_ids, n_loops=n_loops)
            next_token = sample(logits[:, -1, :])
            input_ids = torch.cat([input_ids, next_token], dim=-1)
        return input_ids
```

#### 3.2.3 RecurrentBlock（循环块 - 核心⭐）

```python
class RecurrentBlock(nn.Module):
    """循环块 - OpenMythos核心创新
    
    推导来源：循环深度机制
    
    职责：执行T次循环迭代，每次更新隐状态
    
    利用的属性：
    - InputInjection: 状态注入
    - TransformerBlock: 注意力+FFN层
    - MythosConfig: 获取循环次数配置
    
    核心公式：
    h_{t+1} = A·h_t + B·e + Transformer(h_t, e)
    
    其中：
    - h_t: 第t次循环的隐状态
    - e: 编码后的输入（来自Prelude）
    - A, B: 学习的注入参数
    """
    
    def __init__(self, config: MythosConfig):
        self.config = config
        self.injection = InputInjection(config)  # 注入层
        self.blocks = nn.ModuleList([
            TransformerBlock(config) 
            for _ in range(config.recurrent_layers)
        ])
    
    def forward(self, x: Tensor, n_loops: int = None) -> Tensor:
        """循环前向传播
        
        利用的属性：
        - injection: 状态注入
        - blocks: Transformer块
        
        Args:
            x: 来自Prelude的编码输入
            n_loops: 循环次数（默认使用config.max_loop_iters）
        """
        n_loops = n_loops or self.config.max_loop_iters
        
        # 初始化隐状态
        h = x
        
        # 循环迭代
        for t in range(n_loops):
            # 注入输入信号（防止漂移）
            h = self.injection(h, x, t)
            
            # Transformer块处理
            for block in self.blocks:
                h = block(h)
        
        return h
```

#### 3.2.4 InputInjection（输入注入层）

```python
class InputInjection(nn.Module):
    """输入注入层
    
    推导来源：隐状态注入
    
    职责：每次循环注入原始输入信号，防止信息漂移
    
    核心公式：
    h' = A·h + B·e
    
    其中A、B是可学习参数，控制历史状态和原始输入的权重
    """
    
    def __init__(self, config: MythosConfig):
        self.A = nn.Parameter(torch.randn(config.dim, config.dim))
        self.B = nn.Parameter(torch.randn(config.dim, config.dim))
    
    def forward(self, h: Tensor, e: Tensor, loop_idx: int) -> Tensor:
        """注入前向传播
        
        Args:
            h: 当前隐状态
            e: 原始编码输入
            loop_idx: 当前循环索引
        """
        # 线性组合历史状态和原始输入
        return torch.matmul(h, self.A) + torch.matmul(e, self.B)
    
    def get_A(self) -> Tensor:
        """获取A矩阵（用于检查谱半径）"""
        return self.A
```

#### 3.2.5 MLAAttention（多头潜在注意力）

```python
class MLAAttention(nn.Module):
    """Multi-head Latent Attention (DeepSeek-V2风格)
    
    推导来源：注意力可切换、降低KV缓存
    
    职责：通过低秩压缩减少KV缓存，提高效率
    
    关键创新：
    - Q/K/V都使用低秩分解
    - 分离RoPE和非RoPE部分
    - 显著减少推理时的KV缓存
    """
    
    def __init__(self, config: MythosConfig):
        self.q_lora_rank = config.q_lora_rank
        self.kv_lora_rank = config.kv_lora_rank
        self.qk_rope_head_dim = config.qk_rope_head_dim
        self.qk_nope_head_dim = config.qk_nope_head_dim
        self.v_head_dim = config.v_head_dim
        
        # Q投影（低秩）
        self.q_down = nn.Linear(config.dim, config.q_lora_rank)
        self.q_up = nn.Linear(config.q_lora_rank, config.n_heads * config.qk_nope_head_dim)
        
        # KV投影（低秩）
        self.kv_down = nn.Linear(config.dim, config.kv_lora_rank)
        self.k_up = nn.Linear(config.kv_lora_rank, config.n_heads * config.qk_nope_head_dim)
        self.v_up = nn.Linear(config.kv_lora_rank, config.n_heads * config.v_head_dim)
        
        # RoPE投影
        self.q_rope = nn.Linear(config.dim, config.n_heads * config.qk_rope_head_dim)
        self.k_rope = nn.Linear(config.dim, config.n_heads * config.qk_rope_head_dim)
    
    def forward(self, x: Tensor, mask: Tensor = None) -> Tensor:
        # 低秩Q投影
        q_nope = self.q_up(self.q_down(x))
        q_rope = self.q_rope(x)
        
        # 低秩KV投影
        kv = self.kv_down(x)
        k_nope = self.k_up(kv)
        v = self.v_up(kv)
        k_rope = self.k_rope(x)
        
        # 合并RoPE和非RoPE部分
        q = torch.cat([q_nope, q_rope], dim=-1)
        k = torch.cat([k_nope, k_rope], dim=-1)
        
        # 标准注意力计算
        scores = torch.matmul(q, k.transpose(-2, -1)) / sqrt(dim)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float("-inf"))
        attn = F.softmax(scores, dim=-1)
        out = torch.matmul(attn, v)
        
        return out
```

#### 3.2.6 MoE（混合专家网络）

```python
class MoE(nn.Module):
    """Mixture of Experts
    
    推导来源：MoE前馈、稀疏激活
    
    职责：通过专家路由实现稀疏激活，提高参数效率
    
    关键组件：
    - Router: 决定每个token路由到哪些专家
    - Experts: 多个前馈专家网络
    - SharedExperts: 所有token都会经过的共享专家
    """
    
    def __init__(self, config: MythosConfig):
        self.n_experts = config.n_experts
        self.n_experts_per_tok = config.n_experts_per_tok
        self.n_shared_experts = config.n_shared_experts
        
        # 路由网络
        self.router = nn.Linear(config.dim, config.n_experts)
        
        # 专家网络
        self.experts = nn.ModuleList([
            nn.Sequential(
                nn.Linear(config.dim, config.expert_dim),
                nn.GELU(),
                nn.Linear(config.expert_dim, config.dim)
            )
            for _ in range(config.n_experts)
        ])
        
        # 共享专家
        self.shared_experts = nn.ModuleList([
            nn.Sequential(
                nn.Linear(config.dim, config.expert_dim),
                nn.GELU(),
                nn.Linear(config.expert_dim, config.dim)
            )
            for _ in range(config.n_shared_experts)
        ])
    
    def forward(self, x: Tensor) -> Tensor:
        """MoE前向传播
        
        Args:
            x: 输入张量 [batch, seq_len, dim]
        
        Returns:
            输出张量 [batch, seq_len, dim]
        """
        batch, seq_len, dim = x.shape
        
        # 路由计算
        router_logits = self.router(x)  # [batch, seq_len, n_experts]
        weights, selected_experts = torch.topk(
            F.softmax(router_logits, dim=-1), 
            k=self.n_experts_per_tok,
            dim=-1
        )  # weights: [batch, seq_len, n_experts_per_tok]
        
        # 专家计算（稀疏激活）
        output = torch.zeros_like(x)
        for i, expert in enumerate(self.experts):
            # 找到选择该专家的token
            mask = (selected_experts == i).any(dim=-1)  # [batch, seq_len]
            if mask.any():
                expert_input = x[mask]  # [n_selected, dim]
                expert_output = expert(expert_input)  # [n_selected, dim]
                
                # 加权聚合
                expert_weights = weights[mask][selected_experts[mask] == i]
                output[mask] += expert_weights.unsqueeze(-1) * expert_output
        
        # 共享专家（所有token都经过）
        for shared_expert in self.shared_experts:
            output += shared_expert(x)
        
        return output
```

### 3.3 路由设计（方法的多场景分发）

**场景：注意力机制选择**

```python
class AttentionRouter:
    """注意力路由 - 同一场景，不同策略"""
    
    def __init__(self, config: MythosConfig):
        self.config = config
    
    def get_attention(self) -> nn.Module:
        """根据配置返回对应的注意力模块
        
        Args:
            config.attn_type: "mla" 或 "gqa"
        
        Returns:
            对应的注意力模块实例
        """
        if self.config.attn_type == "mla":
            return MLAAttention(self.config)
        elif self.config.attn_type == "gqa":
            return GQAAttention(self.config)
        else:
            raise ValueError(f"Unknown attention type: {self.config.attn_type}")
```

### 3.4 接口定义（跨模块契约）

```python
from typing import Protocol, Optional
from dataclasses import dataclass
import torch
from torch import Tensor, nn

class AttentionInterface(Protocol):
    """注意力接口契约"""
    
    def forward(self, x: Tensor, mask: Optional[Tensor] = None) -> Tensor:
        """前向传播
        
        Args:
            x: 输入张量 [batch, seq_len, dim]
            mask: 注意力掩码
        
        Returns:
            输出张量 [batch, seq_len, dim]
        """
        ...

class RecurrentInterface(Protocol):
    """循环块接口契约"""
    
    def forward(self, x: Tensor, n_loops: Optional[int] = None) -> Tensor:
        """循环前向传播
        
        Args:
            x: 输入张量
            n_loops: 循环次数
        
        Returns:
            输出张量
        """
        ...
    
    def get_A(self) -> Tensor:
        """获取注入矩阵A（用于检查谱半径）"""
        ...

class MoEInterface(Protocol):
    """MoE接口契约"""
    
    def forward(self, x: Tensor) -> Tensor:
        """MoE前向传播
        
        Args:
            x: 输入张量
        
        Returns:
            输出张量
        """
        ...
```

### 3.5 数据流图

```
输入token IDs
    ↓
[Embedding层] → 词嵌入向量
    ↓
[Prelude] → 初始编码（标准Transformer层）
    ↓
[RecurrentBlock] → 循环推理（核心）
    ├── 第1次循环: h₁ = A·h₀ + B·e + Transformer(h₀, e)
    ├── 第2次循环: h₂ = A·h₁ + B·e + Transformer(h₁, e)
    ├── ...
    └── 第T次循环: h_T = A·h_{T-1} + B·e + Transformer(h_{T-1}, e)
    ↓
[Coda] → 最终处理（标准Transformer层）
    ↓
[LM Head] → 词表logits
    ↓
输出概率分布
```

---

## 四、❌ 属性详细说明

### Prelude模块
- **职责**：初始编码，将输入转换为适合循环处理的表示
- **实现**：标准Transformer层堆叠
- **特点**：只执行一次，为循环块准备输入

### Coda模块
- **职责**：最终处理，将循环输出转换为适合预测的形式
- **实现**：标准Transformer层堆叠
- **特点**：只执行一次，处理循环块的最终输出

### GQAAttention模块
- **职责**：分组查询注意力实现
- **与MLA区别**：不使用低秩压缩，直接计算Q/K/V
- **适用场景**：对KV缓存要求不高的场景

### MythosTokenizer
- **职责**：文本与token ID的相互转换
- **实现**：基于GPT-2/GPT-4的分词器
- **特点**：支持长上下文编码

---

## 核心公式汇总

```
┌─────────────────────────────────────────────────────────┐
│              OpenMythos 核心公式                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  循环更新公式：                                          │
│  h_{t+1} = A·h_t + B·e + Transformer(h_t, e)           │
│                                                         │
│  其中：                                                  │
│  - h_t: 第t次循环的隐状态                                │
│  - e: 来自Prelude的编码输入（每次循环注入）               │
│  - A, B: 学习的注入参数矩阵                              │
│  - Transformer: 注意力 + MoE前馈                        │
│                                                         │
│  谱半径约束（稳定性）：                                   │
│  ρ(A) < 1                                               │
│                                                         │
│  注意力计算（MLA）：                                     │
│  Q = [Q_up(Q_down(x)); Q_rope(x)]                       │
│  K = [K_up(KV_down(x)); K_rope(x)]                      │
│  V = V_up(KV_down(x))                                   │
│                                                         │
│  MoE路由：                                               │
│  weights, experts = TopK(Softmax(Router(x)), k)         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 设计亮点总结

| 亮点 | 说明 | 价值 |
|-----|------|------|
| **循环深度** | 用循环代替堆叠 | 参数复用，深度可变 |
| **输入注入** | 每次循环注入原始信号 | 防止信息漂移 |
| **MLA注意力** | 低秩压缩KV | 减少缓存，提高效率 |
| **MoE稀疏激活** | 只激活部分专家 | 参数高效 |
| **注意力切换** | MLA/GQA可选 | 灵活适配不同场景 |

---

## 参考链接

- [[99-架构图谱]]
- [[00-GitHub项目索引]]
- [[01-OpenMythos-三阶解构]]
- [[02-RecurrentBlock-三阶解构]]
- [[03-MLAAttention-三阶解构]]
- [[04-MoE-三阶解构]]