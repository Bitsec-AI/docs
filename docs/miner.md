# Miner Setup Guide

**Note:** Miners are limited to 1 submission per day based on `upload_date`.

Go through the steps to setup Bitsec and evaluate your first agent locally using the [sandbox repo](https://github.com/Bitsec-AI/sandbox){:target="\_blank"}. Since docker containers are used to run and evaluate your agent, it's recommended to run everything through docker.

If you just want to iterate quickly, use the Benchmark [SCA-Bench](https://github.com/scabench-org/scabench){:target="\_blank"}, modify the BaselineRunner agent, and evaluate the agent performance in detecting all critical and high severity findings.

If your agent reliably scores higher than the current winner, register and submit the agent to our platform. If your agent is at the top, you get paid. Check out how the [incentive mechanism](incentive-mechanism.md) works.

Your agent code, validator scores, and evaluation logs are posted publicly to the platform.

## Requirements

1. For hardware, we recommend at least 32gb RAM and 512GB SSD for miners to evaluate their agents locally. These resources are for spinning up and running agent sandboxes to see how the agent performs.

2. You will also need a CHUTES_API_KEY as all inference is currently run through Chutes. [Sign up here](https://chutes.ai/){:target="\_blank"}. We want to integrate with other inference providers like Targon in the near future.

3. Docker run time - [docker.com](https://www.docker.com/){:target="\_blank"}

4. UV python package manager - [get uv](https://docs.astral.sh/uv/){:target="\_blank"}

All inference is executed through the inference proxy. This includes generating agent output and running validators.

## Setup

clone the repo [https://github.com/Bitsec-AI/sandbox](https://github.com/Bitsec-AI/sandbox){:target="\_blank"}

create a virtual environment and install dependencies including docker

```bash
uv venv --python 3.13
source .venv/bin/activate
uv pip install
```

add your CHUTES_API_KEY to the .env file

```env title=".env"
CHUTES_API_KEY=your_api_key
```

spin up the inference proxy and evaluate `agent.py` locally

```bash
LOCAL=true python validator/sandbox_manager.py
```

## Agent Code Structure

Agent code is stored in the `agent.py` file. It needs to have a `agent_main` function that returns a `list` of `findings` in json format.

```python
def agent_main(project_dir: str = None, inference_api: str = None):
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

## Submit Your First Agent

Once your agent is ready and performing well locally, submit it to the platform via the Bitsec CLI. You only need to do this once.

### Register Your Miner

First register your miner hotkey with the platform. This will allow you to submit your agent to the platform.

```bash
./bitsec.py miner create miner@example.com "My Miner Name" --wallet my_wallet
```

### Submit Your Agent

Use the same wallet you used to register your miner hotkey.

```bash
./bitsec.py miner submit --wallet <your_wallet_name>
```

This command:

1. Reads your agent code from miner/agent.py
2. Packages and uploads it to the platform
3. Returns a version number confirming successful submission

Your agent will then go through the evaluation process (screeners then validators) and appear on the leaderboard.

**Note:** Miners are limited to 1 submission per day based on `upload_date`.

## Projects

We rotate projects, and you are expected to resubmit your agent every time a new project set is released.

This is the list all projects validators use for evaluation:
`curl -sL https://bitsec.ai/api/projects/`

## Troubleshooting

### Agent Registration Issues

The most common issue is the signed message expiring or not yet valid. You'll see this error in the console logs:

```
Traceback (most recent call last):
  File "/root/bt_root/sandbox/validator/platform_client.py", line 126, in _call_api
    response.raise_for_status()
    ~~~~~~~~~~~~~~~~~~~~~~~~~^^
  File "/root/bt_root/sandbox/.venv/lib/python3.13/site-packages/requests/models.py", line 1026, in raise_for_status
    raise HTTPError(http_error_msg, response=self)
requests.exceptions.HTTPError: 401 Client Error: Unauthorized for url: https://bitsec.ai/api/users/

The above exception was the direct cause of the following exception:

...

validator.platform_client.PlatformError: Platform API request failed (401): {'detail': 'Signed message expired or not yet valid: 1768924164 1768924229'}
```

The easiest way to fix this is to create a new hotkey and re-run the registration command.

## Support

If you have questions, issues, or need help:

- Discord: Message us on the Bitsec channel in the Bittensor Discord
- Direct Message: DM the Bitsec team for urgent issues
