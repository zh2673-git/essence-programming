# bank_analyzer - 三阶解构

> 父文档：[[02-analysis-三阶解构]]
> 源码位置：`src/analysis/bank_analyzer.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | BankAnalyzer |
| **继承** | BaseAnalyzer |
| **模块类型** | 分析器 |
| **分析对象** | 银行交易数据 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个类本质上是什么？**

BankAnalyzer的本质是：**银行交易数据的统计分析引擎**

**核心职责**：
1. **频率分析** - 统计与各方的交易次数
2. **金额分析** - 分析收支分布、识别大额交易
3. **时间分析** - 分析交易时间模式
4. **存取现分析** - 统计存取现频率和金额

### 1.2 关键属性

| 属性名 | 类型 | 说明 |
|-------|------|------|
| **config** | Config | 分析配置（阈值等） |
| **large_amount_threshold** | float | 大额交易阈值 |
| **top_n** | int | Top N统计数量 |

### 1.3 分析结果结构

```python
@dataclass
class BankAnalysisResult:
    frequency_stats: DataFrame      # 频率统计
    amount_stats: Dict              # 金额统计
    time_stats: Dict                # 时间统计
    cash_stats: Dict                # 存取现统计
    large_transactions: DataFrame   # 大额交易明细
    summary: str                    # 分析摘要
```

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 需要从大量银行交易中发现关键关系
    2. 需要识别大额交易和可疑模式
    3. 需要分析存取现行为
    4. 需要为报告提供统计数据

果（解决方案）：
    1. 频率分析：识别高频交易对象
    2. 金额分析：发现大额交易
    3. 时间分析：识别交易时间规律
    4. 存取现分析：统计现金流动
```

---

## 三、怎么做 (How)

### 3.1 类结构

```
BankAnalyzer
├── 继承: BaseAnalyzer
├── 属性
│   ├── config: Config
│   ├── large_amount_threshold: float = 100000
│   └── top_n: int = 20
├── 方法: analyze(data) -> BankAnalysisResult
│   ├── frequency_analysis()
│   ├── amount_analysis()
│   ├── time_analysis()
│   └── cash_analysis()
├── 方法: frequency_analysis(data) -> DataFrame
├── 方法: amount_analysis(data) -> Dict
├── 方法: time_analysis(data) -> Dict
└── 方法: cash_analysis(data) -> Dict
```

### 3.2 核心方法实现

**方法：analyze()**

```python
def analyze(self, data: DataFrame) -> BankAnalysisResult:
    """执行完整银行数据分析
    
    分析流程：
    1. 频率分析
    2. 金额分析
    3. 时间分析
    4. 存取现分析
    """
    return BankAnalysisResult(
        frequency_stats=self.frequency_analysis(data),
        amount_stats=self.amount_analysis(data),
        time_stats=self.time_analysis(data),
        cash_stats=self.cash_analysis(data),
        large_transactions=self._get_large_transactions(data),
        summary=self._generate_summary(data)
    )
```

**方法：frequency_analysis()**

```python
def frequency_analysis(self, data: DataFrame) -> DataFrame:
    """频率分析 - 统计与各方的交易次数
    
    统计维度：
    - 按本方姓名+对方姓名分组
    - 统计交易次数
    - 排序取Top N
    """
    freq_stats = data.groupby(
        ['本方姓名', '对方姓名']
    ).agg({
        '交易金额': ['count', 'sum'],
        '收入金额': 'sum',
        '支出金额': 'sum'
    }).reset_index()
    
    freq_stats.columns = ['本方姓名', '对方姓名', '交易次数', '净额', '总收入', '总支出']
    freq_stats = freq_stats.sort_values('交易次数', ascending=False)
    
    return freq_stats.head(self.top_n)
```

**方法：amount_analysis()**

```python
def amount_analysis(self, data: DataFrame) -> Dict:
    """金额分析
    
    分析内容：
    - 总收支统计
    - 金额分布
    - 大额交易识别
    """
    return {
        '总收入': data['收入金额'].sum(),
        '总支出': data['支出金额'].sum(),
        '净额': data['交易金额'].sum(),
        '最大单笔收入': data['收入金额'].max(),
        '最大单笔支出': data['支出金额'].max(),
        '平均收入': data['收入金额'].mean(),
        '平均支出': data['支出金额'].mean(),
        '大额交易数': len(data[data['交易金额'].abs() > self.large_amount_threshold])
    }
```

**方法：cash_analysis()**

```python
def cash_analysis(self, data: DataFrame) -> Dict:
    """存取现分析
    
    分析内容：
    - 存款次数和金额
    - 取款次数和金额
    - 存取现比例
    """
    cash_data = data[data['现金标志'] == 'cash']
    
    deposits = cash_data[cash_data['存取现类型'] == 'deposit']
    withdraws = cash_data[cash_data['存取现类型'] == 'withdraw']
    
    return {
        '存款次数': len(deposits),
        '存款总额': deposits['交易金额'].sum(),
        '取款次数': len(withdraws),
        '取款总额': abs(withdraws['交易金额'].sum()),
        '存取现次数比': len(deposits) / max(len(withdraws), 1),
        '存取现金额比': deposits['交易金额'].sum() / max(abs(withdraws['交易金额'].sum()), 1)
    }
```

### 3.3 使用示例

```python
from analysis.bank_analyzer import BankAnalyzer
from utils.config import Config

config = Config('config.json')
analyzer = BankAnalyzer(config)

# 执行分析
result = analyzer.analyze(bank_data)

# 查看结果
print("频率统计:")
print(result.frequency_stats)

print("\n金额统计:")
print(result.amount_stats)

print("\n存取现统计:")
print(result.cash_stats)
```

---

## 四、参考链接

- [[02-analysis-三阶解构]]
- [[01-1-bank_model-三阶解构]]