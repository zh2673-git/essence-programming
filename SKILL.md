---
name: 本质编程
description: "基于'三阶方法论'（是什么-为什么-怎么做）和'类-属性-方法-路由'模型，支持新项目开发与现有项目解析两种场景，从本质逻辑推导设计或逆向推导理解系统。"
---

# 本质编程 - 基于三阶方法论的代码设计与分析系统

## 核心思想

> **项目 = 类，模块 = 属性，接口 = 契约，路由 = 方法分发**

软件开发的本质：**属性化的静态结构 + 方法化的动态行为**

- **属性**：静态描述、状态、配置（模块/数据/配置）
- **方法**：动态行为、执行逻辑（对属性的编排和利用）
- **路由**：方法的多场景分发机制（同一场景，不同策略）
- **接口**：跨模块契约（各管各事，通过契约协作）

**关键原则**：所有分析或设计必须从**本质逻辑推导**，杜绝随意性。

---

## 两大使用场景

本Skill支持两种截然不同的使用场景，执行流程和输出目标完全不同：

| 场景 | 方向 | 起点 | 终点 | 核心目标 |
|-----|------|------|------|---------|
| **场景A：开发新项目** | 正向设计 | 问题/需求 | 可运行代码 | 从零构建系统 |
| **场景B：解析现有项目** | 逆向分析 | 现有代码 | 理解文档 | 理解已有系统 |

### 场景对比

```
┌─────────────────────────────────────────────────────────────┐
│                      三阶方法论                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   场景A：开发新项目（正向设计）                              │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐                │
│   │ 是什么  │ →  │ 为什么  │ →  │ 怎么做  │                │
│   │ (定义)  │    │ (决策)  │    │ (实现)  │                │
│   └─────────┘    └─────────┘    └─────────┘                │
│        ↑                              ↓                     │
│     问题/需求                      可运行代码               │
│                                                             │
│   场景B：解析项目（逆向分析）                                │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐                │
│   │ 是什么  │ ←  │ 为什么  │ ←  │ 怎么做  │                │
│   │ (推导)  │    │ (推导)  │    │ (观察)  │                │
│   └─────────┘    └─────────┘    └─────────┘                │
│        ↓                              ↑                     │
│     本质理解                      现有代码                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 核心差异

| 维度 | 场景A：开发新项目 | 场景B：解析项目 |
|-----|------------------|----------------|
| **思维方向** | 正向推导：从需求到实现 | 逆向推导：从代码到本质 |
| **三阶执行** | 是什么→为什么→怎么做 | 怎么做→为什么→是什么 |
| **属性来源** | 从本质特征推导属性 | 从代码结构识别属性 |
| **方法设计** | 设计方法编排属性 | 分析方法如何利用属性 |
| **输出目标** | 生成可运行代码 | 生成理解文档 |
| **文档位置** | 项目代码库内 | Obsidian Vault |

---

## 场景A：开发新项目（正向设计）

### 适用场景

- 从零开始构建新系统
- 设计复杂模块的架构
- 重构现有系统前的重新设计

### 执行流程

```
第1步：是什么 (What) - 定义本质
    ├── 1.1 本质特征（内涵）→ 从问题推导系统根本
    ├── 1.2 关键属性（外延）→ 从本质推导具体组成
    ├── 1.3 区分属性（边界）→ 明确不是什么
    └── 1.4 设计哲学 → 指导思想
           ↓
第2步：为什么 (Why) - 确定架构
    ├── 2.1 存在理由 → 从本质推导为什么需要
    ├── 2.2 技术选型 → 从本质推导技术决策
    └── 2.3 架构设计 → 从本质推导系统结构
           ↓
第3步：怎么做 (How) - 实现交付
    ├── 3.1 模块划分 → 从属性推导模块边界
    ├── 3.2 方法设计 → 从属性推导方法逻辑
    ├── 3.3 路由设计 → 方法的多场景分发
    └── 3.4 接口定义 → 从协作推导接口契约
           ↓
    输出：可运行的代码实现
