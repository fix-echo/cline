# LLM API

<cite>
**本文档中引用的文件**   
- [anthropic.ts](file://src/core/api/providers/anthropic.ts)
- [openai.ts](file://src/core/api/providers/openai.ts)
- [ollama.ts](file://src/core/api/providers/ollama.ts)
- [gemini.ts](file://src/core/api/providers/gemini.ts)
- [vertex.ts](file://src/core/api/providers/vertex.ts)
- [index.ts](file://src/core/api/index.ts)
- [retry.ts](file://src/core/api/retry.ts)
- [api-configuration-conversion.ts](file://src/shared/proto-conversions/models/api-configuration-conversion.ts)
- [validate.ts](file://webview-ui/src/utils/validate.ts)
- [ApiOptions.tsx](file://webview-ui/src/components/settings/ApiOptions.tsx)
</cite>

## 目录
1. [支持的LLM提供商列表](#支持的llm提供商列表)
2. [统一API抽象层](#统一api抽象层)
3. [重试机制](#重试机制)
4. [提供商配置与实现细节](#提供商配置与实现细节)
   1. [Anthropic](#anthropic)
   2. [OpenAI](#openai)
   3. [Ollama](#ollama)
   4. [Google Gemini](#google-gemini)
   5. [GCP Vertex AI](#gcp-vertex-ai)

## 支持的LLM提供商列表

Cline集成了多个大型语言模型（LLM）提供商，为用户提供多样化的模型选择。每个提供商都通过独立的处理程序（Handler）进行集成，确保了功能的一致性和可扩展性。以下是Cline支持的主要LLM提供商：

- **Anthropic**: 提供Claude系列模型，包括Claude 3 Sonnet、Claude 3 Opus等。
- **OpenAI**: 支持OpenAI的兼容API，包括GPT系列模型。
- **Ollama**: 支持本地运行的开源模型，允许用户在本地环境中部署和使用LLM。
- **Google Gemini**: 集成Google的Gemini模型，提供强大的多模态能力。
- **GCP Vertex AI**: 通过Vertex AI平台访问Google的AI服务，包括Gemini和Claude模型。
- **其他提供商**: 还包括OpenRouter、Bedrock、LM Studio、DeepSeek、Requesty、Fireworks、Together、Qwen、Doubao、Mistral、VS Code LM、Cline、LiteLLM、Moonshot、Hugging Face、Nebius、AskSage、Sambanova、Cerebras、Groq、Baseten、SAP AI Core、Huawei Cloud MaaS、Dify、Vercel AI Gateway、Z AI、X AI等。

这些提供商的配置和使用方法在后续章节中详细说明。

## 统一API抽象层

Cline通过`src/core/api/index.ts`中的统一API抽象层，将不同LLM提供商的响应标准化为内部格式。这一抽象层的核心是`ApiHandler`接口，它定义了所有提供商必须实现的方法，确保了API调用的一致性。

### 核心组件

- **ApiHandler接口**: 定义了`createMessage`和`getModel`方法，所有提供商的处理程序都必须实现这些方法。
- **createHandlerForProvider函数**: 根据配置的提供商类型，创建相应的处理程序实例。该函数通过`switch`语句选择正确的处理程序，并传递必要的配置选项。
- **buildApiHandler函数**: 在创建处理程序之前，验证`thinkingBudgetTokens`是否超过模型的最大令牌数，以防止API错误。

### 工作流程

1. **初始化**: 用户在设置中选择一个LLM提供商，并配置相应的API密钥和其他参数。
2. **创建处理程序**: `buildApiHandler`函数根据用户的配置，调用`createHandlerForProvider`生成对应的处理程序实例。
3. **标准化调用**: 所有处理程序通过`createMessage`方法发送请求，返回的响应被标准化为`ApiStream`格式，包含文本、推理和使用情况等信息。
4. **响应解析**: 响应数据被解析并传递给前端，用户可以在界面中查看模型的输出。

此抽象层的设计使得Cline能够轻松集成新的LLM提供商，同时保持代码的整洁和可维护性。

**Section sources**
- [index.ts](file://src/core/api/index.ts#L1-L421)

## 重试机制

Cline的重试机制由`src/core/api/retry.ts`中的`withRetry`装饰器实现，旨在处理网络不稳定或API限流等临时性错误。该机制通过指数退避算法和可配置的重试策略，确保API调用的可靠性。

### 核心组件

- **RetriableError类**: 继承自`Error`，用于表示可重试的错误。它包含`status`和`retryAfter`属性，分别表示HTTP状态码和建议的重试延迟。
- **withRetry装饰器**: 接收`RetryOptions`对象作为参数，定义了最大重试次数、基础延迟、最大延迟和是否对所有错误进行重试等选项。

### 工作流程

1. **错误检测**: 当API调用失败时，检查错误类型。如果错误状态码为429（Too Many Requests）或错误实例为`RetriableError`，则认为是可重试的错误。
2. **计算延迟**: 根据`retry-after`、`x-ratelimit-reset`或`ratelimit-reset`等HTTP头信息，计算重试延迟。如果没有这些头信息，则使用指数退避算法计算延迟。
3. **执行重试**: 在指定的延迟后重新尝试API调用。如果达到最大重试次数仍未成功，则抛出最终错误。
4. **回调通知**: 每次重试前，调用`onRetryAttempt`回调函数，通知用户重试状态。

该机制不仅提高了API调用的成功率，还通过合理的延迟策略避免了对服务器的过度压力。

**Section sources**
- [retry.ts](file://src/core/api/retry.ts#L0-L86)

## 提供商配置与实现细节

### Anthropic

Anthropic提供商通过`AnthropicHandler`类实现，支持Claude系列模型。其主要特性包括：

- **API密钥**: 环境变量名为`CLINE_ANTHROPIC_API_KEY`。
- **基础URL**: 可通过`anthropicBaseUrl`配置自定义基础URL。
- **模型支持**: 支持`claude-3-5-sonnet`、`claude-3-opus`等模型。
- **请求参数**: 使用`thinking`参数控制模型的推理行为，`max_tokens`限制输出长度。
- **响应解析**: 解析`message_start`、`message_delta`等事件，提取输入/输出令牌数和缓存读写信息。

**Section sources**
- [anthropic.ts](file://src/core/api/providers/anthropic.ts#L0-L246)

### OpenAI

OpenAI提供商通过`OpenAiHandler`类实现，支持OpenAI的兼容API。其主要特性包括：

- **API密钥**: 环境变量名为`CLINE_OPENAI_API_KEY`。
- **基础URL**: 可通过`openAiBaseUrl`配置自定义基础URL，支持Azure OpenAI。
- **模型支持**: 支持GPT-3.5、GPT-4等模型。
- **请求参数**: 使用`temperature`、`max_tokens`等参数控制生成行为。
- **响应解析**: 解析`chat.completions.create`的流式响应，提取文本内容和使用情况。

**Section sources**
- [openai.ts](file://src/core/api/providers/openai.ts#L0-L140)

### Ollama

Ollama提供商通过`OllamaHandler`类实现，支持本地运行的开源模型。其主要特性包括：

- **API密钥**: 可选，用于认证Ollama云实例。
- **基础URL**: 可通过`ollamaBaseUrl`配置Ollama服务器地址。
- **模型支持**: 支持任何在本地Ollama服务器上可用的模型。
- **请求参数**: 使用`num_ctx`参数设置上下文窗口大小。
- **响应解析**: 解析Ollama的流式响应，提取生成的文本和令牌使用情况。

**Section sources**
- [ollama.ts](file://src/core/api/providers/ollama.ts#L0-L122)

### Google Gemini

Google Gemini提供商通过`GeminiHandler`类实现，支持Gemini模型。其主要特性包括：

- **API密钥**: 环境变量名为`CLINE_GEMINI_API_KEY`。
- **基础URL**: 可通过`geminiBaseUrl`配置自定义基础URL。
- **模型支持**: 支持`gemini-pro`、`gemini-ultra`等模型。
- **请求参数**: 使用`thinkingConfig`参数控制模型的推理预算。
- **响应解析**: 解析Gemini的流式响应，提取文本、推理内容和使用情况，包括缓存读取和写入令牌数。

**Section sources**
- [gemini.ts](file://src/core/api/providers/gemini.ts#L0-L472)

### GCP Vertex AI

GCP Vertex AI提供商通过`VertexHandler`类实现，支持通过Vertex AI平台访问Google的AI服务。其主要特性包括：

- **项目ID和区域**: 需要配置`vertexProjectId`和`vertexRegion`。
- **API密钥**: 可选，用于Gemini模型的认证。
- **模型支持**: 支持Gemini和Claude模型。
- **请求参数**: 使用`thinking`参数控制Claude模型的推理行为。
- **响应解析**: 对于Gemini模型，委托给`GeminiHandler`处理；对于Claude模型，直接调用Anthropic的Vertex SDK。

**Section sources**
- [vertex.ts](file://src/core/api/providers/vertex.ts#L0-L278)