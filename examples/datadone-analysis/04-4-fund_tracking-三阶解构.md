# fund_tracking - 三阶解构

> 父文档：[[04-utils-三阶解构]]
> 源码位置：`src/utils/fund_tracking.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | FundTracking |
| **功能** | 大额资金追踪 |
| **算法** | 递归追踪 |

---

## 一、是什么 (What)

**本质**：大额资金流向追踪器

**核心职责**：
1. **追踪资金来源** - 查找大额资金的上游来源
2. **追踪资金去向** - 查找大额资金的下游去向
3. **构建流向图** - 构建资金流动网络
4. **层级分析** - 分析资金转移层级

---

## 二、为什么 (Why)

**存在理由**：
- 追踪可疑资金流动
- 识别资金链条
- 发现隐藏关系
- 支持反洗钱分析

---

## 三、怎么做 (How)

### 类结构

```
FundTracking
├── 属性: threshold: float
├── 属性: max_depth: int
├── 方法: track(data, start_person) -> FundFlowResult
├── 方法: find_sources(data, person) -> List[Transaction]
├── 方法: find_destinations(data, person) -> List[Transaction]
└── 方法: build_flow_graph(data, start_person) -> Graph
```

### 核心方法

**track()**

```python
def track(
    self,
    data: DataFrame,
    start_person: str,
    threshold: float = 100000
) -> FundFlowResult:
    """追踪资金流向"""
    # 筛选大额交易
    large_tx = data[data['交易金额'].abs() >= threshold]
    
    # 递归追踪
    flow_graph = self._build_flow_graph(
        large_tx, start_person, current_depth=0
    )
    
    return FundFlowResult(
        start_person=start_person,
        flow_graph=flow_graph,
        total_amount=flow_graph.total_amount,
        involved_persons=flow_graph.nodes
    )
```

**_build_flow_graph()**

```python
def _build_flow_graph(
    self,
    data: DataFrame,
    person: str,
    current_depth: int
) -> FlowGraph:
    """递归构建资金流向图"""
    if current_depth >= self.max_depth:
        return FlowGraph()
    
    # 查找与该人员的交易
    person_tx = data[
        (data['本方姓名'] == person) | 
        (data['对方姓名'] == person)
    ]
    
    graph = FlowGraph()
    for _, tx in person_tx.iterrows():
        graph.add_edge(tx['本方姓名'], tx['对方姓名'], tx['交易金额'])
        
        # 递归追踪对方
        next_person = tx['对方姓名'] if tx['本方姓名'] == person else tx['本方姓名']
        subgraph = self._build_flow_graph(data, next_person, current_depth + 1)
        graph.merge(subgraph)
    
    return graph
```

---

## 四、参考链接

- [[04-utils-三阶解构]]
- [[02-3-comprehensive_analyzer-三阶解构]]