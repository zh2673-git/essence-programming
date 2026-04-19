# key_transactions - 三阶解构

> 父文档：[[04-utils-三阶解构]]
> 源码位置：`src/utils/key_transactions.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | KeyTransactions |
| **功能** | 关键交易识别 |
| **识别方式** | 多条件判断 |

---

## 一、是什么 (What)

**本质**：关键交易模式识别器

**核心职责**：
1. **识别重点收支** - 工作、房产、车辆等
2. **识别特殊金额** - 整数、吉利数字等
3. **识别特殊日期** - 节假日、周末等
4. **交易分类** - 按业务类型分类

---

## 二、为什么 (Why)

**存在理由**：
- 发现重要交易线索
- 识别可疑交易模式
- 支持报告重点展示
- 辅助人工分析

---

## 三、怎么做 (How)

### 类结构

```
KeyTransactions
├── 属性: config: Config
├── 方法: identify(data) -> KeyTransactionResult
├── 方法: classify_income(data) -> DataFrame
├── 方法: classify_expense(data) -> DataFrame
└── 方法: flag_special(data) -> DataFrame
```

### 核心方法

**identify()**

```python
def identify(self, data: DataFrame) -> KeyTransactionResult:
    """识别关键交易"""
    return KeyTransactionResult(
        work_related=self._identify_work_transactions(data),
        property_related=self._identify_property_transactions(data),
        vehicle_related=self._identify_vehicle_transactions(data),
        special_amounts=self._identify_special_amounts(data),
        special_dates=self._identify_special_dates(data)
    )
```

**分类规则示例**

```python
def _identify_work_transactions(self, data: DataFrame) -> DataFrame:
    """识别工作相关交易"""
    work_keywords = ['工资', '奖金', '报销', '补贴']
    mask = data['交易摘要'].str.contains('|'.join(work_keywords), na=False)
    return data[mask]

def _identify_special_amounts(self, data: DataFrame) -> DataFrame:
    """识别特殊金额"""
    # 整数金额
    integer_mask = data['交易金额'] == data['交易金额'].round()
    # 吉利数字
    lucky_mask = data['交易金额'].astype(str).str.contains(r'666|888|999')
    return data[integer_mask | lucky_mask]
```

---

## 四、参考链接

- [[04-utils-三阶解构]]