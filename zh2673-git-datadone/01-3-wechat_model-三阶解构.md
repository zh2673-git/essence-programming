# wechat_model - 三阶解构

> 父文档：[[01-datasource-三阶解构]]
> 源码位置：`src/datasource/payment/wechat_model.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | WechatModel |
| **继承** | BaseModel |
| **模块类型** | 数据模型 |
| **数据类型** | 微信支付账单数据 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个类本质上是什么？**

WechatModel的本质是：**微信支付账单数据的加载器和转换器**

**核心职责**：
1. **加载微信账单CSV文件** - 读取微信导出的账单数据
2. **字段标准化** - 将微信特有字段映射为标准字段
3. **数据清洗** - 处理交易状态、金额格式等
4. **交易分类** - 识别转账、红包、消费等类型

### 1.2 关键属性

| 属性名 | 类型 | 说明 |
|-------|------|------|
| **data_type** | str | 固定为"wechat" |
| **config** | Config | 配置管理器 |
| **column_mapping** | Dict | 字段映射配置 |

### 1.3 标准字段映射

```python
STANDARD_COLUMNS = {
    "本方姓名": "本方姓名",
    "本方微信号": "本方微信号",
    "交易时间": "交易时间",
    "交易类型": "交易类型",      # 转账/红包/消费
    "交易对方": "对方姓名",
    "对方账号": "对方账号",      # 微信号/商户号
    "收入金额": "收入金额",
    "支出金额": "支出金额",
    "交易状态": "交易状态",      # 已入账/已退款
    "交易单号": "交易单号",
    "商户名称": "商户名称",
    "商品名称": "商品名称",
    "交易方式": "交易方式",      # 零钱/银行卡
    "备注": "备注"
}
```

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 微信账单格式与其他数据源不同
    2. 交易类型多样（转账、红包、消费、理财）
    3. 交易状态需要过滤（只保留已入账）
    4. 商户信息和对方信息需要区分

果（解决方案）：
    1. 专门的字段映射配置
    2. 交易类型识别和分类
    3. 交易状态过滤逻辑
    4. 商户/个人交易分别处理
```

### 2.2 设计决策

| 决策项 | 选择 | 理由 |
|-------|------|------|
| **交易状态过滤** | 只保留"已入账" | 避免重复统计退款 |
| **金额处理** | 分离收入和支出 | 便于统计分析 |
| **商户识别** | 通过商户号前缀 | 区分个人和商户交易 |
| **红包处理** | 单独标记 | 便于红包专项分析 |

---

## 三、怎么做 (How)

### 3.1 类结构

```
WechatModel
├── 继承: BaseModel
├── 属性: data_type = "wechat"
├── 方法: load(file_path) -> DataFrame
│   ├── 读取CSV（微信账单为CSV格式）
│   ├── 字段重命名
│   ├── 数据清洗
│   ├── 交易状态过滤
│   └── 交易类型标记
├── 方法: clean(data) -> DataFrame
│   ├── 去除无效记录
│   ├── 金额格式化
│   ├── 日期格式化
│   └── 去除重复
└── 方法: _classify_transaction(row) -> str
    └── 识别交易类型（转账/红包/消费）
```

### 3.2 核心方法实现

**方法：load()**

```python
def load(self, file_path: str) -> DataFrame:
    """加载微信账单数据
    
    微信账单特点：
    - CSV格式，UTF-8编码
    - 第一行为标题
    - 包含交易状态列
    """
    # 1. 读取CSV
    raw_data = pd.read_csv(file_path, encoding='utf-8', dtype=str)
    
    # 2. 字段重命名
    column_mapping = self.config.get_column_mapping('wechat')
    data = raw_data.rename(columns=column_mapping)
    
    # 3. 数据清洗
    data = self.clean(data)
    
    # 4. 交易状态过滤（只保留已入账）
    if '交易状态' in data.columns:
        data = data[data['交易状态'] == '已入账']
    
    # 5. 交易类型分类
    data['交易类型细分'] = data.apply(self._classify_transaction, axis=1)
    
    return data
```

**方法：clean()**

```python
def clean(self, data: DataFrame) -> DataFrame:
    """清洗微信账单数据"""
    # 1. 去除空行
    data = data.dropna(how='all')
    
    # 2. 金额处理（微信金额带¥符号）
    amount_columns = ['收入金额', '支出金额']
    for col in amount_columns:
        if col in data.columns:
            data[col] = data[col].astype(str).str.replace('¥', '')
            data[col] = data[col].str.replace(',', '')
            data[col] = pd.to_numeric(data[col], errors='coerce')
    
    # 3. 计算交易金额（收入为正，支出为负）
    data['交易金额'] = data['收入金额'].fillna(0) - data['支出金额'].fillna(0)
    
    # 4. 时间格式化
    if '交易时间' in data.columns:
        data['交易时间'] = pd.to_datetime(data['交易时间'], errors='coerce')
    
    # 5. 去除重复（按交易单号）
    if '交易单号' in data.columns:
        data = data.drop_duplicates(subset=['交易单号'])
    
    return data
```

**方法：_classify_transaction()**

```python
def _classify_transaction(self, row: Series) -> str:
    """分类微信交易类型
    
    分类规则：
    - 红包：摘要含"红包"
    - 转账：摘要含"转账"
    - 消费：有商户名称
    - 理财：摘要含"理财"
    - 其他：无法识别
    """
    summary = str(row.get('交易摘要', ''))
    merchant = str(row.get('商户名称', ''))
    
    if '红包' in summary:
        return '红包'
    elif '转账' in summary:
        return '转账'
    elif merchant and merchant != 'nan':
        return '消费'
    elif '理财' in summary or '零钱通' in summary:
        return '理财'
    else:
        return '其他'
```

### 3.3 数据流图

```
微信账单CSV
    │
    ↓
┌─────────────────┐
│ pd.read_csv     │
│ (UTF-8编码)     │
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
│ (只保留已入账)   │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 交易类型分类     │
│ (红包/转账/消费) │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 标准化DataFrame │
└─────────────────┘
```

### 3.4 使用示例

```python
from datasource.payment.wechat_model import WechatModel
from utils.config import Config

config = Config('config.json')
wechat_model = WechatModel(config)

# 加载数据
data = wechat_model.load('data/wechat_bill.csv')

# 统计
print(f"总记录数: {len(data)}")
print(f"红包记录: {len(data[data['交易类型细分'] == '红包'])}")
print(f"转账记录: {len(data[data['交易类型细分'] == '转账'])}")
print(f"消费记录: {len(data[data['交易类型细分'] == '消费'])}")

# 红包分析
red_packets = data[data['交易类型细分'] == '红包']
print(f"红包总收入: {red_packets[red_packets['交易金额'] > 0]['交易金额'].sum()}")
print(f"红包总支出: {abs(red_packets[red_packets['交易金额'] < 0]['交易金额'].sum())}")
```

---

## 四、参考链接

- [[01-datasource-三阶解构]]
- [[01-1-bank_model-三阶解构]]
- [[01-2-call_model-三阶解构]]