# call_analyzer - 三阶解构

> 父文档：[[02-analysis-三阶解构]]
> 源码位置：`src/analysis/call_analyzer.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | CallAnalyzer |
| **继承** | BaseAnalyzer |
| **分析对象** | 通话记录数据 |

---

## 一、是什么 (What)

**本质**：通话记录数据的统计分析引擎

**核心职责**：
1. **频率分析** - 统计与各方的通话次数
2. **时间分析** - 分析通话时间分布
3. **联系人分析** - 识别高频联系人
4. **通话时长分析** - 统计通话时长分布

---

## 二、为什么 (Why)

**存在理由**：
- 发现高频通话联系人
- 识别通话时间规律
- 与银行存取现进行关联分析
- 为关系网络构建提供数据

---

## 三、怎么做 (How)

### 类结构

```
CallAnalyzer
├── 继承: BaseAnalyzer
├── 方法: analyze(data) -> CallAnalysisResult
├── 方法: frequency_analysis(data) -> DataFrame
├── 方法: time_analysis(data) -> Dict
└── 方法: contact_analysis(data) -> DataFrame
```

### 核心方法

**frequency_analysis()**

```python
def frequency_analysis(self, data: DataFrame) -> DataFrame:
    """频率分析"""
    freq_stats = data.groupby(['本方号码', '对方号码']).agg({
        '通话时长': ['count', 'sum'],
        '通话类型': lambda x: (x == '主叫').sum()
    }).reset_index()
    
    freq_stats.columns = ['本方号码', '对方号码', '通话次数', '总时长', '主叫次数']
    return freq_stats.sort_values('通话次数', ascending=False).head(self.top_n)
```

**time_analysis()**

```python
def time_analysis(self, data: DataFrame) -> Dict:
    """时间分析"""
    data['小时'] = data['通话时间'].dt.hour
    
    return {
        '时段分布': data.groupby('小时').size().to_dict(),
        '工作日通话': len(data[data['通话时间'].dt.weekday < 5]),
        '周末通话': len(data[data['通话时间'].dt.weekday >= 5]),
        '平均通话时长': data['通话时长'].mean()
    }
```

---

## 四、参考链接

- [[02-analysis-三阶解构]]
- [[01-2-call_model-三阶解构]]