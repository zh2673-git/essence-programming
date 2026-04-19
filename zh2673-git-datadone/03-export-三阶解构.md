# export模块 - 三阶解构

> 父文档：[[00-datadone-三阶解构]]
> 模块路径：`src/export/`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **模块名称** | export |
| **所属项目** | datadone |
| **模块职责** | 报告导出层 - 负责生成Excel和Word格式的分析报告 |
| **分析日期** | 2026-04-19 |

---

## 一、是什么 (What)

### 1.1 本质特征

**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：**专业报告生成**
- 因此该模块的本质是：**报告渲染引擎 - 将分析结果格式化为可读的Excel/Word报告**

**核心职责**：
1. **Excel报告生成** - 生成结构化Excel分析报告
2. **Word报告生成** - 生成专业Word分析报告
3. **格式渲染** - 样式、图表、布局处理
4. **数据格式化** - 数值、日期、文本格式化

### 1.2 关键属性（子模块识别）

```
export/
├── __init__.py              → 模块初始化
├── excel_exporter.py        → Excel导出器 ✅ 需深入
├── word_exporter.py         → Word导出器(旧版) ✅ 需深入
└── word_exporter_new.py     → Word导出器(新版) ✅ 需深入
```

| 属性名 | 源码位置 | 说明 | 标记 |
|-------|---------|------|------|
| **ExcelExporter** | `excel_exporter.py` | Excel报告导出器 | ✅ 需深入 → [[03-1-excel_exporter-三阶解构]] |
| **WordExporter** | `word_exporter.py` | Word报告导出器(旧版) | ✅ 需深入 → [[03-2-word_exporter-三阶解构]] |
| **WordExporterNew** | `word_exporter_new.py` | Word报告导出器(新版) | ✅ 需深入 → [[03-3-word_exporter_new-三阶解构]] |

### 1.3 区分属性

| 对比项 | export模块 | 数据导出 | 模板引擎 |
|-------|-----------|---------|---------|
| **职责** | 报告生成+格式渲染 | 仅数据导出 | 仅模板渲染 |
| **输入** | 分析结果对象 | 原始数据 | 模板+数据 |
| **输出** | Excel/Word文件 | CSV/JSON | HTML/PDF |
| **复杂度** | 高（业务逻辑+格式） | 低 | 中 |

**边界结论**：
- ✅ 做：报告生成、格式渲染、图表绘制、样式设置
- ❌ 不做：数据分析、数据清洗、数据库存储

---

## 二、为什么 (Why)

### 2.1 存在理由

```
因（问题）：
    1. 分析结果需要以专业格式呈现给用户
    2. 金融调查需要可打印、可分享的书面报告
    3. Excel适合数据查看和二次分析
    4. Word适合正式报告和文档归档

果（解决方案）：
    1. Excel导出：8个工作表，结构化数据展示
    2. Word导出：按人员分组，专业报告格式
    3. 双版本支持：旧版保持兼容，新版更详细
    4. 性能优化：v0.1.3生成速度提升75-85%
```

### 2.2 设计决策

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **Excel库** | openpyxl | Python操作Excel标准库 |
| **Word库** | python-docx | Python操作Word标准库 |
| **报告版本** | 双版本支持 | 向后兼容+功能升级 |
| **性能优化** | 数据预计算 | 避免重复计算 |
| **进度显示** | 实时进度条 | 提升用户体验 |

---

## 三、怎么做 (How)

### 3.1 模块划分

```
export (报告导出层)
├── ExcelExporter (Excel导出)
│   ├── 属性：分析结果引用
│   ├── 方法：export() - 主导出入口
│   ├── 方法：_create_summary_sheet() - 分析汇总表
│   ├── 方法：_create_frequency_sheet() - 频率表
│   ├── 方法：_create_cross_analysis_sheet() - 综合分析表
│   ├── 方法：_create_raw_data_sheet() - 原始数据表
│   └── 方法：_create_advanced_sheet() - 高级分析表
│
├── WordExporter (Word导出-旧版)
│   ├── 属性：分析结果引用
│   ├── 方法：export() - 主导出入口
│   ├── 方法：_generate_person_analysis() - 个人分析
│   ├── 方法：_generate_cross_analysis() - 交叉分析
│   └── 方法：_generate_key_transactions() - 重点收支
│
└── WordExporterNew (Word导出-新版)
    ├── 属性：分析结果引用
    ├── 方法：export() - 主导出入口
    ├── 方法：_generate_fund_volume() - 资金体量分析
    ├── 方法：_generate_special_funds() - 特殊资金分析
    ├── 方法：_generate_key_income_expense() - 重点收支分析
    └── 方法：_generate_key_persons() - 重点人员分析
```

