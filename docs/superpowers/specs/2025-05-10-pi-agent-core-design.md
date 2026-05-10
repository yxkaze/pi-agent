# Pi Agent Core - Product Requirements Document (PRD)

**Version:** 1.0  
**Date:** 2025-05-10  
**Author:** AI Agent (with user collaboration)  
**Status:** Draft

---

## 1. Executive Summary

Pi Agent Core is a modular, extensible AI agent framework built with Go and the eino framework. The first phase focuses on LLM integration abstraction, providing a unified interface for multiple LLM providers (OpenAI, Chinese domestic models, and local models) based on the OpenAI API standard.

### Key Principles

- **Modularity**: Each component in its own package, single responsibility
- **Clarity**: No god objects, no monolithic files, clear separation of concerns
- **Extensibility**: Easy to add new providers, tools, and capabilities
- **Go Idiomatic**: Follow Go best practices and conventions

---

## 2. Project Scope

### Phase 1: LLM Integration (Current Focus)

**In Scope:**
- Provider abstraction layer with unified interface
- OpenAI provider implementation
- Chinese domestic model support (Baidu Qianfan)
- Local model support (Ollama)
- Basic message handling and conversation management
- Streaming response support
- Tool calling capability

**Out of Scope:**
- Advanced agent orchestration
- Multi-agent coordination
- Complex workflow composition
- Web UI / Terminal UI
- Session persistence
- Skills system implementation

---

## 3. Architecture Overview

### 3.1 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Application Layer                       │
│              (pi-coding-agent, other agents)                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Pi Agent Core                           │
│  ┌──────────────┐  ┌──────────┐  ┌─────────┐  ┌─────────┐  │
│  │   Provider   │  │  Message │  │  Tools  │  │  Agent  │  │
│  │   Registry   │  │  Builder │  │  System │  │  Core   │  │
│  └──────────────┘  └──────────┘  └─────────┘  └─────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Eino Framework                           │
│        BaseChatModel, ToolCallingChatModel, Schema           │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Package Structure

```
pi-agent/
├── go.mod                    # Go module definition
├── go.sum                    # Dependency checksums
├── README.md                 # Project documentation
├── LICENSE                   # Apache 2.0 License
│
├── pkg/                      # Public packages
│   ├── provider/             # LLM provider abstraction
│   │   ├── provider.go       # Provider interface & registry
│   │   ├── config.go         # Configuration structures
│   │   ├── errors.go         # Provider-specific errors
│   │   ├── openai/           # OpenAI implementation
│   │   │   ├── provider.go
│   │   │   ├── config.go
│   │   │   └── provider_test.go
│   │   ├── qianfan/          # Baidu Qianfan implementation
│   │   │   ├── provider.go
│   │   │   ├── config.go
│   │   │   └── provider_test.go
│   │   └── ollama/           # Ollama implementation
│   │       ├── provider.go
│   │       ├── config.go
│   │       └── provider_test.go
│   │
│   ├── message/              # Message utilities
│   │   ├── builder.go        # Fluent message builder
│   │   ├── convert.go        # Format conversion utilities
│   │   └── builder_test.go
│   │
│   ├── tools/                # Tool system (Phase 2)
│   │   ├── tool.go           # Tool interface
│   │   ├── registry.go       # Tool registry
│   │   └── builtin/          # Built-in tools
│   │
│   └── agent/                # Agent core (Phase 2)
│       ├── agent.go          # Agent interface
│       ├── state.go          # State management
│       └── executor.go       # Execution engine
│
├── internal/                 # Internal packages
│   └── testutil/             # Testing utilities
│       └── mock.go
│
├── examples/                 # Usage examples
│   ├── basic/                # Basic usage examples
│   │   ├── main.go
│   │   └── README.md
│   └── streaming/            # Streaming examples
│       ├── main.go
│       └── README.md
│
└── docs/                     # Documentation
    ├── ARCHITECTURE.md       # Architecture decisions
    ├── PROVIDER.md           # Provider development guide
    └── examples/             # Code examples
        └── quickstart.md
```

