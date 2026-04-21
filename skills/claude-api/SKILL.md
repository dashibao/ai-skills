---
name: claude-api
description: Build apps with the Claude API or Anthropic SDK. TRIGGER when: code imports `anthropic`/`@anthropic-ai/sdk`/`claude_agent_sdk`, or user asks to use Claude API, Anthropic SDKs, or Agent SDK. DO NOT TRIGGER when: code imports `openai`/other AI SDK, general programming, or ML/data-science tasks.
---

# Claude API Skill

Build, debug, and optimize applications using the Claude API or Anthropic SDK.

## When This Skill Applies

**TRIGGER when:**
- Code imports `anthropic`, `@anthropic-ai/sdk`, or `claude_agent_sdk`
- User asks about the Claude API, Anthropic SDK, or Agent SDK
- User adds/modifies/tunes Claude features (caching, thinking, tool use, batch, files)
- Questions about prompt caching or cache hit rates

**SKIP when:**
- Code imports `openai` or other AI provider SDKs
- File names like `*-openai.py` or `*-generic.py`
- Provider-neutral code
- General programming/ML tasks

## Quick Task Reference

| Task | Documentation to Reference |
|------|----------------------------|
| Single text classification/summarization/extraction/Q&A | `{lang}/claude-api/README.md` |
| Chat UI or real-time response display | `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md` |
| Long-running conversations (may exceed context) | `{lang}/claude-api/README.md` — Compaction section |
| Prompt caching / optimize caching | `shared/prompt-caching.md` + `{lang}/claude-api/README.md` |
| Function calling / tool use / agents | `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` |
| Batch processing (non-latency-sensitive) | `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md` |
| File uploads across multiple requests | `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md` |
| Agent with built-in tools (Python & TypeScript only) | `{lang}/agent-sdk/README.md` |
| Error handling | `shared/error-codes.md` |

## Current Models (April 2026)

| Model | ID | Best For |
|-------|----|----------|
| Claude Opus 4.6 | `claude-opus-4-6` | Most capable, complex tasks |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | Balanced performance, default choice |
| Claude Haiku 4.5 | `claude-haiku-4-5` | Fast, cost-effective |

## Prompt Caching

Prompt caching reduces costs and latency for repeated prompts. Key points:

1. **Cacheable content**: System prompts, tool definitions, large context
2. **Minimum size**: 1024 tokens for system prompts, 2048 for tools/context
3. **Cache lifetime**: 5 minutes TTL
4. **Cache control**: Set `cache_control: { type: "ephemeral" }` on cacheable blocks

```python
# Example: Cache system prompt
message = client.messages.create(
    model="claude-sonnet-4-6",
    system=[
        {
            "type": "text",
            "text": "Long system prompt...",
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": "Hello"}]
)
```

## Tool Use Pattern

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            },
            "required": ["location"]
        }
    }
]

response = client.messages.create(
    model="claude-sonnet-4-6",
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in SF?"}]
)

# Handle tool_use response
for block in response.content:
    if block.type == "tool_use":
        # Execute tool and return result
        result = get_weather(block.input["location"])
```

## Streaming

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## Batch Processing

For non-latency-sensitive tasks, use the Messages Batch API:

```python
batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": "request-1",
            "params": {
                "model": "claude-sonnet-4-6",
                "messages": [{"role": "user", "content": "Hello"}]
            }
        }
    ]
)
```

## Common Pitfalls

1. **Not caching repetitive content** — Add `cache_control` to system prompts and tool definitions
2. **Streaming without timeout handling** — Always set reasonable timeouts
3. **Hardcoding model IDs** — Use constants or configuration for model IDs
4. **Missing error handling** — Always handle `APIError`, `RateLimitError`

## Live Documentation Sources

For the latest documentation, fetch from:
- https://docs.anthropic.com/claude/docs
- https://github.com/anthropics/anthropic-sdk-python
- https://github.com/anthropics/anthropic-sdk-typescript

## Usage

```
/claude-api                     # Activate Claude API skill
/claude-api how do I use streaming?
/claude-api optimize my cache hit rate
```