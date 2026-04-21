# MLAAttention 多头潜在注意力 - 三阶解构

> 父文档：[[00-OpenMythos-三阶解构]]
> 模块类型：注意力机制（可选）

---

## 一、是什么 (What) - 模块本质

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**通过低秩压缩减少KV缓存，提高效率**
- 因此该模块的本质是：**低秩注意力压缩器 (Low-Rank Attention Compressor)**

这是DeepSeek-V2引入的创新注意力机制。

| 推导要素 | 逻辑分析 | 结论 |
|---------|---------|------|
| **核心问题** | 长上下文推理时KV缓存占用太大 | 如何减少KV缓存？ |
| **解决思路** | 用低秩矩阵压缩K和V | 类似LoRA的思想 |
| **模块定位** | 注意力计算的可选实现 | 与GQA二选一 |
| **核心价值** | 显著减少推理时的KV缓存 | 内存效率提升 |

**本质特征**：
1. **低秩压缩** - 用低秩矩阵分解压缩K和V
2. **解耦RoPE** - 分离可旋转和不可旋转部分
3. **潜在向量** - 中间潜在表示替代完整KV
4. **缓存优化** - 推理时只缓存压缩后的潜在向量

### 1.2 关键属性

| 属性名 | 推导来源 | 说明 | 标记 |
|-------|---------|------|------|
| **q_down/q_up** | 低秩压缩需要 | Q的降维/升维 | ❌ 线性层 |
| **kv_down** | 低秩压缩需要 | KV的降维 | ❌ 线性层 |
| **k_up/v_up** | 低秩压缩需要 | K/V的升维 | ❌ 线性层 |
| **q_rope/k_rope** | 解耦RoPE需要 | RoPE投影 | ❌ 线性层 |
| **forward** | 注意力计算需要 | 前向传播 | ✅ 核心方法 |

### 1.3 区分属性

| 对比项 | MLA | GQA | MHA (标准) |
|-------|-----|-----|-----------|
| **KV缓存** | 极小（低秩） | 中等（分组） | 大（完整） |
| **Q投影** | 低秩 | 完整 | 完整 |
| **K/V投影** | 低秩 | 分组共享 | 独立 |
| **计算量** | 较小 | 中等 | 大 |
| **适用场景** | 长上下文 | 中等上下文 | 短上下文 |

---

## 二、为什么 (Why) - 设计理由

### 2.1 存在理由

```
因（问题）：
    1. 长上下文推理时，KV缓存占用巨大内存
    2. 标准MHA每个头独立存储K/V，冗余度高
    3. GQA虽然减少缓存，但Q投影仍是完整维度

果（解决方案）：
    1. 用低秩矩阵压缩K和V：KV_down → 潜在向量 → K_up/V_up
    2. Q也用低秩压缩：Q_down → 潜在向量 → Q_up
    3. 解耦RoPE：分离需要位置编码的部分
```

### 2.2 核心公式

**低秩压缩**：
```
# KV压缩（关键！）
c_kv = KV_down(x)  # [batch, seq, kv_lora_rank]
缓存大小 = batch × seq_len × kv_lora_rank  (极大减少！)

# Q压缩
c_q = Q_down(x)    # [batch, seq, q_lora_rank]
```

**解耦RoPE**：
```
Q = [Q_up(c_q); Q_rope(x)]      # 非RoPE部分 + RoPE部分
K = [K_up(c_kv); K_rope(x)]     # 非RoPE部分 + RoPE部分
V = V_up(c_kv)                   # 只有非RoPE部分
```

**压缩比**：
```
原始KV缓存：2 × n_heads × head_dim = 2 × dim
MLA缓存：kv_lora_rank

压缩比 = (2 × dim) / kv_lora_rank

例如：dim=4096, kv_lora_rank=512
压缩比 = 8192 / 512 = 16×
```

---

## 三、怎么做 (How) - 实现细节

### 3.1 模块结构

