# utils模块 - 三阶解构

> 父文档：[[00-datadone-三阶解构]]
> 模块路径：`src/utils/`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **模块名称** | utils |
| **所属项目** | datadone |
| **模块职责** | 工具模块 - 提供通用工具和高级分析功能 |
| **分析日期** | 2026-04-19 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**通用工具和高级分析**
- 因此该模块的本质是：**工具箱 - 提供配置管理、缓存、识别算法等通用能力**

**核心职责**：
1. **配置管理** - 统一读取和管理config.json
2. **缓存管理** - 热重载、数据缓存、性能优化
3. **存取现识别** - 智能识别银行存取现交易
4. **大额资金追踪** - 追踪资金流向路径
5. **关键交易识别** - 识别重要交易模式
6. **异常处理** - 统一异常定义

### 1.2 关键属性（子模块识别）

```
utils/
├── __init__.py              → 模块初始化
├── config.py                → 配置管理 ✅ 需深入
├── cache_manager.py         → 缓存管理器 ✅ 需深入
├── cash_recognition.py      → 存取现识别工具 ✅ 需深入
├── fund_tracking.py         → 大额资金追踪工具 ✅ 需深入
├── key_transactions.py      → 关键交易识别工具 ✅ 需深入
├── advanced_analysis.py     → 高级分析工具 ❌ 见下方
├── exceptions.py            → 异常定义 ❌ 见下方
└── logger.py                → 日志工具 ❌ 见下方
```

| 属性名 | 源码位置 | 说明 | 标记 |
|-------|---------|------|------|
| **Config** | `config.py` | 配置管理器 | ✅ 需深入 → [[04-1-config-三阶解构]] |
| **CacheManager** | `cache_manager.py` | 缓存管理器 | ✅ 需深入 → [[04-2-cache_manager-三阶解构]] |
| **CashRecognition** | `cash_recognition.py` | 存取现识别 | ✅ 需深入 → [[04-3-cash_recognition-三阶解构]] |
| **FundTracking** | `fund_tracking.py` | 大额资金追踪 | ✅ 需深入 → [[04-4-fund_tracking-三阶解构]] |
| **KeyTransactions** | `key_transactions.py` | 关键交易识别 | ✅ 需深入 → [[04-5-key_transactions-三阶解构]] |

### 1.3 区分属性

| 对比项 | utils模块 | 标准库 | 第三方库 |
|-------|----------|--------|---------|
| **定位** | 业务工具 | 通用工具 | 专用工具 |
| **依赖** | 依赖项目配置 | 无依赖 | 独立功能 |
| **复用性** | 项目内复用 | 全局复用 | 特定场景 |
| **业务性** | 强业务相关 | 无业务 | 弱业务 |

**边界结论**：
- ✅ 做：配置管理、缓存、业务识别算法、项目专用工具
- ❌ 不做：通用数据处理（用pandas）、通用文件操作（用标准库）

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 配置分散，难以统一管理
    2. 重复计算影响性能
    3. 存取现识别需要复杂的规则匹配
    4. 资金追踪需要递归算法
    5. 关键交易需要多维度判断

果（解决方案）：
    1. Config：集中管理配置，支持字段映射
    2. CacheManager：热重载、数据缓存
    3. CashRecognition：关键词+排除规则识别
    4. FundTracking：递归追踪+层级分析
    5. KeyTransactions：多条件综合判断
