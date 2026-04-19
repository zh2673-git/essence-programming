# analysis模块 - 三阶解构

> 父文档：[[00-datadone-三阶解构]]
> 模块路径：`src/analysis/`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **模块名称** | analysis |
| **所属项目** | datadone |
| **模块职责** | 分析引擎层 - 负责数据分析和统计计算 |
| **分析日期** | 2026-04-19 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**金融交易分析**
- 因此该模块的本质是：**分析引擎 - 对标准化数据进行统计分析和模式识别**

**核心职责**：
1. **频率分析** - 统计交易/通话频率，识别高频对象
2. **金额分析** - 分析金额分布，识别大额交易
3. **时间分析** - 分析时间模式，识别规律性行为
4. **关联分析** - 跨数据源关联，构建关系网络
5. **异常检测** - 识别可疑交易和行为模式

### 1.2 关键属性（子模块识别）

```
analysis/
├── __init__.py                    → 模块初始化
├── base_analyzer.py (继承)        → 分析器基类
├── bank_analyzer.py               → 银行数据分析器 ✅ 需深入
├── call_analyzer.py               → 话单数据分析器 ✅ 需深入
├── comprehensive_analyzer.py      → 综合分析器 ✅ 需深入
└── payment/                       → 支付分析子模块
    ├── __init__.py
    ├── wechat_analyzer.py         → 微信数据分析器 ✅ 需深入
    └── alipay_analyzer.py         → 支付宝数据分析器 ✅ 需深入
```

| 属性名 | 源码位置 | 说明 | 标记 |
|-------|---------|------|------|
| **BankAnalyzer** | `bank_analyzer.py` | 银行数据分析器 | ✅ 需深入 → [[02-1-bank_analyzer-三阶解构]] |
| **CallAnalyzer** | `call_analyzer.py` | 话单数据分析器 | ✅ 需深入 → [[02-2-call_analyzer-三阶解构]] |
| **ComprehensiveAnalyzer** | `comprehensive_analyzer.py` | 综合分析器 | ✅ 需深入 → [[02-3-comprehensive_analyzer-三阶解构]] |
| **WechatAnalyzer** | `payment/wechat_analyzer.py` | 微信数据分析器 | ✅ 需深入 → [[02-4-wechat_analyzer-三阶解构]] |
| **AlipayAnalyzer** | `payment/alipay_analyzer.py` | 支付宝数据分析器 | ✅ 需深入 → [[02-5-alipay_analyzer-三阶解构]] |

### 1.3 区分属性

| 对比项 | analysis模块 | 数据查询 | 机器学习 |
|-------|-------------|---------|---------|
| **职责** | 业务分析+统计 | 简单筛选 | 模式识别 |
| **输入** | 标准化DataFrame | 原始数据 | 特征向量 |
| **输出** | 分析结果对象 | 查询结果 | 预测结果 |
| **复杂度** | 中等（业务逻辑） | 低 | 高 |

**边界结论**：
- ✅ 做：统计分析、频率计算、关联分析、异常检测
- ❌ 不做：原始数据读取、报告格式渲染、复杂机器学习

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 需要从海量交易中发现关键信息（高频联系人、大额交易）
    2. 需要识别行为模式（交易习惯、时间规律）
    3. 需要跨数据源关联（银行记录+通话记录）
    4. 需要检测异常行为（可疑交易、异常模式）

果（解决方案）：
    1. 频率分析：统计交易次数，排序识别高频对象
    2. 金额分析：计算金额分布，设定阈值识别大额
    3. 时间分析：按时间段聚合，识别时间规律
    4. 综合分析：多数据源关联，构建关系网络
```

### 2.2 设计决策

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **分析粒度** | 按人员分组 | 金融调查以人员为维度 |
| **计算方式** | pandas向量化 | 性能优化，避免循环 |
| **结果结构** | 分析结果对象 | 包含统计数据+明细数据 |
| **缓存策略** | 中间结果缓存 | 避免重复计算 |

---

## 三、怎么做 (How)

### 3.1 模块划分

```
analysis (分析引擎层)
├── BaseAnalyzer (抽象基类)
│   ├── 属性：分析配置
│   ├── 方法：analyze() - 抽象方法
│   ├── 方法：get_statistics() - 抽象方法
│   └── 方法：validate_data() - 数据验证
│
├── BankAnalyzer (银行分析)
│   ├── 方法：analyze() - 主分析入口
│   ├── 方法：frequency_analysis() - 频率分析
│   ├── 方法：amount_analysis() - 金额分析
│   ├── 方法：time_analysis() - 时间分析
│   └── 方法：cash_analysis() - 存取现分析
│
├── CallAnalyzer (话单分析)
│   ├── 方法：analyze() - 主分析入口
│   ├── 方法：frequency_analysis() - 通话频率
│   ├── 方法：time_analysis() - 通话时间分布
│   └── 方法：contact_analysis() - 联系人分析
│
├── ComprehensiveAnalyzer (综合分析)
│   ├── 方法：cross_analysis() - 交叉分析
│   ├── 方法：relation_network() - 关系网络
│   ├── 方法：platform_comparison() - 平台对比
│   └── 方法：fund_tracking() - 资金追踪
│
├── WechatAnalyzer (微信分析)
│   ├── 方法：analyze() - 主分析入口
│   ├── 方法：transfer_analysis() - 转账分析
│   └── 方法：redpacket_analysis() - 红包分析
│
└── AlipayAnalyzer (支付宝分析)
    ├── 方法：analyze() - 主分析入口
    ├── 方法：consumption_analysis() - 消费分析
    └── 方法：transfer_analysis() - 转账分析
