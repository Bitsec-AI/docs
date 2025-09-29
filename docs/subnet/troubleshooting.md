# Troubleshooting Guide

This comprehensive troubleshooting guide helps resolve common issues encountered when operating on the Bitsec-AI subnet, covering both validator and miner operations.

## General Troubleshooting Approach

### Systematic Problem Resolution

1. **Identify the Problem**
   - Gather symptoms and error messages
   - Determine scope (single node vs. network-wide)
   - Check timing (when did it start?)

2. **Gather Information**
   - Check logs and metrics
   - Verify system resources
   - Test network connectivity

3. **Isolate the Cause**
   - Rule out common causes
   - Test components individually
   - Use diagnostic tools

4. **Implement Solution**
   - Apply targeted fixes
   - Monitor for resolution
   - Document the solution

## Common Issues and Solutions

### Connection and Network Issues

#### Issue: Cannot Connect to Subnet

**Symptoms:**
```bash
ERROR: Connection refused to subnet endpoint
ERROR: Timeout connecting to validator
```

**Diagnosis:**
```bash
# Check network connectivity
ping api.bitsec.ai

# Test specific ports
telnet api.bitsec.ai 8080

# Check DNS resolution
nslookup api.bitsec.ai

# Verify firewall settings
sudo ufw status verbose
```

**Solutions:**

1. **Firewall Configuration**
   ```bash
   # Open required ports
   sudo ufw allow 8080/tcp
   sudo ufw allow 9946/tcp
   sudo ufw reload
   ```

2. **Network Configuration**
   ```bash
   # Check network interface
   ip addr show
   
   # Test routing
   traceroute api.bitsec.ai
   
   # Check for proxy settings
   echo $http_proxy
   echo $https_proxy
   ```

3. **DNS Issues**
   ```bash
   # Use alternative DNS
   echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
   
   # Flush DNS cache
   sudo systemctl flush-dns
   ```

#### Issue: High Network Latency

**Symptoms:**
- Slow response times
- Timeout errors
- Poor performance metrics

**Diagnosis:**
```bash
# Measure latency
ping -c 10 api.bitsec.ai

# Check network quality
mtr api.bitsec.ai

# Monitor bandwidth usage
iftop
```

**Solutions:**

1. **Network Optimization**
   ```bash
   # Optimize TCP settings
   echo 'net.core.rmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
   echo 'net.core.wmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
   echo 'net.ipv4.tcp_rmem = 4096 87380 16777216' | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

2. **Use Closer Endpoints**
   ```yaml
   # Update configuration to use regional endpoints
   network:
     primary_endpoint: "us-east.api.bitsec.ai"
     fallback_endpoints:
       - "us-west.api.bitsec.ai"
       - "eu.api.bitsec.ai"
   ```

### Validator-Specific Issues

#### Issue: Validator Not Participating in Consensus

**Symptoms:**
```bash
WARNING: Validator not included in consensus round
ERROR: Consensus participation rate below threshold
```

**Diagnosis:**
```bash
# Check validator registration
btcli subnet list --netuid 1

# Verify validator status
btcli wallet overview --wallet.name my_validator

# Check validator logs
tail -f logs/validator.log

# Monitor consensus participation
grep "consensus" logs/validator.log | tail -20
```

**Solutions:**

1. **Re-register Validator**
   ```bash
   btcli subnet register --wallet.name my_validator --wallet.hotkey my_hotkey --netuid 1
   ```

2. **Check Stake Requirements**
   ```bash
   # Verify minimum stake
   btcli wallet balance --wallet.name my_validator
   
   # Check stake on network
   btcli wallet overview --wallet.name my_validator
   ```

3. **Synchronization Issues**
   ```bash
   # Restart validator
   sudo systemctl restart validator
   
   # Force resync
   python validator.py --resync --config config/validator.yaml
   ```

#### Issue: Low Validation Accuracy

**Symptoms:**
- Poor performance metrics
- Low rewards
- Negative feedback from network

**Diagnosis:**
```python
# Check validation logic
def diagnose_validation_accuracy():
    # Review recent validations
    recent_validations = get_recent_validations(limit=100)
    
    # Calculate accuracy metrics
    accuracy_metrics = calculate_accuracy_metrics(recent_validations)
    
    # Compare with network average
    network_average = get_network_average_accuracy()
    
    return {
        'current_accuracy': accuracy_metrics,
        'network_average': network_average,
        'improvement_needed': network_average - accuracy_metrics
    }
