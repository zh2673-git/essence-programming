# datasource模块 - 三阶解构

> 父文档：[[00-datadone-三阶解构]]
> 模块路径：`src/datasource/`

---

## 元数据

| 属性       | 值                        |
| -------- | ------------------------ |
| **模块名称** | datasource               |
| **所属项目** | datadone                 |
| **模块职责** | 数据模型层 - 负责多源数据的加载、清洗、标准化 |
| **分析日期** | 2026-04-19               |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**多源数据整合**
- 因此该模块的本质是：**数据接入层 - 统一处理不同格式、不同来源的金融数据**

**核心职责**：
1. **数据加载** - 从Excel/CSV文件读取原始数据
2. **字段映射** - 根据配置将不同字段名映射到标准字段
3. **数据清洗** - 处理缺失值、格式转换、类型校验
4. **数据标准化** - 统一数据格式，便于后续分析

### 1.2 关键属性（子模块识别）

```
datasource/
├── __init__.py              → 模块初始化
├── base_model.py (继承)     → 数据模型基类
├── bank_model.py            → 银行数据模型 ✅ 需深入
├── call_model.py            → 话单数据模型 ✅ 需深入
└── payment/                 → 支付数据子模块
    ├── __init__.py
    ├── wechat_model.py      → 微信数据模型 ✅ 需深入
    └── alipay_model.py      → 支付宝数据模型 ✅ 需深入
```

| 属性名 | 源码位置 | 说明 | 标记 |
|-------|---------|------|------|
| **BankModel** | `bank_model.py` | 银行交易数据模型 | ✅ 需深入 → [[01-1-bank_model-三阶解构]] |
| **CallModel** | `call_model.py` | 通话记录数据模型 | ✅ 需深入 → [[01-2-call_model-三阶解构]] |
| **WechatModel** | `payment/wechat_model.py` | 微信支付数据模型 | ✅ 需深入 → [[01-3-wechat_model-三阶解构]] |
| **AlipayModel** | `payment/alipay_model.py` | 支付宝数据模型 | ✅ 需深入 → [[01-4-alipay_model-三阶解构]] |

### 1.3 区分属性

| 对比项 | datasource模块 | pandas读取 | ORM模型 |
|-------|----------------|-----------|---------|
| **职责** | 数据+配置+清洗 | 仅数据读取 | 数据库映射 |
| **配置驱动** | ✅ 支持字段映射 | ❌ 不支持 | ❌ 固定映射 |
| **多源支持** | ✅ 银行/话单/支付 | ❌ 需自行处理 | ❌ 需自行处理 |
| **业务逻辑** | ✅ 内置清洗规则 | ❌ 无 | ❌ 无 |

**边界结论**：
- ✅ 做：数据加载、字段映射、数据清洗、标准化
- ❌ 不做：数据分析、报告生成、数据库存储

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 不同数据源字段命名不一致（如"交易日期"vs"日期"vs"时间"）
    2. 数据格式不统一（日期格式、金额格式、编码格式）
    3. 需要统一的数据接口供分析层使用
    4. 需要处理缺失值和异常数据

果（解决方案）：
    1. 配置驱动的字段映射（config.json定义映射关系）
    2. 统一的数据清洗流程（日期解析、金额转换）
    3. 标准化的DataFrame输出
    4. 内置数据验证和异常处理
```

### 2.2 设计决策

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **输出格式** | pandas.DataFrame | 分析层统一使用DataFrame进行处理 |
| **配置方式** | JSON配置文件 | 无需修改代码即可适配新数据源 |
| **继承结构** | BaseModel基类 | 统一接口，子类实现特定逻辑 |
| **错误处理** | 异常抛出+日志记录 | 上层可以捕获并处理数据问题 |

---

## 三、怎么做 (How)

### 3.1 模块划分

```
datasource (数据模型层)
├── BaseModel (抽象基类)
│   ├── 属性：config配置引用
│   ├── 方法：load() - 抽象方法
│   ├── 方法：clean() - 抽象方法
│   └── 方法：validate() - 抽象方法
│
├── BankModel (银行数据)
│   ├── 属性：银行特有字段映射
│   ├── 方法：load() - 读取Excel/CSV
│   ├── 方法：clean() - 日期解析、金额转换
│   ├── 方法：识别存取现() - 关键词匹配
│   └── 方法：收支分类() - 借贷标识判断
│
├── CallModel (话单数据)
│   ├── 属性：话单特有字段映射
│   ├── 方法：load() - 读取通话记录
│   ├── 方法：clean() - 时间解析、时长计算
│   └── 方法：呼叫类型分类() - 主叫/被叫/短信
│
├── WechatModel (微信数据)
│   ├── 属性：微信特有字段映射
│   ├── 方法：load() - 读取微信账单
│   ├── 方法：clean() - 交易类型解析
│   └── 方法：转账识别() - 识别转账/红包
│
└── AlipayModel (支付宝数据)
    ├── 属性：支付宝特有字段映射
    ├── 方法：load() - 读取支付宝账单
    ├── 方法：clean() - 交易状态过滤
    └── 方法：支付场景识别() - 识别消费场景
```

### 3.2 核心方法设计

**方法：数据加载流程**

```python
def load(self, file_path: str) -> DataFrame:
    """加载数据
    
    利用的属性：
    - config: 字段映射配置
    - file_path: 数据文件路径
    
    逻辑流程：
    1. 读取原始数据（pandas.read_excel/csv）
    2. 字段重命名（根据config映射）
    3. 数据清洗（clean方法）
    4. 数据验证（validate方法）
    """
    # 1. 读取原始数据
    raw_data = pd.read_excel(file_path)
    
    # 2. 字段重命名
    column_mapping = self.config.get_column_mapping(self.data_type)
    data = raw_data.rename(columns=column_mapping)
    
    # 3. 数据清洗
    data = self.clean(data)
    
    # 4. 数据验证
    if not self.validate(data):
        raise DataValidationError("数据验证失败")
    
    return data
