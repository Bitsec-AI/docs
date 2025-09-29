# Miner Setup Guide

This comprehensive guide will help you set up a miner node for the Bitsec-AI subnet on the Bittensor network.

## Prerequisites

Before starting:

- Complete the [installation guide](../getting-started/installation.md)
- Have adequate hardware for mining operations
- Understand the competitive nature of mining
- Have a stable internet connection with good bandwidth

## Step 1: Environment Setup

Create a dedicated directory for your miner:

```bash
mkdir ~/bitsec-miner
cd ~/bitsec-miner
```

Set up a Python virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

## Step 2: Hardware Configuration

### GPU Setup (Recommended)

For optimal performance, configure GPU acceleration:

```bash
# Check GPU availability
nvidia-smi

# Install CUDA toolkit if needed
sudo apt update
sudo apt install nvidia-cuda-toolkit

# Verify CUDA installation
nvcc --version
```

### CPU Mining Setup

If using CPU mining:

```bash
# Set CPU affinity for better performance
taskset -c 0-7 python miner.py
```

## Step 3: Configuration

Create your miner configuration file:

```bash
cp config/miner.example.yaml config/miner.yaml
```

Edit the configuration with your specific settings:

```yaml
# miner.yaml
miner:
  netuid: 1  # Adjust based on your subnet
  wallet_name: "my_miner"
  hotkey_name: "my_hotkey"
  
performance:
  batch_size: 16
  max_workers: 4
  timeout: 30
  
hardware:
  device: "cuda"  # or "cpu"
  memory_fraction: 0.8
  
network:
  port: 8081
  external_ip: "your.external.ip"
  
logging:
  level: "INFO"
  file: "logs/miner.log"
```

## Step 4: Wallet Setup

Create and configure your miner wallet:

```bash
btcli wallet new_coldkey --wallet.name my_miner
btcli wallet new_hotkey --wallet.name my_miner --wallet.hotkey my_hotkey
```

## Step 5: Registration

Register your miner on the subnet:

```bash
btcli subnet register --wallet.name my_miner --wallet.hotkey my_hotkey --netuid 1
```

## Step 6: Install Dependencies

Install required Python packages:

```bash
pip install -r requirements.txt
```

For GPU acceleration:

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

## Step 7: Start Mining

Launch your miner:

```bash
python miner.py --config config/miner.yaml
```

For background operation:

```bash
nohup python miner.py --config config/miner.yaml > logs/miner.log 2>&1 &
```

## Step 8: Monitoring

Monitor your miner's performance:

```bash
# Check miner status
btcli wallet overview --wallet.name my_miner

# View real-time logs
tail -f logs/miner.log

# Monitor system resources
htop
nvidia-smi  # For GPU monitoring
```

## Performance Optimization

### GPU Optimization

For NVIDIA GPUs:

```bash
# Set GPU power mode
sudo nvidia-smi -pm 1

# Set memory and core clocks
sudo nvidia-smi -ac 5001,1506

# Monitor GPU utilization
watch -n 1 nvidia-smi
```

### Memory Management

Optimize memory usage:

```python
# In your configuration
memory_settings:
  max_memory_gb: 8
  garbage_collection: true
  memory_mapped_files: true
```

### Network Optimization

Improve network performance:

```bash
# Increase network buffer sizes
echo 'net.core.rmem_default = 262144' | sudo tee -a /etc/sysctl.conf
echo 'net.core.rmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Troubleshooting

### Common Issues

**GPU not detected:**
```bash
# Check CUDA installation
python -c "import torch; print(torch.cuda.is_available())"

# Reinstall GPU drivers if needed
sudo apt purge nvidia*
sudo apt autoremove
sudo apt install nvidia-driver-525
```

**Memory errors:**
```bash
# Reduce batch size in configuration
batch_size: 8  # Instead of 16

# Clear GPU memory
python -c "import torch; torch.cuda.empty_cache()"
```

**Connection issues:**
```bash
# Check network connectivity
ping -c 4 api.bitsec.ai

# Verify firewall settings
sudo ufw status
```

## Advanced Configuration

### Model Customization

Configure custom AI models:

```yaml
model:
  name: "custom_model_v1"
  path: "/path/to/model"
  precision: "fp16"
  optimization: "tensorrt"
```

### Load Balancing

For multiple GPU setup:

```yaml
multi_gpu:
  enabled: true
  devices: [0, 1, 2, 3]
  strategy: "data_parallel"
```

## Security Considerations

### Wallet Security

1. **Separate Funds**: Keep mining rewards separate from main holdings
2. **Regular Withdrawals**: Don't accumulate large amounts in hot wallets
3. **Key Management**: Secure backup of all wallet keys

### System Security

1. **Access Control**: Limit SSH access to mining systems
2. **Monitoring**: Set up alerts for unusual activity
3. **Updates**: Keep system and mining software updated

## Next Steps

After successful setup:

1. **[Optimization Guide](optimization.md)** - Maximize your mining efficiency
2. **[Monitoring Setup](../subnet/monitoring.md)** - Set up comprehensive monitoring
3. **Community Engagement** - Join miner discussions and share experiences

## Support

For miner-specific support:

- Check the [optimization guide](optimization.md)
- Review [troubleshooting documentation](../subnet/troubleshooting.md)
- Join the miner community forums
- Contact technical support through official channels