# excel_exporter - 三阶解构

> 父文档：[[03-export-三阶解构]]
> 源码位置：`src/export/excel_exporter.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | ExcelExporter |
| **导出格式** | Excel (.xlsx) |
| **库依赖** | openpyxl |

---

## 一、是什么 (What)

**本质**：Excel格式分析报告生成器

**核心职责**：
1. **生成多工作表Excel** - 8个工作表结构化展示
2. **数据格式化** - 数值、日期格式处理
3. **样式设置** - 单元格样式、列宽调整
4. **图表插入** - 统计图表可视化

---

## 二、为什么 (Why)

**存在理由**：
- Excel适合数据查看和二次分析
- 多工作表便于分类查看
- 支持筛选和排序
- 便于分享和打印

---

## 三、怎么做 (How)

### 类结构

```
ExcelExporter
├── 方法: export(analysis_results, output_path) -> str
├── 方法: _create_summary_sheet(wb, results)
├── 方法: _create_frequency_sheet(wb, results)
├── 方法: _create_cross_analysis_sheet(wb, results)
├── 方法: _create_raw_data_sheet(wb, results)
└── 方法: _apply_styles(wb)
```

### 工作表列表

| 工作表 | 内容 |
|-------|------|
| 分析汇总 | 各平台数据概览 |
| 账单类频率 | 银行/微信/支付宝频率 |
| 话单类频率 | 通话记录频率 |
| 综合分析 | 交叉分析结果 |
| 平台原始数据 | 原始数据展示 |
| 高级分析 | 高级分析结果 |

### 核心方法

**export()**

```python
def export(
    self,
    analysis_results: Dict[str, AnalysisResult],
    output_path: str
) -> str:
    """导出Excel报告"""
    wb = Workbook()
    
    # 生成各工作表
    self._create_summary_sheet(wb, analysis_results)
    self._create_frequency_sheet(wb, analysis_results)
    self._create_cross_analysis_sheet(wb, analysis_results)
    self._create_raw_data_sheet(wb, analysis_results)
    
    # 应用样式
    self._apply_styles(wb)
    
    # 保存
    wb.save(output_path)
    return output_path
```

---

## 四、参考链接

- [[03-export-三阶解构]]