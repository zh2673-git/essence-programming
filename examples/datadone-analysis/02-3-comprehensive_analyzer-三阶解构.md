# comprehensive_analyzer - 三阶解构

> 父文档：[[02-analysis-三阶解构]]
> 源码位置：`src/analysis/comprehensive_analyzer.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | ComprehensiveAnalyzer |
| **继承** | BaseAnalyzer |
| **分析对象** | 多平台综合数据 |

---

## 一、是什么 (What)

**本质**：跨数据源综合分析引擎

**核心职责**：
1. **交叉分析** - 多平台数据关联
2. **关系网络** - 构建人员关系图
3. **资金追踪** - 追踪资金流向
4. **异常检测** - 识别可疑模式

---

## 二、为什么 (Why)

**存在理由**：
- 发现跨平台关联关系
- 追踪资金完整路径
- 识别存取现与话单的关联
- 构建全面的关系网络

---

## 三、怎么做 (How)

### 类结构

```
ComprehensiveAnalyzer
├── 继承: BaseAnalyzer
├── 方法: cross_analysis(bank, call, wechat, alipay) -> CrossResult
├── 方法: relation_network(data) -> NetworkGraph
├── 方法: fund_tracking(data, start_person) -> FundFlow
└── 方法: platform_comparison(data) -> ComparisonResult
```

### 核心方法

**cross_analysis()**

```python
def cross_analysis(
    self,
    bank_data: DataFrame,
    call_data: DataFrame,
    wechat_data: DataFrame,
    alipay_data: DataFrame
) -> CrossAnalysisResult:
    """跨平台交叉分析"""
    # 提取所有人员
    all_persons = self._extract_all_persons(
        bank_data, call_data, wechat_data, alipay_data
    )
    
    # 按人员分析
    person_analysis = {}
    for person in all_persons:
        person_analysis[person] = {
            'bank': self._analyze_bank_for_person(person, bank_data),
            'call': self._analyze_call_for_person(person, call_data),
            'wechat': self._analyze_wechat_for_person(person, wechat_data),
            'alipay': self._analyze_alipay_for_person(person, alipay_data)
        }
    
    return CrossAnalysisResult(
        persons=all_persons,
        analysis=person_analysis,
        cross_matches=self._find_cross_platform_matches(...)
    )
```

**relation_network()**

```python
def relation_network(self, data: Dict[str, DataFrame]) -> NetworkGraph:
    """构建关系网络"""
    G = nx.Graph()
    
    # 添加节点（人员）
    for person in self._extract_all_persons(**data):
        G.add_node(person)
    
    # 添加边（交易/通话关系）
    for df in data.values():
        for _, row in df.iterrows():
            if '对方姓名' in row and row['对方姓名']:
                G.add_edge(row['本方姓名'], row['对方姓名'])
    
    return G
```

---

## 四、参考链接

- [[02-analysis-三阶解构]]