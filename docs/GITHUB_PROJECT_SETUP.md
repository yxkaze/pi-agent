# GitHub Project 设置指南

本文档指导您如何为Pi Agent项目创建GitHub Project看板，管理PRD中的需求。

---

## 一、创建Project

### 步骤1：访问仓库

1. 打开您的GitHub仓库主页：
   ```
   https://github.com/您的用户名/pi-agent
   ```

2. 点击顶部的 **"Projects"** 标签页

3. 点击右侧绿色的 **"New project"** 按钮

### 步骤2：选择模板

推荐选择 **"Team planning"** 模板，它提供：
- ✅ 预配置的看板列（Todo、In Progress、Done）
- ✅ 自动化的状态流转
- ✅ 团队协作功能

或者选择 **"Blank project"** 完全自定义。

### 步骤3：填写信息

- **Project name:** `Pi Agent Development`
- **Description:** `Pi Agent Core开发跟踪 - 基于eino框架的Go语言Agent系统`
- **Visibility:** 建议选择 **Public**（与仓库一致）
- 点击 **"Create project"**

---

## 二、配置看板结构

### 方案A：使用预设模板（推荐新手）

如果选择了"Team planning"模板，GitHub会自动创建：
- 📋 Todo
- 🔧 In Progress  
- ✅ Done

### 方案B：自定义看板列（推荐进阶）

创建以下列结构：

```
📋 Backlog
   └─ 所有待办任务

🔧 In Progress
   └─ 当前正在开发

👀 In Review
   └─ 代码审查中

✅ Done
   └─ 已完成

🚫 Blocked
   └─ 有阻塞问题
```

**添加自定义字段：**

1. 点击看板右上角的 **"+"** 号
2. 选择 **"New field"**
3. 添加以下字段：

| 字段名 | 类型 | 选项值 |
|--------|------|--------|
| **Phase** | Single select | Phase 1, Phase 2, Phase 3, Phase 4 |
| **Priority** | Single select | P0 (Critical), P1 (High), P2 (Medium), P3 (Low) |
| **Size** | Single select | XS, S, M, L, XL |
| **Start date** | Date | - |
| **End date** | Date | - |

---

## 三、创建第一个Issue

### 方法1：从PRD创建Issue

**Phase 1 - Provider管理：**

1. 在Project页面，点击 **"New item"**
2. 选择 **"Create new issue"**
3. 填写以下信息：

**Issue #1: Provider注册中心**
```markdown
**标题:** [FEATURE] 实现Provider注册中心

**描述:**
实现Provider接口和Registry，支持Provider的注册、发现和管理。

**前置条件:**
- Go 1.26环境已配置
- eino框架依赖已添加

**业务流程:**
1. 定义Provider接口
2. 实现Registry结构
3. 实现Register方法
4. 实现Get方法
5. 实现List方法
6. 编写单元测试

**验收标准:**
- ✅ Provider接口定义完整
- ✅ Registry支持并发安全
- ✅ 单元测试覆盖率 ≥ 80%
- ✅ 有使用示例

**标签:** `feature`, `phase-1`, `provider`
**阶段:** Phase 1
**优先级:** P0
**工作量:** M
```

**Issue #2: OpenAI Provider实现**
```markdown
**标题:** [PROVIDER] 添加OpenAI Provider支持

**描述:**
实现OpenAI Provider，支持GPT-4、GPT-3.5等模型。

**前置条件:**
- Provider接口已定义
- Registry已实现

**业务流程:**
1. 创建openai包
2. 定义Config结构
3. 实现Provider接口
4. 实现CreateModel方法
5. 实现CreateToolCallingModel方法
6. 实现ValidateConfig方法
7. 实现ListModels方法
8. 编写测试

**API配置:**
- API Key: 必填
- Model: 必填
- Temperature: 可选
- MaxTokens: 可选

**支持的模型:**
- gpt-4o (推荐)
- gpt-4o-mini
- gpt-4-turbo
- gpt-3.5-turbo

**验收标准:**
- ✅ 支持Generate和Stream
- ✅ 支持工具调用
- ✅ 配置验证完善
- ✅ 错误处理完整
- ✅ 测试覆盖率 ≥ 80%

**标签:** `provider`, `openai`, `phase-1`
**阶段:** Phase 1
**优先级:** P0
**工作量:** M
```

**Issue #3: 百度千帆Provider实现**
```markdown
**标题:** [PROVIDER] 添加百度千帆Provider支持

**描述:**
实现百度千帆Provider，支持GLM-5、MiniMax、Kimi等国产模型。

**前置条件:**
- Provider接口已定义
- OpenAI Provider已实现（可参考）

**业务流程:**
1. 创建qianfan包
2. 定义Config结构
3. 实现Provider接口
4. 适配千帆API（OpenAI兼容）
5. 实现ListModels方法
6. 编写测试

**API配置:**
- API Key: 必填
- BaseURL: 可选（默认千帆API地址）
- Model: 必填

**支持的模型:**
- glm-5 (推荐)
- minimax-m2.5
- kimi-k2.5

**验收标准:**
- ✅ API兼容OpenAI格式
- ✅ 支持主流国产模型
- ✅ 测试覆盖率 ≥ 80%

**标签:** `provider`, `qianfan`, `phase-1`
**阶段:** Phase 1
**优先级:** P1
**工作量:** M
```