```

### 完整示例

见下文【第1-3步完整示例：任务管理系统】

---

## 场景B：解析现有项目（逆向分析）

### 适用场景

- 学习优秀开源项目
- 接手遗留系统需要快速理解
- 进行代码审查或架构评估

### 执行流程

```
第1步：怎么做 (How) - 观察实现
    ├── 1.1 获取项目信息 → GitHub元数据、README
    ├── 1.2 浏览源码结构 → 目录组织、文件分布
    ├── 1.3 识别模块划分 → 从代码结构识别属性
    │                    └── 标记 ✅需深入 的模块
    └── 1.4 分析方法实现 → 方法如何利用属性
           ↓
第2步：为什么 (Why) - 推导设计
    ├── 2.1 技术选型分析 → 从源码推导决策原因
    ├── 2.2 架构设计推导 → 从结构推导设计思想
    └── 2.3 设计亮点识别 → 可借鉴的模式
           ↓
第3步：是什么 (What) - 推导本质
    ├── 3.1 本质特征推导 → 从代码推导系统根本
    ├── 3.2 关键属性确认 → 验证识别的属性
    └── 3.3 区分属性界定 → 明确系统边界
           ↓
    输出：项目理解文档（保存到Obsidian）
           ↓
第4步：递归深入 - 分析标记模块（自动执行）
    ├── 对每个 ✅需深入 的模块
    │   ├── 4.1 深入源码分析该模块
    │   ├── 4.2 执行逆向三阶分析
    │   └── 4.3 生成模块级文档（01-{模块}-三阶解构.md）
    └── 直到没有 ✅需深入 的模块或达到深度限制
           ↓
    最终输出：完整的项目文档集合
```

### 分析检查清单

**项目级分析（必须完成）**
- [ ] 已获取GitHub项目元数据（star、语言、README）
- [ ] 已在线浏览项目目录结构
- [ ] 已识别核心模块和入口文件
- [ ] 已执行逆向三阶分析（怎么做→为什么→是什么）
- [ ] 已生成项目级文档（00-{repo}-三阶解构.md）
- [ ] 已识别并标记需深入分析的模块（✅）

**模块级分析（递归自动执行）**
- [ ] 对每个 ✅需深入 的模块执行深入分析
- [ ] 已生成模块级文档（01-{模块A}-三阶解构.md、02-{模块B}-三阶解构.md...）
- [ ] 模块文档中也标记了需深入分析的子模块（如有）
- [ ] 没有更多 ✅需深入 的模块时停止递归

**汇总输出（最后生成）**
- [ ] 已生成架构图谱（99-架构图谱.md）
- [ ] 已更新项目索引（00-GitHub项目索引.md）
- [ ] 所有文档已保存到Obsidian路径

### 输出文档结构

**Obsidian Vault路径**：`C:\Users\LX\Documents\Obsidian Vault\github`

```
C:\Users\LX\Documents\Obsidian Vault\github/
├── {org}-{repo}/                          # 项目文件夹
│   ├── 00-{repo}-三阶解构.md               # 项目级分析（逆向推导结果）
│   ├── 01-{模块A}-三阶解构.md              # 模块级分析
│   ├── 02-{模块B}-三阶解构.md              # 模块级分析
│   └── 99-架构图谱.md                      # 架构汇总
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

---

## 第1-3步完整示例：任务管理系统（场景A）

以下用一个完整的示例展示**开发新项目**的正向推导过程。

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
| **用户模块** | 状态追踪需要身份 | 管理用户信息、权限 | ❌ 当前文档说明 |
| **任务模块** | 记忆辅助的核心 | 任务CRUD、状态管理 | ✅ 需深入分析 |
| **标签模块** | 分类组织需要 | 标签定义、关联 | ❌ 当前文档说明 |
| **项目模块** | 分类组织需要 | 项目分组、权限 | ❌ 当前文档说明 |
| **历史模块** | 状态追踪需要审计 | 操作记录、变更历史 | ❌ 当前文档说明 |
| **调度模块** | 提醒通知需要触发 | 定时任务、触发器 | ❌ 当前文档说明 |
| **通知模块** | 提醒通知需要渠道 | 多渠道通知发送 | ❌ 当前文档说明 |

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
| **❌ 当前文档说明** | 简单属性 | 当前文档详细展开 |