```

### 3.2 核心方法设计

**方法：频率分析**

```python
def frequency_analysis(self, data: DataFrame) -> FrequencyResult:
    """频率分析
    
    利用的属性：
    - data: 交易/通话数据
    - group_by: 分组字段（本方姓名+对方姓名）
    
    逻辑流程：
    1. 按人员分组统计
    2. 计算交易/通话次数
    3. 排序取Top N
    4. 返回频率统计结果
    """
    # 1. 分组统计
    freq_stats = data.groupby(
        [self.name_column, self.opposite_name_column]
    ).size().reset_index(name='count')
    
    # 2. 排序
    freq_stats = freq_stats.sort_values('count', ascending=False)
    
    # 3. 构建结果
    return FrequencyResult(
        top_contacts=freq_stats.head(self.top_n),
        total_records=len(data),
        unique_contacts=freq_stats[self.opposite_name_column].nunique()
    )
```

**方法：金额分析**

```python
def amount_analysis(self, data: DataFrame) -> AmountResult:
    """金额分析
    
    利用的属性：
    - data: 交易数据
    - amount_column: 金额字段
    - threshold: 大额阈值
    
    逻辑流程：
    1. 计算金额统计（总和、平均、最大、最小）
    2. 识别大额交易（超过阈值）
    3. 分析金额分布
    4. 返回金额统计结果
    """
    # 1. 基础统计
    amount_stats = {
        'total': data[self.amount_column].sum(),
        'mean': data[self.amount_column].mean(),
        'max': data[self.amount_column].max(),
        'min': data[self.amount_column].min(),
        'median': data[self.amount_column].median()
    }
    
    # 2. 大额交易识别
    large_transactions = data[
        data[self.amount_column] > self.large_amount_threshold
    ]
    
    # 3. 金额分布
    amount_distribution = pd.cut(
        data[self.amount_column],
        bins=[0, 1000, 5000, 10000, 50000, float('inf')],
        labels=['<1K', '1K-5K', '5K-10K', '10K-50K', '>50K']
    ).value_counts()
    
    return AmountResult(
        stats=amount_stats,
        large_transactions=large_transactions,
        distribution=amount_distribution
    )
```

**方法：综合分析 - 跨数据源关联**

```python
def cross_analysis(
    self,
    bank_data: DataFrame,
    call_data: DataFrame,
    wechat_data: DataFrame,
    alipay_data: DataFrame
) -> CrossAnalysisResult:
    """综合分析 - 跨数据源关联
    
    利用的属性：
    - 各平台数据DataFrame
    - 关联字段（本方姓名、对方姓名、日期）
    
    逻辑流程：
    1. 提取所有人员列表
    2. 按人员关联各平台数据
    3. 识别跨平台关联关系
    4. 构建关系网络
    """
    # 1. 提取人员列表
    all_persons = self._extract_all_persons(
        bank_data, call_data, wechat_data, alipay_data
    )
    
    # 2. 按人员分析
    person_relations = {}
    for person in all_persons:
        relations = self._analyze_person_relations(
            person, bank_data, call_data, wechat_data, alipay_data
        )
        person_relations[person] = relations
    
    # 3. 构建关系网络
    relation_network = self._build_relation_network(person_relations)
    
    return CrossAnalysisResult(
        person_relations=person_relations,
        relation_network=relation_network,
        cross_platform_matches=self._find_cross_platform_matches(
            bank_data, call_data, wechat_data, alipay_data
        )
    )
```

### 3.3 路由设计

**场景：分析类型选择**

```python
class AnalysisRouter:
    """分析路由 - 同一场景，不同策略"""
    
    def analyze(
        self,
        data_type: str,
        data: DataFrame,
        analysis_type: str,
        **kwargs
    ) -> AnalysisResult:
        """统一分析入口
        
        Args:
            data_type: 数据类型（bank/call/wechat/alipay/comprehensive）
            data: 待分析数据
            analysis_type: 分析类型
                - "frequency": 频率分析
                - "amount": 金额分析
                - "time": 时间分析
                - "full": 全部分析
        """
        # 获取分析器
        analyzers = {
            "bank": BankAnalyzer,
            "call": CallAnalyzer,
            "wechat": WechatAnalyzer,
            "alipay": AlipayAnalyzer,
            "comprehensive": ComprehensiveAnalyzer
        }
        
        analyzer_class = analyzers.get(data_type)
        if not analyzer_class:
            raise ValueError(f"Unknown data type: {data_type}")
        
        analyzer = analyzer_class(self.config)
        
        # 路由分发
        if analysis_type == "frequency":
            return analyzer.frequency_analysis(data)
        elif analysis_type == "amount":
            return analyzer.amount_analysis(data)
        elif analysis_type == "time":
            return analyzer.time_analysis(data)
        elif analysis_type == "full":
            return analyzer.analyze(data, **kwargs)
        else:
            raise ValueError(f"Unknown analysis type: {analysis_type}")
