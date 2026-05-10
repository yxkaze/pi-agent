# Pi Agent Core - 产品需求文档 (PRD)

**版本:** 1.0  
**日期:** 2025-05-10  
**作者:** AI Agent (与用户协作)  
**状态:** 草稿

---

## 1. 执行摘要

Pi Agent Core 是一个基于 Go 语言和 eino 框架构建的模块化、可扩展的 AI Agent 框架。第一阶段聚焦于 LLM 集成抽象，基于 OpenAI API 标准，为多个 LLM 提供商（OpenAI、国产模型、本地模型）提供统一接口。

### 核心原则

- **模块化**：每个组件独立成包，单一职责
- **清晰性**：拒绝上帝对象，拒绝单文件几千行，关注点清晰分离
- **可扩展性**：易于添加新的提供商、工具和能力
- **Go语言惯例**：遵循 Go 最佳实践和惯例

---

## 2. 项目范围

### 第一阶段：LLM 集成（当前重点）

**范围内：**
- Provider 抽象层与统一接口
- OpenAI Provider 实现
- 国产模型支持（百度千帆）
- 本地模型支持（Ollama）
- 基础消息处理与对话管理
- 流式响应支持
- 工具调用能力

**范围外：**
- 高级 Agent 编排
- 多 Agent 协调
- 复杂工作流组合
- Web UI / 终端 UI
- 会话持久化
- Skills 系统实现

---

## 3. 架构概览

### 3.1 系统架构

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

### 3.2 包结构

```
pi-agent/
├── go.mod                    # Go 模块定义
├── go.sum                    # 依赖校验和
├── README.md                 # 项目文档
├── LICENSE                   # Apache 2.0 许可证
│
├── pkg/                      # 公共包
│   ├── provider/             # LLM Provider 抽象
│   │   ├── provider.go       # Provider 接口与注册中心
│   │   ├── config.go         # 配置结构
│   │   ├── errors.go         # Provider 特定错误
│   │   ├── openai/           # OpenAI 实现
│   │   ├── qianfan/          # 百度千帆实现
│   │   └── ollama/           # Ollama 实现
│   │
│   ├── message/              # 消息工具
│   │   ├── builder.go        # 流畅的消息构建器
│   │   ├── convert.go        # 格式转换工具
│   │   └── builder_test.go
│   │
│   ├── tools/                # 工具系统（第二阶段）
│   │   ├── tool.go           # 工具接口
│   │   ├── registry.go       # 工具注册中心
│   │   └── builtin/          # 内置工具
│   │
│   └── agent/                # Agent 核心（第二阶段）
│       ├── agent.go          # Agent 接口
│       ├── state.go          # 状态管理
│       └── executor.go       # 执行引擎
│
├── internal/                 # 内部包
│   └── testutil/             # 测试工具
│       └── mock.go
│
├── examples/                 # 使用示例
│   ├── basic/                # 基础使用示例
│   └── streaming/            # 流式示例
│
└── docs/                     # 文档
    ├── ARCHITECTURE.md       # 架构决策
    └── PROVIDER.md           # Provider 开发指南
```

---

## 4. 组件规格

### 4.1 Provider 包

#### 4.1.1 核心接口设计

**文件：** `pkg/provider/provider.go`

Provider 接口是顶层工厂，用于创建聊天模型实例。每个 Provider 实现（OpenAI、Qianfan、Ollama）都实现此接口。

**主要方法：**

- `Name()` - 返回 Provider 的唯一标识符（如 "openai"、"qianfan"）
- `CreateModel(ctx, config)` - 创建 BaseChatModel 实例
- `CreateToolCallingModel(ctx, config)` - 创建支持工具调用的 ToolCallingChatModel 实例
- `ValidateConfig(config)` - 验证配置是否有效
- `ListModels(ctx)` - 返回该 Provider 可用的模型列表

**ModelInfo 结构：**

