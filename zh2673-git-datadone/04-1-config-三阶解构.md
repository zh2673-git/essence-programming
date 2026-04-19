# config - 三阶解构

> 父文档：[[04-utils-三阶解构]]
> 源码位置：`src/utils/config.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | Config |
| **功能** | 配置管理 |
| **配置格式** | JSON |

---

## 一、是什么 (What)

**本质**：项目配置集中管理器

**核心职责**：
1. **加载配置** - 读取config.json
2. **配置访问** - 提供配置项读取接口
3. **字段映射** - 管理数据源字段映射
4. **阈值管理** - 管理分析阈值参数

---

## 二、为什么 (Why)

**存在理由**：
- 集中管理配置，避免硬编码
- 支持不同数据源格式适配
- 便于修改阈值等参数
- 无需修改代码即可适配新格式

---

## 三、怎么做 (How)

### 类结构

```
Config
├── 属性: config_path: str
├── 属性: config: Dict
├── 方法: load() -> Dict
├── 方法: get(key, default) -> Any
├── 方法: get_column_mapping(data_type) -> Dict
└── 方法: get_threshold(name) -> float
```

### 核心方法

**get()**

```python
def get(self, key: str, default=None) -> Any:
    """获取配置项（支持点号路径）"""
    keys = key.split('.')
    value = self.config
    for k in keys:
        value = value.get(k, default)
        if value is None:
            return default
    return value
```

**get_column_mapping()**

```python
def get_column_mapping(self, data_type: str) -> Dict[str, str]:
    """获取字段映射配置"""
    return self.get(f'data_sources.{data_type}.column_mapping', {})
```

---

## 四、参考链接

- [[04-utils-三阶解构]]