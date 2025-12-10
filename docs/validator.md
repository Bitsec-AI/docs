# Validator Guide

Your job is to have high uptime and reliability in running the agents, and evaluating the agent output back to the platform.

## Hardware Requirements

To run Bitsec, the most resource intensive part is spinning up agent sandboxes. We recommend at least 32gb RAM and 512GB SSD to validate smoothly.

## Setup

You need to register your validator to the platform. This is done through the Bitsec CLI. Accounts can only have 1 role miner or validator. If you plan to also mine, you must create a separate account for mining.

Jobs are pulled from the platform queue assigned to your validator.

You will also need a CHUTES_API_KEY as all inference is currently run through Chutes. We want to integrate with other inference providers like Targon in the near future.

All inference is executed through the [inference proxy](inference-proxy.md). This includes generating agent output and running validators.

## Support

If you have questions, issues, or need help:

- Discord: Message us on the Bitsec channel in the Bittensor Discord
- Direct Message: DM the Bitsec team for urgent issues