描述模型的元数据和能力：
- `ID` - 模型标识符（如 "gpt-4o"）
- `Name` - 显示名称
- `Provider` - Provider 名称
- `MaxTokens` - 最大上下文长度
- `Capabilities` - 能力列表（如 chat、tools、vision）
- `Metadata` - 额外的 Provider 特定信息

**Registry（注册中心）：**

管理所有已注册的 Provider：
- `NewRegistry()` - 创建新的注册中心
- `Register(p)` - 注册 Provider
- `Get(name)` - 按名称获取 Provider
- `List()` - 列出所有注册的 Provider 名称

#### 4.1.2 配置结构设计

**文件：** `pkg/provider/config.go`

**CommonConfig（通用配置）：**

包含所有 Provider 共享的配置字段：
- `Model` - 模型标识符
- `Temperature` - 控制随机性（0.0-2.0）
- `MaxTokens` - 限制响应长度
- `TopP` - 通过核采样控制多样性
- `Timeout` - API 请求超时时间
- `BaseURL` - 允许覆盖默认 API 端点

**ModelConfig（模型配置）：**

创建模型实例的标准配置：
- `Provider` - 指定使用的 Provider（"openai"、"qianfan"、"ollama"）
- 包含 CommonConfig
- `ProviderConfig` - Provider 特定配置（APIKey 等）

#### 4.1.3 OpenAI Provider 实现

**文件：** `pkg/provider/openai/provider.go`

**职责：**
- 实现 Provider 接口
- 封装 eino-ext 的 OpenAI 实现
- 处理 OpenAI 特定的配置和错误

**特定配置：**
- `APIKey` - OpenAI API 密钥（必需）
- `Organization` - OpenAI 组织 ID（可选）

**模型列表：**
- GPT-4o（支持 chat、tools、vision）
- GPT-4o Mini（支持 chat、tools）
- GPT-4 Turbo（支持 chat、tools、vision）
- GPT-3.5 Turbo（支持 chat、tools）

**实现要点：**
- 使用 eino-ext/components/model/openai
- BaseChatModel 已实现 ToolCallingChatModel 接口
- 支持所有标准 OpenAI 参数

#### 4.1.4 百度千帆 Provider 实现

**文件：** `pkg/provider/qianfan/provider.go`

**职责：**
- 实现 Provider 接口
- 通过 OpenAI 兼容 API 接入千帆
- 处理千帆特定的配置

**特定配置：**
- `APIKey` - 百度千帆 API 密钥（必需）
- `BaseURL` - 千帆 API 端点（默认：https://qianfan.baidubce.com/v2）

**模型列表：**
- GLM-5（最大上下文 198K）
- MiniMax-M2.5（最大上下文 192K）
- Kimi-K2.5（最大上下文 256K）

**实现要点：**
- 千帆 API 与 OpenAI 兼容，复用 eino-ext 的 openai 组件
- 正确设置 BaseURL
- 处理千帆特有的错误码

#### 4.1.5 Ollama Provider 实现

**文件：** `pkg/provider/ollama/provider.go`

**职责：**
- 实现 Provider 接口
- 连接本地 Ollama 服务
- 支持本地开源模型

**特定配置：**
- `BaseURL` - Ollama 服务地址（默认：http://localhost:11434）
- `Model` - 本地模型名称

**实现要点：**
- Ollama 提供 OpenAI 兼容 API
- 无需 APIKey
- 支持模型自动下载

### 4.2 Message Builder

**文件：** `pkg/message/builder.go`

**设计目标：**

提供流畅的接口构建消息序列，基于 eino schema。

**Builder 方法：**

- `NewBuilder()` - 创建新的消息构建器
- `System(content)` - 添加系统消息
- `User(content)` - 添加用户消息
- `UserWithImages(content, images)` - 添加带图片的用户消息
- `Assistant(content)` - 添加助手消息
- `Tool(toolCallID, content)` - 添加工具结果消息
- `Build()` - 返回构建的消息列表

**ImageContent 结构：**
- `Data` - 图片数据（base64 解码后的字节）
- `MimeType` - MIME 类型（如 "image/jpeg"、"image/png"）

