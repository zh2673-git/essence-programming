# bank_model - 三阶解构

> 父文档：[[01-datasource-三阶解构]]
> 源码位置：`src/datasource/bank_model.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | BankModel |
| **继承** | BaseModel |
| **模块类型** | 数据模型 |
| **数据类型** | 银行交易数据 |
| **分析日期** | 2026-04-19 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个类本质上是什么？**

从父文档推导：
- 父文档定义datasource模块负责：**金融交易数据标准化**
- 因此BankModel的本质是：**银行交易数据的加载器和转换器**

**核心职责**：
1. **加载银行Excel/CSV文件** - 读取原始银行交易数据
2. **字段标准化** - 将不同银行的字段名映射为统一标准
3. **数据清洗** - 处理缺失值、格式转换、异常值处理
4. **交易分类** - 识别存取现、转账等交易类型

### 1.2 关键属性

| 属性名 | 类型 | 说明 | 来源 |
|-------|------|------|------|
| **data_type** | str | 数据类型标识，固定为"bank" | 类定义 |
| **config** | Config | 配置管理器实例 | 构造函数注入 |
| **file_path** | str | 数据文件路径 | load()方法参数 |
| **data** | DataFrame | 加载后的数据 | load()方法生成 |
| **column_mapping** | Dict | 字段映射配置 | config获取 |

### 1.3 标准字段映射

```python
# 银行数据标准字段
STANDARD_COLUMNS = {
    "本方姓名": "本方姓名",      # 账户持有人姓名
    "本方卡号": "本方卡号",      # 本方银行卡号
    "本方账号": "本方账号",      # 本方银行账号
    "交易时间": "交易时间",      # 交易发生时间
    "交易日期": "交易日期",      # 交易日期
    "交易金额": "交易金额",      # 交易金额（带正负号）
    "收入金额": "收入金额",      # 收入金额
    "支出金额": "支出金额",      # 支出金额
    "账户余额": "账户余额",      # 交易后余额
    "对方姓名": "对方姓名",      # 交易对方姓名
    "对方账号": "对方账号",      # 交易对方账号
    "对方银行": "对方银行",      # 交易对方银行
    "交易摘要": "交易摘要",      # 交易描述/备注
    "交易类型": "交易类型",      # 交易类型分类
    "交易渠道": "交易渠道",      # 交易渠道（网银、ATM等）
    "交易场所": "交易场所",      # 交易发生地点
    "交易状态": "交易状态",      # 交易状态（成功、失败等）
    "币种": "币种",              # 交易币种
    "现金标志": "现金标志"       # 是否现金交易
}
```

### 1.4 区分属性

| 对比项 | BankModel | CallModel | WechatModel |
|-------|-----------|-----------|-------------|
| **数据来源** | 银行Excel/CSV | 运营商话单 | 微信账单 |
| **核心字段** | 金额、余额、对方信息 | 通话时长、类型 | 交易类型、商户 |
| **数据特点** | 有收支方向、有余额 | 有主被叫、有时长 | 有交易单号、有商户 |
| **清洗重点** | 金额符号、日期格式 | 电话号码格式 | 交易状态过滤 |

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 不同银行导出的数据格式不同（字段名、顺序、格式）
    2. 银行数据包含大量冗余信息需要清洗
    3. 需要统一的数据结构供后续分析使用
    4. 存取现交易需要特殊识别和处理

果（解决方案）：
    1. 通过config.json配置字段映射，适配不同银行格式
    2. 实现数据清洗逻辑：去重、处理缺失值、格式标准化
    3. 继承BaseModel，实现统一的load()接口
    4. 集成CashRecognition工具识别存取现
```

### 2.2 设计决策

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **继承基类** | BaseModel | 统一数据源接口，支持多态 |
| **配置驱动** | config.json字段映射 | 无需修改代码即可适配新银行格式 |
| **数据类型** | pandas DataFrame | 便于后续分析和处理 |
| **金额处理** | 分离收入和支出 | 便于统计分析和可视化 |
| **日期格式** | 统一为datetime | 支持时间序列分析 |

---

## 三、怎么做 (How)

### 3.1 类结构

