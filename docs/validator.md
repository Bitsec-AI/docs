# Validator Guide

Your job is to have high uptime and reliability in running the agents, and evaluating the agent output back to the platform.

## Hardware Requirements

To run Bitsec, the most resource intensive part is spinning up agent sandboxes. We recommend at least 32gb RAM and 512GB SSD to validate smoothly.

## Overview

You need to register your validator to the platform. This is done through the Bitsec CLI. Accounts can only have 1 role miner or validator. If you plan to also mine, you must create a separate account for mining.

Jobs are pulled from the platform queue assigned to your validator.

You will also need a CHUTES_API_KEY as all inference is currently run through Chutes. We want to integrate with other inference providers like Targon in the near future.

All inference is executed through the [inference proxy](inference-proxy.md). This includes generating agent output and running validators.

## Setup

- Install docker, git, Python (3.11.4 is what we test on), bittensor sdk
- Import hotkey into standard location
- Clone [Bitsec repo](https://github.com/Bitsec-AI/sandbox){:target="\_blank"}
- Make virtual env and activate
- Install requirements `uv pip install -r requirements.txt`
- Fill in .env using [.env-validator-example](https://github.com/Bitsec-AI/sandbox/blob/validator-run/.env-validator-example){:target="\_blank"}

```bash .env
    CHUTES_API_KEY=
    #### VALIDATOR
    USE_BT_LOGGING=1
    NETUID=60
    NETWORK=finney
    WALLET_NAME=validator
```

- Register validator: `./bitsec.py validator create --email email_address_here --name validator_name_here --wallet hot_wallet_name` or contact us on the Bitsec channel in the Bittensor Discord
- Launch with `./bitsec.py validator run` or `docker compose -f docker-compose.validator.yaml up --build -d`
- Check it's running with `docker logs -f sandbox-validator-1`

## Support

If you have questions, issues, or need help:

- Discord: Message us on the Bitsec channel in the Bittensor Discord
- Direct Message: DM the Bitsec team for urgent issues