**使用场景：**
- 简化消息构建过程
- 支持链式调用
- 类型安全的消息构造

### 4.3 错误处理

**文件：** `pkg/provider/errors.go`

**Error 结构：**

Provider 特定的错误类型：
- `Provider` - Provider 名称
- `Code` - 错误代码
- `Message` - 错误消息
- `Cause` - 底层错误（如果有）

**错误代码：**
- `INVALID_CONFIG` - 配置无效
- `MISSING_API_KEY` - 缺少 API 密钥
- `INVALID_MODEL` - 模型无效
- `REQUEST_FAILED` - 请求失败
- `RATE_LIMITED` - 被限流
- `TIMEOUT` - 请求超时

**错误处理原则：**
- 所有错误都包含 Provider 名称
- 支持错误链（Unwrap）
- 清晰的错误消息便于调试

---

## 5. 数据流

### 5.1 基础聊天流程

```
用户代码
    │
    ▼
创建 Provider 实例 (Provider.New())
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

### 5.2 工具调用流程

```
用户代码
    │
    ▼
创建 Provider 实例
    │
    ▼
创建 ToolCallingChatModel (Provider.CreateToolCallingModel())
    │
    ▼
绑定工具 (ToolCallingChatModel.WithTools())
    │
    ▼
构建消息
    │
    ▼
调用模型
    │
    ▼
检查响应中的 ToolCalls
    │
    ├─ 有 ToolCalls？
    │     │
    │     ▼
    │  执行工具
    │     │
    │     ▼
    │  添加 ToolResult 消息
    │     │
    │     └─> 循环回到"调用模型"
    │
    └─ 无 ToolCalls → 返回最终响应