```

### 2.2 设计决策

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **配置格式** | JSON | 易读易修改，支持嵌套 |
| **缓存策略** | 文件缓存+超时机制 | 简单有效，支持热重载 |
| **识别算法** | 关键词匹配+排除规则 | 可配置，准确率可控 |
| **追踪算法** | 递归+层级限制 | 防止无限递归，控制深度 |

---

## 三、怎么做 (How)

### 3.1 模块划分

```
utils (工具模块)
├── Config (配置管理)
│   ├── 属性：config.json内容
│   ├── 方法：load() - 加载配置
│   ├── 方法：get() - 获取配置项
│   ├── 方法：get_column_mapping() - 获取字段映射
│   └── 方法：get_threshold() - 获取阈值参数
│
├── CacheManager (缓存管理)
│   ├── 属性：缓存目录、超时时间
│   ├── 方法：get() - 获取缓存
│   ├── 方法：set() - 设置缓存
│   ├── 方法：invalidate() - 失效缓存
│   ├── 方法：clear() - 清空缓存
│   └── 方法：check_hot_reload() - 热重载检查
│
├── CashRecognition (存取现识别)
│   ├── 属性：存取现关键词、排除关键词
│   ├── 方法：recognize() - 识别存取现
│   ├── 方法：is_deposit() - 判断存款
│   ├── 方法：is_withdraw() - 判断取款
│   └── 方法：classify() - 分类标记
│
├── FundTracking (大额资金追踪)
│   ├── 属性：追踪阈值、最大深度
│   ├── 方法：track() - 追踪资金
│   ├── 方法：find_sources() - 查找资金来源
│   ├── 方法：find_destinations() - 查找资金去向
│   └── 方法：build_flow() - 构建资金流向
│
└── KeyTransactions (关键交易识别)
    ├── 属性：关键交易规则
    ├── 方法：identify() - 识别关键交易
    ├── 方法：classify_income() - 收入分类
    ├── 方法：classify_expense() - 支出分类
    └── 方法：flag_special() - 标记特殊交易
```

### 3.2 核心方法设计

**方法：存取现识别**

```python
def recognize(self, transaction: Dict) -> str:
    """识别交易是否为存取现
    
    利用的属性：
    - deposit_keywords: 存款关键词列表
    - withdraw_keywords: 取款关键词列表
    - exclude_keywords: 排除关键词列表
    
    逻辑流程：
    1. 提取交易摘要
    2. 检查排除关键词
    3. 匹配存款关键词
    4. 匹配取款关键词
    5. 返回分类结果
    """
    summary = transaction.get('summary', '')
    
    # 1. 检查排除关键词
    for keyword in self.exclude_keywords:
        if keyword in summary:
            return 'transfer'  # 转账类，非存取现
    
    # 2. 匹配存款关键词
    for keyword in self.deposit_keywords:
        if keyword in summary:
            return 'deposit'  # 存款
    
    # 3. 匹配取款关键词
    for keyword in self.withdraw_keywords:
        if keyword in summary:
            return 'withdraw'  # 取款
    
    return 'unknown'  # 未知类型
```

**方法：大额资金追踪**

```python
def track(
    self,
    data: DataFrame,
    start_person: str,
    threshold: float = 100000,
    max_depth: int = 3
) -> FundFlowResult:
    """追踪资金流向
    
    利用的属性：
    - data: 交易数据
    - threshold: 金额阈值
    - max_depth: 最大追踪深度
    
    逻辑流程：
    1. 筛选大额交易
    2. 递归追踪资金来源
    3. 递归追踪资金去向
    4. 构建资金流向图
    """
    # 1. 筛选大额交易
    large_transactions = data[data['amount'] >= threshold]
    
    # 2. 递归追踪
    flow_graph = self._build_flow_graph(
        large_transactions,
        start_person,
        current_depth=0,
        max_depth=max_depth
    )
    
    return FundFlowResult(
        start_person=start_person,
        flow_graph=flow_graph,
        total_amount=flow_graph.total_amount,
        involved_persons=flow_graph.nodes
    )

def _build_flow_graph(
    self,
    data: DataFrame,
    person: str,
    current_depth: int,
    max_depth: int
) -> FlowGraph:
    """递归构建资金流向图"""
    if current_depth >= max_depth:
        return FlowGraph()
    
    # 查找与该人员的交易
    person_transactions = data[
        (data['本方姓名'] == person) | 
        (data['对方姓名'] == person)
    ]
    
    graph = FlowGraph()
    for _, tx in person_transactions.iterrows():
        # 添加节点和边
        graph.add_edge(tx['本方姓名'], tx['对方姓名'], tx['amount'])
        
        # 递归追踪对方
        if tx['本方姓名'] == person:
            next_person = tx['对方姓名']
        else:
            next_person = tx['本方姓名']
        
        subgraph = self._build_flow_graph(
            data, next_person, current_depth + 1, max_depth
        )
        graph.merge(subgraph)
    
    return graph