### 停止条件

递归分析在以下任一条件满足时停止：

| 优先级 | 停止条件 | 说明 |
|-------|---------|------|
| **1** | 当前层级没有"✅ 需深入"的属性 | 所有模块都已分析完毕 |
| **2** | 属性已经是基础类型 | 如字符串、数字、布尔值等不可再分的基础数据类型 |
| **3** | 模块职责清晰且简单 | 模块功能单一，无需进一步拆解 |

**注意**：不以固定深度（如3层）作为强制停止条件，而是以"是否还有需要深入分析的复杂属性"为判断标准。

---

## 核心公式

```
┌─────────────────────────────────────────────────────────┐
│                      本质编程公式                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   三阶推导：                                            │
│   场景A：是什么（定义）→ 为什么（决策）→ 怎么做（实现）  │
│   场景B：怎么做（观察）→ 为什么（推导）→ 是什么（理解）  │
│                                                         │
│   开发模式：                                            │
│   项目开发 = 属性化静态开发 + 方法化动态开发             │
│                                                         │
│   方法扩展：                                            │
│   多场景 = 同核心方法的路由分支                          │
│                                                         │
│   接口本质：                                            │
│   接口 = 跨模块协作契约                                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 附录A：场景B文档模板

### 项目级分析模板（00-{repo}-三阶解构.md）

```markdown
# {repo} - 三阶解构

> GitHub: {github_url}
> 分析日期：{date}
> 分析场景：场景B - 解析现有项目

## 元数据
- **项目名称**：{repo}
- **所属组织**：{org}
- **GitHub地址**：{github_url}
- **主要语言**：{primary_language}
- **Star数**：{stars}
- **分析日期**：{date}

---

## 一、怎么做 (How) - 观察实现

### 1.1 项目概览

**README描述**：
{readme_summary}

**源码结构**：
```
{repo}/
├── src/
│   ├── core/           → 核心模块
│   ├── utils/          → 工具模块
│   └── ...
```

### 1.2 模块划分（从代码识别）

| 模块名 | 源码位置 | 职责 | 标记 |
|-------|---------|------|------|
| **core** | `src/core/` | 核心逻辑 | ✅ 需深入 → [[01-core-三阶解构]] |
| **utils** | `src/utils/` | 工具函数 | ❌ 当前文档说明 |

### 1.3 核心方法分析

**方法：{方法名}**
- 位置：`{文件路径}:{行号}`
- 功能：...
- 利用的属性：...

---

## 二、为什么 (Why) - 推导设计

### 2.1 技术栈决策（从源码推导）

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

## 三、是什么 (What) - 推导本质

### 3.1 本质特征（从代码推导）

| 推导要素 | 源码证据 | 结论 |
|---------|---------|------|
| **核心问题** | 从README和issue分析 | ... |
| **目标用户** | 从文档和示例分析 | ... |
| **核心价值** | 从功能列表分析 | ... |

**本质特征**：
1. **...** - 证据：{文件路径}
2. **...** - 证据：{文件路径}

### 3.2 区分属性（与竞品对比）

| 对比项 | {repo} | 竞品A | 竞品B |
|-------|--------|------|------|
| ... | ... | ... | ... |

---

## 四、❌ 属性详细说明

### utils模块
...

---

## 参考链接

- [[99-架构图谱]]
- [[00-GitHub项目索引]]
```

### 模块级分析模板（{序号}-{模块名}-三阶解构.md）

```markdown
# {模块名} - 三阶解构

> 所属项目：{repo}
> 源码位置：{source_path}
> 分析日期：{date}
> 分析场景：场景B - 模块级逆向分析

---

## 模块概览

### 职责定位
{一句话描述该模块的核心职责}

### 源码位置
```
{source_path}/
├── file1.js          → 功能A
├── file2.js          → 功能B
└── ...
```

