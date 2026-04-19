---
name: 本质编程
description: "开始新项目、设计系统架构或重构复杂模块时使用。基于'三阶方法论'（是什么-为什么-怎么做）和'类-属性-方法-路由'模型，从本质逻辑推导设计，构建高质量代码。"
---

# 本质编程 - 基于三阶方法论的代码设计与实现系统

## 核心思想

> **项目 = 类，模块 = 属性，接口 = 契约，路由 = 方法分发**

软件开发的本质：**属性化的静态开发 + 方法化的动态开发**

- **属性**：静态描述、状态、配置（模块/数据/配置）
- **方法**：动态行为、执行逻辑（对属性的编排和利用）
- **路由**：方法的多场景分发机制（同一场景，不同策略）
- **接口**：跨模块契约（各管各事，通过契约协作）

**关键原则**：所有设计必须从**本质逻辑推导**，杜绝随意性。

---

## 何时使用此Skill

- **开始新项目**：从零设计系统架构，明确核心问题
- **重构现有系统**：重新梳理设计思路，优化架构
- **设计复杂模块**：深入分析模块本质，避免过度设计
- **代码审查**：验证实现是否符合设计本质

---

## 三阶方法论：逻辑推导框架

```
┌─────────────────────────────────────────────────────────┐
│                    三阶逻辑推导                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   第1阶：是什么 (What)                                   │
│   ├── 本质特征（内涵）→ 定义系统根本                     │
│   ├── 关键属性（外延）→ 推导出具体组成                   │
│   └── 区分属性（边界）→ 明确不是什么                     │
│              ↓                                          │
│   第2阶：为什么 (Why)                                    │
│   ├── 存在理由 → 从本质推导为什么需要                   │
│   ├── 技术选型 → 从本质推导技术决策                     │
│   └── 架构设计 → 从本质推导系统结构                     │
│              ↓                                          │
│   第3阶：怎么做 (How)                                    │
│   ├── 模块划分 → 从属性推导模块边界                     │
│   ├── 方法设计 → 从属性推导方法逻辑                     │
│   └── 接口定义 → 从协作推导接口契约                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**核心原则**：每一阶的推导必须基于上一阶的结论，形成严密的逻辑链条。

---

## 完整示例：任务管理系统

以下用一个完整的示例贯穿全文，展示从本质定义到代码实现的完整推导过程。

---

## 第1步：是什么 (What) - 定义本质

**核心问题**：这个系统本质上是什么？

### 1.1 本质特征（内涵）

从根本问题出发，推导系统本质：

| 推导要素 | 逻辑分析 | 结论 |
|---------|---------|------|
| **核心问题** | 用户面临什么问题？ | 任务太多，容易遗漏；任务分散，难以追踪 |
| **目标用户** | 为谁解决？ | 个人用户、小型团队 |
| **核心价值** | 解决后带来什么？ | 提高效率、减少遗漏、清晰进度 |
| **一句话定义** | 类比理解 | "这是一个任务管家，帮你记住该做的事并跟踪进度" |

**本质特征推导**：
1. **记忆辅助** - 系统必须记住用户创建的所有任务
2. **状态追踪** - 系统必须能追踪任务从创建到完成的全生命周期
3. **分类组织** - 系统必须支持按不同维度组织任务
4. **提醒通知** - 系统必须在关键时刻提醒用户

### 1.2 关键属性（外延的具体表现）

从本质特征**逻辑推导**出系统必须具备的属性：

```
本质特征 "记忆辅助" 
    → 需要存储任务数据
    → 属性：任务模块（Task）

本质特征 "状态追踪"
    → 需要记录谁在操作
    → 属性：用户模块（User）
    → 需要记录状态变化历史
    → 属性：历史记录模块（History）

本质特征 "分类组织"
    → 需要分类标签
    → 属性：标签模块（Tag）
    → 需要项目/列表分组
    → 属性：项目模块（Project）

本质特征 "提醒通知"
    → 需要定时检查机制
    → 属性：调度模块（Scheduler）
    → 需要通知渠道
    → 属性：通知模块（Notification）
