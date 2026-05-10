# GitHub Project 快速创建指南

**仓库地址：** https://github.com/yxkaze/pi-agent

---

## 第一步：创建Project

### 1.1 访问仓库

在浏览器中打开：
```
https://github.com/yxkaze/pi-agent
```

### 1.2 创建新Project

1. 点击顶部的 **"Projects"** 标签页
2. 点击右侧绿色的 **"New project"** 按钮
3. 选择 **"Team planning"** 模板

### 1.3 填写信息

- **Project name:** `Pi Agent Development`
- **Description:** `Pi Agent Core开发跟踪 - 基于eino框架的Go语言Agent系统`
- 点击 **"Create project"**

---

## 第二步：配置看板字段

### 2.1 添加自定义字段

点击看板右上角的 **"+"** → **"New field"**

**字段1：Phase（阶段）**
- Field name: `Phase`
- Field type: `Single select`
- Options:
  - `Phase 1` - LLM集成
  - `Phase 2` - Agent核心
  - `Phase 3` - 聊天频道
  - `Phase 4` - 高级特性

**字段2：Priority（优先级）**
- Field name: `Priority`
- Field type: `Single select`
- Options:
  - `P0` - Critical（紧急）
  - `P1` - High（高）
  - `P2` - Medium（中）
  - `P3` - Low（低）

**字段3：Size（工作量）**
- Field name: `Size`
- Field type: `Single select`
- Options:
  - `XS` - < 2小时
  - `S` - 2-4小时
  - `M` - 1-2天
  - `L` - 3-5天
  - `XL` - > 5天

---

## 第三步：创建Phase 1的Issue

### Issue #1: Provider注册中心

**标题：**
```
[FEATURE] 实现Provider注册中心
```

**内容（直接复制粘贴）：**
```markdown
## 功能描述

实现Provider接口和Registry，支持Provider的注册、发现和管理。

## 动机

为不同的LLM提供商提供统一的抽象层，降低集成成本，支持灵活切换。

## 详细说明

### 前置条件
- Go 1.26 环境已配置
- eino 框架依赖已添加

### 业务流程

1. 定义 Provider 接口
   - Name() - 返回Provider名称
   - CreateModel() - 创建基础聊天模型
   - CreateToolCallingModel() - 创建支持工具的模型
   - ValidateConfig() - 验证配置
   - ListModels() - 返回可用模型列表

2. 实现 Registry 结构
   - providers map 存储注册的Provider
   - 支持并发安全访问

3. 实现 Registry 方法
   - Register(p Provider) - 注册Provider
   - Get(name string) - 获取Provider
   - List() - 列出所有Provider

4. 定义错误类型
   - ProviderNotFoundError
   - DuplicateProviderError
   - InvalidConfigError

5. 编写单元测试

### 预期结果

- Provider 接口定义清晰
- Registry 支持并发安全
- 错误处理完善
- 单元测试覆盖率 ≥ 80%

## 实现建议

**文件结构：**
```
pkg/provider/
├── provider.go      # Provider接口定义
├── registry.go      # Registry实现
├── errors.go        # 错误类型
└── provider_test.go # 单元测试
```

**关键代码点：**
- 使用 sync.RWMutex 保证并发安全
- 接口设计遵循 Go 惯例
- 错误类型支持 Unwrap

## 验收标准

- [ ] Provider 接口定义完整
- [ ] Registry 支持并发安全
- [ ] 单元测试覆盖率 ≥ 80%
- [ ] 有使用示例
- [ ] 错误处理完善
- [ ] 文档注释完整

## 参考资料

- PRD文档：`docs/superpowers/specs/2025-05-10-pi-agent-core-design.md`
- eino框架：https://github.com/cloudwego/eino
- pi agent参考：https://github.com/earendil-works/pi

## 检查清单

- [ ] 已阅读CONTRIBUTING.md
- [ ] 已搜索相关Issue
- [ ] 功能符合项目范围
```

**标签：** `feature`, `phase-1`, `provider`
**Phase:** Phase 1
**Priority:** P0
**Size:** M

---

### Issue #2: OpenAI Provider实现

**标题：**
```
[PROVIDER] 添加OpenAI Provider支持
```

