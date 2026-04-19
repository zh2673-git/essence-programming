# call_model - 三阶解构

> 父文档：[[01-datasource-三阶解构]]
> 源码位置：`src/datasource/call_model.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | CallModel |
| **继承** | BaseModel |
| **模块类型** | 数据模型 |
| **数据类型** | 通话记录数据 |
| **分析日期** | 2026-04-19 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个类本质上是什么？**

从父文档推导：
- 父文档定义datasource模块负责：**金融交易数据标准化**
- 因此CallModel的本质是：**通话记录数据的加载器和转换器**

**核心职责**：
1. **加载运营商话单文件** - 读取通话记录Excel/CSV
2. **字段标准化** - 将不同运营商的字段名映射为统一标准
3. **数据清洗** - 处理电话号码格式、时间格式等
4. **通话类型分类** - 识别主叫、被叫、短信等类型

### 1.2 关键属性

| 属性名 | 类型 | 说明 | 来源 |
|-------|------|------|------|
| **data_type** | str | 数据类型标识，固定为"call" | 类定义 |
| **config** | Config | 配置管理器实例 | 构造函数注入 |
| **file_path** | str | 数据文件路径 | load()方法参数 |
| **data** | DataFrame | 加载后的数据 | load()方法生成 |
| **column_mapping** | Dict | 字段映射配置 | config获取 |

### 1.3 标准字段映射

```python
# 话单数据标准字段
STANDARD_COLUMNS = {
    "本方姓名": "本方姓名",          # 机主姓名
    "本方号码": "本方号码",          # 机主电话号码
    "对方姓名": "对方姓名",          # 对方姓名（如有）
    "对方号码": "对方号码",          # 对方电话号码
    "通话时间": "通话时间",          # 通话开始时间
    "通话日期": "通话日期",          # 通话日期
    "通话时长": "通话时长",          # 通话时长（秒）
    "通话类型": "通话类型",          # 主叫/被叫/短信
    "通话地点": "通话地点",          # 通话发生地点
    "基站信息": "基站信息",          # 基站LAC/CI
    "IMEI": "IMEI",                  # 设备IMEI
    "IMSI": "IMSI",                  # SIM卡IMSI
    "归属地": "归属地",              # 号码归属地
    "运营商": "运营商"               # 运营商信息
}
```

### 1.4 区分属性

| 对比项 | CallModel | BankModel | WechatModel |
|-------|-----------|-----------|-------------|
| **数据来源** | 运营商话单 | 银行Excel | 微信账单 |
| **核心字段** | 通话时长、类型 | 金额、余额 | 交易类型、商户 |
| **数据特点** | 有主被叫、有时长 | 有收支方向 | 有交易单号 |
| **清洗重点** | 电话号码格式 | 金额符号 | 交易状态过滤 |

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 不同运营商的话单格式不同（字段名、顺序）
    2. 电话号码格式不统一（+86、区号等）
    3. 通话时长格式多样（秒、分:秒、小时:分:秒）
    4. 需要与银行数据进行关联分析

果（解决方案）：
    1. 通过config.json配置字段映射
    2. 实现电话号码标准化（去除+86、区号等）
    3. 统一通话时长为秒
    4. 标准化字段名便于跨数据源关联
```

### 2.2 设计决策

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **电话号码格式** | 统一为11位 | 便于匹配和关联 |
| **通话时长** | 统一为秒（int） | 便于计算和统计 |
| **通话类型** | 枚举值：主叫/被叫/短信 | 便于分类统计 |
| **时间格式** | datetime | 支持时间序列分析 |

---

## 三、怎么做 (How)

### 3.1 类结构

```
CallModel (话单数据模型)
├── 继承: BaseModel
│   └── 抽象方法: load(), validate(), clean()
│
├── 属性
│   ├── data_type = "call"
│   ├── config: Config
│   └── column_mapping: Dict[str, str]
│
├── 方法: load(file_path: str) -> DataFrame
│   ├── 1. 读取原始数据
│   ├── 2. 字段重命名
│   ├── 3. 数据清洗
│   ├── 4. 数据验证
│   └── 5. 返回标准化DataFrame
│
├── 方法: clean(data: DataFrame) -> DataFrame
│   ├── 1. 去除空行
│   ├── 2. 电话号码标准化
│   ├── 3. 通话时长转换
│   ├── 4. 时间格式转换
│   ├── 5. 去除重复
│   └── 6. 通话类型标准化
│
├── 方法: validate(data: DataFrame) -> bool
│   ├── 1. 检查必需字段
│   ├── 2. 检查电话号码格式
│   └── 3. 检查通话时长
│
├── 方法: _normalize_phone_number(phone: str) -> str
│   └── 标准化电话号码格式
│
└── 方法: _parse_duration(duration: str) -> int
    └── 解析通话时长为秒
```