```

**关键属性表**：

| 属性名 | 推导来源 | 说明 | 是否需深入 |
|-------|---------|------|----------|
| **用户模块** | 状态追踪需要身份 | 管理用户信息、权限 | ❌ 见下方 |
| **任务模块** | 记忆辅助的核心 | 任务CRUD、状态管理 | ✅ 需深入 |
| **标签模块** | 分类组织需要 | 标签定义、关联 | ❌ 见下方 |
| **项目模块** | 分类组织需要 | 项目分组、权限 | ❌ 见下方 |
| **历史模块** | 状态追踪需要审计 | 操作记录、变更历史 | ❌ 见下方 |
| **调度模块** | 提醒通知需要触发 | 定时任务、触发器 | ❌ 见下方 |
| **通知模块** | 提醒通知需要渠道 | 多渠道通知发送 | ❌ 见下方 |

### 1.3 区分属性（边界定义）

从本质特征推导**不是什么**：

| 对比项 | 任务管理系统 | 日历应用 | 项目管理工具 |
|-------|-------------|---------|-------------|
| **核心定位** | 个人任务追踪 | 时间安排 | 团队协作 |
| **时间维度** | 截止时间提醒 | 日程安排为主 | 里程碑规划 |
| **协作深度** | 简单分配 | 无 | 复杂工作流 |
| **数据粒度** | 任务级 | 事件级 | 项目级 |

**边界结论**：
- ✅ 做：个人任务管理、简单分配、状态追踪、提醒通知
- ❌ 不做：复杂日历视图、甘特图、审批工作流、资源管理

### 1.4 设计哲学（从本质推导）

基于本质特征推导设计指导思想：

```
本质1：记忆辅助
    → 设计哲学：简单录入，快速记录
    
本质2：状态追踪
    → 设计哲学：状态清晰，进度可视
    
本质3：分类组织
    → 设计哲学：灵活标签，多维筛选
    
本质4：提醒通知
    → 设计哲学：及时提醒，不打扰
```

**核心设计哲学**：**简单优先，快速记录，清晰追踪**

---

## 第2步：为什么 (Why) - 确定架构

**核心问题**：为什么这样设计？

### 2.1 存在理由（因果逻辑）

```
因（问题）：
    1. 用户大脑记忆有限，容易忘记待办事项
    2. 任务散落在各处（邮件、聊天、脑中）
    3. 不知道当前该优先做什么
    4. 任务完成了但没有成就感

果（解决方案）：
    1. 提供统一入口记录所有任务
    2. 支持多维度分类（标签、项目、优先级）
    3. 清晰展示任务状态和进度
    4. 完成时给予反馈（统计、成就）
```

### 2.2 技术栈决策（从本质推导）

| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| **架构模式** | 单体应用 | 本质是个人工具，非高并发，单体足够 |
| **编程语言** | Python | 开发效率高，生态丰富，适合快速迭代 |
| **Web框架** | FastAPI | 异步支持好，自动生成API文档，类型安全 |
| **数据库** | PostgreSQL | 关系型数据适合任务关联，支持JSON扩展 |
| **前端** | React | 组件化适合状态管理，生态成熟 |
| **部署** | Docker | 简单部署，适合个人/小团队 |

### 2.3 参考项目分析

| 项目 | 可借鉴点 | 需避免 |
|-----|---------|-------|
| Todoist | 标签系统、自然语言输入 | 功能过于复杂 |
| Microsoft To Do | 与Office集成 | 生态锁定 |
| Notion | 灵活组织 | 学习成本高 |

---

## 第3步：怎么做 (How) - 实现交付

**核心问题**：如何实现？

### 3.1 模块划分（从属性推导）

基于第1步识别的属性，划分模块：

```
项目（任务管理系统）
├── 用户模块（User）
│   ├── 属性：user_id, username, email, settings
│   └── 方法：注册、登录、更新设置
│
├── 任务模块（Task）✅ 需深入
│   ├── 属性：task_id, title, description, status, priority, due_date
│   └── 方法：创建、查询、更新、删除、状态流转
│
├── 标签模块（Tag）
│   ├── 属性：tag_id, name, color
│   └── 方法：创建、关联、查询
│
├── 项目模块（Project）
│   ├── 属性：project_id, name, description, owner_id
│   └── 方法：创建、归档、统计
│
├── 历史模块（History）
│   ├── 属性：history_id, task_id, action, old_value, new_value, timestamp
│   └── 方法：记录、查询、回滚
│
├── 调度模块（Scheduler）
│   ├── 属性：job_id, task_id, trigger_time, status
│   └── 方法：创建定时任务、触发检查
│
└── 通知模块（Notification）
    ├── 属性：notification_id, user_id, type, content, status
    └── 方法：发送、标记已读、批量处理