**内容（直接复制粘贴）：**
```markdown
## Provider信息

**Provider名称：** openai

**Provider类型：**
- [x] 云端LLM服务
- [ ] 本地LLM服务
- [ ] 开源模型托管平台

**官方网站：** https://platform.openai.com

## 支持功能

**基础功能：**
- [x] 文本对话
- [x] 流式响应
- [x] 多轮对话
- [x] 工具调用

**高级功能：**
- [x] 多模态（图片）
- [ ] 多模态（音频）
- [ ] 多模态（视频）

## API信息

**API类型：**
- [x] OpenAI兼容API
- [ ] Anthropic API
- [ ] 自定义API

**Base URL：** https://api.openai.com/v1

**认证方式：**
- [x] API Key
- [ ] OAuth
- [ ] 其他

## 实现计划

### 配置结构

```go
type Config struct {
    // 通用配置
    Model       string     // 模型名称（必填）
    Temperature *float32   // 温度参数
    MaxTokens   *int       // 最大token数
    TopP        *float32   // Top P
    Timeout     Duration   // 超时时间
    
    // OpenAI特定配置
    APIKey       string    // API密钥（必填）
    Organization string    // 组织ID（可选）
    BaseURL      string    // API端点（可选）
}
```

### 接口实现

- [ ] Provider接口实现
  - [ ] Name() → "openai"
  - [ ] CreateModel()
  - [ ] CreateToolCallingModel()
  - [ ] ValidateConfig()
  - [ ] ListModels()

### 支持的模型

| 模型ID | 显示名称 | 上下文 | 能力 |
|--------|---------|--------|------|
| gpt-4o | GPT-4o | 128K | chat, tools, vision |
| gpt-4o-mini | GPT-4o Mini | 128K | chat, tools |
| gpt-4-turbo | GPT-4 Turbo | 128K | chat, tools, vision |
| gpt-3.5-turbo | GPT-3.5 Turbo | 16K | chat, tools |

### 测试计划

- [ ] 配置验证测试
- [ ] 模型创建测试
- [ ] 流式响应测试
- [ ] 工具调用测试
- [ ] 错误处理测试

## 参考实现

- eino-ext OpenAI实现：`github.com/cloudwego/eino-ext/components/model/openai`
- pi agent OpenAI Provider：https://github.com/earendil-works/pi

## 验收标准

- [ ] 支持Generate和Stream
- [ ] 支持工具调用
- [ ] 配置验证完善
- [ ] 错误处理完整
- [ ] 测试覆盖率 ≥ 80%
- [ ] 有使用示例

## 检查清单

- [x] 已阅读Provider开发指南
- [x] API文档已准备好
- [ ] 测试环境已就绪
- [ ] 符合Provider接口规范
```

**标签：** `provider`, `openai`, `phase-1`
**Phase:** Phase 1
**Priority:** P0
**Size:** M

---

### Issue #3: 百度千帆Provider实现

**标题：**
```
[PROVIDER] 添加百度千帆Provider支持
```

**内容（直接复制粘贴）：**
```markdown
## Provider信息

**Provider名称：** qianfan

**Provider类型：**
- [x] 云端LLM服务
- [ ] 本地LLM服务
- [ ] 开源模型托管平台

**官方网站：** https://cloud.baidu.com/product/wenxinworkshop.html

## 支持功能

**基础功能：**
- [x] 文本对话
- [x] 流式响应
- [x] 多轮对话
- [x] 工具调用

**高级功能：**
- [ ] 多模态（图片）
- [ ] 多模态（音频）
- [ ] 多模态（视频）

## API信息

**API类型：**
- [x] OpenAI兼容API
- [ ] Anthropic API
- [ ] 自定义API

**Base URL：** https://qianfan.baidubce.com/v2

**认证方式：**
- [x] API Key
- [ ] OAuth
- [ ] 其他

## 实现计划

### 配置结构

```go
type Config struct {
    // 通用配置
    Model       string     // 模型名称（必填）
    Temperature *float32   // 温度参数
    MaxTokens   *int       // 最大token数
    TopP        *float32   // Top P
    Timeout     Duration   // 超时时间
    
    // 千帆特定配置
    APIKey  string         // API密钥（必填）
    BaseURL string         // API端点（可选）
}
```

### 接口实现

- [ ] Provider接口实现
  - [ ] Name() → "qianfan"
  - [ ] CreateModel()
  - [ ] CreateToolCallingModel()
  - [ ] ValidateConfig()
  - [ ] ListModels()

### 支持的模型

| 模型ID | 显示名称 | 上下文 | 能力 |
|--------|---------|--------|------|
| glm-5 | GLM-5 | 198K | chat, tools |
| minimax-m2.5 | MiniMax-M2.5 | 192K | chat, tools |
| kimi-k2.5 | Kimi-K2.5 | 256K | chat, tools |

### 测试计划

- [ ] 配置验证测试
- [ ] 模型创建测试
- [ ] API兼容性测试
- [ ] 流式响应测试
- [ ] 工具调用测试

## 参考实现

- OpenAI Provider实现（本项目）
- 千帆API文档：https://cloud.baidu.com/doc/WENXINWORKSHOP/index.html

## 验收标准

- [ ] API兼容OpenAI格式
- [ ] 支持主流国产模型
- [ ] 测试覆盖率 ≥ 80%
- [ ] 有使用示例

## 检查清单

- [ ] 已阅读Provider开发指南
- [ ] API文档已准备好
- [ ] 测试环境已就绪
- [ ] 符合Provider接口规范
```