### 3.2 核心方法设计

**方法：Excel导出**

```python
def export(
    self,
    analysis_results: Dict[str, AnalysisResult],
    output_path: str,
    **kwargs
) -> str:
    """导出Excel报告
    
    利用的属性：
    - analysis_results: 各平台分析结果
    - output_path: 输出文件路径
    
    逻辑流程：
    1. 创建工作簿
    2. 生成各工作表
    3. 设置样式格式
    4. 保存文件
    """
    # 1. 创建工作簿
    wb = Workbook()
    
    # 2. 生成各工作表
    self._create_summary_sheet(wb, analysis_results)
    self._create_frequency_sheet(wb, analysis_results)
    self._create_cross_analysis_sheet(wb, analysis_results)
    self._create_raw_data_sheet(wb, analysis_results)
    self._create_advanced_sheet(wb, analysis_results)
    
    # 3. 设置样式
    self._apply_styles(wb)
    
    # 4. 保存
    wb.save(output_path)
    return output_path
```

**方法：Word导出（新版）**

```python
def export(
    self,
    analysis_results: Dict[str, AnalysisResult],
    output_path: str,
    **kwargs
) -> str:
    """导出Word报告（新版）
    
    利用的属性：
    - analysis_results: 综合分析结果
    - output_path: 输出文件路径
    
    逻辑流程：
    1. 创建文档
    2. 生成资金体量分析
    3. 生成特殊资金分析
    4. 生成重点收支分析
    5. 生成重点人员分析
    6. 保存文档
    """
    # 1. 创建文档
    doc = Document()
    
    # 2. 添加标题
    doc.add_heading('多源数据分析报告', 0)
    
    # 3. 生成各部分内容
    self._generate_fund_volume(doc, analysis_results)
    self._generate_special_funds(doc, analysis_results)
    self._generate_key_income_expense(doc, analysis_results)
    self._generate_key_persons(doc, analysis_results)
    
    # 4. 保存
    doc.save(output_path)
    return output_path
```

### 3.3 报告结构

**Excel报告结构（8个工作表）**：

| 工作表 | 内容 | 数据来源 |
|-------|------|---------|
| 分析汇总 | 各平台汇总数据概览 | 所有分析结果 |
| 账单类频率 | 银行、微信、支付宝频率 | 各平台frequency_analysis |
| 话单类频率 | 通话记录频率 | call_analyzer |
| 综合分析 | 多基准交叉分析 | comprehensive_analyzer |
| 平台原始数据 | 各平台原始数据展示 | datasource原始数据 |
| 高级分析 | 高级分析结果 | advanced_analysis |

**Word报告结构（新版）**：

```
多源数据分析报告
├── 资金体量分析
│   ├── 总资金统计
│   ├── 活跃时间分析
│   ├── 存取现分析
│   └── 大额资金分析
├── 特殊资金分析
│   ├── 纯进出资分析
│   ├── 特殊金额分析
│   └── 特殊日期分析
├── 重点收支分析
│   ├── 工作收支分析
│   ├── 房产车辆收支分析
│   └── 理财收入分析
└── 重点人员分析
    ├── 存取现匹配人员
    ├── 大额资金跟踪人员
    └── 层级区分
```

### 3.4 性能优化设计

**v0.1.3优化策略**：

```python
class PerformanceOptimizedExporter:
    """性能优化导出器"""
    
    def __init__(self):
        self.cache = PerformanceCache()
        self.data_processor = DataProcessor()
    
    def export_with_optimization(self, data, output_path):
        """优化导出流程
        
        优化策略：
        1. 数据预计算 - 提前计算所有需要的数据
        2. 批量处理 - 使用pandas向量化操作
        3. 缓存复用 - 避免重复计算
        4. 进度显示 - 实时反馈生成进度
        """
        # 1. 数据预计算
        precomputed_data = self._precompute_data(data)
        
        # 2. 批量处理
        processed_data = self.data_processor.batch_process(precomputed_data)
        
        # 3. 使用缓存
        cached_results = self.cache.get_or_compute(
            key="report_data",
            compute_func=lambda: self._compute_report_data(processed_data)
        )
        
        # 4. 生成报告（带进度显示）
        with ProgressBar() as progress:
            self._generate_report(cached_results, output_path, progress)
```