```

### 3.2 方法设计（对属性的编排）

**示例：创建任务的完整方法设计**

```python
# 方法：创建任务
# 利用的属性：User, Task, Tag, History, Scheduler
def create_task(
    user_id: str,           # 来自User模块
    title: str,             # Task属性
    description: str,       # Task属性
    priority: str,          # Task属性
    due_date: datetime,     # Task属性
    tag_ids: List[str],     # 关联Tag模块
    project_id: str = None  # 关联Project模块
) -> Task:
    """创建新任务
    
    利用的属性：
    - User：验证用户存在
    - Task：存储任务数据
    - Tag：关联标签
    - History：记录创建操作
    - Scheduler：创建提醒任务
    
    逻辑流程：
    1. 验证用户存在（User.get）
    2. 创建任务记录（Task.create）
    3. 关联标签（Tag.link_to_task）
    4. 记录历史（History.record）
    5. 创建提醒（Scheduler.create_reminder）
    """
    # 1. 获取属性
    user = User.get(user_id)
    if not user:
        raise UserNotFoundError(user_id)
    
    # 2. 创建任务（核心属性操作）
    task = Task.create(
        user_id=user_id,
        title=title,
        description=description,
        priority=priority,
        due_date=due_date,
        project_id=project_id,
        status="todo"
    )
    
    # 3. 关联标签（利用Tag属性）
    if tag_ids:
        Tag.link_to_task(task.task_id, tag_ids)
    
    # 4. 记录历史（利用History属性）
    History.record(
        task_id=task.task_id,
        action="create",
        new_value=task.to_dict()
    )
    
    # 5. 创建提醒（利用Scheduler属性）
    if due_date:
        Scheduler.create_reminder(
            task_id=task.task_id,
            trigger_time=due_date - timedelta(hours=1)
        )
    
    return task
```

### 3.3 路由设计（方法的多场景分发）

**场景：任务状态变更**

同一场景（状态变更），不同策略：

```python
class TaskStatusRouter:
    """任务状态路由 - 同一场景，不同策略"""
    
    def transition(self, action: str, task_id: str, **kwargs) -> Task:
        """统一入口
        
        Args:
            action: 场景标识
                - "start": 开始任务
                - "complete": 完成任务
                - "pause": 暂停任务
                - "resume": 恢复任务
                - "cancel": 取消任务
            task_id: 任务ID
        """
        # 路由分发
        handlers = {
            "start": self._start_task,
            "complete": self._complete_task,
            "pause": self._pause_task,
            "resume": self._resume_task,
            "cancel": self._cancel_task,
        }
        
        handler = handlers.get(action)
        if not handler:
            raise ValueError(f"Unknown action: {action}")
        
        # 执行策略 + 组装结果
        result = handler(task_id, **kwargs)
        return self._format_result(result)
    
    def _start_task(self, task_id: str, **kwargs):
        """策略：开始任务"""
        task = Task.get(task_id)
        task.status = "in_progress"
        task.started_at = datetime.now()
        History.record(task_id, "start", old="todo", new="in_progress")
        return task.save()
    
    def _complete_task(self, task_id: str, **kwargs):
        """策略：完成任务"""
        task = Task.get(task_id)
        task.status = "completed"
        task.completed_at = datetime.now()
        History.record(task_id, "complete", old=task.status, new="completed")
        # 发送完成通知
        Notification.send(task.user_id, f"任务完成: {task.title}")
        return task.save()
    
    def _pause_task(self, task_id: str, reason: str = None, **kwargs):
        """策略：暂停任务"""
        task = Task.get(task_id)
        task.status = "paused"
        task.pause_reason = reason
        History.record(task_id, "pause", old="in_progress", new="paused")
        return task.save()
    
    def _resume_task(self, task_id: str, **kwargs):
        """策略：恢复任务"""
        task = Task.get(task_id)
        task.status = "in_progress"
        History.record(task_id, "resume", old="paused", new="in_progress")
        return task.save()
    
    def _cancel_task(self, task_id: str, reason: str = None, **kwargs):
        """策略：取消任务"""
        task = Task.get(task_id)
        task.status = "cancelled"
        task.cancel_reason = reason
        History.record(task_id, "cancel", old=task.status, new="cancelled")
        return task.save()
