# Pi Agent Core - 产品需求文档 (PRD)

---

## 文档修订记录

| 版本 | 日期 | 作者 | 修订内容 |
|------|------|------|----------|
| v1.0 | 2025-05-10 | AI Agent | 初始版本，定义第一阶段范围 |

---

## 设计理念

本项目借鉴两个优秀的开源项目，融合其设计精华：

### 参考项目

**1. pi agent (TypeScript实现)**
- **架构清晰**：Provider、Message、Tools、Skills分层明确
- **模块化设计**：每个包职责单一，易于理解和扩展
- **不可变设计**：使用WithTools而非BindTools，避免并发问题
- **配置灵活**：统一的配置结构，Provider特定配置分离

**2. nanobot (Python实现)**
- **超轻量级**：保持核心agent循环小而可读
- **实用主义**：内置聊天频道、记忆、MCP等实用功能
- **渐进式学习**：代码简单易懂，易于研究和修改
- **生产就绪**：从本地开发到长期运行，最小化开销

### 融合方案

**渐进式发展路径：**
- **第一阶段**：专注LLM集成，构建稳定的基础层（借鉴pi agent的Provider抽象）
- **第二阶段**：Agent核心与工具系统，保持轻量级（借鉴nanobot的小核心理念）
- **后续阶段**：渐进式添加聊天频道、记忆、MCP等特性（参考nanobot的实用功能）

**技术优势：**
- 语言：Go 1.26（并发性能强、类型安全、部署简单）
- 框架：基于eino（cloudwego开源，生产就绪）
- 目标：通用agent框架（支持多种应用场景）

**核心原则：**
- **模块化**：每个组件独立成包，单一职责
- **清晰性**：拒绝上帝对象，拒绝单文件几千行，关注点清晰分离
- **可扩展性**：易于添加新的提供商、工具和能力
- **Go语言惯例**：遵循 Go 最佳实践和惯例

---

## 目录

