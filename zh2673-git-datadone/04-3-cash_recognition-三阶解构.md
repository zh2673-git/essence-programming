# cash_recognition - 三阶解构

> 父文档：[[04-utils-三阶解构]]
> 源码位置：`src/utils/cash_recognition.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | CashRecognition |
| **功能** | 存取现识别 |
| **识别方式** | 关键词匹配 |

---

## 一、是什么 (What)

**本质**：银行存取现交易智能识别器

**核心职责**：
1. **识别存款** - 通过关键词识别存款交易
2. **识别取款** - 通过关键词识别取款交易
3. **排除非现金** - 过滤转账、利息等非现金交易
4. **分类标记** - 为交易添加存取现标记

---

## 二、为什么 (Why)

**存在理由**：
- 存取现是资金追踪的关键节点
- 需要与话单进行关联分析
- 识别可疑的现金流动
- 统计现金收支情况

---

## 三、怎么做 (How)

### 类结构

```
CashRecognition
├── 属性: deposit_keywords: List[str]
├── 属性: withdraw_keywords: List[str]
├── 属性: exclude_keywords: List[str]
├── 方法: recognize(transaction) -> str
├── 方法: is_deposit(summary) -> bool
└── 方法: is_withdraw(summary) -> bool
```

### 核心方法

**recognize()**

```python
def recognize(self, transaction: Dict) -> str:
    """识别交易类型
    
    Returns:
        'deposit' - 存款
        'withdraw' - 取款
        'transfer' - 转账（非现金）
    """
    summary = transaction.get('summary', '')
    
    # 检查排除关键词
    for keyword in self.exclude_keywords:
        if keyword in summary:
            return 'transfer'
    
    # 匹配存款关键词
    for keyword in self.deposit_keywords:
        if keyword in summary:
            return 'deposit'
    
    # 匹配取款关键词
    for keyword in self.withdraw_keywords:
        if keyword in summary:
            return 'withdraw'
    
    return 'transfer'
```

### 关键词配置

```json
{
  "cash_recognition": {
    "deposit_keywords": ["存入", "存款", "现金存入", "ATM存款"],
    "withdraw_keywords": ["取出", "取款", "现金支取", "ATM取款"],
    "exclude_keywords": ["转账", "汇款", "利息", "手续费"]
  }
}
```

---

## 四、参考链接

- [[04-utils-三阶解构]]
- [[01-1-bank_model-三阶解构]]