```

### 3.4 接口设计（跨模块契约）

**任务服务接口契约**：

```python
from typing import Protocol, List, Optional
from dataclasses import dataclass
from datetime import datetime

@dataclass
class TaskCreateRequest:
    """创建任务请求"""
    title: str
    description: str = ""
    priority: str = "medium"  # low/medium/high
    due_date: Optional[datetime] = None
    tag_ids: List[str] = None
    project_id: Optional[str] = None

@dataclass
class TaskResponse:
    """任务响应"""
    task_id: str
    title: str
    description: str
    status: str
    priority: str
    due_date: Optional[datetime]
    created_at: datetime
    updated_at: datetime

class TaskService(Protocol):
    """任务服务接口契约
    
    职责：提供任务管理能力
    """
    
    def create_task(self, user_id: str, request: TaskCreateRequest) -> TaskResponse:
        """创建新任务
        
        Args:
            user_id: 用户ID
            request: 任务创建参数
            
        Returns:
            创建的任务
            
        Raises:
            UserNotFoundError: 用户不存在
            ValidationError: 参数校验失败
        """
        ...
    
    def get_task(self, task_id: str) -> Optional[TaskResponse]:
        """获取任务详情
        
        Args:
            task_id: 任务ID
            
        Returns:
            任务详情，不存在返回None
        """
        ...
    
    def list_tasks(
        self, 
        user_id: str, 
        status: Optional[str] = None,
        project_id: Optional[str] = None,
        tag_ids: Optional[List[str]] = None
    ) -> List[TaskResponse]:
        """查询任务列表
        
        Args:
            user_id: 用户ID
            status: 状态筛选
            project_id: 项目筛选
            tag_ids: 标签筛选
            
        Returns:
            任务列表
        """
        ...
    
    def transition_status(
        self, 
        task_id: str, 
        action: str, 
        **kwargs
    ) -> TaskResponse:
        """状态流转
        
        Args:
            task_id: 任务ID
            action: 操作类型（start/complete/pause/resume/cancel）
            **kwargs: 附加参数
            
        Returns:
            更新后的任务
        """
        ...
```

### 3.5 统一接口层

```python
def task_api_endpoint(request: Request) -> Response:
    """任务API统一入口
    
    职责：参数收敛 → 路由分发 → 结果组装
    """
    # 1. 参数收敛
    action = request.headers.get("X-Action", "create")
    user_id = request.user.id
    
    # 2. 路由分发
    if action == "create":
        result = TaskService.create_task(user_id, TaskCreateRequest(**request.json))
    elif action == "get":
        result = TaskService.get_task(request.json["task_id"])
    elif action == "list":
        result = TaskService.list_tasks(user_id, **request.json)
    elif action in ["start", "complete", "pause", "resume", "cancel"]:
        result = TaskService.transition_status(
            request.json["task_id"], 
            action, 
            **request.json
        )
    else:
        raise ValueError(f"Unknown action: {action}")
    
    # 3. 结果组装
    return Response(
        code=0,
        message="success",
        data=result
    )
