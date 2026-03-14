# Provider Abstraction

Markdown OS uses AI capabilities that exist across at least Claude and OpenAI. If both providers support it, it's fair game as a language primitive. This guarantees provider independence.

## Common Capability Surface

**Tool / Function Calling** — Both providers support structured tool definitions. This is the entire execution model.

**Structured Output / JSON Mode** — Both produce validated JSON. The response envelope is structured JSON.

**System Prompts** — Both support system-level instructions. The compiled markup *becomes* the system prompt.

**Multi-turn Context** — Both support conversation history. Used for multi-step workflows.

**Vision / Multimodal Input** — Both accept images. Applications processing visual input work without special handling.

**Streaming** — Both support SSE. Real-time applications use this transparently.

## Adapter Interface

```
Provider Adapter
├── sendCompletion(systemPrompt, messages, tools) → response
├── parseToolCalls(response) → ToolCall[]
├── parseTextOutput(response) → string
├── supportsVision() → boolean
├── supportsStreaming() → boolean
└── estimateTokens(content) → number
```

| Aspect | Claude Adapter | OpenAI Adapter |
|---|---|---|
| Tool definition format | `tools[]` with `input_schema` | `tools[]` with `parameters` |
| Tool call response | `content[].type === "tool_use"` | `tool_calls[]` in message |
| System prompt | `system` parameter | `{"role": "system"}` message |
| Image input | `content[].type === "image"` base64 | `content[].type === "image_url"` |
| Streaming format | SSE `content_block_delta` | SSE `delta` |

## Provider Selection Strategy

The platform routes intelligently: cost optimization (simple requests to cheaper models), latency optimization (fastest responding provider), capability matching (vision-heavy to best vision model), and availability (failover between providers). The author can optionally express a preference but doesn't have to.
