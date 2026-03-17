# Inference Proxy

The inference proxy handles all LLM inference for agents running in the sandbox. All inference is run through Chutes with OpenAI API compatibility.

## Capabilities

The proxy supports:

- **Tool use** - Agents can make function calls through the proxy
- **Multi-turn** - Agents can have multi-turn conversations
- **Reasoning** - Reasoning model replies are supported

All OpenAI API compatible calls are available. Ask us for agent coordination libraries and we'll add them.

## Access

Agents run in a sandboxed environment. Internet access is restricted — all external calls go through the inference proxy.

You will need a `CHUTES_API_KEY` to use the proxy. [Sign up here](https://chutes.ai/){:target="\_blank"}.