---

## 4. Component Specifications

### 4.1 Provider Package

#### 4.1.1 Core Interface

**File:** `pkg/provider/provider.go`

```go
package provider

import (
    "context"
    "github.com/cloudwego/eino/components/model"
    "github.com/cloudwego/eino/schema"
)

// Provider is the top-level factory for creating chat model instances.
// Each provider implementation (OpenAI, Qianfan, Ollama) implements this interface.
type Provider interface {
    // Name returns the provider's unique identifier (e.g., "openai", "qianfan")
    Name() string
    
    // CreateModel creates a BaseChatModel instance with the given configuration.
    // The config parameter must be of the provider's specific Config type.
    CreateModel(ctx context.Context, config interface{}) (model.BaseChatModel, error)
    
    // CreateToolCallingModel creates a ToolCallingChatModel with tool support.
    CreateToolCallingModel(ctx context.Context, config interface{}) (model.ToolCallingChatModel, error)
    
    // ValidateConfig checks if the configuration is valid for this provider.
    ValidateConfig(config interface{}) error
    
    // ListModels returns available models for this provider (optional, can return empty list)
    ListModels(ctx context.Context) ([]ModelInfo, error)
}

// ModelInfo describes a model's capabilities and metadata
type ModelInfo struct {
    ID          string            // Model identifier (e.g., "gpt-4o")
    Name        string            // Display name
    Provider    string            // Provider name
    MaxTokens   int               // Maximum context length
    Capabilities []string         // ["chat", "tools", "vision", etc.]
    Metadata    map[string]string // Additional provider-specific info
}

// Registry manages all registered providers
type Registry struct {
    providers map[string]Provider
}

// NewRegistry creates a new provider registry
func NewRegistry() *Registry

// Register adds a provider to the registry
func (r *Registry) Register(p Provider) error

// Get retrieves a provider by name
func (r *Registry) Get(name string) (Provider, error)

// List returns all registered provider names
func (r *Registry) List() []string
```

#### 4.1.2 Configuration Structure

**File:** `pkg/provider/config.go`

```go
package provider

import "time"

// CommonConfig contains shared configuration fields for all providers
type CommonConfig struct {
    // Model is the model identifier
    Model string
    
    // Temperature controls randomness (0.0-2.0)
    Temperature *float32
    
    // MaxTokens limits the response length
    MaxTokens *int
    
    // TopP controls diversity via nucleus sampling
    TopP *float32
    
    // Timeout for API requests
    Timeout time.Duration
    
    // BaseURL allows overriding the default API endpoint
    BaseURL string
}

// ModelConfig is the standard configuration for creating a model instance
type ModelConfig struct {
    // Provider specifies which provider to use ("openai", "qianfan", "ollama")
    Provider string
    
    // Common fields shared across providers
    CommonConfig
    
    // Provider-specific configuration (APIKey, etc.)
    // Must be type-asserted by each provider implementation
    ProviderConfig interface{}
}
```

#### 4.1.3 OpenAI Provider Implementation

**File:** `pkg/provider/openai/provider.go`

