# API参考

<cite>
**本文档中引用的文件**  
- [account.proto](file://proto/cline/account.proto)
- [browser.proto](file://proto/cline/browser.proto)
- [file.proto](file://proto/cline/file.proto)
- [mcp.proto](file://proto/cline/mcp.proto)
- [models.proto](file://proto/cline/models.proto)
- [task.proto](file://proto/cline/task.proto)
- [ui.proto](file://proto/cline/ui.proto)
- [anthropic.ts](file://src/core/api/providers/anthropic.ts)
- [gemini.ts](file://src/core/api/providers/gemini.ts)
- [openai.ts](file://src/core/api/providers/openai.ts)
- [together.ts](file://src/core/api/providers/together.ts)
- [mistral.ts](file://src/core/api/providers/mistral.ts)
- [groq.ts](file://src/core/api/providers/groq.ts)
- [huggingface.ts](file://src/core/api/providers/huggingface.ts)
- [doubao.ts](file://src/core/api/providers/doubao.ts)
- [qwen.ts](file://src/core/api/providers/qwen.ts)
- [xai.ts](file://src/core/api/providers/xai.ts)
- [zai.ts](file://src/core/api/providers/zai.ts)
- [nebius.ts](file://src/core/api/providers/nebius.ts)
- [fireworks.ts](file://src/core/api/providers/fireworks.ts)
- [baseten.ts](file://src/core/api/providers/baseten.ts)
- [cerebras.ts](file://src/core/api/providers/cerebras.ts)
- [huawei-cloud-maas.ts](file://src/core/api/providers/huawei-cloud-maas.ts)
- [sambanova.ts](file://src/core/api/providers/sambanova.ts)
- [sapaicore.ts](file://src/core/api/providers/sapaicore.ts)
- [litellm.ts](file://src/core/api/providers/litellm.ts)
- [ollama.ts](file://src/core/api/providers/ollama.ts)
- [lmstudio.ts](file://src/core/api/providers/lmstudio.ts)
- [vscode-lm.ts](file://src/core/api/providers/vscode-lm.ts)
- [requesty.ts](file://src/core/api/providers/requesty.ts)
- [vercel-ai-gateway.ts](file://src/core/api/providers/vercel-ai-gateway.ts)
- [vertex.ts](file://src/core/api/providers/vertex.ts)
- [asksage.ts](file://src/core/api/providers/asksage.ts)
- [deepseek.ts](file://src/core/api/providers/deepseek.ts)
- [dify.ts](file://src/core/api/providers/dify.ts)
- [openrouter.ts](file://src/core/api/providers/openrouter.ts)
- [moonshot.ts](file://src/core/api/providers/moonshot.ts)
- [bedrock.ts](file://src/core/api/providers/bedrock.ts)
- [openai-native.ts](file://src/core/api/providers/openai-native.ts)
- [claude-code.ts](file://src/core/api/providers/claude-code.ts)
- [qwen-code.ts](file://src/core/api/providers/qwen-code.ts)
- [cline.ts](file://src/core/api/providers/cline.ts)
</cite>

## 目录
1. [gRPC API](#grpc-api)
2. [LLM API](#llm-api)
3. [MCP API](#mcp-api)
4. [错误码说明](#错误码说明)
5. [最佳实践](#最佳实践)

## gRPC API

Cline通过gRPC协议暴露了一系列服务，用于与核心功能进行交互。每个服务定义在`proto/cline/`目录下的`.proto`文件中。

### AccountService

提供账户管理功能，包括登录、登出、认证状态监听和信用额度查询。

**RPC方法：**

- `accountLoginClicked`: 触发登录流程，打开外部浏览器进行身份验证。
- `accountLogoutClicked`: 清除用户状态并登出。
- `subscribeToAuthStatusUpdate`: 订阅认证状态变更事件（流式响应）。
- `authStateChanged`: 处理由Firebase上下文触发的认证状态变更。
- `getUserCredits`: 获取用户的信用额度、使用记录和支付记录。
- `getOrganizationCredits`: 获取指定组织的信用额度和使用记录。
- `getUserOrganizations`: 获取用户所属的所有组织列表。
- `setUserOrganization`: 设置当前活跃的组织。
- `openrouterAuthClicked`: 触发OpenRouter认证流程。

**消息结构：**
- `AuthStateChangedRequest`: 包含元数据和用户信息。
- `UserInfo`: 用户基本信息（UID、邮箱、头像等）。
- `UserOrganization`: 用户组织信息（ID、名称、角色等）。
- `UserCreditsData`: 用户信用数据，包含余额、使用和支付交易记录。
- `OrganizationCreditsData`: 组织信用数据。

**调用上下文：**
此服务通常由UI组件调用，用于处理用户身份验证和账户信息展示。`subscribeToAuthStatusUpdate`用于实时更新UI中的登录状态。

**Section sources**
- [account.proto](file://proto/cline/account.proto#L15-L132)

### BrowserService

提供与浏览器交互的功能，包括连接信息获取、连接测试和浏览器发现。

**RPC方法：**

- `getBrowserConnectionInfo`: 获取当前浏览器连接状态（是否连接、是否远程）。
- `testBrowserConnection`: 测试指定端点的浏览器连接。
- `discoverBrowser`: 自动发现本地Chrome浏览器实例。
- `getDetectedChromePath`: 获取检测到的Chrome可执行文件路径。
- `relaunchChromeDebugMode`: 重启Chrome以启用调试模式。

**消息结构：**
- `BrowserConnectionInfo`: 浏览器连接信息，包含连接状态和主机地址。
- `BrowserConnection`: 连接测试结果，包含成功标志和消息。
- `ChromePath`: Chrome可执行文件路径信息。

**调用上下文：**
该服务在需要与浏览器进行自动化交互时被调用，例如执行网页操作或调试任务。`discoverBrowser`和`relaunchChromeDebugMode`常用于初始化浏览器环境。

**Section sources**
- [browser.proto](file://proto/cline/browser.proto#L15-L51)

### FileService

提供文件系统操作功能，包括文件打开、搜索、规则管理等。

**RPC方法：**

- `copyToClipboard`: 将文本复制到剪贴板。
- `openFile`: 在编辑器中打开文件。
- `openImage`: 在系统查看器中打开图片。
- `openMention`: 打开提及的资源（文件、URL等）。
- `deleteRuleFile`: 删除规则文件。
- `createRuleFile`: 创建规则文件。
- `searchCommits`: 搜索Git提交记录。
- `selectFiles`: 从文件系统选择文件。
- `getRelativePaths`: 将URI转换为工作区相对路径。
- `searchFiles`: 在工作区中模糊搜索文件。
- `toggleClineRule`: 切换Cline规则的启用状态。
- `toggleCursorRule`: 切换Cursor规则的启用状态。
- `toggleWindsurfRule`: 切换Windsurf规则的启用状态。
- `refreshRules`: 刷新所有规则的开关状态。
- `openTaskHistory`: 打开任务的历史记录文件。
- `toggleWorkflow`: 切换工作流的启用状态。
- `ifFileExistsRelativePath`: 检查相对路径下的文件是否存在。
- `openFileRelativePath`: 通过相对路径在编辑器中打开文件。
- `openFocusChainFile`: 打开或创建焦点链检查清单文件。

**消息结构：**
- `RuleFileRequest`: 统一的规则文件操作请求，包含是否为全局规则、规则路径等。
- `RuleFile`: 规则文件操作结果，包含文件路径和显示名称。
- `FileSearchRequest`: 文件搜索请求，包含查询字符串和类型过滤器。
- `FileSearchResults`: 文件搜索结果，包含匹配的文件/文件夹列表。
- `GitCommits`: Git提交记录列表。

**调用上下文：**
此服务被广泛用于各种文件操作场景，如代码编辑、任务执行和规则管理。`searchFiles`和`openFile`是高频调用方法，用于实现代码导航和文件操作。

**Section sources**
- [file.proto](file://proto/cline/file.proto#L15-L186)

### McpService

提供MCP（Model Control Protocol）服务器和工具的管理功能。

**RPC方法：**

- `toggleMcpServer`: 启用或禁用MCP服务器。
- `updateMcpTimeout`: 更新MCP服务器的超时设置。
- `addRemoteMcpServer`: 添加远程MCP服务器。
- `downloadMcp`: 下载MCP服务器。
- `restartMcpServer`: 重启MCP服务器。
- `deleteMcpServer`: 删除MCP服务器。
- `toggleToolAutoApprove`: 切换工具的自动批准状态。
- `refreshMcpMarketplace`: 刷新MCP市场目录。
- `openMcpSettings`: 打开MCP设置界面。
- `subscribeToMcpMarketplaceCatalog`: 订阅MCP市场目录更新（流式响应）。
- `getLatestMcpServers`: 获取最新的MCP服务器列表。
- `subscribeToMcpServers`: 订阅MCP服务器更新（流式响应）。

**消息结构：**
- `McpServer`: MCP服务器信息，包含名称、配置、状态、工具列表等。
- `McpTool`: MCP工具信息，包含名称、描述、输入模式和自动批准状态。
- `McpResource`: MCP资源信息。
- `McpMarketplaceItem`: MCP市场项目信息，包含ID、GitHub URL、作者、描述等。
- `McpMarketplaceCatalog`: MCP市场目录，包含项目列表。

**调用上下文：**
该服务用于集成和管理外部AI工具和服务器。`subscribeToMcpMarketplaceCatalog`用于实时更新市场内容，`toggleMcpServer`用于启用/禁用特定的MCP服务器。

**Section sources**
- [mcp.proto](file://proto/cline/mcp.proto#L15-L132)

### ModelsService

提供模型管理功能，包括获取可用模型列表和更新API配置。

**RPC方法：**

- `getOllamaModels`: 获取Ollama可用模型列表。
- `getLmStudioModels`: 获取LM Studio可用模型列表。
- `getVsCodeLmModels`: 获取VS Code LM API可用模型列表。
- `refreshOpenRouterModels`: 刷新并返回OpenRouter模型信息。
- `refreshHuggingFaceModels`: 刷新并返回Hugging Face模型信息。
- `refreshOpenAiModels`: 刷新并返回OpenAI模型信息。
- `refreshVercelAiGatewayModels`: 刷新并返回Vercel AI Gateway模型信息。
- `refreshRequestyModels`: 刷新并返回Requesty模型信息。
- `subscribeToOpenRouterModels`: 订阅OpenRouter模型更新（流式响应）。
- `updateApiConfigurationProto`: 更新API配置。
- `refreshGroqModels`: 刷新并返回Groq模型信息。
- `refreshBasetenModels`: 刷新并返回Baseten模型信息。
- `getSapAiCoreModels`: 获取SAP AI Core可用模型列表。

**消息结构：**
- `ModelsApiConfiguration`: 主API配置消息，包含各提供商的API密钥、基础URL等全局和模式特定配置。
- `OpenRouterModelInfo`: OpenRouter模型信息，包含上下文窗口、价格、功能支持等。
- `OpenAiCompatibleModelInfo`: OpenAI兼容模型信息。
- `LiteLLMModelInfo`: LiteLLM模型信息。
- `SapAiCoreModelsResponse`: SAP AI Core模型响应，包含部署列表和编排可用性。

**调用上下文：**
此服务是模型选择和配置的核心。`updateApiConfigurationProto`用于持久化用户的API密钥和模型偏好设置，`refresh*Models`系列方法用于动态获取各提供商的最新模型列表。

**Section sources**
- [models.proto](file://proto/cline/models.proto#L15-L349)

### TaskService

提供任务管理功能，包括任务创建、历史记录管理和反馈收集。

**RPC方法：**

- `cancelTask`: 取消当前正在运行的任务。
- `clearTask`: 清除当前任务。
- `getTotalTasksSize`: 获取所有任务的总大小。
- `deleteTasksWithIds`: 删除指定ID的任务。
- `newTask`: 创建新任务。
- `showTaskWithId`: 显示指定ID的任务。
- `exportTaskWithId`: 将指定ID的任务导出为Markdown。
- `toggleTaskFavorite`: 切换任务的收藏状态。
- `getTaskHistory`: 获取过滤后的任务历史记录。
- `askResponse`: 发送对先前询问操作的响应。
- `taskFeedback`: 记录任务反馈（点赞/点踩）。
- `taskCompletionViewChanges`: 在视图中显示任务完成变更的差异。
- `executeQuickWin`: 执行快速获胜任务。
- `deleteAllTaskHistory`: 删除所有任务历史记录。

**消息结构：**
- `NewTaskRequest`: 创建新任务的请求，包含任务文本和可选的图片/文件。
- `TaskFavoriteRequest`: 切换任务收藏状态的请求。
- `TaskResponse`: 任务详情响应。
- `GetTaskHistoryRequest`: 获取任务历史记录的请求，支持按收藏、搜索、排序等条件过滤。
- `TaskHistoryArray`: 任务历史记录数组。

**调用上下文：**
该服务是任务生命周期管理的核心。`newTask`和`cancelTask`用于任务的启动和终止，`getTaskHistory`用于在UI中展示任务列表，`taskFeedback`用于收集用户对任务结果的评价。

**Section sources**
- [task.proto](file://proto/cline/task.proto#L15-L116)

### UiService

提供UI交互功能，包括事件订阅和界面控制。

**RPC方法：**

- `scrollToSettings`: 滚动到设置视图的特定部分。
- `onDidShowAnnouncement`: 标记公告为已显示。
- `subscribeToAddToInput`: 订阅“添加到输入”事件（流式响应）。
- `subscribeToMcpButtonClicked`: 订阅MCP按钮点击事件（流式响应）。
- `subscribeToHistoryButtonClicked`: 订阅历史记录按钮点击事件（流式响应）。
- `subscribeToChatButtonClicked`: 订阅聊天按钮点击事件（流式响应）。
- `subscribeToAccountButtonClicked`: 订阅账户按钮点击事件（流式响应）。
- `subscribeToSettingsButtonClicked`: 订阅设置按钮点击事件（流式响应）。
- `subscribeToPartialMessage`: 订阅部分消息更新（流式响应），用于实时接收Cline消息。
- `initializeWebview`: 初始化Webview。
- `subscribeToRelinquishControl`: 订阅放弃控制权事件（流式响应）。
- `subscribeToFocusChatInput`: 订阅聚焦聊天输入事件（流式响应）。
- `subscribeToDidBecomeVisible`: 订阅Webview可见性变更事件（流式响应）。
- `getWebviewHtml`: 获取Webview的HTML内容（仅用于外部客户端）。
- `openUrl`: 在默认浏览器中打开URL。
- `openWalkthrough`: 打开Cline引导流程。

**消息结构：**
- `ClineMessage`: Cline消息的主类型，包含时间戳、类型、文本、图片等。
- `ClineMessageType`: 消息类型（ASK, SAY）。
- `ClineAsk`: ASK消息的子类型（FOLLOWUP, PLAN_MODE_RESPOND等）。
- `ClineSay`: SAY消息的子类型（TASK, ERROR, TEXT等）。
- `ClineSayTool`: 工具使用消息，包含工具类型、路径、差异等。

**调用上下文：**
此服务是前端UI与后端逻辑的桥梁。`subscribeToPartialMessage`是关键方法，用于实现消息的实时流式传输，`openUrl`和`openWalkthrough`用于导航和引导。

**Section sources**
- [ui.proto](file://proto/cline/ui.proto#L15-L272)

## LLM API

Cline支持多种大型语言模型提供商，通过在`src/core/api/providers/`目录下的专用处理程序进行集成。

### 配置提供商

每个LLM提供商都需要配置API密钥和基础URL（如果适用）。配置通过`ModelsService`的`updateApiConfigurationProto` RPC方法进行持久化。

#### Anthropic
- **API密钥**: `anthropic_api_key` (在`ModelsApiConfiguration`中)
- **基础URL**: `anthropic_base_url` (可选，用于自定义端点)
- **特性**: 支持Claude系列模型（Opus, Sonnet, Haiku），支持扩展思考（Extended Thinking）和1M上下文窗口（实验性）。
- **限制**: 需要有效的Anthropic账户和API密钥。

```typescript
// 示例：配置Anthropic
const config: ModelsApiConfiguration = {
  anthropic_api_key: "your-anthropic-api-key",
  anthropic_base_url: "https://api.anthropic.com", // 可选
  plan_mode_api_provider: ApiProvider.ANTHROPIC,
  plan_mode_api_model_id: "claude-3-5-sonnet-20241022"
};
```

**Section sources**
- [anthropic.ts](file://src/core/api/providers/anthropic.ts#L1-L246)

#### Google Gemini
- **API密钥**: `gemini_api_key`
- **基础URL**: `gemini_base_url` (可选)
- **特性**: 支持Gemini Pro和Flash模型，支持图像输入。
- **限制**: 需要Google Cloud项目和启用Gemini API。

```typescript
// 示例：配置Gemini
const config: ModelsApiConfiguration = {
  gemini_api_key: "your-gemini-api-key",
  gemini_base_url: "https://generativelanguage.googleapis.com", // 可选
  act_mode_api_provider: ApiProvider.GEMINI,
  act_mode_api_model_id: "gemini-1.5-pro"
};
```

**Section sources**
- [gemini.ts](file://src/core/api/providers/gemini.ts)

#### OpenAI
- **API密钥**: `open_ai_api_key`
- **基础URL**: `open_ai_base_url` (可选，用于自定义或兼容端点)
- **特性**: 支持GPT-4、GPT-3.5等系列模型，功能全面。
- **限制**: 需要有效的OpenAI账户和API密钥。

```typescript
// 示例：配置OpenAI
const config: ModelsApiConfiguration = {
  open_ai_api_key: "your-openai-api-key",
  open_ai_base_url: "https://api.openai.com/v1", // 可选
  plan_mode_api_provider: ApiProvider.OPENAI,
  plan_mode_api_model_id: "gpt-4o"
};
```

**Section sources**
- [openai.ts](file://src/core/api/providers/openai.ts)

#### Together AI
- **API密钥**: `together_api_key`
- **基础URL**: 默认为 `https://api.together.xyz/v1`
- **特性**: 提供广泛的开源模型选择，如Llama、Mixtral等。
- **限制**: 需要Together AI账户。

```typescript
// 示例：配置Together AI
const config: ModelsApiConfiguration = {
  together_api_key: "your-together-api-key",
  act_mode_api_provider: ApiProvider.TOGETHER,
  act_mode_api_model_id: "meta-llama/Llama-3-70b-chat-hf"
};
```

**Section sources**
- [together.ts](file://src/core/api/providers/together.ts)

#### Mistral AI
- **API密钥**: `mistral_api_key`
- **基础URL**: 默认为 `https://api.mistral.ai/v1`
- **特性**: 支持Mistral、Mixtral等自家模型，性能优异。
- **限制**: 需要Mistral AI账户。

```typescript
// 示例：配置Mistral AI
const config: ModelsApiConfiguration = {
  mistral_api_key: "your-mistral-api-key",
  plan_mode_api_provider: ApiProvider.MISTRAL,
  plan_mode_api_model_id: "mistral-large-latest"
};
```

**Section sources**
- [mistral.ts](file://src/core/api/providers/mistral.ts)

#### Groq
- **API密钥**: `groq_api_key`
- **基础URL**: 默认为 `https://api.groq.com/openai/v1`
- **特性**: 以极快的推理速度著称，支持Llama 3等模型。
- **限制**: 需要Groq账户。

```typescript
// 示例：配置Groq
const config: ModelsApiConfiguration = {
  groq_api_key: "your-groq-api-key",
  act_mode_api_provider: ApiProvider.GROQ,
  act_mode_api_model_id: "llama3-70b-8192"
};
```

**Section sources**
- [groq.ts](file://src/core/api/providers/groq.ts)

#### Hugging Face
- **API密钥**: `hugging_face_api_key`
- **基础URL**: `https://api-inference.huggingface.co/v1/`
- **特性**: 可访问Hugging Face Hub上的海量开源模型。
- **限制**: 需要Hugging Face账户，部分模型需要付费或申请。

```typescript
// 示例：配置Hugging Face
const config: ModelsApiConfiguration = {
  hugging_face_api_key: "your-hf-api-key",
  act_mode_api_provider: ApiProvider.HUGGINGFACE,
  act_mode_api_model_id: "mistralai/Mistral-7B-Instruct-v0.2"
};
```

**Section sources**
- [huggingface.ts](file://src/core/api/providers/huggingface.ts)

#### 其他提供商
Cline还支持以下提供商，配置方式类似：
- **Doubao (豆包)**: `doubao_api_key`
- **Qwen (通义千问)**: `qwen_api_key`
- **X.ai (Grok)**: `xai_api_key`
- **Z.AI**: `zai_api_key`
- **Nebius AI**: `nebius_api_key`
- **Fireworks AI**: `fireworks_api_key`
- **Baseten**: `baseten_api_key`
- **Cerebras**: `cerebras_api_key`
- **Huawei Cloud MaaS**: `huawei_cloud_maas_api_key`
- **SambaNova**: `sambanova_api_key`
- **SAP AI Core**: `sap_ai_core_client_id`, `sap_ai_core_client_secret`
- **LiteLLM**: `lite_llm_base_url`, `lite_llm_api_key` (用于路由到多个后端)
- **Ollama**: `ollama_base_url` (用于本地模型)
- **LM Studio**: `lm_studio_base_url` (用于本地模型)
- **VS Code LM**: 通过VS Code内置API，无需密钥
- **Requesty**: `requesty_api_key`, `requesty_base_url`
- **Vercel AI Gateway**: `vercel_ai_gateway_api_key`
- **Google Vertex AI**: `vertex_project_id`, `vertex_region`
- **AskSage**: `asksage_api_url`, `asksage_api_key`
- **DeepSeek**: `deep_seek_api_key`
- **Dify**: `dify_api_key`, `dify_base_url`
- **OpenRouter**: `open_router_api_key` (聚合多个提供商)
- **Moonshot**: `moonshot_api_key`
- **AWS Bedrock**: `aws_access_key`, `aws_secret_key` (或使用配置文件)
- **OpenAI Native**: `open_ai_native_api_key` (特定于某些部署)
- **Claude Code**: `claude_code_path` (用于本地Claude Code模型)
- **Qwen Code**: `qwen_code_oauth_path` (用于OAuth认证)
- **Cline**: `cline_api_key` (用于Cline自有模型服务)

**Section sources**
- [doubao.ts](file://src/core/api/providers/doubao.ts)
- [qwen.ts](file://src/core/api/providers/qwen.ts)
- [xai.ts](file://src/core/api/providers/xai.ts)
- [zai.ts](file://src/core/api/providers/zai.ts)
- [nebius.ts](file://src/core/api/providers/nebius.ts)
- [fireworks.ts](file://src/core/api/providers/fireworks.ts)
- [baseten.ts](file://src/core/api/providers/baseten.ts)
- [cerebras.ts](file://src/core/api/providers/cerebras.ts)
- [huawei-cloud-maas.ts](file://src/core/api/providers/huawei-cloud-maas.ts)
- [sambanova.ts](file://src/core/api/providers/sambanova.ts)
- [sapaicore.ts](file://src/core/api/providers/sapaicore.ts)
- [litellm.ts](file://src/core/api/providers/litellm.ts)
- [ollama.ts](file://src/core/api/providers/ollama.ts)
- [lmstudio.ts](file://src/core/api/providers/lmstudio.ts)
- [vscode-lm.ts](file://src/core/api/providers/vscode-lm.ts)
- [requesty.ts](file://src/core/api/providers/requesty.ts)
- [vercel-ai-gateway.ts](file://src/core/api/providers/vercel-ai-gateway.ts)
- [vertex.ts](file://src/core/api/providers/vertex.ts)
- [asksage.ts](file://src/core/api/providers/asksage.ts)
- [deepseek.ts](file://src/core/api/providers/deepseek.ts)
- [dify.ts](file://src/core/api/providers/dify.ts)
- [openrouter.ts](file://src/core/api/providers/openrouter.ts)
- [moonshot.ts](file://src/core/api/providers/moonshot.ts)
- [bedrock.ts](file://src/core/api/providers/bedrock.ts)
- [openai-native.ts](file://src/core/api/providers/openai-native.ts)
- [claude-code.ts](file://src/core/api/providers/claude-code.ts)
- [qwen-code.ts](file://src/core/api/providers/qwen-code.ts)
- [cline.ts](file://src/core/api/providers/cline.ts)

## MCP API

MCP (Model Control Protocol) 是Cline用于集成和调用外部AI工具和服务器的协议。

### 协议规范

MCP基于JSON-RPC 2.0规范，通过HTTP或WebSocket进行通信。一个MCP服务器暴露一组工具（Tools），每个工具都有一个名称、描述和JSON Schema定义的输入参数。Cline通过`McpService`与MCP服务器交互。

### 定义和注册新的MCP服务器

1.  **创建MCP服务器**: 开发一个HTTP服务，实现MCP规范。该服务应提供一个`/mcp.json`端点，返回服务器的元数据（名称、版本、工具列表等）。
2.  **实现工具**: 在服务器中定义具体的工具函数。每个工具应能接收符合其Schema的输入并返回结果。
3.  **启动服务器**: 确保MCP服务器在指定端口上运行。
4.  **注册到Cline**:
    *   **方式一 (手动)**: 使用`McpService.addRemoteMcpServer` RPC，提供服务器名称和URL。
    *   **方式二 (市场)**: 将MCP服务器发布到MCP市场。用户可以通过`McpService.refreshMcpMarketplace`发现并安装。
5.  **启用服务器**: 使用`McpService.toggleMcpServer` RPC启用服务器。
6.  **使用工具**: 一旦服务器启用，Cline的AI模型就可以在规划中请求使用其工具。用户可以通过`McpService.toggleToolAutoApprove`设置工具的自动批准策略。

**示例：注册远程MCP服务器**
```protobuf
// RPC调用
rpc addRemoteMcpServer(AddRemoteMcpServerRequest) returns (McpServers);

// 请求消息
message AddRemoteMcpServerRequest {
  Metadata metadata = 1;
  string server_name = 2; // 例如 "MyCustomToolServer"
  string server_url = 3;  // 例如 "http://localhost:8080"
}
```

**调用上下文：**
MCP API允许Cline的功能通过插件化的方式无限扩展。开发者可以创建专门的MCP服务器来集成数据库、执行代码、调用专有API等，极大地增强了Cline的自动化能力。

**Section sources**
- [mcp.proto](file://proto/cline/mcp.proto#L15-L132)

## 错误码说明

Cline的API在发生错误时会返回结构化的错误信息。gRPC服务通常返回标准的gRPC状态码，而LLM API的错误则封装在具体的响应流中。

| 错误码 | 含义 | 常见原因 | 建议操作 |
| :--- | :--- | :--- | :--- |
| `UNAUTHENTICATED` (gRPC) | 未通过身份验证 | API密钥缺失或无效 | 检查并正确配置相关提供商的API密钥 |
| `NOT_FOUND` (gRPC) | 资源未找到 | 请求的文件、任务或MCP服务器不存在 | 确认资源ID或路径的正确性 |
| `INVALID_ARGUMENT` (gRPC) | 参数无效 | 请求消息中的字段值不符合要求 | 检查请求消息的结构和字段值 |
| `FAILED_PRECONDITION` (gRPC) | 前提条件失败 | 操作无法在当前状态下执行（如在无任务时取消任务） | 确保系统处于正确的状态再执行操作 |
| `DEADLINE_EXCEEDED` (gRPC) | 操作超时 | RPC调用耗时过长 | 检查网络连接，或增加超时设置 |
| `INTERNAL` (gRPC) | 内部错误 | 服务器内部发生未知错误 | 重试操作，若持续发生则报告问题 |
| `UNAVAILABLE` (gRPC) | 服务不可用 | 后端服务暂时不可用 | 稍后重试 |
| `RESOURCE_EXHAUSTED` (gRPC) | 资源耗尽 | 达到速率限制或信用额度不足 | 等待速率限制重置或充值信用额度 |
| `MODEL_NOT_FOUND` (LLM) | 模型未找到 | 指定的模型ID在提供商处不存在 | 检查模型ID的拼写，或使用`refresh*Models`获取有效模型列表 |
| `AUTHENTICATION_FAILED` (LLM) | 认证失败 | LLM提供商拒绝了API密钥 | 验证API密钥的有效性，检查提供商账户状态 |
| `RATE_LIMITED` (LLM) | 被限速 | 超过了提供商的请求速率限制 | 降低请求频率，或申请更高的速率限制 |
| `CONTEXT_LENGTH_EXCEEDED` (LLM) | 上下文长度超限 | 请求的token数量超过了模型的最大限制 | 缩短输入内容，或选择上下文窗口更大的模型 |

**Section sources**
- [common.proto](file://proto/cline/common.proto) (隐含gRPC状态码)
- [anthropic.ts](file://src/core/api/providers/anthropic.ts) (LLM特定错误处理)
- [openai.ts](file://src/core/api/providers/openai.ts) (LLM特定错误处理)

## 最佳实践

1.  **安全保管API密钥**: 所有API密钥都应通过`ModelsService.updateApiConfigurationProto`进行配置，避免在代码或日志中硬编码。
2.  **使用流式API**: 对于`UiService.subscribeToPartialMessage`等流式RPC，使用流式处理来实时更新UI，提供更好的用户体验。
3.  **处理gRPC错误**: 在调用gRPC方法时，始终处理可能的错误状态码，并向用户提供有意义的错误信息。
4.  **管理MCP服务器**: 定期通过`refreshMcpMarketplace`检查MCP市场的更新，并谨慎批准新的工具，以保障安全。
5.  **监控信用额度**: 使用`AccountService.getUserCredits`监控信用额度使用情况，避免因额度不足导致服务中断。
6.  **选择合适的模型**: 根据任务需求（速度、成本、能力）在Plan模式和Act模式中选择合适的LLM提供商和模型。
7.  **利用缓存**: 对于支持Prompt Caching的模型（如Anthropic），设计系统提示（System Prompt）以利用缓存，降低延迟和成本。
8.  **异步操作**: 对于长时间运行的任务（如`newTask`），使用`TaskService`的异步方法，并通过订阅消息来跟踪进度。

**Section sources**
- 本文档所有相关文件