### 3.2 核心方法实现

**方法：load()**

```python
def load(self, file_path: str) -> DataFrame:
    """加载话单数据
    
    利用的属性：
    - config: 获取字段映射配置
    - column_mapping: 运营商字段到标准字段的映射
    
    逻辑流程：
    1. 根据文件扩展名选择读取方式
    2. 读取原始数据
    3. 字段重命名
    4. 数据清洗
    5. 数据验证
    """
    # 1. 读取原始数据
    if file_path.endswith('.xlsx') or file_path.endswith('.xls'):
        raw_data = pd.read_excel(file_path, dtype=str)
    elif file_path.endswith('.csv'):
        raw_data = pd.read_csv(file_path, dtype=str, encoding='utf-8')
    else:
        raise ValueError(f"不支持的文件格式: {file_path}")
    
    # 2. 字段重命名
    column_mapping = self.config.get_column_mapping('call')
    data = raw_data.rename(columns=column_mapping)
    
    # 3. 数据清洗
    data = self.clean(data)
    
    # 4. 数据验证
    if not self.validate(data):
        raise DataValidationError("话单数据验证失败")
    
    self.data = data
    return data
```

**方法：clean()**

```python
def clean(self, data: DataFrame) -> DataFrame:
    """清洗话单数据
    
    利用的属性：
    - data: 原始DataFrame
    
    逻辑流程：
    1. 去除空行
    2. 电话号码标准化
    3. 通话时长转换
    4. 时间格式转换
    5. 通话类型标准化
    6. 去除重复
    """
    # 1. 去除空行
    data = data.dropna(how='all')
    
    # 2. 电话号码标准化
    phone_columns = ['本方号码', '对方号码']
    for col in phone_columns:
        if col in data.columns:
            data[col] = data[col].apply(self._normalize_phone_number)
    
    # 3. 通话时长转换
    if '通话时长' in data.columns:
        data['通话时长'] = data['通话时长'].apply(self._parse_duration)
    
    # 4. 时间格式转换
    time_columns = ['通话时间', '通话日期']
    for col in time_columns:
        if col in data.columns:
            data[col] = pd.to_datetime(data[col], errors='coerce')
    
    # 5. 通话类型标准化
    if '通话类型' in data.columns:
        data['通话类型'] = data['通话类型'].apply(self._normalize_call_type)
    
    # 6. 去除重复
    data = data.drop_duplicates()
    
    # 7. 过滤必需字段非空记录
    required_cols = ['本方号码', '对方号码']
    data = data.dropna(subset=required_cols)
    
    return data
```

**方法：_normalize_phone_number()**

```python
def _normalize_phone_number(self, phone: str) -> str:
    """标准化电话号码
    
    处理规则：
    1. 去除空格和特殊字符
    2. 去除+86前缀
    3. 去除区号（0开头）
    4. 保留11位手机号
    
    Args:
        phone: 原始电话号码
        
    Returns:
        标准化后的11位手机号
    """
    if pd.isna(phone):
        return None
    
    phone = str(phone).strip()
    
    # 去除所有非数字字符
    phone = re.sub(r'\D', '', phone)
    
    # 去除+86前缀
    if phone.startswith('86') and len(phone) > 11:
        phone = phone[2:]
    
    # 去除区号（0开头）
    if phone.startswith('0') and len(phone) > 11:
        phone = phone[1:]
    
    # 验证长度
    if len(phone) != 11:
        return None
    
    return phone
```

**方法：_parse_duration()**

```python
def _parse_duration(self, duration: str) -> int:
    """解析通话时长为秒
    
    支持格式：
    - 纯数字（秒）
    - 分:秒（如 5:30）
    - 时:分:秒（如 1:30:00）
    
    Args:
        duration: 原始时长字符串
        
    Returns:
        时长（秒）
    """
    if pd.isna(duration):
        return 0
    
    duration = str(duration).strip()
    
    # 尝试直接转换为数字（秒）
    try:
        return int(float(duration))
    except ValueError:
        pass
    
    # 解析时:分:秒格式
    parts = duration.split(':')
    if len(parts) == 2:  # 分:秒
        minutes, seconds = map(int, parts)
        return minutes * 60 + seconds
    elif len(parts) == 3:  # 时:分:秒
        hours, minutes, seconds = map(int, parts)
        return hours * 3600 + minutes * 60 + seconds
    
    return 0
```