```

---

## 递归执行

### 递归逻辑

当属性标记为"✅ 需深入"时，对该属性递归执行三阶分析：

```
项目（任务管理系统）
├── 用户模块 ❌ → 当前文档详细说明
├── 任务模块 ✅ → 深入分析
│       └── 00-任务模块-三阶解构.md
│           ├── 是什么：任务模块本质是什么？
│           │   ├── 本质特征：任务数据的生命周期管理
│           │   ├── 关键属性：标题、描述、状态、优先级、时间
│           │   └── 区分属性：不做子任务、不做依赖关系
│           ├── 为什么：为什么这样设计？
│           │   └── 技术选型、状态机设计
│           └── 怎么做：如何实现？
│               ├── 模块划分：Task实体、Status枚举
│               ├── 方法设计：CRUD、状态流转
│               └── 接口定义：TaskService
├── 标签模块 ❌ → 当前文档详细说明
└── ...
```

### 判断标准

| 标记 | 条件 | 后续动作 |
|-----|------|---------|
| **✅ 需深入** | 复杂度高/独立性强/复用性高 | 生成子文档，递归执行三阶分析 |
| **❌ 见下方** | 简单属性 | 当前文档详细展开 |

### 停止条件

- 递归深度达到3层
- 当前层级没有"✅ 需深入"的属性
- 属性已经是基础类型（字符串、数字等）

---

## 核心公式

```
┌─────────────────────────────────────────────────────────┐
│                      本质编程公式                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   三阶推导：                                            │
│   是什么（本质特征）→ 为什么（架构决策）→ 怎么做（实现） │
│                                                         │
│   开发模式：                                            │
│   项目开发 = 属性化静态开发 + 方法化动态开发             │
│                                                         │
│   方法扩展：                                            │
│   多场景 = 同核心方法的路由分支                          │
│                                                         │
│   接口本质：                                            │
│   统一接口 = 参数收敛 + 路由分发 + 结果组装              │
│                                                         │
│   系统构建：                                            │
│   递归 = 确定属性 → 方法 → 接口 → 循环往复               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 设计检查清单

### 是什么检查（逻辑严密性）
- [ ] 本质特征是否从核心问题推导？
- [ ] 关键属性是否从本质特征推导？
- [ ] 区分属性是否明确边界？
- [ ] 设计哲学是否从本质推导？

### 为什么检查
- [ ] 存在理由是否形成因果链条？
- [ ] 技术选型是否有推导逻辑？

### 怎么做检查
- [ ] 模块划分是否对应属性？
- [ ] 方法是否明确利用哪些属性？
- [ ] 路由是否覆盖所有场景？
- [ ] 接口契约是否完整？

---

## 代码模板

### 属性定义模板

```python
@dataclass
class ModuleName:
    """模块属性定义
    
    推导来源：本质特征XXX
    职责：描述该模块管理的数据/状态
    """
    id: str
    name: str
    config: dict
    status: str
```

### 方法定义模板

```python
def method_name(param1: Type1, param2: Type2) -> Result:
    """方法说明
    
    推导来源：属性XXX需要YYY操作
    
    利用的属性：
    - 属性A：用途说明
    - 属性B：用途说明
    
    Args:
        param1: 参数说明
        param2: 参数说明
    
    Returns:
        返回说明
    """
    # 1. 获取属性
    attr_a = get_attr_a()
    attr_b = get_attr_b()
    
    # 2. 编排处理
    result = process(attr_a, attr_b, param1, param2)
    
    # 3. 返回结果
    return result
```

### 路由定义模板

```python
class Router:
    """路由分发器
    
    推导来源：同一场景XXX，不同策略YYY
    """
    
    def dispatch(self, action: str, **kwargs) -> Result:
        """统一入口"""
        handlers = {
            "action1": self._handle_action1,
            "action2": self._handle_action2,
        }
        
        handler = handlers.get(action)
        if not handler:
            raise ValueError(f"Unknown action: {action}")
        
        result = handler(**kwargs)
        return self._format(result)
```

### 接口定义模板

```python
from typing import Protocol

class ServiceInterface(Protocol):
    """服务接口契约
    
    推导来源：模块XXX对外提供的能力
    """
    
    def method1(self, param: Type) -> Result:
        """方法说明
        
        Args:
            param: 参数
            
        Returns:
            结果
            
        Raises:
            异常说明
        """
        ...
```

---

## 文档生成规范

### 生成流程

调用此Skill后，**不要直接开始编码**，而是按照以下流程生成设计文档：

```
1. 分析项目需求
   └── 与用户沟通，明确核心问题、目标用户

2. 生成项目级设计文档
   └── docs/00-{项目名}-三阶解构.md

3. 识别需深入的属性（标记✅）
   └── 对每个✅属性生成子模块文档

4. 生成模块级设计文档
   └── docs/01-{模块A}-三阶解构.md
   └── docs/02-{模块B}-三阶解构.md
   └── ...

5. 生成架构汇总文档
   └── docs/99-架构图谱.md

6. 确认设计完成
   └── 与用户确认文档内容后，才开始编码
```

### 文档命名规范

