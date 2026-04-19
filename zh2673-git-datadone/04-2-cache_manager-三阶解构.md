# cache_manager - 三阶解构

> 父文档：[[04-utils-三阶解构]]
> 源码位置：`src/utils/cache_manager.py`

---

## 元数据

| 属性 | 值 |
|------|-----|
| **类名** | CacheManager |
| **功能** | 缓存管理 |
| **缓存类型** | 文件缓存 |

---

## 一、是什么 (What)

**本质**：数据缓存和性能优化管理器

**核心职责**：
1. **数据缓存** - 缓存计算结果
2. **热重载** - 检测源码变更自动刷新
3. **超时管理** - 缓存过期处理
4. **性能优化** - 避免重复计算

---

## 二、为什么 (Why)

**存在理由**：
- 避免重复计算，提升性能
- 支持开发时热重载
- 减少报告生成时间
- v0.1.3性能优化核心组件

---

## 三、怎么做 (How)

### 类结构

```
CacheManager
├── 属性: cache_dir: str
├── 属性: timeout: int
├── 方法: get_or_compute(key, compute_func, force_refresh)
├── 方法: invalidate(key)
├── 方法: clear()
└── 方法: check_hot_reload(key)
```

### 核心方法

**get_or_compute()**

```python
def get_or_compute(
    self,
    key: str,
    compute_func: Callable,
    force_refresh: bool = False
) -> Any:
    """获取缓存或计算"""
    cache_path = os.path.join(self.cache_dir, f"{key}.pkl")
    
    # 检查缓存有效性
    if not force_refresh and self._is_cache_valid(cache_path):
        if not self._is_source_modified(key):
            return self._load_cache(cache_path)
    
    # 重新计算并缓存
    result = compute_func()
    self._save_cache(cache_path, result)
    return result
```

---

## 四、参考链接

- [[04-utils-三阶解构]]