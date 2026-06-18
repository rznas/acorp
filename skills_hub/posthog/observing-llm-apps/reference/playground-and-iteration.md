# Playground: testing prompts and comparing models

**PM question it answers:** "Which model/prompt should we ship?" and "Can I reproduce this bad production response and fix it?" — without writing or deploying code.

The Playground (AI Observability > Playground) lets you test prompts, compare models side by side, tune parameters, test tool calling, replay from real traces, and save results to prompt management or evaluations.

## Getting started

1. Go to **AI Observability > Playground**.
2. Pick a model, write a system prompt, add messages, click **Run** — response streams in.
3. Works out of the box with **trial models** (PostHog-provided, rate-limited, requires a valid payment method on file). Add your own keys (**Settings > AI Observability > BYOK**: OpenAI, Azure OpenAI, Anthropic, Google, OpenRouter, Fireworks, Together AI, etc.) to remove PostHog rate limits and unlock more models.

> Self-hosted note: trial models are PostHog-hosted; on self-hosted you may need BYOK API keys. Availability/config may differ — verify with your admin.

## Building a prompt

- **System prompt** — sets behavior (default "You are a helpful AI assistant.").
- **Messages** — user/assistant turns with a role toggle.
- **Tools** (wrench icon) — define function tools in JSON (OpenAI tool-calling format: `type: "function"` + `function` with `name`, `description`, `parameters`). Tool calls the model makes are shown in the result.

## Parameters (Settings/gear)

| Parameter | What it does | Range |
|---|---|---|
| Max tokens | Max response length | 1–16,384 |
| Temperature | Randomness (lower = more deterministic) | 0–2 |
| Top p | Nucleus sampling threshold | 0–1 |
| Reasoning effort | How much the model reasons first | None/Minimal/Low/Medium/High |
| Thinking | Enable extended-thinking stream | On/Off |

## Reading the result

Each run shows the markdown response, any tool calls, **token usage** (prompt, completion, cache read/write), **latency**, and **time to first token**. **Stop** aborts mid-stream.

## Comparing (the PM decision tool)

Click **Add prompt** to add columns — each with its own model, system prompt, messages, tools, and parameters. **Run** executes all in parallel for side-by-side comparison. Use it to:
- Compare the same prompt across models (GPT-5 vs Claude Sonnet vs Gemini) on quality, latency, and tokens.
- Test system-prompt variants on one model.
- See how temperature/reasoning-effort changes affect output.

**Decision:** pick the model/prompt with the best quality-vs-cost-vs-latency tradeoff for the feature, backed by the cost data in AI Observability.

## Replay from production and save

- **Replay a trace:** open any generation from Traces in the playground — it pre-populates model, system prompt, and messages so you can reproduce and iterate on a real bad response.
- **Save:** save the system prompt as a **managed prompt** (prompt management) or as a new **evaluation**; or load an existing prompt/evaluation. Opening the playground from traces/prompt-management/evaluations pre-fills it and links back so you can publish a new prompt version or update the eval config directly. See `running-ai-evals`.