```

**Solutions:**

1. **Update Validation Logic**
   ```python
   # Implement improved validation algorithms
   class ImprovedValidator:
       def __init__(self):
           self.validation_models = load_latest_models()
           self.benchmark_data = load_benchmark_data()
       
       def validate_output(self, miner_output):
           # Use multiple validation methods
           scores = []
           for model in self.validation_models:
               score = model.evaluate(miner_output)
               scores.append(score)
           
           return np.mean(scores)
   ```

2. **Calibrate Thresholds**
   ```yaml
   # Update validation thresholds
   validation:
     quality_threshold: 0.85
     performance_threshold: 0.80
     innovation_threshold: 0.75
   ```

### Miner-Specific Issues

#### Issue: Low Mining Rewards

**Symptoms:**
- Earnings below expectations
- Few successful validations
- Poor ranking in subnet

**Diagnosis:**
```bash
# Check mining performance
python -c "
import json
with open('logs/miner_metrics.json', 'r') as f:
    metrics = json.load(f)
    print(f'Success rate: {metrics[\"success_rate\"]}')
    print(f'Average response time: {metrics[\"avg_response_time\"]}')
    print(f'Quality score: {metrics[\"quality_score\"]}')
"

# Compare with network stats
btcli subnet list --netuid 1 | grep -A 5 -B 5 $(btcli wallet overview --wallet.name my_miner | grep "Hotkey" | cut -d: -f2)
```

**Solutions:**

1. **Optimize Performance**
   ```python
   # Implement performance optimizations
   class OptimizedMiner:
       def __init__(self):
           self.model = load_optimized_model()
           self.cache = LRUCache(maxsize=1000)
       
       def process_request(self, request):
           # Check cache first
           cache_key = hash(str(request))
           if cache_key in self.cache:
               return self.cache[cache_key]
           
           # Process and cache result
           result = self.model.process(request)
           self.cache[cache_key] = result
           return result
   ```

2. **Improve Model Quality**
   ```bash
   # Update to latest models
   pip install --upgrade your-ai-models
   
   # Retrain with recent data
   python train_model.py --data recent_training_data.json
   ```

#### Issue: GPU/Hardware Issues

**Symptoms:**
```bash
ERROR: CUDA out of memory
ERROR: GPU not detected
WARNING: High GPU temperature
```

**Diagnosis:**
```bash
# Check GPU status
nvidia-smi

# Monitor GPU temperature
watch -n 1 nvidia-smi

# Check CUDA installation
python -c "import torch; print(torch.cuda.is_available())"

# Check memory usage
nvidia-smi --query-gpu=memory.used,memory.total --format=csv
```

**Solutions:**

1. **Memory Management**
   ```python
   # Implement memory optimization
   import torch
   
   def optimize_gpu_memory():
       # Clear cache
       torch.cuda.empty_cache()
       
       # Set memory fraction
       torch.cuda.set_per_process_memory_fraction(0.8)
       
       # Use gradient checkpointing
       torch.utils.checkpoint.checkpoint_sequential()
   ```

2. **Temperature Management**
   ```bash
   # Check cooling system
   sensors
   
   # Reduce GPU power limit
   sudo nvidia-smi -pl 200  # 200W limit
   
   # Improve ventilation
   # Ensure proper airflow in server room
   ```

3. **Driver Issues**
   ```bash
   # Update NVIDIA drivers
   sudo apt update
   sudo apt install nvidia-driver-525
   sudo reboot
   
   # Reinstall CUDA toolkit
   sudo apt install nvidia-cuda-toolkit
   ```

### Authentication and Wallet Issues

#### Issue: Wallet Authentication Failures

**Symptoms:**
```bash
ERROR: Invalid wallet credentials
ERROR: Hotkey not found
ERROR: Insufficient permissions
```

**Diagnosis:**
```bash
# Check wallet files
ls -la ~/.bittensor/wallets/

# Verify wallet integrity
btcli wallet check --wallet.name my_wallet

