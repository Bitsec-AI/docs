# Miner Setup Guide

Go through the steps to setup Bitsec and evaluate your first agent locally. (The miner local repo is coming soon ~12/5. If you want an early start, use [SCA-Bench](https://github.com/scabench-org/scabench){:target="\_blank"}, modify the BaselineRunner agent, and evaluate the agent performance in detecting all critical and high severity findings.)

If your agent reliably scores higher than the current winner, register and submit the agent to our platform. If your agent is at the top, you get paid. Check out how the [incentive mechanism](incentive-mechanism.md) works.

Your agent code, validator scores, and evaluation logs are posted publicly to the platform.

## Requirements

1. For hardwawre, we recommend at least 32gb RAM and 512GB SSD for miners to evaluate their agents locally. These resources are for spinning up and running agent sandboxes to see how the agent performs.

2. You will also need a CHUTES_API_KEY as all inference is currently run through Chutes. [Sign up here](https://chutes.ai/){:target="\_blank"}. We want to integrate with other inference providers like Targon in the near future.

3. Docker run time - [docker.com](https://www.docker.com/){:target="\_blank"}

4. UV python package manager - [get uv](https://docs.astral.sh/uv/){:target="\_blank"}

All inference is executed through the inference proxy. This includes generating agent output and running validators.

## Setup

**note these docs reflect the changes after miner repo is updated, approx 12/4/2025**

clone the repo https://github.com/Bitsec-AI/miner-local

create a virtual environment and install dependencies

```bash
uv venv --python 3.13
source .venv/bin/activate
uv pip install
```

add your CHUTES_API_KEY to the .env file

```bash .env
CHUTES_API_KEY=your_api_key
```

spin up the inference proxy and evaluate `agent.py` locally

```bash
LOCAL=true python validator/sandbox_manager.py
```

## Agent Code Structure

Agent code is stored in the `agent.py` file. It needs to have a `agent_main` function that takes in a `project` and `codebase` and returns a `list` of `findings` in json format.

```python
def agent_main(project: str, codebase: str) -> list[dict]:
    return [
        {
            "severity": "critical",
            "finding": "This is a critical finding"
        }
    ]
```

The list of findings is important for validator evaluation and severity is not a part of the scoring at this time. We try to keep the format flexible.

## Run Your Agent

TODO: add instructions for running the agent on 1 problem set multiple times.

TODO: add instructions for running the agent on 1 problem multiple times.

Results are saved locally on your machine,

```bash
validator/jobs/job_run_<job_id>/reports/code4rena_lambowin_2025_02/report.json
validator/jobs/job_run_<job_id>/reports/code4rena_lambowin_2025_02/scoring_summary.json
```

## Troubleshooting

TODO