```go
package openai

import (
    "context"
    "fmt"
    
    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/components/model"
    "github.com/cloudwego/eino/schema"
    
    piprovider "pi-agent/pkg/provider"
)

const ProviderName = "openai"

type Provider struct{}

func New() *Provider {
    return &Provider{}
}

func (p *Provider) Name() string {
    return ProviderName
}

type Config struct {
    piprovider.CommonConfig
    
    // APIKey is the OpenAI API key (required)
    APIKey string
    
    // Organization is the OpenAI organization ID (optional)
    Organization string
}

func (p *Provider) CreateModel(ctx context.Context, config interface{}) (model.BaseChatModel, error) {
    cfg, ok := config.(*Config)
    if !ok {
        return nil, fmt.Errorf("invalid config type: expected *openai.Config")
    }
    
    if err := p.ValidateConfig(cfg); err != nil {
        return nil, err
    }
    
    opts := []openai.Option{
        openai.WithModel(cfg.Model),
        openai.WithAPIKey(cfg.APIKey),
    }
    
    if cfg.BaseURL != "" {
        opts = append(opts, openai.WithBaseURL(cfg.BaseURL))
    }
    if cfg.Organization != "" {
        opts = append(opts, openai.WithOrganization(cfg.Organization))
    }
    if cfg.Temperature != nil {
        opts = append(opts, openai.WithTemperature(*cfg.Temperature))
    }
    if cfg.MaxTokens != nil {
        opts = append(opts, openai.WithMaxTokens(*cfg.MaxTokens))
    }
    if cfg.TopP != nil {
        opts = append(opts, openai.WithTopP(*cfg.TopP))
    }
    if cfg.Timeout > 0 {
        opts = append(opts, openai.WithTimeout(cfg.Timeout))
    }
    
    return openai.NewChatModel(ctx, opts...)
}

func (p *Provider) CreateToolCallingModel(ctx context.Context, config interface{}) (model.ToolCallingChatModel, error) {
    // For OpenAI, BaseChatModel implements ToolCallingChatModel
    m, err := p.CreateModel(ctx, config)
    if err != nil {
        return nil, err
    }
    
    // Type assertion is safe for OpenAI as it implements ToolCallingChatModel
    return m.(model.ToolCallingChatModel), nil
}

func (p *Provider) ValidateConfig(config interface{}) error {
    cfg, ok := config.(*Config)
    if !ok {
        return fmt.Errorf("invalid config type: expected *openai.Config")
    }
    
    if cfg.APIKey == "" {
        return fmt.Errorf("APIKey is required")
    }
    if cfg.Model == "" {
        return fmt.Errorf("Model is required")
    }
    
    return nil
}

func (p *Provider) ListModels(ctx context.Context) ([]piprovider.ModelInfo, error) {
    // Return common OpenAI models
    return []piprovider.ModelInfo{
        {ID: "gpt-4o", Name: "GPT-4o", Provider: ProviderName, MaxTokens: 128000, Capabilities: []string{"chat", "tools", "vision"}},
        {ID: "gpt-4o-mini", Name: "GPT-4o Mini", Provider: ProviderName, MaxTokens: 128000, Capabilities: []string{"chat", "tools"}},
        {ID: "gpt-4-turbo", Name: "GPT-4 Turbo", Provider: ProviderName, MaxTokens: 128000, Capabilities: []string{"chat", "tools", "vision"}},
        {ID: "gpt-3.5-turbo", Name: "GPT-3.5 Turbo", Provider: ProviderName, MaxTokens: 16385, Capabilities: []string{"chat", "tools"}},
    }, nil
}
```

#### 4.1.4 Baidu Qianfan Provider

**File:** `pkg/provider/qianfan/provider.go`

