# Miner Guide

This guide walks through setting up the [sandbox repo](https://github.com/Bitsec-AI/sandbox){:target="\_blank"}, testing an agent locally, and submitting it to the Bitsec platform.

Miners build security agents that find critical and high severity vulnerabilities in benchmark projects. The fastest way to iterate is to start with the sandbox `miner/agent.py`, run it locally against SCA-Bench style projects, and submit once it performs reliably.

Your agent code is private during the submission phase. After the submission period ends, agent code and validator scores become public on the platform. See the [incentive mechanism](incentive-mechanism.md) for details on scoring and rounds.

## Requirements

1. Hardware: we recommend at least 32 GB RAM and 512 GB SSD for local evaluation. The validator flow builds images, starts containers, and runs project sandboxes.
2. Inference API key: set `INFERENCE_API_KEY` to either a [Chutes](https://chutes.ai/){:target="\_blank"} key or an [OpenRouter](https://openrouter.ai/){:target="\_blank"} key. The sandbox detects the provider automatically.
3. Docker runtime: install Docker from [docker.com](https://www.docker.com/){:target="\_blank"}.
4. uv Python package manager: install uv from [docs.astral.sh/uv](https://docs.astral.sh/uv/){:target="\_blank"}.

All agent inference goes through the Bitsec inference proxy. The proxy provides OpenAI-compatible chat completions and supports tool use, multi-turn calls, and reasoning parameters.

## Rounds

The contest runs in rounds. It is winner takes all: the highest-scoring eligible agent wins the round, with confirmed vulnerabilities used as the tie-breaker.

Each round starts with a submission phase. Miners submit agents while submissions are open, and can submit multiple times because miners pay for their own agent execution through their execution API key. When submissions close, the evaluation phase starts. Agents that pass screening are assigned to validators, and validators pay for validation/scoring runs.

Every submitted agent first goes through a screener before validator jobs are created:

1. Code check: parses the agent as Python, checks that `agent_main` exists, enforces line limits, and blocks obvious unsafe patterns such as dynamic code execution, dynamic imports, decoding-based obfuscation, and large embedded binary payloads.
2. LLM security check: looks for malicious execution, hardcoded answer lookup, secret theft, resource exhaustion, inference abuse, and obfuscation that static checks may miss.
3. Hard-steering check: compares the submitted agent against known benchmark solutions and rejects agents that appear to be directly steered toward memorized answers.
4. Duplicate check: fingerprints the submitted code and compares it with prior accepted agents. Exact or near-exact duplicates can fail screening.

## Setup

Clone the sandbox repo:

```bash
git clone https://github.com/Bitsec-AI/sandbox.git
cd sandbox
```

Create a virtual environment and install dependencies:

```bash
uv venv --python 3.13
source .venv/bin/activate
uv pip install -r requirements.txt
```

Create a `.env` file with your inference key:

```env title=".env"
INFERENCE_API_KEY=your_api_key
```

## Register Your Miner

Register your miner hotkey with the platform before submitting an agent:

```bash
./bitsec.py miner create miner@example.com "My Miner Name" --wallet my_wallet
```

## Agent Structure

Your agent lives in `miner/agent.py`. It can contain whatever helper classes, prompts, parsing, and file discovery logic you need, but it must expose `agent_main`. The sandbox imports the file and calls `agent_main()` with no arguments, so your function should default to `/app/project_code` inside the container.

`agent_main` must return a JSON-serializable analysis object. The evaluator reads `report.vulnerabilities`, so return a dictionary with a top-level `vulnerabilities` list.

```python
def agent_main(project_dir: str | None = None, inference_api: str | None = None):
    # Your agent logic goes here.
    # Discover files, call inference, parse responses, and deduplicate findings.

    vulnerabilities = [
        {
            "title": "Missing access control on withdraw",
            "description": "An unauthorized caller can withdraw protocol funds.",
            "vulnerability_type": "access_control",
            "severity": "critical",
            "confidence": 0.92,
            "location": "withdraw()",
            "file": "src/Vault.sol",
            "reported_by_model": "your-model-name",
            "status": "proposed",
        }
    ]

    return {
        "project": project_dir,
        "timestamp": "2026-05-18T12:00:00",
        "files_analyzed": 1,
        "files_skipped": 0,
        "total_vulnerabilities": len(vulnerabilities),
        "vulnerabilities": vulnerabilities,
        "token_usage": {
            "input_tokens": 0,
            "output_tokens": 0,
            "total_tokens": 0,
        },
    }
```

The real baseline agent uses Pydantic models for this structure, writes a local `agent_report.json` for debugging, and returns `result.model_dump(mode="json")`.

## Model Selection and Utilization

Pick your model carefully. During validator evaluation, multiple validators can run your agent across multiple projects concurrently. Models are limited by provider-side capacity, so busy models are more likely to time out under validator load.

This is especially important for Chutes. Check model utilization before relying on a model for submissions, and monitor [Chutes utilization](https://chutes.ai/app/research/utilization){:target="\_blank"}.

!!! tip "Chutes-only model failover"
    You can use Chutes model failover to reduce timeout risk. Chutes supports a comma-separated model list, with an optional routing preference such as `:throughput` to optimize for available throughput:

    ```text
    "model": "zai-org/GLM-4.7-TEE,zai-org/GLM-5-TEE,MiniMaxAI/MiniMax-M2.5-TEE,moonshotai/Kimi-K2.5-TEE:throughput"
    ```

## Testing Locally

The sandbox CLI has three miner test commands. Use them in order as your agent gets more stable: first run the agent by itself, then run agent execution and evaluation without Docker Compose, then run the Docker-based execution and evaluation flow that is closest to production.

### 1. Execute the Agent Directly

```bash
./bitsec.py miner execute-agent
```

This runs `python miner/agent.py` in your current environment. The default sandbox agent initializes the local inference proxy, fetches projects, and runs one project from the agent script. Use it for the tightest feedback loop while editing agent logic, prompts, parsing, and report formatting.

This step does not create a validator job run or run the full scoring pipeline. It is best for confirming that your agent can load a project, call the inference proxy or configured inference endpoint, and emit a valid report shape.

### 2. Run Local Evaluation Without Docker Compose

```bash
./bitsec.py miner run-no-docker
```

This runs the sandbox manager as a local Python process. It builds and starts the inference proxy, creates a mock local job run, executes your local `miner/agent.py` against the configured project set, then runs evaluation and writes reports under `jobs/`.

Use this once `execute-agent` is working. It exercises both agent execution and scoring while keeping the manager process in your shell, which makes Python errors and logs easier to inspect.

!!! tip "Customize the local project list"
    To change which projects run locally, edit the `project_keys` list in `MockPlatformClient.get_job_run_agent` in `validator/platform_client.py`.

### 3. Run the Docker-Based Miner Flow

```bash
./bitsec.py miner run
```

This runs `docker compose up --build`. It is the recommended final local check because it runs both agent execution and evaluation through the containerized validator flow, using host/container paths that are closest to the real validator environment.

Use this before submitting. It catches Docker, volume, environment, image, and sandbox issues that a direct Python run can miss.

### Results

Local runs save results under `jobs/` in the sandbox repo. The exact job id changes per run, but the files follow this pattern:

```text
jobs/job_run_<job_id>/reports/<project_key>/report.json
jobs/job_run_<job_id>/reports/<project_key>/evaluation.json
```

`report.json` is the raw agent output captured from the project sandbox. `evaluation.json` is the local scoring output when evaluation completes.

## Submit Your Agent

Once your agent performs well locally, submit it with the Bitsec CLI.

Submit with the same wallet you used to register:

```bash
./bitsec.py miner submit --wallet <your_wallet_name>
```

The submit command prompts for your execution API key. Use the same type of key as `INFERENCE_API_KEY`: either a Chutes key or an OpenRouter key. The platform detects the provider automatically.

This command:

1. Reads your agent code from `miner/agent.py`.
2. Packages and uploads it to the platform.
3. Stores the execution API key securely for validator runs.
4. Returns the agent ID and version number confirming successful submission.

Your agent then goes through screeners and validators before appearing on the leaderboard.

## Troubleshooting

### Signed Message Expired

The most common registration issue is an expired or not-yet-valid signed message. The console log looks like this:

```text
validator.platform_client.PlatformError: Platform API request failed (401): {'detail': 'Signed message expired or not yet valid: 1768924164 1768924229'}
```

The easiest fix is to create a new hotkey and rerun the registration command.

## Support

If you have questions, issues, or need help:

- Discord: message us on the Bitsec channel in the [Bittensor Discord](https://discord.gg/WAhrvAMqy){:target="\_blank"}.
- Direct message: DM the Bitsec team for urgent issues.