**Issue #4: Message Builder实现**
```markdown
**标题:** [FEATURE] 实现Message Builder

**描述:**
提供流畅的消息构建接口，简化对话消息的创建。

**前置条件:**
- eino schema已了解

**业务流程:**
1. 创建Builder结构
2. 实现System方法
3. 实现User方法
4. 实现UserWithImages方法
5. 实现Assistant方法
6. 实现Tool方法
7. 实现Build方法
8. 编写测试

**使用示例:**
```go
msgs := message.NewBuilder().
    System("You are a helpful assistant.").
    User("Hello!").
    Build()
```

**验收标准:**
- ✅ 支持链式调用
- ✅ 支持多模态消息
- ✅ 类型安全
- ✅ 测试覆盖率 ≥ 80%

**标签:** `feature`, `message`, `phase-1`
**阶段:** Phase 1
**优先级:** P1
**工作量:** S
```

### 方法2：批量导入

如果需要批量创建Issue，可以使用GitHub CLI：

```bash
# 安装GitHub CLI
brew install gh  # macOS
# 或
choco install gh  # Windows

# 创建Issue
gh issue create --title "[FEATURE] Provider注册中心" \
  --body "实现Provider注册中心..." \
  --label "feature,phase-1,provider" \
  --project "Pi Agent Development"
```

---

## 四、组织看板视图

### 创建多个视图

**1. 按阶段视图**
- 视图名称：`By Phase`
- 分组字段：`Phase`
- 显示：Phase 1, Phase 2, Phase 3, Phase 4

**2. 按优先级视图**
- 视图名称：`By Priority`
- 分组字段：`Priority`
- 显示：P0, P1, P2, P3

**3. 时间线视图**
- 视图名称：`Roadmap`
- 视图类型：Roadmap
- 字段：Start date, End date

### 设置过滤器

**查看Phase 1任务：**
```
Phase: Phase 1
```

**查看高优先级任务：**
```
Priority: P0 OR Priority: P1
```

**查看进行中的任务：**
```
Status: In Progress
```

---

## 五、自动化工作流（可选）

GitHub Projects支持自动化，可以配置：

**自动化规则示例：**

1. **新Issue自动添加到Backlog**
   - 触发：新Issue创建
   - 动作：添加到Project，设置Status为"Todo"

2. **Pull Request自动移动到In Review**
   - 触发：新PR创建
   - 动作：移动到"In Review"列

3. **合并后自动标记为Done**
   - 触发：PR合并
   - 动作：移动到"Done"列，关闭关联Issue

**配置方法：**
1. 点击Project右上角的 **"..."** 菜单
2. 选择 **"Workflows"**
3. 启用或自定义工作流

---

## 六、团队协作

### 分配任务

1. 点击Issue卡片
2. 在右侧面板的 **"Assignees"** 字段
3. 输入团队成员的GitHub用户名

### 添加评论和更新

- 在Issue中添加评论，同步进展
- 使用Markdown格式化内容
- 可以@提及团队成员

### 关联Pull Request

在PR描述中使用关键字：
```
Fixes #123
Closes #456
```

PR合并后，相关Issue会自动关闭。

---

## 七、最佳实践

### Issue命名规范

```
[FEATURE] 功能描述
[PROVIDER] 添加XXX Provider支持
[BUG] 问题描述
[DOCS] 文档更新
[TEST] 测试相关
```

### 标签使用

| 标签 | 含义 | 颜色建议 |
|------|------|----------|
| `phase-1` | 第一阶段 | 蓝色 |
| `phase-2` | 第二阶段 | 绿色 |
| `phase-3` | 第三阶段 | 黄色 |
| `phase-4` | 第四阶段 | 紫色 |
| `provider` | Provider相关 | 橙色 |
| `feature` | 新功能 | 亮蓝 |
| `bug` | Bug修复 | 红色 |
| `documentation` | 文档 | 浅蓝 |
| `test` | 测试 | 绿色 |

### 工作量估算

| 标识 | 工作量 | 时间估算 |
|------|--------|----------|
| XS | 极小 | < 2小时 |
| S | 小 | 2-4小时 |
| M | 中 | 1-2天 |
| L | 大 | 3-5天 |
| XL | 极大 | > 5天 |

---

## 八、示例看板截图

创建完成后，您的看板应该类似这样：

```
📋 Backlog                  🔧 In Progress          ✅ Done
┌─────────────────────┐   ┌─────────────────────┐   ┌────────────────┐
│ Phase 1             │   │ Provider接口定义     │   │ 项目初始化     │
│ ├─ Provider注册中心  │   │ 负责人: @you        │   │ 文档结构创建   │
│ ├─ OpenAI Provider   │   │ 标签: phase-1       │   │                │
│ ├─ 百度千帆Provider   │   │ 优先级: P0         │   │                │
│ ├─ Message Builder   │   │                     │   │                │
│ Phase 2             │   │                     │   │                │
│ ├─ Agent核心        │   │                     │   │                │
│ └─ 工具系统          │   │                     │   │                │
└─────────────────────┘   └─────────────────────┘   └────────────────┘
```

---

## 九、下一步

1. ✅ 创建GitHub Project
2. ✅ 配置看板列和字段
3. ✅ 创建Phase 1的Issue（4-5个）
4. ✅ 设置视图和过滤器
5. ✅ 开始第一个任务

---

## 十、获取帮助

- **GitHub Projects文档:** https://docs.github.com/en/issues/planning-and-tracking-with-projects
- **Issue模板:** `.github/ISSUE_TEMPLATE/` 目录
- **项目讨论:** 使用GitHub Discussions提问

---

**祝您项目管理顺利！** 🚀