1. [设计理念](#设计理念)
2. [全局说明](#1-全局说明)
3. [项目背景](#2-项目背景)
4. [项目范围](#3-项目范围)
5. [用户角色分析](#4-用户角色分析)
6. [系统架构](#5-系统架构)
7. [业务流程](#6-业务流程)
8. [功能需求](#7-功能需求)
9. [非功能需求](#8-非功能需求)
10. [数据字典](#9-数据字典)
11. [风险与应对](#10-风险与应对)
12. [附录](#11-附录)

---

## 1. 全局说明

### 1.1 名词解释

| 术语 | 说明 |
|------|------|
| **Provider** | LLM提供商，如OpenAI、百度千帆、Ollama等 |
| **BaseChatModel** | eino框架的基础聊天模型接口，支持Generate和Stream方法 |
| **ToolCallingChatModel** | eino框架的支持工具调用的聊天模型接口，使用不可变的WithTools方法 |
| **schema.Message** | eino框架的消息结构，包含角色、内容、工具调用信息等 |
| **Tool** | 工具，LLM可以调用的外部能力，如天气查询、计算器等 |
| **Stream** | 流式响应，逐步返回生成内容而非一次性返回完整结果 |
| **Context** | 上下文，对话历史记录 |

### 1.2 统一异常处理规则

**网络异常：**
- 请求超时：默认30秒，可通过配置调整
- 网络不可达：返回明确错误提示，建议检查网络连接
- DNS解析失败：记录错误日志，返回友好错误消息

**API异常：**
- 认证失败（401）：提示API密钥无效或过期
- 权限不足（403）：提示账户权限问题
- 资源不存在（404）：提示模型或端点不存在
- 限流（429）：建议等待后重试，返回Retry-After时间
- 服务器错误（5xx）：建议稍后重试，记录详细错误日志

**配置异常：**
- 必填字段缺失：明确指出缺失的字段名称
- 字段类型错误：指出期望类型和实际类型
- 值范围错误：指出有效范围

### 1.3 列表默认数据规则

- 模型列表默认按名称字母序排序
- 分页查询默认每页20条，最大100条
- 空列表返回空数组而非null
- 时间字段统一使用RFC3339格式

---

## 2. 项目背景

### 2.1 现状

**问题：**
- 开发AI应用时，不同LLM提供商的API差异大，集成成本高
- 缺乏统一的Provider抽象层，代码重复严重
- 配置管理混乱，API密钥等敏感信息处理不规范
- 错误处理不统一，调试困难

**痛点：**
- 切换LLM提供商需要大量代码修改
- 新增Provider支持需要了解底层细节
- 测试时难以mock不同的LLM响应
- 缺乏统一的监控和日志标准

### 2.2 方案

构建模块化的Pi Agent框架，第一阶段聚焦LLM集成抽象：

**核心设计：**
- 统一的Provider接口，隔离不同LLM的差异
- 基于eino框架的BaseChatModel抽象
- 清晰的配置结构和验证机制
- 统一的错误处理和日志规范

**技术选型：**
- 语言：Go 1.26
- 核心框架：eino（cloudwego开源）
- 设计原则：模块化、单一职责、代码清晰

### 2.3 目标

**功能目标：**
- 支持至少3个主流LLM提供商（OpenAI、百度千帆、Ollama）
- 提供统一的消息构建接口
- 支持流式响应和工具调用

**质量目标：**
- 代码覆盖率 ≥ 80%
- 单文件代码行数 < 500行
- 添加新Provider时间 < 30分钟

**价值目标：**
- 减少LLM集成开发时间 60%
- 降低Provider切换成本至接近零
- 为后续Agent开发提供坚实基础

---

## 3. 项目范围

### 3.1 功能结构图

```
Pi Agent Core (Phase 1)
├── Provider管理
│   ├── Provider注册中心
│   ├── Provider接口定义
│   └── Provider实现
│       ├── OpenAI Provider
│       ├── Qianfan Provider
│       └── Ollama Provider
├── Message处理
│   ├── Message Builder
│   └── Message转换工具
├── 配置管理
│   ├── 通用配置
│   └── Provider特定配置
└── 错误处理
    ├── 错误类型定义
    └── 错误码规范
```

### 3.2 范围内

**核心功能：**
- Provider抽象层与统一接口
- OpenAI Provider实现
- 国产模型支持（百度千帆）
- 本地模型支持（Ollama）
- 基础消息处理与对话管理
- 流式响应支持
- 工具调用能力

**文档和测试：**
- Provider开发指南
- API使用示例
- 单元测试和集成测试

### 3.3 范围外

**后续阶段功能（参考pi agent和nanobot设计）：**

**第二阶段 - Agent核心与工具系统：**
- Agent状态管理与执行引擎
- 工具注册中心与内置工具
- 会话管理与持久化
- 记忆系统（长期记忆、上下文管理）

**第三阶段 - 聊天频道集成：**
- 多平台聊天应用支持（Telegram、Discord、WeChat、飞书等）
- 流式响应与多模态支持
- WebSocket通道
- WebUI界面

**第四阶段 - 高级特性：**
- MCP（Model Context Protocol）支持
- Skills系统与发现机制
- 多Agent协调与编排
- OpenAI兼容API服务
- Python/Go SDK

**不涉及：**
- 用户管理系统
- 权限控制与认证系统
- 计费系统
- 企业级部署运维平台

---

## 4. 用户角色分析

### 4.1 用户角色定义

| 角色 | 描述 | 核心诉求 |
|------|------|----------|
| **应用开发者** | 使用Pi Agent开发AI应用 | 快速集成LLM，切换Provider容易，API简单易懂 |
| **Provider开发者** | 为Pi Agent开发新的Provider | 接口清晰，开发指南详细，测试工具完善 |
| **测试工程师** | 测试基于Pi Agent的应用 | 易于mock，错误消息清晰，测试用例完整 |
| **运维工程师** | 部署和监控Pi Agent应用 | 日志规范，监控指标明确，配置灵活 |

### 4.2 用户场景

**场景1：应用开发者快速集成OpenAI**
- 作为应用开发者，我想在5分钟内完成OpenAI集成
- 配置API密钥和模型名称即可开始使用
- 支持流式响应显示进度

**场景2：应用开发者切换Provider**
- 作为应用开发者，我想从OpenAI切换到百度千帆
- 只需修改配置，无需改动业务代码
- API行为保持一致

**场景3：Provider开发者新增Provider**
- 作为Provider开发者，我想添加对Claude的支持
- 实现Provider接口的5个方法
- 参考现有Provider实现，30分钟内完成

**场景4：测试工程师编写测试**
- 作为测试工程师，我想mock LLM响应
- 使用统一的接口mock，无需了解Provider细节
- 错误场景易于模拟

---

## 5. 系统架构

### 5.1 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层                                  │
│              (pi-coding-agent, 其他 agents)                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Pi Agent Core                             │
│  ┌──────────────┐  ┌──────────┐  ┌─────────┐  ┌─────────┐  │
│  │   Provider   │  │  Message │  │  Tools  │  │  Agent  │  │
│  │   Registry   │  │  Builder │  │  System │  │  Core   │  │
│  └──────────────┘  └──────────┘  └─────────┘  └─────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Eino 框架                                │
│        BaseChatModel, ToolCallingChatModel, Schema           │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 包结构

```
pi-agent/
├── go.mod                    # Go模块定义
├── go.sum                    # 依赖校验和
├── README.md                 # 项目文档
├── LICENSE                   # Apache 2.0许可证
│
├── pkg/                      # 公共包
│   ├── provider/             # LLM Provider抽象
│   │   ├── provider.go       # Provider接口与注册中心
│   │   ├── config.go         # 配置结构
│   │   ├── errors.go         # Provider特定错误
│   │   ├── openai/           # OpenAI实现
│   │   ├── qianfan/          # 百度千帆实现
│   │   └── ollama/           # Ollama实现
│   │
│   ├── message/              # 消息工具
│   │   ├── builder.go        # 流畅的消息构建器
│   │   └── convert.go        # 格式转换工具
│   │
│   ├── tools/                # 工具系统（第二阶段）
│   │   ├── tool.go           # 工具接口
│   │   └── registry.go       # 工具注册中心
│   │
│   └── agent/                # Agent核心（第二阶段）
│       ├── agent.go          # Agent接口
│       └── state.go          # 状态管理
│
├── internal/                 # 内部包
│   └── testutil/             # 测试工具
│
├── examples/                 # 使用示例
│   ├── basic/                # 基础使用示例
│   └── streaming/            # 流式示例
│
└── docs/                     # 文档
    ├── ARCHITECTURE.md       # 架构决策
    └── PROVIDER.md           # Provider开发指南
```

---

## 6. 业务流程

### 6.1 核心业务流程

#### 6.1.1 基础聊天流程

```
用户代码
    │
    ▼
创建Provider实例 (Provider.New())
    │
    ▼
创建模型实例 (Provider.CreateModel())
    │
    ▼
构建消息 (message.Builder)
    │
    ▼
调用模型 (BaseChatModel.Generate/Stream)
    │
    ▼
接收响应 (*schema.Message)
    │
    ▼
用户代码（处理响应）
```

#### 6.1.2 工具调用流程

```
用户代码
    │
    ▼
创建Provider实例
    │
    ▼
创建ToolCallingChatModel
    │
    ▼
绑定工具 (WithTools)
    │
    ▼
构建消息
    │
    ▼
调用模型
    │
    ▼
检查响应中的ToolCalls
    │
    ├─ 有ToolCalls？
    │     │
    │     ▼
    │  执行工具
    │     │
    │     ▼
    │  添加ToolResult消息
    │     │
    │     └─> 循环回到"调用模型"
    │
    └─ 无ToolCalls → 返回最终响应
```

### 6.2 Provider注册流程

```
应用启动
    │
    ▼
创建Registry实例
    │
    ▼
注册Provider (Register)
    │
    ├─ 验证Provider名称唯一性
    ├─ 保存到providers map
    │
    ▼
使用时获取Provider (Get)
    │
    ▼
创建模型实例
```

---

## 7. 功能需求

### 7.1 Provider管理

#### 用例1：注册Provider

**描述：** 将新的Provider添加到注册中心

**前置条件：**
- Registry已创建
- Provider实例已初始化

**后置条件：**
- Provider被保存到Registry
- 可通过Name()方法获取该Provider

**业务流程：**
1. 调用Registry.Register(provider)
2. 检查Provider.Name()是否已存在
3. 如已存在，返回错误
4. 如不存在，保存到providers map
5. 返回成功

**异常流程：**
- Provider名称已存在 → 返回DuplicateProviderError
- Provider为nil → 返回InvalidParameterError

#### 用例2：获取Provider

**描述：** 从Registry中获取指定名称的Provider

**前置条件：**
- Registry已创建
- Provider已注册

**后置条件：**
- 返回Provider实例
- 可用于创建模型

**业务流程：**
1. 调用Registry.Get(name)
2. 检查name是否存在于providers map
3. 如存在，返回Provider
4. 如不存在，返回错误

**异常流程：**
- Provider不存在 → 返回ProviderNotFoundError

#### 用例3：创建模型实例

**描述：** 使用Provider创建聊天模型实例

**前置条件：**
- Provider已获取
- 配置已准备

**后置条件：**
- 返回BaseChatModel实例
- 可用于生成对话

**业务流程：**
1. 准备配置参数
2. 调用Provider.ValidateConfig(config)
3. 如验证失败，返回错误
4. 如验证成功，调用Provider.CreateModel(ctx, config)
5. 返回模型实例

**异常流程：**
- 配置类型错误 → 返回InvalidConfigTypeError
- API密钥缺失 → 返回MissingAPIKeyError
- 模型名称无效 → 返回InvalidModelError

### 7.2 消息构建

#### 用例4：构建简单消息

**描述：** 使用Builder构建基础对话消息

**前置条件：**
- Builder已创建

**后置条件：**
- 返回消息列表
- 可传递给模型

**业务流程：**
1. 创建Builder实例
2. 调用System()添加系统消息
3. 调用User()添加用户消息
4. 调用Build()返回消息列表

**数据验证：**
- 消息内容不能为空（空字符串）
- 角色必须是有效值（system/user/assistant/tool）

#### 用例5：构建多模态消息

**描述：** 构建包含图片的用户消息

**前置条件：**
- 图片数据已准备
- Builder已创建

**后置条件：**
- 返回包含多模态内容的消息
- 图片以base64编码存储

**业务流程：**
1. 准备图片数据（字节）
2. 指定MIME类型（image/jpeg等）
3. 调用UserWithImages(content, images)
4. 调用Build()返回消息

**数据验证：**
- 图片大小 ≤ 20MB
- 支持的格式：JPEG、PNG、GIF、WebP

### 7.3 工具调用

#### 用例6：绑定工具

**描述：** 为ToolCallingChatModel绑定可用工具

**前置条件：**
- ToolCallingChatModel已创建
- 工具信息已定义

**后置条件：**
- 返回新的ToolCallingChatModel实例
- 原实例不变（不可变设计）

**业务流程：**
1. 定义ToolInfo结构
2. 调用WithTools(tools)
3. 返回新的模型实例

**数据验证：**
- 工具名称必须唯一
- 参数定义必须符合JSON Schema

---

## 8. 非功能需求

### 8.1 性能需求

| 指标 | 要求 | 说明 |
|------|------|------|
| Provider注册 | < 1ms | 纯内存操作 |
| 获取Provider | < 1ms | Map查找 |
| 创建模型实例 | < 100ms | 主要耗时在配置验证 |
| 流式响应首字节 | 依赖LLM | 通常 < 2秒 |
| 并发请求 | 支持 | 每个模型实例独立，无共享状态 |

### 8.2 可靠性需求

- **错误恢复**：网络错误支持自动重试（可配置）
- **超时控制**：所有外部调用支持超时设置
- **资源清理**：StreamReader必须显式关闭
- **状态隔离**：每个模型实例独立，不共享可变状态

### 8.3 可维护性需求

- **代码规范**：遵循golangci-lint规则
- **文档完整**：所有公共API必须有GoDoc注释
- **测试覆盖**：单元测试覆盖率 ≥ 80%
- **示例代码**：每个主要功能提供可运行示例

### 8.4 监控需求

**日志记录：**
- Provider注册/获取操作
- 模型创建成功/失败
- API调用耗时
- 错误详情（不含敏感信息）

**监控指标：**
- API调用次数（按Provider、模型分组）
- API调用成功率
- 平均响应时间
- 错误类型分布

**告警规则：**
- 错误率 > 5% 告警
- 平均响应时间 > 10s 告警
- 认证失败次数 > 10次/分钟 告警

### 8.5 安全需求

- **密钥管理**：不记录完整API密钥，仅显示前后4位
- **敏感数据**：请求/响应日志脱敏处理
- **输入验证**：所有外部输入必须验证
- **依赖安全**：使用govulncheck扫描依赖漏洞

---

## 9. 数据字典

### 9.1 Provider相关

**Provider接口**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Name() | string | 是 | Provider唯一标识，如"openai" |
| CreateModel() | BaseChatModel | 是 | 创建基础聊天模型 |
| CreateToolCallingModel() | ToolCallingChatModel | 是 | 创建支持工具的模型 |
| ValidateConfig() | error | 是 | 验证配置有效性 |
| ListModels() | []ModelInfo | 否 | 返回可用模型列表 |

**ModelInfo结构**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ID | string | 是 | 模型标识符，如"gpt-4o" |
| Name | string | 是 | 显示名称 |
| Provider | string | 是 | 所属Provider |
| MaxTokens | int | 是 | 最大上下文长度 |
| Capabilities | []string | 是 | 能力列表（chat/tools/vision） |
| Metadata | map[string]string | 否 | 扩展元数据 |

### 9.2 配置相关

**CommonConfig**

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| Model | string | 是 | - | 模型标识符 |
| Temperature | *float32 | 否 | nil | 随机性控制（0.0-2.0） |
| MaxTokens | *int | 否 | nil | 最大输出token数 |
| TopP | *float32 | 否 | nil | 核采样参数 |
| Timeout | Duration | 否 | 30s | 请求超时时间 |
| BaseURL | string | 否 | Provider默认值 | API端点地址 |

**OpenAI配置**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| APIKey | string | 是 | OpenAI API密钥 |
| Organization | string | 否 | 组织ID |

**百度千帆配置**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| APIKey | string | 是 | 千帆API密钥 |
| BaseURL | string | 否 | 默认为千帆API地址 |

**Ollama配置**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| BaseURL | string | 否 | 默认localhost:11434 |
| Model | string | 是 | 本地模型名称 |

### 9.3 Message相关

**Message结构（基于eino schema）**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Role | RoleType | 是 | 角色（system/user/assistant/tool） |
| Content | Content | 是 | 内容（字符串或多模态） |
| ToolCalls | []ToolCall | 否 | 工具调用信息（仅assistant） |
| ToolCallID | string | 条件必填 | 工具调用ID（仅tool角色） |

**ToolInfo结构**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Name | string | 是 | 工具名称 |
| Desc | string | 是 | 工具描述 |
| ParamsOneOf | ParamsOneOf | 是 | 参数定义（JSON Schema） |

### 9.4 错误码

| 错误码 | 说明 | HTTP状态码对应 |
|--------|------|----------------|
| INVALID_CONFIG | 配置无效 | 400 |
| MISSING_API_KEY | 缺少API密钥 | 401 |
| INVALID_MODEL | 模型无效 | 404 |
| REQUEST_FAILED | 请求失败 | 500/502/503 |
| RATE_LIMITED | 被限流 | 429 |
| TIMEOUT | 请求超时 | - |
| PROVIDER_NOT_FOUND | Provider不存在 | - |
| DUPLICATE_PROVIDER | Provider已存在 | - |

---

## 10. 风险与应对

### 10.1 技术风险

| 风险 | 影响 | 概率 | 应对措施 |
|------|------|------|----------|
| eino API变更 | 高 | 低 | 固定版本号，监控上游Release Notes |
| Provider API不兼容 | 中 | 中 | 详细的集成测试，版本兼容性检查 |
| 流式响应性能问题 | 中 | 低 | 早期性能测试，优化数据流处理 |
| Go版本兼容性 | 低 | 低 | 明确支持的Go版本范围 |

### 10.2 业务风险

| 风险 | 影响 | 概率 | 应对措施 |
|------|------|------|----------|
| 第三方API限流 | 中 | 高 | 实现退避重试机制，限流提示 |
| API密钥泄露 | 高 | 低 | 文档强调安全实践，示例代码脱敏 |
| 成本控制 | 中 | 中 | 提供token计数工具，成本估算API |

### 10.3 项目风险

| 风险 | 影响 | 概率 | 应对措施 |
|------|------|------|----------|
| 需求变更 | 中 | 中 | 模块化设计，易于扩展 |
| 依赖库问题 | 中 | 低 | 选择成熟稳定的依赖，定期更新 |
| 文档不完善 | 低 | 中 | PRD评审，示例代码先行 |

---

## 11. 附录

### 11.1 设计决策记录

#### 决策1：使用BaseChatModel而非废弃的ChatModel

**背景：** eino的ChatModel接口已废弃，BindTools()方法会修改实例本身，并发不安全。

**决策：** 使用新的BaseChatModel和ToolCallingChatModel接口。

**理由：**
- 避免并发安全问题
- WithTools()返回新实例，不可变设计
- 符合函数式编程原则

**影响：**
- 所有Provider实现返回ToolCallingChatModel时需类型断言
- 用户使用WithTools()而非BindTools()

#### 决策2：Provider接口分离创建方法

**背景：** 是否应该统一CreateModel和CreateToolCallingModel？

**决策：** 保持两个独立的创建方法。

**理由：**
- 不是所有Provider都支持工具调用
- 类型安全，避免运行时类型断言
- 调用者明确知道得到什么类型的模型

**影响：**
- Provider接口包含两个方法
- 文档需明确说明何时使用哪个方法

#### 决策3：配置使用interface{}

**背景：** Go 1.26支持泛型，是否应该使用泛型类型化配置？

**决策：** 使用interface{}配合ValidateConfig。

**理由：**
- 更简单的接口定义
- Registry可以统一管理所有Provider
- Provider内部进行类型断言和验证
- 符合Go惯例

**影响：**
- Provider实现需进行类型断言
- 错误消息需明确指出期望的类型

#### 决策4：Message基于eino schema

**背景：** 是否应该自定义消息结构？

**决策：** 直接使用eino的schema.Message。

**理由：**
- 避免重复造轮子
- 与eino生态系统无缝集成
- schema.Message已经很完善（支持多模态、工具调用等）

**影响：**
- Message Builder是对schema.Message的便捷封装
- 用户需了解eino schema的基本概念

### 11.2 参考资料

- [eino官方文档](https://www.cloudwego.io/docs/eino/)
- [eino GitHub仓库](https://github.com/cloudwego/eino)
- [eino-ext GitHub仓库](https://github.com/cloudwego/eino-ext)
- [OpenAI API文档](https://platform.openai.com/docs)
- [百度千帆文档](https://cloud.baidu.com/doc/WENXINWORKSHOP/index.html)
- [Ollama文档](https://github.com/ollama/ollama)

### 11.3 相关文档

- ARCHITECTURE.md - 架构设计详细说明
- PROVIDER.md - Provider开发指南
- MIGRATION.md - 从其他框架迁移指南

---

**文档结束**