```

**方法：数据清洗流程**

```python
def clean(self, data: DataFrame) -> DataFrame:
    """数据清洗
    
    利用的属性：
    - data: 原始DataFrame
    - config: 清洗规则配置
    
    逻辑流程：
    1. 日期字段解析
    2. 金额字段转换
    3. 缺失值处理
    4. 异常值过滤
    """
    # 1. 日期解析
    date_column = self.config.get_date_column()
    data[date_column] = pd.to_datetime(data[date_column], format='mixed')
    
    # 2. 金额转换
    amount_column = self.config.get_amount_column()
    data[amount_column] = pd.to_numeric(data[amount_column], errors='coerce')
    
    # 3. 缺失值处理
    data = data.dropna(subset=[date_column, amount_column])
    
    return data
```

### 3.3 字段映射机制

**配置驱动设计**：

```json
{
  "data_sources": {
    "bank": {
      "name_column": "本方姓名",
      "date_column": "交易日期",
      "time_column": "交易时间",
      "amount_column": "交易金额",
      "type_column": "交易类型",
      "opposite_name_column": "对方姓名"
    },
    "call": {
      "name_column": "本方姓名",
      "date_column": "呼叫日期",
      "time_column": "时间",
      "duration_column": "通话时长",
      "opposite_name_column": "对方姓名"
    }
  }
}
```

**映射逻辑**：

```python
class BaseModel:
    def __init__(self, config: Config):
        self.config = config
        self.column_mapping = config.get_column_mapping(self.data_type)
    
    def _map_columns(self, data: DataFrame) -> DataFrame:
        """字段映射"""
        # 反向映射：将配置中的标准名映射到实际列名
        reverse_mapping = {v: k for k, v in self.column_mapping.items()}
        return data.rename(columns=reverse_mapping)
```

### 3.4 接口定义

```python
from typing import Protocol, DataFrame
from abc import abstractmethod

class BaseModel(Protocol):
    """数据模型基类接口
    
    推导来源：所有数据源模型需要统一的加载接口
    """
    
    @property
    @abstractmethod
    def data_type(self) -> str:
        """数据类型标识"""
        ...
    
    @abstractmethod
    def load(self, file_path: str) -> DataFrame:
        """加载数据
        
        Args:
            file_path: 数据文件路径
            
        Returns:
            标准化的DataFrame
            
        Raises:
            FileNotFoundError: 文件不存在
            DataValidationError: 数据验证失败
        """
        ...
    
    @abstractmethod
    def clean(self, data: DataFrame) -> DataFrame:
        """数据清洗
        
        Args:
            data: 原始DataFrame
            
        Returns:
            清洗后的DataFrame
        """
        ...
    
    @abstractmethod
    def validate(self, data: DataFrame) -> bool:
        """数据验证
        
        Args:
            data: 待验证的DataFrame
            
        Returns:
            验证是否通过
        """
        ...
```

### 3.5 数据流图

```
Excel/CSV文件
      │
      ↓
┌─────────────────┐
│   BaseModel     │
│   .load()       │
│                 │
│  1. pandas读取   │
│  2. 字段映射     │
│  3. 调用clean()  │
│  4. 调用validate()│
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   DataFrame     │
│  (标准化数据)    │
│                 │
│ - 统一字段名     │
│ - 正确数据类型   │
│ - 已清洗数据     │
└─────────────────┘
```

---

## 四、子模块详细说明

### 4.1 BankModel (银行数据模型)

**职责**：处理银行交易数据

**特有功能**：
- 存取现识别（关键词匹配）
- 收支分类（借贷标识判断）
- 账户余额处理

**关键字段**：
- 本方姓名、本方账号
- 交易日期、交易时间
- 交易金额、账户余额
- 借贷标识（借/贷）
- 对方姓名、对方账号
- 交易摘要、交易备注

### 4.2 CallModel (话单数据模型)

**职责**：处理通话记录数据

**特有功能**：
- 通话时长计算
- 呼叫类型分类（主叫/被叫/短信）
- 通话地点解析

**关键字段**：
- 本方姓名、本方号码
- 呼叫日期、时间
- 通话时长
- 对方姓名、对方号码
- 呼叫类型（主叫/被叫）

### 4.3 WechatModel (微信数据模型)

**职责**：处理微信支付数据

**特有功能**：
- 交易类型识别（转账/红包/消费）
- 微信账号解析
- 商户名称处理

**关键字段**：
- 本方姓名、本方微信账号
- 交易日期、交易时间
- 交易金额、账户余额
- 交易类型（入/出）
- 对方姓名、对方微信账号
- 交易说明、商户名称

### 4.4 AlipayModel (支付宝数据模型)

**职责**：处理支付宝交易数据

**特有功能**：
- 交易状态过滤（成功/失败）
- 支付方式识别
- 商品名称解析

**关键字段**：
- 本方姓名、本方账号
- 交易日期、交易时间
- 交易金额
- 交易状态
- 对方姓名、对方账号
- 交易备注、商品名称

---

## 五、参考链接

- [[00-datadone-三阶解构]] - 项目级设计
- [[99-架构图谱]] - 架构汇总
- [[01-1-bank_model-三阶解构]] - 银行数据模型深入分析
- [[01-2-call_model-三阶解构]] - 话单数据模型深入分析
- [[01-3-wechat_model-三阶解构]] - 微信数据模型深入分析
- [[01-4-alipay_model-三阶解构]] - 支付宝数据模型深入分析