```

---

## 6. 测试策略

### 6.1 单元测试

**Provider 测试：**
- 测试配置验证
- 测试各种配置下的模型创建
- 测试无效输入的错误处理

**Message Builder 测试：**
- 测试消息构建
- 测试多模态内容
- 测试边界情况（空内容等）

### 6.2 集成测试

- 测试实际 API 调用（使用 mock 服务器或 CI secrets 中的真实 API 密钥）
- 测试流式响应
- 测试工具调用工作流
- 测试多个 Provider 互换使用

### 6.3 测试覆盖率目标

- 最低 80% 代码覆盖率
- 所有错误路径必须测试
- 所有公共 API 必须有示例

---

## 7. 依赖关系

### 7.1 核心依赖

- **go 1.18+**：语言版本
- **github.com/cloudwego/eino**：核心框架（BaseChatModel、Schema 等）
- **github.com/cloudwego/eino-ext**：Provider 实现

### 7.2 开发依赖

- **golangci-lint**：代码检查和质量
- **mockgen**：测试用的 Mock 生成
- **gotestsum**：改进测试输出

---

## 8. 演进路线

### 第一阶段（当前）
- Provider 抽象
- OpenAI、Qianfan、Ollama 支持
- Message builder
- 基础工具调用

### 第二阶段
- Agent 核心与状态管理
- 工具注册中心
- 高级对话管理
- 会话持久化

### 第三阶段
- Skills 系统
- 多 Agent 协调
- 高级工作流组合
- UI 组件

---

## 9. 成功标准

### 第一阶段成功指标

1. **功能性**
   - ✅ 成功调用 OpenAI API
   - ✅ 成功调用百度千帆 API
   - ✅ 成功调用 Ollama API
   - ✅ 流式响应正常工作
   - ✅ 工具调用正常工作

2. **代码质量**
   - ✅ 所有包的文件行数 < 500 行
   - ✅ 清晰的关注点分离
   - ✅ 无循环依赖
   - ✅ 80%+ 测试覆盖率

3. **文档**
   - ✅ README 包含快速开始
   - ✅ 可运行的代码示例
   - ✅ Provider 开发指南

4. **开发者体验**
   - ✅ 添加新 Provider 容易（< 30 分钟）
   - ✅ 清晰的错误消息
   - ✅ 直观的 API

---

## 10. 开放问题

1. **配置管理**
   - 是否应该支持基于环境变量的配置？
   - 如何处理 API 密钥轮换？

2. **错误恢复**
   - Provider 是否应该实现自动重试逻辑？
   - 如何优雅地处理限流？

3. **性能**
   - 是否应该实现连接池？
   - 如何高效处理并发请求？

4. **可观测性**
   - 是否应该添加日志中间件？
   - 如何追踪 API 使用和成本？

---

## 11. 时间估算

### 第一阶段：LLM 集成（2-3 周）

**第一周：**
- Provider 接口设计
- OpenAI Provider 实现
- Message builder
- 基础测试

**第二周：**
- Qianfan Provider 实现
- Ollama Provider 实现
- 错误处理
- 集成测试

**第三周：**
- 文档编写
- 示例代码
- 代码审查和优化
- 发布准备

---

## 12. 风险与缓解措施

| 风险 | 影响 | 概率 | 缓解措施 |
|------|------|------|----------|
| eino API 变更 | 高 | 低 | 固定版本，监控上游变更 |
| Provider API 不兼容 | 中 | 中 | 详细的集成测试 |
| 流式响应性能问题 | 中 | 低 | 早期基准测试，按需优化 |
| 配置复杂度 | 低 | 中 | 保持配置最小化，使用合理默认值 |

---

## 13. 设计决策记录

### 决策 1：使用 BaseChatModel 而非废弃的 ChatModel

**背景：** eino 的 ChatModel 接口已废弃，原因是 BindTools() 方法会修改实例本身，在并发环境下不安全。

**决策：** 使用新的 BaseChatModel 和 ToolCallingChatModel 接口。

**理由：**
- 避免并发安全问题
- WithTools() 返回新实例，不可变设计
- 符合函数式编程原则

### 决策 2：Provider 接口分离创建方法

**背景：** 是否应该统一 CreateModel 和 CreateToolCallingModel？

**决策：** 保持两个独立的创建方法。

**理由：**
- 不是所有 Provider 都支持工具调用
- 类型安全，避免运行时类型断言
- 调用者明确知道得到什么类型的模型

### 决策 3：配置使用 interface{} 而非泛型

**背景：** Go 1.18+ 支持泛型，是否应该使用泛型来类型化配置？

**决策：** 使用 interface{} 配合 ValidateConfig。

**理由：**
- 更简单的接口定义
- Registry 可以统一管理所有 Provider
- Provider 内部进行类型断言和验证
- 兼容 Go 1.18 之前的惯例

### 决策 4：Message Builder 基于 eino schema

**背景：** 是否应该自定义消息结构？

**决策：** 直接使用 eino 的 schema.Message。

**理由：**
- 避免重复造轮子
- 与 eino 生态系统无缝集成
- schema.Message 已经很完善（支持多模态、工具调用等）

---

## 14. 附录

### A. 使用场景示例

**场景 1：基础聊天**
- 用户创建 OpenAI Provider
- 配置 API 密钥和模型
- 构建系统消息和用户消息
- 调用 Generate 获取响应

**场景 2：流式对话**
- 用户创建 Qianfan Provider
- 使用流式响应实时显示助手回复
- 支持中途取消

**场景 3：工具调用**
- 用户创建 Ollama Provider
- 定义天气查询工具
- 绑定工具到模型
- 模型自动决定何时调用工具
- 执行工具并返回结果

**场景 4：多模型切换**
- 注册多个 Provider 到 Registry
- 根据任务需求选择不同模型
- 统一的错误处理和日志

### B. Provider 开发者指南大纲

1. 实现 Provider 接口
2. 定义 Provider 特定配置结构
3. 实现 CreateModel 和 CreateToolCallingModel
4. 处理 Provider 特定错误
5. 编写单元测试和集成测试
6. 提供模型列表和元数据
7. 编写文档和示例

### C. 错误处理最佳实践

- 所有错误都使用 Error 结构
- 包含足够的上下文信息
- 支持错误链追踪
- 提供可操作的错误消息
- 区分用户错误和系统错误

---

**文档结束**