# Test wallet access
btcli wallet balance --wallet.name my_wallet
```

**Solutions:**

1. **Wallet Recovery**
   ```bash
   # Restore from mnemonic
   btcli wallet regen_coldkey --wallet.name my_wallet --mnemonic "your mnemonic phrase"
   
   # Restore hotkey
   btcli wallet regen_hotkey --wallet.name my_wallet --wallet.hotkey my_hotkey
   ```

2. **Permission Issues**
   ```bash
   # Fix file permissions
   chmod 600 ~/.bittensor/wallets/my_wallet/coldkey
   chmod 600 ~/.bittensor/wallets/my_wallet/hotkeys/my_hotkey
   
   # Check ownership
   chown -R $USER:$USER ~/.bittensor/wallets/
   ```

### Performance and Resource Issues

#### Issue: High CPU/Memory Usage

**Symptoms:**
- System slowdown
- Out of memory errors
- High load averages

**Diagnosis:**
```bash
# Monitor resource usage
top -p $(pgrep -f "validator\|miner")

# Check memory usage
free -h

# Analyze process memory
ps aux --sort=-%mem | head

# Check for memory leaks
valgrind --tool=memcheck --leak-check=full python miner.py
```

**Solutions:**

1. **Resource Optimization**
   ```python
   # Implement resource monitoring
   import psutil
   import gc
   
   class ResourceMonitor:
       def __init__(self, memory_limit_gb=8):
           self.memory_limit = memory_limit_gb * 1024 * 1024 * 1024
       
       def check_resources(self):
           # Check memory usage
           memory_usage = psutil.virtual_memory().used
           if memory_usage > self.memory_limit:
               self.cleanup_memory()
           
           # Check CPU usage
           cpu_usage = psutil.cpu_percent(interval=1)
           if cpu_usage > 90:
               self.reduce_load()
       
       def cleanup_memory(self):
           gc.collect()
           torch.cuda.empty_cache()
   ```

2. **Configuration Tuning**
   ```yaml
   # Optimize configuration
   performance:
     batch_size: 16  # Reduce if memory issues
     max_workers: 4  # Limit concurrent operations
     memory_limit_gb: 8
     
   logging:
     level: "WARNING"  # Reduce logging overhead
     max_file_size: "100MB"
   ```

### Database and Storage Issues

#### Issue: Database Corruption

**Symptoms:**
```bash
ERROR: Database connection failed
ERROR: Corrupted index file
WARNING: Inconsistent data detected
```

**Diagnosis:**
```bash
# Check database integrity
sqlite3 database.db "PRAGMA integrity_check;"

# Check disk space
df -h

# Check file system errors
sudo fsck /dev/sda1

# Monitor I/O operations
iotop
```

**Solutions:**

1. **Database Recovery**
   ```bash
   # Backup current database
   cp database.db database.db.backup
   
   # Attempt repair
   sqlite3 database.db ".recover" | sqlite3 database_recovered.db
   
   # Replace with recovered version
   mv database_recovered.db database.db
   ```

2. **Prevent Future Corruption**
   ```python
   # Implement database health monitoring
   import sqlite3
   
   class DatabaseMonitor:
       def __init__(self, db_path):
           self.db_path = db_path
       
       def health_check(self):
           try:
               conn = sqlite3.connect(self.db_path)
               cursor = conn.cursor()
               cursor.execute("PRAGMA integrity_check;")
               result = cursor.fetchone()
               conn.close()
               
               return result[0] == "ok"
           except Exception as e:
               return False
       
       def backup_database(self):
           # Implement regular backups
           pass
   ```

## Diagnostic Tools and Scripts

### Comprehensive Health Check Script

```bash
#!/bin/bash
# health_check.sh

echo "=== Bitsec-AI Node Health Check ==="

# System health
echo "1. System Resources:"
echo "   CPU: $(top -bn1 | grep 'Cpu(s)' | awk '{print $2}' | cut -d'%' -f1)% used"
echo "   Memory: $(free | grep Mem | awk '{printf("%.1f%%", $3/$2 * 100.0)}')"
echo "   Disk: $(df -h / | awk 'NR==2{printf "%s", $5}')"

# Network connectivity
echo "2. Network Connectivity:"
if ping -c 1 api.bitsec.ai &> /dev/null; then
    echo "   ✓ Internet connectivity: OK"
else
    echo "   ✗ Internet connectivity: FAILED"
fi

# Service status
echo "3. Service Status:"
if pgrep -f "validator\|miner" > /dev/null; then
    echo "   ✓ Node process: Running"