```

### 3.4 接口定义

```python
from typing import Protocol, DataFrame, Dict, Any
from dataclasses import dataclass
from abc import abstractmethod

@dataclass
class AnalysisResult:
    """分析结果基类"""
    data_type: str
    analysis_type: str
    statistics: Dict[str, Any]
    details: DataFrame
    summary: str

class BaseAnalyzer(Protocol):
    """分析器基类接口
    
    推导来源：所有分析器需要统一的分析接口
    """
    
    @property
    @abstractmethod
    def data_type(self) -> str:
        """数据类型标识"""
        ...
    
    @abstractmethod
    def analyze(self, data: DataFrame, **kwargs) -> AnalysisResult:
        """执行完整分析
        
        Args:
            data: 待分析的DataFrame
            **kwargs: 附加参数
            
        Returns:
            分析结果对象
        """
        ...
    
    @abstractmethod
    def frequency_analysis(self, data: DataFrame) -> AnalysisResult:
        """频率分析"""
        ...
    
    @abstractmethod
    def get_statistics(self) -> Dict[str, Any]:
        """获取统计信息"""
        ...
```

### 3.5 数据流图

```
┌─────────────────┐
│  DataFrame      │
│ (标准化数据)     │
│ 来自datasource   │
└────────┬────────┘
         │
         ↓
┌─────────────────────────────────────────────────────┐
│                  AnalysisRouter                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Bank      │  │    Call     │  │ Comprehensive│ │
│  │  Analyzer   │  │  Analyzer   │  │  Analyzer    │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘ │
└─────────┼────────────────┼────────────────┼────────┘
          │                │                │
          ↓                ↓                ↓
┌─────────────────────────────────────────────────────┐
│                  分析方法分发                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │  frequency  │  │   amount    │  │    time     │ │
│  │  _analysis  │  │  _analysis  │  │  _analysis  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                  ┌─────────────────┐
                  │  AnalysisResult │
                  │   (分析结果)     │
                  │  - statistics   │
                  │  - details      │
                  │  - summary      │
                  └─────────────────┘
```

---

## 四、子模块详细说明

### 4.1 BankAnalyzer (银行数据分析器)

**职责**：分析银行交易数据

**分析方法**：
- **频率分析**：统计与各方的交易次数
- **金额分析**：收支分布、大额交易识别
- **时间分析**：工作日/周末分布、时段分析
- **存取现分析**：存取现频率、金额统计

**特有指标**：
- 存取现比例
- 大额交易占比
- 收支平衡情况

### 4.2 CallAnalyzer (话单数据分析器)

**职责**：分析通话记录数据

**分析方法**：
- **频率分析**：通话次数统计
- **时间分析**：通话时间分布（时段、时长）
- **联系人分析**：高频联系人、通话时长Top

**特有指标**：
- 平均通话时长
- 主叫/被叫比例
- 短信频率

### 4.3 ComprehensiveAnalyzer (综合分析器)

**职责**：跨数据源综合分析

**分析方法**：
- **交叉分析**：多平台数据关联
- **关系网络**：构建人员关系图
- **资金追踪**：追踪资金流向
- **异常检测**：识别可疑模式

**特有功能**：
- 存取现与话单匹配
- 大额资金跨平台追踪
- 关系网络可视化

### 4.4 WechatAnalyzer (微信数据分析器)

**职责**：分析微信支付数据

**分析方法**：
- **转账分析**：转账频率、金额
- **红包分析**：红包收发统计
- **消费分析**：消费场景识别

### 4.5 AlipayAnalyzer (支付宝数据分析器)

**职责**：分析支付宝交易数据

**分析方法**：
- **消费分析**：消费类别统计
- **转账分析**：转账记录分析
- **理财分析**：理财交易识别

---

## 五、参考链接

- [[00-datadone-三阶解构]] - 项目级设计
- [[01-datasource-三阶解构]] - 数据源模块
- [[99-架构图谱]] - 架构汇总
- [[02-1-bank_analyzer-三阶解构]] - 银行分析器深入分析
- [[02-2-call_analyzer-三阶解构]] - 话单分析器深入分析
- [[02-3-comprehensive_analyzer-三阶解构]] - 综合分析器深入分析