| 文档类型 | 命名格式 | 示例 | 说明 |
|---------|---------|------|------|
| **项目级设计** | `00-{项目名}-三阶解构.md` | `00-任务管理系统-三阶解构.md` | 项目整体分析 |
| **模块级设计** | `{序号}-{模块名}-三阶解构.md` | `01-任务模块-三阶解构.md` | 子模块深入分析 |
| **架构图谱** | `99-架构图谱.md` | `99-架构图谱.md` | 汇总所有模块关系 |

**编号规则**：
- `00`：项目级文档（唯一）
- `01-98`：模块级文档（按依赖顺序或重要性排序）
- `99`：架构汇总文档（唯一）

### 文档存放位置

```
项目根目录/
├── docs/                          # 设计文档目录
│   ├── 00-任务管理系统-三阶解构.md  # 项目级设计
│   ├── 01-任务模块-三阶解构.md     # 模块级设计
│   ├── 02-用户模块-三阶解构.md     # 模块级设计
│   └── 99-架构图谱.md              # 架构汇总
├── src/                           # 源代码目录
├── tests/                         # 测试目录
└── README.md                      # 项目说明
```

### 文档内容模板

#### 项目级文档模板（00-{项目名}-三阶解构.md）

```markdown
# {项目名} - 三阶解构

## 元数据
- **项目名称**：{项目名}
- **分析日期**：{日期}
- **版本**：v1.0

---

## 一、是什么 (What)

### 1.1 本质特征
| 推导要素 | 逻辑分析 | 结论 |
|---------|---------|------|
| 核心问题 | ... | ... |
| 目标用户 | ... | ... |
| 核心价值 | ... | ... |

**本质特征**：
1. ...
2. ...

### 1.2 关键属性
```
本质特征 "..." → 属性：...
```

| 属性名 | 推导来源 | 说明 | 标记 |
|-------|---------|------|------|
| 模块A | ... | ... | ✅ 需深入 → [[01-模块A-三阶解构]] |
| 模块B | ... | ... | ❌ 见下方 |

### 1.3 区分属性
| 对比项 | 本项目 | 竞品A | 竞品B |
|-------|-------|------|------|
| ... | ... | ... | ... |

**边界结论**：
- ✅ 做：...
- ❌ 不做：...

### 1.4 设计哲学
...

---

## 二、为什么 (Why)

### 2.1 存在理由
...

### 2.2 技术栈决策
| 决策项 | 选择 | 推导逻辑 |
|-------|------|---------|
| ... | ... | ... |

---

## 三、怎么做 (How)

### 3.1 模块划分
```
项目结构图
```

### 3.2 关键方法设计
...

### 3.3 路由设计
...

### 3.4 接口定义
...

---

## 四、❌ 属性详细说明

### 模块B
...
```

#### 模块级文档模板（{序号}-{模块名}-三阶解构.md）

```markdown
# {模块名} - 三阶解构

> 父文档：[[00-{项目名}-三阶解构]]

## 元数据
- **模块名称**：{模块名}
- **所属项目**：{项目名}
- **分析日期**：{日期}

---

## 一、是什么 (What)

### 1.1 本质特征
**这个模块本质上是什么？**

从父文档推导：
- 父文档定义该模块负责：...
- 因此该模块的本质是：...

### 1.2 关键属性
...

### 1.3 区分属性
...

---

## 二、为什么 (Why)
...

---

## 三、怎么做 (How)
...

---

## 四、子模块（如有）
| 属性名 | 说明 | 标记 |
|-------|------|------|
| 子模块A | ... | ✅ 需深入 → [[{序号}-子模块A-三阶解构]] |
```

#### 架构图谱文档模板（99-架构图谱.md）

```markdown
# 架构图谱

> 汇总所有模块关系

## 项目概览
- **项目名称**：{项目名}
- **模块数量**：{数量}

## 模块关系图
```
[项目]
├── [模块A] → [[01-模块A-三阶解构]]
├── [模块B] → [[02-模块B-三阶解构]]
└── ...
```

## 数据流图
...

## 接口清单
| 接口名 | 所属模块 | 文档链接 |
|-------|---------|---------|
| ... | ... | ... |

## 文档索引
| 序号 | 文档名 | 说明 |
|-----|-------|------|
| 00 | [[00-{项目名}-三阶解构]] | 项目级设计 |
| 01 | [[01-模块A-三阶解构]] | 模块A设计 |
| ... | ... | ... |
```

