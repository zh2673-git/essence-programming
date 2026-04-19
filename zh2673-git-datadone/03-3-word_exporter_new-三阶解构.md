# word_exporter_new - 三阶解构

> 父文档：[[03-export-三阶解构]]
> 源码位置：`src/export/word_exporter_new.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | WordExporterNew |
| **导出格式** | Word (.docx) |
| **库依赖** | python-docx |
| **版本** | 新版（v0.1.2+） |

---

## 一、是什么 (What)

**本质**：Word格式分析报告生成器（新版）

**核心职责**：
1. **结构化报告** - 按主题组织的详细分析
2. **四大分析模块** - 资金体量、特殊资金、重点收支、重点人员
3. **性能优化** - v0.1.3速度提升75-85%
4. **进度显示** - 实时反馈生成进度

---

## 二、为什么 (Why)

**存在理由**：
- 更详细的分析内容
- 结构化报告便于阅读
- 支持缓存优化
- 提升生成速度

---

## 三、怎么做 (How)

### 报告结构

```
多源数据分析报告（新版）
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

### 核心方法

**export()**

```python
def export(
    self,
    analysis_results: Dict[str, AnalysisResult],
    output_path: str,
    progress_callback: Callable = None
) -> str:
    """导出Word报告（新版）"""
    doc = Document()
    
    # 添加标题
    doc.add_heading('多源数据分析报告', 0)
    
    # 生成各部分内容（带进度）
    self._generate_fund_volume(doc, analysis_results, progress_callback)
    self._generate_special_funds(doc, analysis_results, progress_callback)
    self._generate_key_income_expense(doc, analysis_results, progress_callback)
    self._generate_key_persons(doc, analysis_results, progress_callback)
    
    # 保存
    doc.save(output_path)
    return output_path
```

### 性能优化

```python
# v0.1.3优化策略
class PerformanceOptimizedExporter:
    def __init__(self):
        self.cache = PerformanceCache()
    
    def export_with_cache(self, data, output_path):
        # 使用缓存避免重复计算
        cached_data = self.cache.get_or_compute(
            key="report_data",
            compute_func=lambda: self._precompute_data(data)
        )
        return self._generate_report(cached_data, output_path)
```

---

## 四、参考链接

- [[03-export-三阶解构]]
- [[03-2-word_exporter-三阶解构]]