else
    echo "   ✗ Node process: Not running"
fi

# Wallet status
echo "4. Wallet Status:"
if btcli wallet balance --wallet.name my_wallet &> /dev/null; then
    echo "   ✓ Wallet access: OK"
else
    echo "   ✗ Wallet access: FAILED"
fi

echo "=== Health Check Complete ==="
```

### Log Analysis Script

```python
#!/usr/bin/env python3
# analyze_logs.py

import re
import json
from datetime import datetime, timedelta
from collections import defaultdict

class LogAnalyzer:
    def __init__(self, log_file):
        self.log_file = log_file
        self.error_patterns = {
            'connection_errors': r'Connection.*failed|Timeout|Connection refused',
            'memory_errors': r'out of memory|MemoryError|OOM',
            'gpu_errors': r'CUDA.*error|GPU.*error',
            'wallet_errors': r'wallet.*error|authentication.*failed'
        }
    
    def analyze(self):
        errors = defaultdict(int)
        recent_errors = []
        
        with open(self.log_file, 'r') as f:
            for line in f:
                for error_type, pattern in self.error_patterns.items():
                    if re.search(pattern, line, re.IGNORECASE):
                        errors[error_type] += 1
                        
                        # Extract timestamp and add to recent errors
                        if 'ERROR' in line:
                            recent_errors.append({
                                'timestamp': self.extract_timestamp(line),
                                'type': error_type,
                                'message': line.strip()
                            })
        
        return {
            'error_summary': dict(errors),
            'recent_errors': recent_errors[-10:],  # Last 10 errors
            'recommendations': self.get_recommendations(errors)
        }
    
    def get_recommendations(self, errors):
        recommendations = []
        
        if errors['connection_errors'] > 10:
            recommendations.append("Check network connectivity and firewall settings")
        
        if errors['memory_errors'] > 5:
            recommendations.append("Consider reducing batch size or adding more RAM")
        
        if errors['gpu_errors'] > 3:
            recommendations.append("Check GPU drivers and CUDA installation")
        
        return recommendations

if __name__ == "__main__":
    analyzer = LogAnalyzer('logs/node.log')
    results = analyzer.analyze()
    print(json.dumps(results, indent=2))
```

## Advanced Troubleshooting

### Remote Debugging

```python
# Enable remote debugging for complex issues
import pdb
import sys

def enable_remote_debugging():
    """Enable remote debugging capability"""
    try:
        import remote_pdb
        remote_pdb.set_trace()
    except ImportError:
        print("Install remote-pdb for remote debugging")
        pdb.set_trace()

# Usage in your code
if DEBUG_MODE:
    enable_remote_debugging()
```

### Performance Profiling

```python
import cProfile
import pstats
from functools import wraps

def profile_performance(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        
        result = func(*args, **kwargs)
        
        profiler.disable()
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(10)  # Top 10 functions
        
        return result
    return wrapper

# Usage
@profile_performance
def mining_function():
    # Your mining logic
    pass
```

## Getting Help

### Support Channels

1. **Community Forums**
   - Discord: [Bitsec-AI Discord](https://discord.gg/bitsec-ai)
   - Telegram: [Bitsec-AI Telegram](https://t.me/bitsec_ai)

2. **Documentation**
   - GitHub: [Bitsec-AI/docs](https://github.com/Bitsec-AI/docs)
   - Official Docs: [docs.bitsec.ai](https://docs.bitsec.ai)

3. **Technical Support**
   - Create GitHub issue with:
     - Detailed problem description
     - Error messages and logs
     - System configuration
     - Steps to reproduce

### Reporting Issues

When reporting issues, include:

```bash
# System information
uname -a
python --version
pip list | grep -E "(torch|bittensor|numpy)"

# Hardware information
lscpu
free -h
nvidia-smi  # if using GPU

# Network configuration
ip addr show
netstat -tuln | grep -E "(8080|9946)"

# Error logs (last 50 lines)
tail -50 logs/node.log
```

## Conclusion

Effective troubleshooting requires a systematic approach and good understanding of the system components. Use the diagnostic tools and scripts provided, and don't hesitate to reach out to the community for help with complex issues.

Keep your logs detailed, monitor your systems proactively, and maintain good documentation of any custom configurations or modifications you've made.