```go
package qianfan

import (
    "context"
    "fmt"
    
    "github.com/cloudwego/eino-ext/components/model/openai" // Qianfan is OpenAI-compatible
    "github.com/cloudwego/eino/components/model"
    
    piprovider "pi-agent/pkg/provider"
)

const ProviderName = "qianfan"

type Provider struct{}

func New() *Provider {
    return &Provider{}
}

func (p *Provider) Name() string {
    return ProviderName
}

type Config struct {
    piprovider.CommonConfig
    
    // APIKey is the Baidu Qianfan API key (required)
    APIKey string
    
    // BaseURL is the Qianfan API endpoint
    // Default: https://qianfan.baidubce.com/v2
    BaseURL string
}

func (p *Provider) CreateModel(ctx context.Context, config interface{}) (model.BaseChatModel, error) {
    cfg, ok := config.(*Config)
    if !ok {
        return nil, fmt.Errorf("invalid config type: expected *qianfan.Config")
    }
    
    if err := p.ValidateConfig(cfg); err != nil {
        return nil, err
    }
    
    baseURL := cfg.BaseURL
    if baseURL == "" {
        baseURL = "https://qianfan.baidubce.com/v2"
    }
    
    opts := []openai.Option{
        openai.WithModel(cfg.Model),
        openai.WithAPIKey(cfg.APIKey),
        openai.WithBaseURL(baseURL),
    }
    
    if cfg.Temperature != nil {
        opts = append(opts, openai.WithTemperature(*cfg.Temperature))
    }
    if cfg.MaxTokens != nil {
        opts = append(opts, openai.WithMaxTokens(*cfg.MaxTokens))
    }
    
    return openai.NewChatModel(ctx, opts...)
}

func (p *Provider) CreateToolCallingModel(ctx context.Context, config interface{}) (model.ToolCallingChatModel, error) {
    m, err := p.CreateModel(ctx, config)
    if err != nil {
        return nil, err
    }
    return m.(model.ToolCallingChatModel), nil
}

func (p *Provider) ValidateConfig(config interface{}) error {
    cfg, ok := config.(*Config)
    if !ok {
        return fmt.Errorf("invalid config type: expected *qianfan.Config")
    }
    
    if cfg.APIKey == "" {
        return fmt.Errorf("APIKey is required")
    }
    if cfg.Model == "" {
        return fmt.Errorf("Model is required")
    }
    
    return nil
}

func (p *Provider) ListModels(ctx context.Context) ([]piprovider.ModelInfo, error) {
    return []piprovider.ModelInfo{
        {ID: "glm-5", Name: "GLM-5", Provider: ProviderName, MaxTokens: 198000, Capabilities: []string{"chat", "tools"}},
        {ID: "minimax-m2.5", Name: "MiniMax-M2.5", Provider: ProviderName, MaxTokens: 192000, Capabilities: []string{"chat", "tools"}},
        {ID: "kimi-k2.5", Name: "Kimi-K2.5", Provider: ProviderName, MaxTokens: 256000, Capabilities: []string{"chat", "tools"}},
    }, nil
}
```

### 4.2 Message Builder

**File:** `pkg/message/builder.go`

```go
package message

import (
    "github.com/cloudwego/eino/schema"
)

// Builder provides a fluent interface for constructing messages
type Builder struct {
    messages []*schema.Message
}

// NewBuilder creates a new message builder
func NewBuilder() *Builder {
    return &Builder{
        messages: make([]*schema.Message, 0),
    }
}

// System adds a system message
func (b *Builder) System(content string) *Builder {
    b.messages = append(b.messages, &schema.Message{
        Role:    schema.System,
        Content: content,
    })
    return b
}

// User adds a user message
func (b *Builder) User(content string) *Builder {
    b.messages = append(b.messages, &schema.Message{
        Role:    schema.User,
        Content: content,
    })
    return b
}

// UserWithImages adds a user message with images
func (b *Builder) UserWithImages(content string, images []ImageContent) *Builder {
    parts := []schema.ContentPart{
        &schema.TextContent{Text: content},
    }
    for _, img := range images {
        parts = append(parts, &schema.ImageContent{
            Data:     img.Data,
           MimeType: img.MimeType,
        })
    }
    
    b.messages = append(b.messages, &schema.Message{
        Role:    schema.User,
        Content: parts,
    })
    return b
}

// Assistant adds an assistant message
func (b *Builder) Assistant(content string) *Builder {
    b.messages = append(b.messages, &schema.Message{
        Role:    schema.Assistant,
        Content: content,
    })
    return b
}

// Tool adds a tool result message
func (b *Builder) Tool(toolCallID, content string) *Builder {
    b.messages = append(b.messages, &schema.Message{
        Role:       schema.Tool,
        Content:    content,
        ToolCallID: toolCallID,
    })
    return b
}

// Build returns the constructed messages
func (b *Builder) Build() []*schema.Message {
    return b.messages
}

// ImageContent represents an image in a message
type ImageContent struct {
    Data     []byte // Image data (base64-decoded)
    MimeType string // "image/jpeg", "image/png", etc.
}
```