```
BankModel (银行数据模型)
├── 继承: BaseModel
│   └── 抽象方法: load(), validate(), clean()
│
├── 属性
│   ├── data_type = "bank"
│   ├── config: Config
│   └── column_mapping: Dict[str, str]
│
├── 方法: load(file_path: str) -> DataFrame
│   ├── 1. 读取原始数据 (pd.read_excel/csv)
│   ├── 2. 字段重命名 (根据column_mapping)
│   ├── 3. 数据清洗 (clean方法)
│   ├── 4. 数据验证 (validate方法)
│   ├── 5. 存取现识别 (CashRecognition)
│   └── 6. 返回标准化DataFrame
│
├── 方法: clean(data: DataFrame) -> DataFrame
│   ├── 1. 去除空行空列
│   ├── 2. 处理金额格式 (去除逗号、货币符号)
│   ├── 3. 处理日期格式 (统一为datetime)
│   ├── 4. 处理缺失值
│   └── 5. 去除重复记录
│
├── 方法: validate(data: DataFrame) -> bool
│   ├── 1. 检查必需字段
│   ├── 2. 检查数据类型
│   ├── 3. 检查数值范围
│   └── 4. 返回验证结果
│
└── 方法: _recognize_cash_transactions(data: DataFrame) -> DataFrame
    ├── 1. 提取交易摘要
    ├── 2. 调用CashRecognition
    ├── 3. 添加"现金标志"字段
    └── 4. 返回标记后的数据
```

### 3.2 核心方法实现

**方法：load()**

```python
def load(self, file_path: str) -> DataFrame:
    """加载银行数据
    
    利用的属性：
    - config: 获取字段映射配置
    - column_mapping: 银行字段到标准字段的映射
    
    逻辑流程：
    1. 根据文件扩展名选择读取方式
    2. 读取原始数据
    3. 字段重命名
    4. 数据清洗
    5. 数据验证
    6. 存取现识别
    """
    # 1. 确定文件类型并读取
    if file_path.endswith('.xlsx') or file_path.endswith('.xls'):
        raw_data = pd.read_excel(file_path, dtype=str)
    elif file_path.endswith('.csv'):
        raw_data = pd.read_csv(file_path, dtype=str, encoding='utf-8')
    else:
        raise ValueError(f"不支持的文件格式: {file_path}")
    
    # 2. 字段重命名
    column_mapping = self.config.get_column_mapping('bank')
    data = raw_data.rename(columns=column_mapping)
    
    # 3. 数据清洗
    data = self.clean(data)
    
    # 4. 数据验证
    if not self.validate(data):
        raise DataValidationError("银行数据验证失败")
    
    # 5. 存取现识别
    data = self._recognize_cash_transactions(data)
    
    self.data = data
    return data
```

**方法：clean()**

```python
def clean(self, data: DataFrame) -> DataFrame:
    """清洗银行数据
    
    利用的属性：
    - data: 原始DataFrame
    
    逻辑流程：
    1. 去除完全空行
    2. 处理金额字段
    3. 处理日期字段
    4. 处理缺失值
    5. 去除重复
    """
    # 1. 去除空行
    data = data.dropna(how='all')
    
    # 2. 处理金额字段
    amount_columns = ['交易金额', '收入金额', '支出金额', '账户余额']
    for col in amount_columns:
        if col in data.columns:
            # 去除逗号和货币符号
            data[col] = data[col].astype(str).str.replace(',', '')
            data[col] = data[col].str.replace('￥', '').str.replace('$', '')
            # 转换为数值
            data[col] = pd.to_numeric(data[col], errors='coerce')
    
    # 3. 处理日期字段
    date_columns = ['交易时间', '交易日期']
    for col in date_columns:
        if col in data.columns:
            data[col] = pd.to_datetime(data[col], errors='coerce')
    
    # 4. 处理缺失值（保留关键字段非空的记录）
    required_cols = ['本方姓名', '交易金额']
    data = data.dropna(subset=required_cols)
    
    # 5. 去除重复记录
    data = data.drop_duplicates()
    
    return data
```

**方法：validate()**

```python
def validate(self, data: DataFrame) -> bool:
    """验证银行数据
    
    验证规则：
    1. 必需字段存在
    2. 数据类型正确
    3. 数值范围合理
    """
    # 1. 检查必需字段
    required_columns = ['本方姓名', '交易金额']
    for col in required_columns:
        if col not in data.columns:
            logger.error(f"缺少必需字段: {col}")
            return False
    
    # 2. 检查数据类型
    if not pd.api.types.is_numeric_dtype(data['交易金额']):
        logger.error("交易金额字段类型错误")
        return False
    
    # 3. 检查数值范围
    if data['交易金额'].isna().all():
        logger.error("交易金额全部为空")
        return False
    
    return True
```

**方法：_recognize_cash_transactions()**

