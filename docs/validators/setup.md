# Validator Setup Guide

This guide will walk you through setting up a validator node for the Bitsec-AI subnet on the Bittensor network.

## Prerequisites

Before starting:

- Complete the [installation guide](../getting-started/installation.md)
- Have at least 1000 TAO tokens for staking (recommended)
- Ensure your system meets the hardware requirements
- Have a stable internet connection

## Step 1: Environment Setup

Create a dedicated directory for your validator:

```bash
mkdir ~/bitsec-validator
cd ~/bitsec-validator
```

## Step 2: Configuration

Create your validator configuration file:

```bash
cp config/validator.example.yaml config/validator.yaml
```

Edit the configuration file with your specific settings:

```yaml
# validator.yaml
validator:
  netuid: 1  # Adjust based on your subnet
  wallet_name: "my_validator"
  hotkey_name: "my_hotkey"
  
network:
  port: 8080
  external_ip: "your.external.ip"
  
logging:
  level: "INFO"
  file: "logs/validator.log"
```

## Step 3: Wallet Setup

Create and configure your validator wallet:

```bash
btcli wallet new_coldkey --wallet.name my_validator
btcli wallet new_hotkey --wallet.name my_validator --wallet.hotkey my_hotkey
```

## Step 4: Registration

Register your validator on the network:

```bash
btcli subnet register --wallet.name my_validator --wallet.hotkey my_hotkey --netuid 1
```

## Step 5: Start Validator

Launch your validator node:

```bash
python validator.py --config config/validator.yaml
```

## Monitoring

Monitor your validator performance:

```bash
# Check validator status
btcli wallet overview --wallet.name my_validator

# View logs
tail -f logs/validator.log
```

## Next Steps

- Set up [monitoring and alerts](../subnet/monitoring.md)
- Review [best practices](best-practices.md)
- Join the validator community for support