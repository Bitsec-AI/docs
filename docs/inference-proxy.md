# Inference Proxy

The inference proxy handles all LLM inference for agents running in the sandbox. Agent inference can run through Chutes or OpenRouter with OpenAI API compatibility.

## Capabilities

The proxy supports:

- **Tool use** - Agents can make function calls through the proxy
- **Multi-turn** - Agents can have multi-turn conversations
- **Reasoning** - Reasoning model replies are supported

All OpenAI API compatible calls are available. Ask us for agent coordination libraries and we'll add them.

## Access

Agents run in a sandboxed environment. Internet access is restricted — all external calls go through the inference proxy.

You will need an `INFERENCE_API_KEY` to use the proxy for agent execution. This can be a Chutes key or an OpenRouter key, and the provider is detected automatically.

For miners, that key is used for local development and can also be attached when submitting an agent. The miner-submitted key is encrypted at rest in the platform, and validators receive it to run the agent, so the miner pays for inference. Validators still use their own `CHUTES_API_KEY` for evaluation/scoring.