---

## 一、怎么做 (How) - 观察实现

### 1.1 子模块/组件划分

| 组件名 | 源码文件 | 职责 | 标记 |
|-------|---------|------|------|
| **ComponentA** | `file1.js` | 功能A | ✅ 需深入 → [[{next}-ComponentA-三阶解构]] |
| **ComponentB** | `file2.js` | 功能B | ❌ 当前文档说明 |

### 1.2 核心方法分析

**方法：{方法名}**
- 位置：`{文件路径}:{行号}`
- 功能：...
- 利用的属性：...
- 调用的其他方法：...

### 1.3 对外接口

```python
# 从源码提取的接口定义
class {Module}Interface:
    def method(self, ...):
        ...
```

---

## 二、为什么 (Why) - 推导设计

### 2.1 设计决策分析

| 决策 | 选择 | 源码证据 | 推导逻辑 |
|-----|------|---------|---------|
| ... | ... | ... | ... |

### 2.2 与项目其他模块的关系

```
{模块名}
    ├── 依赖 → [[00-{repo}-三阶解构]] 中的 {其他模块}
    ├── 被依赖 ← {其他模块}
    └── 协作 ↔ {其他模块}
```

---

## 三、是什么 (What) - 推导本质

### 3.1 模块本质

从代码推导该模块的本质：
- **核心问题**：这个模块解决什么问题？
- **设计思想**：作者的设计意图是什么？

### 3.2 关键属性确认

验证从代码识别的属性是否正确。

---

## 四、❌ 属性详细说明

### ComponentB
...

---

## 参考链接

- [[00-{repo}-三阶解构]] - 返回项目级分析
- [[99-架构图谱]]
- [[00-GitHub项目索引]]
```

### 项目索引模板（00-GitHub项目索引.md）

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

---

## 附录B：快速参考

### 场景选择决策树

```
开始
  │
  ├─ 有现成代码？
  │     │
  │     ├─ 是 → 场景B：解析项目
  │     │         │
  │     │         ├─ 第1步：观察实现（怎么做）
  │     │         │   └── 识别模块，标记 ✅需深入
  │     │         │
  │     │         ├─ 第2步：推导设计（为什么）
  │     │         │
  │     │         ├─ 第3步：推导本质（是什么）
  │     │         │   └── 生成 00-{repo}-三阶解构.md
  │     │         │
  │     │         └─ 第4步：递归深入（自动执行）
  │     │             │
  │     │             ├─ 对模块A执行逆向三阶分析
  │     │             │   └── 生成 01-{模块A}-三阶解构.md
  │     │             │       └── 如有 ✅ 标记，继续递归
  │     │             │
  │     │             ├─ 对模块B执行逆向三阶分析
  │     │             │   └── 生成 02-{模块B}-三阶解构.md
    │     │             │
    │     │             └─ 直到没有 ✅ 需深入的模块
    │     │                 └── 生成 99-架构图谱.md
  │     │
  │     └─ 否 → 场景A：开发新项目
  │               └─ 执行正向三阶设计
  │                   是什么→为什么→怎么做
```

### 关键问题清单

**场景A：开发新项目时问**
- 这个系统本质上要解决什么问题？
- 从本质能推导出哪些关键属性？
- 为什么选择这个技术栈？
- 模块如何划分？
- 方法如何编排属性？

**场景B：解析项目时问**

**项目级分析：**
- 从代码能看出哪些模块？
- 哪些模块需要深入分析（✅标记）？
- 每个方法利用了哪些属性？
- 为什么选择这种设计？
- 这个系统的本质是什么？
- 有哪些值得借鉴的地方？

**模块级分析（递归执行）：**
- 这个模块的子组件/子模块有哪些？
- 模块的核心方法是如何实现的？
- 模块与项目其他模块的关系是什么？
- 这个模块的设计意图是什么？
- 是否还有需要深入分析的子模块？

**递归停止检查：**
- 是否还有 ✅需深入 的模块？
- 模块是否已经是基础类型？
- 模块职责是否清晰且简单？