### 4.3 Error Handling

**File:** `pkg/provider/errors.go`

```go
package provider

import "fmt"

// Error represents a provider-specific error
type Error struct {
    Provider string // Provider name
    Code     string // Error code
    Message  string // Error message
    Cause    error  // Underlying error (if any)
}

func (e *Error) Error() string {
    if e.Cause != nil {
        return fmt.Sprintf("[%s] %s: %s (cause: %v)", e.Provider, e.Code, e.Message, e.Cause)
    }
    return fmt.Sprintf("[%s] %s: %s", e.Provider, e.Code, e.Message)
}

func (e *Error) Unwrap() error {
    return e.Cause
}

// Common error codes
const (
    ErrCodeInvalidConfig = "INVALID_CONFIG"
    ErrCodeMissingAPIKey = "MISSING_API_KEY"
    ErrCodeInvalidModel  = "INVALID_MODEL"
    ErrCodeRequestFailed = "REQUEST_FAILED"
    ErrCodeRateLimited   = "RATE_LIMITED"
    ErrCodeTimeout       = "TIMEOUT"
)

// NewError creates a new provider error
func NewError(provider, code, message string, cause error) *Error {
    return &Error{
        Provider: provider,
        Code:     code,
        Message:  message,
        Cause:    cause,
    }
}
```

---

## 5. Data Flow

### 5.1 Basic Chat Flow

```
User Code
    │
    ▼
Create Provider Instance (Provider.New())
    │
    ▼
Create Model Instance (Provider.CreateModel())
    │
    ▼
Build Messages (message.Builder)
    │
    ▼
Call Model (BaseChatModel.Generate/Stream)
    │
    ▼
Receive Response (*schema.Message)
    │
    ▼
User Code (process response)
```

### 5.2 Tool Calling Flow

```
User Code
    │
    ▼
Create Provider Instance
    │
    ▼
Create ToolCallingChatModel (Provider.CreateToolCallingModel())
    │
    ▼
Bind Tools (ToolCallingChatModel.WithTools())
    │
    ▼
Build Messages
    │
    ▼
Call Model
    │
    ▼
Check Response for ToolCalls
    │
    ├─ Has ToolCalls?
    │     │
    │     ▼
    │  Execute Tool
    │     │
    │     ▼
    │  Add ToolResult Message
    │     │
    │     └─> Loop back to "Call Model"
    │
    └─ No ToolCalls → Return Final Response
```

---

## 6. Testing Strategy

### 6.1 Unit Tests

**Provider Tests:**
- Test configuration validation
- Test model creation with various configs
- Test error handling for invalid inputs

**Message Builder Tests:**
- Test message construction
- Test multi-modal content
- Test edge cases (empty content, etc.)

### 6.2 Integration Tests

- Test actual API calls (with mock server or real API keys in CI secrets)
- Test streaming responses
- Test tool calling workflow
- Test multiple providers interchangeably

### 6.3 Test Coverage Goals

- Minimum 80% code coverage
- All error paths must be tested
- All public APIs must have examples

---

## 7. Dependencies

### 7.1 Core Dependencies

- **go 1.18+**: Language version
- **github.com/cloudwego/eino**: Core framework (BaseChatModel, Schema, etc.)
- **github.com/cloudwego/eino-ext**: Provider implementations

### 7.2 Development Dependencies

- **golangci-lint**: Linting and code quality
- **mockgen**: Mock generation for testing
- **gotestsum**: Improved test output

---

## 8. Migration Path

### Phase 1 (Current)
- Provider abstraction
- OpenAI, Qianfan, Ollama support
- Message builder
- Basic tool calling