**标签：** `provider`, `qianfan`, `phase-1`
**Phase:** Phase 1
**Priority:** P1
**Size:** M

---

### Issue #4: Message Builder实现

**标题：**
```
[FEATURE] 实现Message Builder
```

**内容（直接复制粘贴）：**
```markdown
## 功能描述

提供流畅的消息构建接口，简化对话消息的创建，支持文本和多模态内容。

## 动机

直接使用eino schema.Message构建消息比较繁琐，需要一个更友好的Builder接口。

## 详细说明

### 前置条件
- 了解 eino schema.Message 结构
- 理解 Message 的 Role 和 Content 类型

### 业务流程

**Builder 方法列表：**

1. **NewBuilder()** - 创建新的构建器
2. **System(content string)** - 添加系统消息
3. **User(content string)** - 添加用户消息
4. **UserWithImages(content string, images []ImageContent)** - 添加带图片的用户消息
5. **Assistant(content string)** - 添加助手消息
6. **Tool(toolCallID, content string)** - 添加工具结果消息
7. **Build()** - 返回构建的消息列表

**ImageContent 结构：**
```go
type ImageContent struct {
    Data     []byte // 图片数据（base64解码后）
    MimeType string // MIME类型，如 "image/jpeg"
}
```

### 使用示例

```go
// 简单对话
msgs := message.NewBuilder().
    System("You are a helpful assistant.").
    User("Hello!").
    Build()

// 带图片的消息
msgs := message.NewBuilder().
    System("You are a helpful assistant.").
    UserWithImages("What's in this image?", []message.ImageContent{
        {Data: imageData, MimeType: "image/jpeg"},
    }).
    Build()

// 多轮对话
msgs := message.NewBuilder().
    System("You are a helpful assistant.").
    User("Hi").
    Assistant("Hello! How can I help you?").
    User("Tell me a joke").
    Build()
```

### 预期结果

- 支持链式调用，代码简洁
- 类型安全，编译时检查
- 支持所有消息类型
- 支持多模态内容

## 实现建议

**文件结构：**
```
pkg/message/
├── builder.go       # Builder实现
├── builder_test.go  # 单元测试
└── example_test.go  # 使用示例
```

**关键设计点：**
- Builder 持有消息切片
- 每个方法返回 *Builder 支持链式调用
- Build() 返回消息切片的副本，避免外部修改

## 验收标准

- [ ] 支持所有消息类型（system/user/assistant/tool）
- [ ] 支持多模态消息（图片）
- [ ] 链式调用流畅
- [ ] 类型安全
- [ ] 测试覆盖率 ≥ 80%
- [ ] 有使用示例

## 参考资料

- eino schema：https://github.com/cloudwego/eino/tree/main/schema
- pi agent message builder：https://github.com/earendil-works/pi

## 检查清单

- [ ] 已阅读CONTRIBUTING.md
- [ ] 已搜索相关Issue
- [ ] 功能符合项目范围
```

**标签：** `feature`, `message`, `phase-1`
**Phase:** Phase 1
**Priority:** P1
**Size:** S

---

## 第四步：创建视图

### 4.1 按阶段视图

1. 点击 **"New view"**
2. 视图名称：`By Phase`
3. Group by: `Phase`
4. 保存

### 4.2 按优先级视图

1. 点击 **"New view"**
2. 视图名称：`By Priority`
3. Group by: `Priority`
4. 保存

### 4.3 时间线视图

1. 点击 **"New view"**
2. 视图名称：`Roadmap`
3. Layout: 选择 `Roadmap`
4. Start date: 选择开始日期字段
5. End date: 选择结束日期字段
6. 保存

---

## 第五步：添加到看板

创建Issue后，在Project页面：

1. 点击 **"Add item"**
2. 选择刚创建的Issue
3. 设置字段：
   - Status: `Todo`
   - Phase: `Phase 1`
   - Priority: `P0` 或 `P1`
   - Size: `M` 或 `S`

---

## 快速操作清单

**创建顺序：**

- [ ] 1. 创建Project（Team planning模板）
- [ ] 2. 添加3个自定义字段（Phase、Priority、Size）
- [ ] 3. 创建Issue #1 - Provider注册中心
- [ ] 4. 创建Issue #2 - OpenAI Provider
- [ ] 5. 创建Issue #3 - 百度千帆Provider
- [ ] 6. 创建Issue #4 - Message Builder
- [ ] 7. 创建3个视图（By Phase、By Priority、Roadmap）
- [ ] 8. 将Issue添加到看板并设置字段

---

**准备好开始了吗？打开浏览器访问：**

```
https://github.com/yxkaze/pi-agent
```

然后点击 **Projects** 标签开始创建！