### 3.5 接口定义

```python
from typing import Protocol, Dict, Any
from abc import abstractmethod

class BaseExporter(Protocol):
    """导出器基类接口
    
    推导来源：所有导出器需要统一的导出接口
    """
    
    @property
    @abstractmethod
    def export_format(self) -> str:
        """导出格式标识"""
        ...
    
    @abstractmethod
    def export(
        self,
        analysis_results: Dict[str, AnalysisResult],
        output_path: str,
        **kwargs
    ) -> str:
        """导出报告
        
        Args:
            analysis_results: 分析结果字典
            output_path: 输出文件路径
            **kwargs: 附加参数（模板选择、样式配置等）
            
        Returns:
            生成的文件路径
            
        Raises:
            ExportError: 导出失败
        """
        ...
    
    @abstractmethod
    def validate_output_path(self, output_path: str) -> bool:
        """验证输出路径"""
        ...
```

### 3.6 数据流图

```
┌─────────────────────────────────────────────────────────┐
│                    分析结果输入                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ BankResult  │  │ CallResult  │  │ CrossResult │     │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘     │
└─────────┼────────────────┼────────────────┼────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                      Export Router                       │
│  ┌─────────────────┐  ┌─────────────────┐              │
│  │   ExcelExporter │  │   WordExporter  │              │
│  │   .export()     │  │   .export()     │              │
│  └────────┬────────┘  └────────┬────────┘              │
└───────────┼────────────────────┼────────────────────────┘
            │                    │
            ↓                    ↓
┌─────────────────┐      ┌─────────────────┐
│   openpyxl      │      │   python-docx   │
│   Workbook      │      │   Document      │
└────────┬────────┘      └────────┬────────┘
         │                        │
         ↓                        ↓
┌─────────────────┐      ┌─────────────────┐
│  report.xlsx    │      │  report.docx    │
└─────────────────┘      └─────────────────┘
```

---

## 四、子模块详细说明

### 4.1 ExcelExporter (Excel导出器)

**职责**：生成Excel格式的分析报告

**工作表列表**：
1. **分析汇总表** - 各平台数据概览
2. **账单类频率表** - 银行/微信/支付宝频率
3. **话单类频率表** - 通话频率统计
4. **综合分析表** - 交叉分析结果
5. **平台原始数据** - 原始数据展示
6. **高级分析表** - 高级分析结果

**技术特点**：
- 使用openpyxl库
- 支持单元格样式设置
- 支持列宽自动调整
- 支持冻结首行

### 4.2 WordExporter (Word导出器-旧版)

**职责**：生成旧版格式的Word报告

**报告结构**：
- 个人详细分析（按人员分组）
- 综合交叉分析
- 重点收支分析

**技术特点**：
- 使用python-docx库
- 支持段落样式
- 支持表格插入
- 保持向后兼容

### 4.3 WordExporterNew (Word导出器-新版)

**职责**：生成新版格式的Word报告

**报告结构**：
1. **资金体量分析**
   - 总资金统计
   - 活跃时间分析
   - 存取现分析
   - 大额资金分析

2. **特殊资金分析**
   - 纯进出资分析
   - 特殊金额分析
   - 特殊日期分析

3. **重点收支分析**
   - 工作收支分析
   - 房产车辆收支分析
   - 理财收入分析

4. **重点人员分析**
   - 存取现匹配人员
   - 大额资金跟踪人员
   - 层级区分

**技术特点**：
- 更详细的分析内容
- 结构化报告格式
- 支持缓存优化
- 实时进度显示

---

## 五、参考链接

- [[00-datadone-三阶解构]] - 项目级设计
- [[01-datasource-三阶解构]] - 数据源模块
- [[02-analysis-三阶解构]] - 分析模块
- [[99-架构图谱]] - 架构汇总
- [[03-1-excel_exporter-三阶解构]] - Excel导出器深入分析
- [[03-2-word_exporter-三阶解构]] - Word导出器(旧版)深入分析
- [[03-3-word_exporter_new-三阶解构]] - Word导出器(新版)深入分析