```python
class MLAAttention(nn.Module):
    """Multi-head Latent Attention"""
    
    # 属性定义
    config: MythosConfig
    
    # Q低秩投影
    q_down: nn.Linear
    q_up: nn.Linear
    q_rope: nn.Linear
    
    # KV低秩投影
    kv_down: nn.Linear
    k_up: nn.Linear
    v_up: nn.Linear
    k_rope: nn.Linear
    
    # 方法定义
    def __init__(self, config: MythosConfig) -> None
    def forward(self, x: Tensor, mask: Tensor = None) -> Tensor
```

### 3.2 核心方法

#### `__init__` 方法

```python
def __init__(self, config: MythosConfig):
    """初始化MLA注意力
    
    维度说明：
    - dim: 模型维度
    - q_lora_rank: Q的潜在维度（通常64-128）
    - kv_lora_rank: KV的潜在维度（通常512）
    - qk_rope_head_dim: 每个头的RoPE维度
    - qk_nope_head_dim: 每个头的非RoPE维度
    - v_head_dim: V的头维度
    """
    super().__init__()
    self.config = config
    
    # Q低秩投影
    self.q_down = nn.Linear(config.dim, config.q_lora_rank, bias=False)
    self.q_up = nn.Linear(config.q_lora_rank, config.n_heads * config.qk_nope_head_dim, bias=False)
    self.q_rope = nn.Linear(config.dim, config.n_heads * config.qk_rope_head_dim, bias=False)
    
    # KV低秩投影（缓存的关键！）
    self.kv_down = nn.Linear(config.dim, config.kv_lora_rank, bias=False)
    self.k_up = nn.Linear(config.kv_lora_rank, config.n_heads * config.qk_nope_head_dim, bias=False)
    self.v_up = nn.Linear(config.kv_lora_rank, config.n_heads * config.v_head_dim, bias=False)
    self.k_rope = nn.Linear(config.dim, config.n_heads * config.qk_rope_head_dim, bias=False)
    
    # 输出投影
    self.o_proj = nn.Linear(config.n_heads * config.v_head_dim, config.dim, bias=False)
```

#### `forward` 方法

```python
def forward(self, x: Tensor, mask: Tensor = None) -> Tensor:
    """MLA前向传播
    
    计算流程：
    1. Q低秩压缩和解耦
    2. KV低秩压缩和解耦
    3. 合并RoPE和非RoPE部分
    4. 标准注意力计算
    5. 输出投影
    """
    batch_size, seq_len, _ = x.shape
    
    # ========== 1. Q低秩压缩和解耦 ==========
    q_compressed = self.q_down(x)  # [batch, seq, q_lora_rank]
    q_nope = self.q_up(q_compressed)  # [batch, seq, n_heads × qk_nope_head_dim]
    q_nope = q_nope.view(batch_size, seq_len, self.config.n_heads, self.config.qk_nope_head_dim)
    
    q_rope = self.q_rope(x)  # [batch, seq, n_heads × qk_rope_head_dim]
    q_rope = q_rope.view(batch_size, seq_len, self.config.n_heads, self.config.qk_rope_head_dim)
    q_rope = apply_rotary_pos_emb(q_rope, seq_len)
    
    q = torch.cat([q_nope, q_rope], dim=-1)  # [batch, seq, n_heads, head_dim_total]
    
    # ========== 2. KV低秩压缩和解耦 ==========
    kv_compressed = self.kv_down(x)  # [batch, seq, kv_lora_rank] ⭐ 缓存这个！
    
    k_nope = self.k_up(kv_compressed)  # [batch, seq, n_heads × qk_nope_head_dim]
    k_nope = k_nope.view(batch_size, seq_len, self.config.n_heads, self.config.qk_nope_head_dim)
    
    v = self.v_up(kv_compressed)  # [batch, seq, n_heads × v_head_dim]
    v = v.view(batch_size, seq_len, self.config.n_heads, self.config.v_head_dim)
    
    k_rope = self.k_rope(x)  # [batch, seq, n_heads × qk_rope_head_dim]
    k_rope = k_rope.view(batch_size, seq_len, self.config.n_heads, self.config.qk_rope_head_dim)
    k_rope = apply_rotary_pos_emb(k_rope, seq_len)
    
    k = torch.cat([k_nope, k_rope], dim=-1)  # [batch, seq, n_heads, head_dim_total]
    
    # ========== 3. 标准注意力计算 ==========
    q = q.transpose(1, 2)  # [batch, n_heads, seq, head_dim]
    k = k.transpose(1, 2)
    v = v.transpose(1, 2)
    
    scores = torch.matmul(q, k.transpose(-2, -1)) / math.sqrt(q.size(-1))
    if mask is not None:
        scores = scores.masked_fill(mask == 0, float("-inf"))
    
    attn_weights = F.softmax(scores, dim=-1)
    attn_output = torch.matmul(attn_weights, v)
    
    # ========== 4. 输出投影 ==========
    attn_output = attn_output.transpose(1, 2).contiguous().view(batch_size, seq_len, -1)
    output = self.o_proj(attn_output)
    
    return output
```

