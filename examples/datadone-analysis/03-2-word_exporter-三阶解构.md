# word_exporter - 三阶解构

> 父文档：[[03-export-三阶解构]]
> 源码位置：`src/export/word_exporter.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | WordExporter |
| **导出格式** | Word (.docx) |
| **库依赖** | python-docx |
| **版本** | 旧版（兼容） |

---

## 一、是什么 (What)

**本质**：Word格式分析报告生成器（旧版）

**核心职责**：
1. **生成Word报告** - 按人员分组的详细分析
2. **段落格式化** - 标题、正文样式
3. **表格插入** - 统计数据表格
4. **保持兼容** - 支持旧版报告格式

---

## 二、为什么 (Why)

**存在理由**：
- 向后兼容旧版报告
- 按人员分组的详细分析
- 适合打印和归档

---

## 三、怎么做 (How)

### 报告结构

```
多源数据分析报告（旧版）
├── 个人详细分析
│   ├── 人员A分析
│   ├── 人员B分析
│   └── ...
├── 综合交叉分析
└── 重点收支分析
```

### 核心方法

**export()**

```python
def export(
    self,
    analysis_results: Dict[str, AnalysisResult],
    output_path: str
) -> str:
    """导出Word报告（旧版）"""
    doc = Document()
    
    # 添加标题
    doc.add_heading('多源数据分析报告', 0)
    
    # 生成各部分内容
    self._generate_person_analysis(doc, analysis_results)
    self._generate_cross_analysis(doc, analysis_results)
    self._generate_key_transactions(doc, analysis_results)
    
    # 保存
    doc.save(output_path)
    return output_path
```

---

## 四、参考链接

- [[03-export-三阶解构]]
- [[03-3-word_exporter_new-三阶解构]]