### 文档关联关系

```
00-项目-三阶解构.md
├── 引用 → 01-模块A-三阶解构.md（✅标记）
├── 引用 → 02-模块B-三阶解构.md（✅标记）
└── 包含 → 模块C详细说明（❌标记）

01-模块A-三阶解构.md
├── 反向引用 → 00-项目-三阶解构.md
├── 引用 → 01-1-子模块A1-三阶解构.md（✅标记）
└── 包含 → 子模块A2详细说明（❌标记）

99-架构图谱.md
├── 汇总 → 所有模块关系
└── 索引 → 所有文档链接
```

### 使用Obsidian双向链接

文档中使用 `[[文档名]]` 格式创建双向链接：

```markdown
# 示例：在00-任务管理系统-三阶解构.md中

## 1.2 关键属性

| 属性名 | 推导来源 | 说明 | 标记 |
|-------|---------|------|------|
| 任务模块 | 记忆辅助需要 | 核心模块 | ✅ 需深入 → [[01-任务模块-三阶解构]] |
| 用户模块 | 状态追踪需要 | 身份管理 | ❌ 见下方 |
```

### 重要提醒

**调用此Skill后，必须先生成设计文档，确认后再编码。**

**禁止行为**：
- ❌ 不与用户确认需求直接开始编码
- ❌ 不生成设计文档直接写代码
- ❌ 跳过三阶分析直接实现

**正确行为**：
- ✅ 先与用户沟通明确需求
- ✅ 按规范生成所有设计文档
- ✅ 文档中明确标记✅/❌
- ✅ 使用Obsidian双向链接关联文档
- ✅ 确认设计后再开始编码

---

## GitHub项目分析规范

当用户提供GitHub链接要求分析项目时，按照以下规范生成项目笔记。

### 分析流程

```
1. 获取项目信息
   └── 从GitHub API获取项目元数据（star数、语言、README等）

2. 在线分析源码结构
   └── 通过GitHub Web界面浏览项目结构、核心文件

3. 识别模块划分
   └── 从目录结构、文件组织识别模块

4. 执行三阶分析
   └── 是什么：从代码推导本质特征
   └── 为什么：从设计推导架构决策
   └── 怎么做：从源码推导实现细节

5. 生成项目笔记
   └── 保存到Obsidian指定路径

6. 生成架构图谱
   └── 汇总模块关系、数据流
```

### 文档存放路径

**Obsidian Vault路径**：`C:\Users\LX\Documents\Obsidian Vault\github`

```
C:\Users\LX\Documents\Obsidian Vault\github/
├── {org}-{repo}/                          # 项目文件夹
│   ├── 00-{repo}-三阶解构.md               # 项目级分析
│   ├── 01-{模块A}-三阶解构.md              # 模块级分析
│   ├── 02-{模块B}-三阶解构.md              # 模块级分析
│   ├── 99-架构图谱.md                      # 架构汇总
│   └── assets/                             # 图片资源
│       ├── architecture.png
│       └── dataflow.png
├── {org}-{repo}/                           # 另一个项目
│   └── ...
└── 00-GitHub项目索引.md                     # 所有项目索引
```

### 文档命名规范

| 文档类型 | 命名格式 | 示例 |
|---------|---------|------|
| **项目文件夹** | `{org}-{repo}/` | `facebook-react/`, `microsoft-vscode/` |
| **项目级分析** | `00-{repo}-三阶解构.md` | `00-react-三阶解构.md` |
| **模块级分析** | `{序号}-{模块名}-三阶解构.md` | `01-core-三阶解构.md` |
| **架构图谱** | `99-架构图谱.md` | `99-架构图谱.md` |
| **项目索引** | `00-GitHub项目索引.md` | `00-GitHub项目索引.md` |

### GitHub项目分析文档模板

#### 项目级分析模板（00-{repo}-三阶解构.md）