```

**方法：缓存管理 - 热重载**

```python
class CacheManager:
    """缓存管理器 - 支持热重载"""
    
    def __init__(self, cache_dir: str = '.cache', timeout: int = 86400):
        self.cache_dir = cache_dir
        self.timeout = timeout
        self.cache_metadata = {}
    
    def get_or_compute(
        self,
        key: str,
        compute_func: Callable,
        force_refresh: bool = False
    ) -> Any:
        """获取缓存或计算
        
        利用的属性：
        - key: 缓存键
        - compute_func: 计算函数
        - force_refresh: 强制刷新
        
        逻辑流程：
        1. 检查缓存是否存在
        2. 检查缓存是否过期
        3. 检查源代码是否变更
        4. 返回缓存或重新计算
        """
        cache_path = os.path.join(self.cache_dir, f"{key}.pkl")
        
        # 1. 检查缓存存在性和有效性
        if not force_refresh and self._is_cache_valid(cache_path):
            # 2. 检查源代码变更
            if not self._is_source_modified(key):
                return self._load_cache(cache_path)
        
        # 3. 重新计算
        result = compute_func()
        
        # 4. 保存缓存
        self._save_cache(cache_path, result)
        self._update_metadata(key)
        
        return result
    
    def _is_cache_valid(self, cache_path: str) -> bool:
        """检查缓存是否有效（未过期）"""
        if not os.path.exists(cache_path):
            return False
        
        cache_time = os.path.getmtime(cache_path)
        current_time = time.time()
        
        return (current_time - cache_time) < self.timeout
    
    def _is_source_modified(self, key: str) -> bool:
        """检查源代码是否变更"""
        # 获取源代码文件的最后修改时间
        source_mtime = self._get_source_mtime()
        cache_mtime = self.cache_metadata.get(key, {}).get('source_mtime', 0)
        
        return source_mtime > cache_mtime
```

### 3.3 配置管理设计

**Config类设计**：

```python
class Config:
    """配置管理器"""
    
    def __init__(self, config_path: str = 'config.json'):
        self.config_path = config_path
        self.config = self._load_config()
    
    def _load_config(self) -> Dict:
        """加载配置文件"""
        with open(self.config_path, 'r', encoding='utf-8') as f:
            return json.load(f)
    
    def get(self, key: str, default=None) -> Any:
        """获取配置项"""
        keys = key.split('.')
        value = self.config
        for k in keys:
            value = value.get(k, default)
            if value is None:
                return default
        return value
    
    def get_column_mapping(self, data_type: str) -> Dict[str, str]:
        """获取字段映射"""
        return self.get(f'data_sources.{data_type}', {})
    
    def get_threshold(self, threshold_name: str) -> float:
        """获取阈值参数"""
        return self.get(f'analysis.thresholds.{threshold_name}', 0.0)
```

### 3.4 接口定义

```python
from typing import Protocol, Dict, Any, Callable
from abc import abstractmethod

class CashRecognitionInterface(Protocol):
    """存取现识别接口"""
    
    @abstractmethod
    def recognize(self, transaction: Dict) -> str:
        """识别交易类型
        
        Returns:
            'deposit' | 'withdraw' | 'transfer' | 'unknown'
        """
        ...

class FundTrackingInterface(Protocol):
    """资金追踪接口"""
    
    @abstractmethod
    def track(
        self,
        data: DataFrame,
        start_person: str,
        threshold: float,
        max_depth: int
    ) -> FundFlowResult:
        """追踪资金流向"""
        ...

class CacheManagerInterface(Protocol):
    """缓存管理接口"""
    
    @abstractmethod
    def get_or_compute(
        self,
        key: str,
        compute_func: Callable,
        force_refresh: bool = False
    ) -> Any:
        """获取缓存或计算"""
        ...
    
    @abstractmethod
    def invalidate(self, key: str) -> None:
        """使缓存失效"""
        ...
```

### 3.5 数据流图

**存取现识别流程**：

```
交易记录
    │
    ↓
┌─────────────────┐
│ 提取交易摘要     │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 检查排除关键词   │
│ (转账、利息等)   │
└────────┬────────┘
         │
    ┌────┴────┐
    ↓         ↓