**方法：_normalize_call_type()**

```python
def _normalize_call_type(self, call_type: str) -> str:
    """标准化通话类型
    
    映射规则：
    - 主叫/呼出/Outgoing -> "主叫"
    - 被叫/呼入/Incoming -> "被叫"
    - 短信/SMS -> "短信"
    - 其他 -> "其他"
    """
    if pd.isna(call_type):
        return "其他"
    
    call_type = str(call_type).strip()
    
    # 主叫关键词
    if any(keyword in call_type for keyword in ['主叫', '呼出', 'Outgoing', 'outgoing', '去电']):
        return "主叫"
    
    # 被叫关键词
    if any(keyword in call_type for keyword in ['被叫', '呼入', 'Incoming', 'incoming', '来电']):
        return "被叫"
    
    # 短信关键词
    if any(keyword in call_type for keyword in ['短信', 'SMS', 'sms', '短消息']):
        return "短信"
    
    return "其他"
```

### 3.3 数据流图

```
运营商话单文件
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
│  - 号码标准化    │
│  - 时长转换      │
│  - 时间格式化    │
│  - 类型标准化    │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  validate()     │
│  - 字段检查      │
│  - 格式验证      │
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
from datasource.call_model import CallModel
from utils.config import Config

# 1. 初始化配置
config = Config('config.json')

# 2. 创建模型实例
call_model = CallModel(config)

# 3. 加载数据
data = call_model.load('data/call_records.xlsx')

# 4. 查看数据
print(f"记录数: {len(data)}")
print(f"字段: {list(data.columns)}")

# 5. 通话统计
print(f"主叫次数: {len(data[data['通话类型'] == '主叫'])}")
print(f"被叫次数: {len(data[data['通话类型'] == '被叫'])}")
print(f"总通话时长: {data['通话时长'].sum()}秒")

# 6. 高频联系人
top_contacts = data.groupby('对方号码').size().sort_values(ascending=False).head(10)
print("高频联系人:")
print(top_contacts)
```

### 3.5 配置示例

```json
{
  "data_sources": {
    "call": {
      "字段映射": {
        "本方姓名": "本方姓名",
        "本方号码": "本方号码",
        "对方姓名": "对方姓名",
        "对方号码": "对方号码",
        "通话时间": "通话时间",
        "通话日期": "通话日期",
        "通话时长": "通话时长",
        "通话类型": "通话类型",
        "通话地点": "通话地点",
        "基站信息": "基站信息",
        "IMEI": "IMEI",
        "IMSI": "IMSI",
        "归属地": "归属地",
        "运营商": "运营商"
      }
    }
  }
}
```

---

## 四、关键设计点

### 4.1 电话号码匹配策略

```python
# 标准化后便于跨数据源关联
# 银行数据中的电话号码 <-> 话单中的电话号码

# 示例：
# 银行数据: "138****1234" -> 需要模糊匹配
# 话单数据: "13812341234" -> 标准化后

# 关联逻辑
bank_phones = bank_data['对方账号'].apply(extract_phone)
call_phones = call_data['对方号码']

# 模糊匹配
matches = fuzzy_match(bank_phones, call_phones)
```

### 4.2 通话时长统计

```python
# 按通话类型统计
stats = data.groupby('通话类型').agg({
    '通话时长': ['count', 'sum', 'mean']
})

# 按时间段统计
data['小时'] = data['通话时间'].dt.hour
hourly_stats = data.groupby('小时').size()
```

### 4.3 与银行数据关联

```python
# 通过电话号码关联存取现交易
cash_with_call = bank_data[
    (bank_data['现金标志'] == 'cash') &
    (bank_data['对方账号'].isin(call_data['对方号码']))
]

# 分析存取现前后的话单
def analyze_cash_and_call(cash_time, phone_number, time_window=30):
    """分析存取现时间前后的话单"""
    nearby_calls = call_data[
        (call_data['对方号码'] == phone_number) &
        (abs((call_data['通话时间'] - cash_time).dt.total_seconds()) < time_window * 60)
    ]
    return nearby_calls
```

---

## 五、参考链接

- [[01-datasource-三阶解构]] - 父模块设计
- [[00-datadone-三阶解构]] - 项目级设计
- [[99-架构图谱]] - 架构汇总
- [[01-1-bank_model-三阶解构]] - 银行数据模型