### 3.3 维度变化图

```
输入 x: [batch, seq_len, dim]
    │
    ├──→ q_down ──→ [batch, seq_len, q_lora_rank]
    │       │
    │       └──→ q_up ──→ [batch, seq_len, n_heads × qk_nope_head_dim]
    │               │
    │               └──→ reshape ──→ [batch, seq_len, n_heads, qk_nope_head_dim]
    │                       │
    │                       └──→ q_nope
    │
    ├──→ q_rope ──→ [batch, seq_len, n_heads × qk_rope_head_dim]
    │       │
    │       └──→ reshape ──→ [batch, seq_len, n_heads, qk_rope_head_dim]
    │               │
    │               └──→ apply_rotary_pos_emb ──→ q_rope
    │
    └──→ concat([q_nope, q_rope], dim=-1) ──→ q: [batch, seq_len, n_heads, head_dim_total]
    
    ├──→ kv_down ──→ [batch, seq_len, kv_lora_rank] ⭐ 缓存这个！
    │       │
    │       ├──→ k_up ──→ [batch, seq_len, n_heads × qk_nope_head_dim]
    │       │       │
    │       │       └──→ reshape ──→ k_nope: [batch, seq_len, n_heads, qk_nope_head_dim]
    │       │
    │       └──→ v_up ──→ [batch, seq_len, n_heads × v_head_dim]
    │               │
    │               └──→ reshape ──→ v: [batch, seq_len, n_heads, v_head_dim]
    │
    ├──→ k_rope ──→ [batch, seq_len, n_heads × qk_rope_head_dim]
    │       │
    │       └──→ reshape + RoPE ──→ k_rope: [batch, seq_len, n_heads, qk_rope_head_dim]
    │
    └──→ concat([k_nope, k_rope], dim=-1) ──→ k: [batch, seq_len, n_heads, head_dim_total]

注意力计算:
    q: [batch, n_heads, seq_len, head_dim]
    k: [batch, n_heads, seq_len, head_dim]
    v: [batch, n_heads, seq_len, v_head_dim]
    
    scores = q @ k.T / sqrt(dim) ──→ [batch, n_heads, seq_len, seq_len]
    attn = softmax(scores) @ v ──→ [batch, n_heads, seq_len, v_head_dim]
    
    reshape ──→ [batch, seq_len, n_heads × v_head_dim]
    │
    └──→ o_proj ──→ [batch, seq_len, dim]
```

---

## 四、关键设计决策

| 决策 | 选择 | 理由 |
|-----|------|------|
| **Q压缩** | 使用q_lora_rank | 进一步减少计算量 |
| **KV共享降维** | 同一个kv_down | K和V共享压缩表示，更高效 |
| **RoPE解耦** | 分离q_nope/q_rope | 只有部分需要位置编码 |
| **缓存内容** | kv_compressed + k_rope | 最小化缓存大小 |
| **bias** | 不使用 | 减少参数量，现代LLM趋势 |

---

## 参考链接

- [[00-OpenMythos-三阶解构]] - 项目级分析
- [[99-架构图谱]] - 架构汇总