### Phase 2
- Agent core with state management
- Tool registry
- Advanced conversation management
- Session persistence

### Phase 3
- Skills system
- Multi-agent coordination
- Advanced workflow composition
- UI components

---

## 9. Success Criteria

### Phase 1 Success Metrics

1. **Functionality**
   - ✅ Successfully call OpenAI API
   - ✅ Successfully call Qianfan API
   - ✅ Successfully call Ollama API
   - ✅ Streaming responses work correctly
   - ✅ Tool calling works correctly

2. **Code Quality**
   - ✅ All packages have < 500 lines per file
   - ✅ Clear separation of concerns
   - ✅ No circular dependencies
   - ✅ 80%+ test coverage

3. **Documentation**
   - ✅ README with quickstart
   - ✅ Working code examples
   - ✅ Provider development guide

4. **Developer Experience**
   - ✅ Easy to add new provider (< 30 minutes)
   - ✅ Clear error messages
   - ✅ Intuitive API

---

## 10. Open Questions

1. **Configuration Management**
   - Should we support environment variable-based configuration?
   - How to handle API key rotation?

2. **Error Recovery**
   - Should providers implement automatic retry logic?
   - How to handle rate limiting gracefully?

3. **Performance**
   - Should we implement connection pooling?
   - How to handle concurrent requests efficiently?

4. **Observability**
   - Should we add logging middleware?
   - How to track API usage and costs?

---

## 11. Timeline Estimate

### Phase 1: LLM Integration (2-3 weeks)

**Week 1:**
- Provider interface design
- OpenAI provider implementation
- Message builder
- Basic tests

**Week 2:**
- Qianfan provider implementation
- Ollama provider implementation
- Error handling
- Integration tests

**Week 3:**
- Documentation
- Examples
- Code review and refinement
- Release preparation

---

## 12. Risks and Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| API changes in eino | High | Low | Pin version, monitor upstream changes |
| Provider API incompatibilities | Medium | Medium | Extensive integration testing |
| Performance issues with streaming | Medium | Low | Benchmark early, optimize if needed |
| Configuration complexity | Low | Medium | Keep config minimal, use sensible defaults |

---

## 13. Appendix

### A. Example Usage

```go
package main

import (
    "context"
    "fmt"
    "log"
    
    "pi-agent/pkg/message"
    "pi-agent/pkg/provider"
    "pi-agent/pkg/provider/openai"
)

func main() {
    // Create provider registry
    registry := provider.NewRegistry()
    
    // Register providers
    registry.Register(openai.New())
    
    // Get OpenAI provider
    p, err := registry.Get("openai")
    if err != nil {
        log.Fatal(err)
    }
    
    // Create model
    model, err := p.CreateModel(context.Background(), &openai.Config{
        APIKey: "sk-...",
        Model:  "gpt-4o",
    })
    if err != nil {
        log.Fatal(err)
    }
    
    // Build messages
    msgs := message.NewBuilder().
        System("You are a helpful assistant.").
        User("Hello!").
        Build()
    
    // Generate response
    response, err := model.Generate(context.Background(), msgs)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println(response.Content)
}
```

### B. Tool Calling Example

```go
// Create tool-calling model
toolModel, err := p.CreateToolCallingModel(ctx, &openai.Config{
    APIKey: "sk-...",
    Model:  "gpt-4o",
})

// Define tools
weatherTool := &schema.ToolInfo{
    Name: "get_weather",
    Desc: "Get current weather for a location",
    ParamsOneOf: schema.NewParamsOneOfByParams(map[string]any{
        "location": map[string]any{
            "type": "string",
            "description": "City name",
        },
    }),
}

// Bind tool
toolModel, err = toolModel.WithTools([]*schema.ToolInfo{weatherTool})
if err != nil {
    log.Fatal(err)
}

// Use the model...
```

---

**End of PRD**