```python
def _recognize_cash_transactions(self, data: DataFrame) -> DataFrame:
    """识别存取现交易
    
    利用的属性：
    - data: 清洗后的DataFrame
    - CashRecognition: 存取现识别工具
    
    逻辑流程：
    1. 初始化识别器
    2. 遍历交易记录
    3. 识别存取现类型
    4. 添加标记字段
    """
    from utils.cash_recognition import CashRecognition
    
    recognizer = CashRecognition(self.config)
    
    # 添加现金标志字段
    data['现金标志'] = 'transfer'  # 默认为转账
    data['存取现类型'] = None
    
    for idx, row in data.iterrows():
        transaction = {
            'summary': row.get('交易摘要', ''),
            'amount': row.get('交易金额', 0),
            'channel': row.get('交易渠道', '')
        }
        
        result = recognizer.recognize(transaction)
        
        if result in ['deposit', 'withdraw']:
            data.at[idx, '现金标志'] = 'cash'
            data.at[idx, '存取现类型'] = result
    
    return data
```

### 3.3 数据流图

```
银行Excel/CSV文件
      │
      ↓
┌─────────────────┐
│ pd.read_excel/  │
│    read_csv     │
│  (原始DataFrame) │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 字段重命名       │
│ (column_mapping)│
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   clean()       │
│  - 金额格式化    │
│  - 日期格式化    │
│  - 去重          │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  validate()     │
│  - 字段检查      │
│  - 类型检查      │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 存取现识别       │
│ CashRecognition │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 标准化DataFrame │
│ (返回给调用者)   │
└─────────────────┘
```

### 3.4 使用示例

```python
from datasource.bank_model import BankModel
from utils.config import Config

# 1. 初始化配置
config = Config('config.json')

# 2. 创建模型实例
bank_model = BankModel(config)

# 3. 加载数据
data = bank_model.load('data/bank_transactions.xlsx')

# 4. 查看数据
print(f"记录数: {len(data)}")
print(f"字段: {list(data.columns)}")
print(f"存取现记录数: {len(data[data['现金标志'] == 'cash'])}")

# 5. 获取存取现明细
cash_transactions = data[data['现金标志'] == 'cash']
print(cash_transactions[['交易时间', '交易金额', '交易摘要', '存取现类型']])
```

### 3.5 配置示例

```json
{
  "data_sources": {
    "bank": {
      "字段映射": {
        "本方姓名": "本方姓名",
        "本方卡号": "本方卡号",
        "本方账号": "本方账号",
        "交易时间": "交易时间",
        "交易日期": "交易日期",
        "交易金额": "交易金额",
        "收入金额": "收入金额",
        "支出金额": "支出金额",
        "账户余额": "账户余额",
        "对方姓名": "对方姓名",
        "对方账号": "对方账号",
        "对方银行": "对方银行",
        "交易摘要": "交易摘要",
        "交易类型": "交易类型",
        "交易渠道": "交易渠道",
        "交易场所": "交易场所",
        "交易状态": "交易状态",
        "币种": "币种"
      }
    }
  }
}
```

---

## 四、关键设计点

### 4.1 字段映射的灵活性

通过config.json配置字段映射，可以适配不同银行的导出格式：

```python
# 工商银行映射示例
{
    "本方户名": "本方姓名",
    "本方账号": "本方账号",
    "交易日期": "交易日期",
    "借方发生额": "支出金额",
    "贷方发生额": "收入金额",
    "余额": "账户余额",
    "对方户名": "对方姓名",
    "对方账号": "对方账号",
    "摘要": "交易摘要"
}

# 建设银行映射示例
{
    "客户名称": "本方姓名",
    "账号": "本方账号",
    "交易时间": "交易时间",
    "支出": "支出金额",
    "收入": "收入金额",
    "账户余额": "账户余额",
    "对方户名": "对方姓名",
    "对方账号": "对方账号",
    "用途": "交易摘要"
}
```

### 4.2 金额处理策略

```python
# 策略1：同时存在收入和支出字段
data['交易金额'] = data['收入金额'].fillna(0) - data['支出金额'].fillna(0)

# 策略2：只有交易金额字段（带正负号）
# 直接使用，无需处理

# 策略3：金额和方向分开
data['交易金额'] = data.apply(
    lambda row: row['金额'] if row['方向'] == '收入' else -row['金额'],
    axis=1
)
```

### 4.3 存取现识别集成

```python
# 在clean()之后自动调用存取现识别
data = self._recognize_cash_transactions(data)

# 识别结果添加到DataFrame
# - 现金标志: 'cash' | 'transfer'
# - 存取现类型: 'deposit' | 'withdraw' | None
```

---

## 五、参考链接

- [[01-datasource-三阶解构]] - 父模块设计
- [[00-datadone-三阶解构]] - 项目级设计
- [[99-架构图谱]] - 架构汇总
- [[04-3-cash_recognition-三阶解构]] - 存取现识别工具