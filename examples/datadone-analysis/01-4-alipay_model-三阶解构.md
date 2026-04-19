# alipay_model - 三阶解构

> 父文档：[[01-datasource-三阶解构]]
> 源码位置：`src/datasource/payment/alipay_model.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | AlipayModel |
| **继承** | BaseModel |
| **模块类型** | 数据模型 |
| **数据类型** | 支付宝账单数据 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个类本质上是什么？**

AlipayModel的本质是：**支付宝账单数据的加载器和转换器**

**核心职责**：
1. **加载支付宝账单CSV文件** - 读取支付宝导出的账单数据
2. **字段标准化** - 将支付宝特有字段映射为标准字段
3. **数据清洗** - 处理交易状态、金额格式等
4. **交易分类** - 识别转账、消费、理财等类型

### 1.2 关键属性

| 属性名 | 类型 | 说明 |
|-------|------|------|
| **data_type** | str | 固定为"alipay" |
| **config** | Config | 配置管理器 |
| **column_mapping** | Dict | 字段映射配置 |

### 1.3 标准字段映射

```python
STANDARD_COLUMNS = {
    "本方姓名": "本方姓名",
    "本方账号": "本方账号",          # 支付宝账号
    "交易时间": "交易时间",
    "交易类型": "交易类型",          # 转账/消费/理财
    "交易对方": "对方姓名",
    "对方账号": "对方账号",          # 支付宝账号/商户号
    "收入金额": "收入金额",
    "支出金额": "支出金额",
    "交易状态": "交易状态",          # 交易成功/关闭
    "交易单号": "交易单号",
    "商户订单号": "商户订单号",
    "商品名称": "商品名称",
    "交易分类": "交易分类",          # 餐饮/购物/交通等
    "交易方式": "交易方式",          # 余额/花呗/银行卡
    "备注": "备注"
}
```

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 支付宝账单格式与微信、银行不同
    2. 交易分类更详细（餐饮、购物、交通等）
    3. 包含花呗、余额宝等金融产品
    4. 需要与微信、银行数据统一分析

果（解决方案）：
    1. 独立的字段映射配置
    2. 交易分类映射和标准化
    3. 金融产品识别和标记
    4. 统一的数据结构输出
```

### 2.2 设计决策

| 决策项 | 选择 | 理由 |
|-------|------|------|
| **交易分类** | 保留原始分类 | 支持消费场景分析 |
| **金融产品** | 单独标记 | 便于理财分析 |
| **交易状态** | 过滤"关闭"交易 | 只统计成功交易 |
| **商户识别** | 通过商户订单号 | 区分线上/线下消费 |

---

## 三、怎么做 (How)

### 3.1 类结构

```
AlipayModel
├── 继承: BaseModel
├── 属性: data_type = "alipay"
├── 方法: load(file_path) -> DataFrame
│   ├── 读取CSV（GBK编码）
│   ├── 字段重命名
│   ├── 数据清洗
│   ├── 交易状态过滤
│   └── 交易分类映射
├── 方法: clean(data) -> DataFrame
│   ├── 去除无效记录
│   ├── 金额格式化
│   ├── 日期格式化
│   └── 去除重复
└── 方法: _classify_transaction(row) -> str
    └── 识别交易类型（转账/消费/理财）
```

### 3.2 核心方法实现

**方法：load()**

```python
def load(self, file_path: str) -> DataFrame:
    """加载支付宝账单数据
    
    支付宝账单特点：
    - CSV格式，GBK编码
    - 包含交易分类（餐饮、购物等）
    - 包含交易方式（余额、花呗等）
    """
    # 1. 读取CSV（支付宝使用GBK编码）
    raw_data = pd.read_csv(file_path, encoding='gbk', dtype=str)
    
    # 2. 字段重命名
    column_mapping = self.config.get_column_mapping('alipay')
    data = raw_data.rename(columns=column_mapping)
    
    # 3. 数据清洗
    data = self.clean(data)
    
    # 4. 交易状态过滤（只保留成功交易）
    if '交易状态' in data.columns:
        data = data[data['交易状态'] == '交易成功']
    
    # 5. 交易类型分类
    data['交易类型细分'] = data.apply(self._classify_transaction, axis=1)
    
    return data
```

**方法：clean()**

```python
def clean(self, data: DataFrame) -> DataFrame:
    """清洗支付宝账单数据"""
    # 1. 去除空行
    data = data.dropna(how='all')
    
    # 2. 金额处理
    amount_columns = ['收入金额', '支出金额']
    for col in amount_columns:
        if col in data.columns:
            data[col] = data[col].astype(str).str.replace(',', '')
            data[col] = pd.to_numeric(data[col], errors='coerce')
    
    # 3. 计算交易金额
    data['交易金额'] = data['收入金额'].fillna(0) - data['支出金额'].fillna(0)
    
    # 4. 时间格式化
    if '交易时间' in data.columns:
        data['交易时间'] = pd.to_datetime(data['交易时间'], errors='coerce')
    
    # 5. 去除重复
    if '交易单号' in data.columns:
        data = data.drop_duplicates(subset=['交易单号'])
    
    return data
```

**方法：_classify_transaction()**

```python
def _classify_transaction(self, row: Series) -> str:
    """分类支付宝交易类型
    
    分类规则：
    - 转账：交易类型含"转账"
    - 消费：有商品名称或交易分类
    - 理财：交易类型含"余额宝"、"基金"
    - 还款：交易类型含"还款"
    - 其他：无法识别
    """
    trans_type = str(row.get('交易类型', ''))
    category = str(row.get('交易分类', ''))
    product = str(row.get('商品名称', ''))
    
    if '转账' in trans_type:
        return '转账'
    elif any(keyword in trans_type for keyword in ['余额宝', '基金', '理财']):
        return '理财'
    elif '还款' in trans_type:
        return '还款'
    elif category and category != 'nan':
        return '消费'
    elif product and product != 'nan':
        return '消费'
    else:
        return '其他'
```

### 3.3 数据流图

```
支付宝账单CSV
    │
    ↓
┌─────────────────┐
│ pd.read_csv     │
│ (GBK编码)       │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 字段重命名       │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   clean()       │
│  - 金额格式化    │
│  - 计算交易金额  │
│  - 去重          │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 交易状态过滤     │
│ (只保留成功)     │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 交易类型分类     │
│ (转账/消费/理财) │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 标准化DataFrame │
└─────────────────┘
```

### 3.4 使用示例

```python
from datasource.payment.alipay_model import AlipayModel
from utils.config import Config

config = Config('config.json')
alipay_model = AlipayModel(config)

# 加载数据
data = alipay_model.load('data/alipay_bill.csv')

# 统计
print(f"总记录数: {len(data)}")
print(f"消费记录: {len(data[data['交易类型细分'] == '消费'])}")
print(f"转账记录: {len(data[data['交易类型细分'] == '转账'])}")
print(f"理财记录: {len(data[data['交易类型细分'] == '理财'])}")

# 消费分类分析
consumption = data[data['交易类型细分'] == '消费']
category_stats = consumption.groupby('交易分类')['支出金额'].sum()
print("消费分类统计:")
print(category_stats)
```

---

## 四、参考链接

- [[01-datasource-三阶解构]]
- [[01-1-bank_model-三阶解构]]
- [[01-2-call_model-三阶解构]]
- [[01-3-wechat_model-三阶解构]]