```markdown
# {repo} - 三阶解构

> GitHub: {github_url}
> 分析日期：{date}

## 元数据
- **项目名称**：{repo}
- **所属组织**：{org}
- **GitHub地址**：{github_url}
- **主要语言**：{primary_language}
- **Star数**：{stars}
- **分析日期**：{date}

---

## 一、是什么 (What) - 从源码推导本质

### 1.1 项目概览

**README描述**：
{readme_summary}

**核心功能**（从源码分析）：
- 功能1：...
- 功能2：...

### 1.2 本质特征（从代码推导）

分析入口文件、核心模块，推导本质：

| 推导要素 | 源码证据 | 结论 |
|---------|---------|------|
| **核心问题** | 从README和issue分析 | ... |
| **目标用户** | 从文档和示例分析 | ... |
| **核心价值** | 从功能列表分析 | ... |

**本质特征**：
1. **...** - 证据：{文件路径}
2. **...** - 证据：{文件路径}

### 1.3 关键属性（模块识别）

从源码结构识别模块：

```
{repo}/
├── src/
│   ├── core/           → 属性：核心模块
│   ├── utils/          → 属性：工具模块
│   └── ...
```

| 属性名 | 源码位置 | 说明 | 标记 |
|-------|---------|------|------|
| **core** | `src/core/` | 核心逻辑 | ✅ 需深入 → [[01-core-三阶解构]] |
| **utils** | `src/utils/` | 工具函数 | ❌ 见下方 |

### 1.4 区分属性（与竞品对比）

| 对比项 | {repo} | 竞品A | 竞品B |
|-------|--------|------|------|
| ... | ... | ... | ... |

---

## 二、为什么 (Why) - 从设计推导架构

### 2.1 技术栈决策（从源码分析）

| 决策项 | 选择 | 源码证据 | 推导逻辑 |
|-------|------|---------|---------|
| **语言** | {lang} | `package.json` | ... |
| **框架** | {framework} | `dependencies` | ... |
| **架构** | {pattern} | 目录结构 | ... |

### 2.2 设计亮点（可借鉴点）

1. **...**：{说明}（证据：{文件路径}）
2. **...**：{说明}（证据：{文件路径}）

### 2.3 潜在问题（需避免）

1. **...**：{说明}（证据：{文件路径}）

---

## 三、怎么做 (How) - 从源码推导实现

### 3.1 模块划分

```
项目结构
```

### 3.2 核心方法分析

**方法：{方法名}**
- 位置：`{文件路径}:{行号}`
- 功能：...
- 利用的属性：...

### 3.3 接口定义（从源码提取）

```python
# 从 {文件路径} 提取的接口
class CoreInterface:
    ...
```

### 3.4 路由设计（如有）

...

---

## 四、❌ 属性详细说明

### utils模块
...

---

## 参考链接

- [[99-架构图谱]]
- [[00-GitHub项目索引]]
```

#### 项目索引模板（00-GitHub项目索引.md）

```markdown
# GitHub项目索引

> 所有分析的GitHub项目汇总

## 项目列表

| 序号 | 项目名称 | 组织 | 语言 | Star数 | 分析文档 |
|-----|---------|------|------|--------|---------|
| 1 | react | facebook | JavaScript | 200k+ | [[00-react-三阶解构]] |
| 2 | vscode | microsoft | TypeScript | 150k+ | [[00-vscode-三阶解构]] |
| ... | ... | ... | ... | ... | ... |

## 分类索引

### 前端框架
- [[00-react-三阶解构]]
- [[00-vue-三阶解构]]

### 后端框架
- ...

### 工具库
- ...

## 分析统计

- 总项目数：{count}
- 最近更新：{date}
```

### 分析检查清单

- [ ] 已获取GitHub项目元数据（star、语言、README）
- [ ] 已在线浏览项目目录结构
- [ ] 已识别核心模块和入口文件
- [ ] 已执行三阶分析（是什么/为什么/怎么做）
- [ ] 已生成项目级文档（00-{repo}-三阶解构.md）
- [ ] 已生成模块级文档（✅标记的属性）
- [ ] 已生成架构图谱（99-架构图谱.md）
- [ ] 已更新项目索引（00-GitHub项目索引.md）
- [ ] 文档已保存到Obsidian路径

### 重要提醒

**分析GitHub项目时**：
- ✅ 必须从源码推导结论，不凭空猜测
- ✅ 必须标注源码证据（文件路径、行号）
- ✅ 必须使用Obsidian双向链接关联文档
- ✅ 必须更新项目索引文档
- ❌ 不要只分析README，要深入源码
- ❌ 不要遗漏✅标记的模块分析