┌────────┐  ┌────────┐
│是排除词 │  │否      │
└───┬────┘  └───┬────┘
    │           │
    ↓           ↓
┌────────┐  ┌─────────────────┐
│transfer│  │ 匹配存款关键词   │
└────────┘  └────────┬────────┘
                     │
                ┌────┴────┐
                ↓         ↓
            ┌────────┐  ┌────────┐
            │匹配    │  │不匹配  │
            └───┬────┘  └───┬────┘
                │           │
                ↓           ↓
            ┌────────┐  ┌─────────────────┐
            │deposit │  │ 匹配取款关键词   │
            └────────┘  └────────┬────────┘
                                 │
                            ┌────┴────┐
                            ↓         ↓
                        ┌────────┐  ┌────────┐
                        │匹配    │  │不匹配  │
                        └───┬────┘  └───┬────┘
                            │           │
                            ↓           ↓
                        ┌────────┐  ┌────────┐
                        │withdraw│  │unknown │
                        └────────┘  └────────┘
```

**缓存管理流程**：

```
获取数据请求
      │
      ↓
┌─────────────────┐
│ 检查缓存存在？   │
└────────┬────────┘
         │
    ┌────┴────┐
    ↓         ↓
┌────────┐  ┌────────┐
│不存在  │  │存在    │
└───┬────┘  └───┬────┘
    │           │
    │           ↓
    │    ┌─────────────────┐
    │    │ 检查缓存过期？   │
    │    └────────┬────────┘
    │             │
    │        ┌────┴────┐
    │        ↓         ↓
    │    ┌────────┐  ┌────────┐
    │    │过期    │  │未过期  │
    │    └───┬────┘  └───┬────┘
    │        │           │
    │        │           ↓
    │        │    ┌─────────────────┐
    │        │    │ 检查源码变更？   │
    │        │    └────────┬────────┘
    │        │             │
    │        │        ┌────┴────┐
    │        │        ↓         ↓
    │        │    ┌────────┐  ┌────────┐
    │        │    │已变更  │  │未变更  │
    │        │    └───┬────┘  └───┬────┘
    │        │        │           │
    │        └────────┘           ↓
    │                    ┌─────────────────┐
    │                    │ 返回缓存数据     │
    │                    └─────────────────┘
    │
    └──────────────────────────→
                                 │
                                 ↓
                        ┌─────────────────┐
                        │ 执行计算函数     │
                        └────────┬────────┘
                                 │
                                 ↓
                        ┌─────────────────┐
                        │ 保存到缓存       │
                        └────────┬────────┘
                                 │
                                 ↓
                        ┌─────────────────┐
                        │ 返回计算结果     │
                        └─────────────────┘
```

---

## 四、❌ 属性详细说明

### 4.1 advanced_analysis.py (高级分析工具)

**职责**：提供高级分析功能

**功能**：
- 时间模式分析（工作日/周末分布）
- 金额模式分析（金额分布统计）
- 异常检测算法

### 4.2 exceptions.py (异常定义)

**职责**：定义项目专用异常

**异常类型**：
- `DataValidationError` - 数据验证失败
- `ExportError` - 导出失败
- `ConfigError` - 配置错误
- `CacheError` - 缓存错误

### 4.3 logger.py (日志工具)

**职责**：提供日志记录功能

**功能**：
- 日志级别设置
- 文件日志+控制台日志
- 日志格式化

---

## 五、参考链接

- [[00-datadone-三阶解构]] - 项目级设计
- [[01-datasource-三阶解构]] - 数据源模块
- [[02-analysis-三阶解构]] - 分析模块
- [[03-export-三阶解构]] - 导出模块
- [[99-架构图谱]] - 架构汇总
- [[04-1-config-三阶解构]] - 配置管理深入分析
- [[04-2-cache_manager-三阶解构]] - 缓存管理深入分析
- [[04-3-cash_recognition-三阶解构]] - 存取现识别深入分析
- [[04-4-fund_tracking-三阶解构]] - 大额资金追踪深入分析
- [[04-5-key_transactions-三阶解构]] - 关键交易识别深入分析