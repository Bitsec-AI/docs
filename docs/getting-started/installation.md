# Installation

This guide will walk you through installing the necessary components to get started with Bitsec-AI.

## System Requirements

### Minimum Requirements
- **OS**: Ubuntu 20.04 LTS or later, macOS 10.15+, or Windows 10+
- **RAM**: 8GB minimum, 16GB recommended
- **Storage**: 100GB available space
- **Network**: Stable internet connection with good bandwidth

### Recommended Requirements
- **RAM**: 32GB or more
- **Storage**: 500GB SSD
- **CPU**: Multi-core processor (8+ cores recommended)

## Installation Steps

### 1. Install Python

Ensure you have Python 3.8 or higher:

```bash
python3 --version
```

If you need to install Python:

=== "Ubuntu/Debian"
    ```bash
    sudo apt update
    sudo apt install python3 python3-pip python3-venv
    ```

=== "macOS"
    ```bash
    # Using Homebrew
    brew install python3
    ```

=== "Windows"
    Download Python from [python.org](https://python.org) and follow the installation wizard.

### 2. Install Git

```bash
git --version
```

If Git is not installed:

=== "Ubuntu/Debian"
    ```bash
    sudo apt install git
    ```

=== "macOS"
    ```bash
    brew install git
    ```

=== "Windows"
    Download Git from [git-scm.com](https://git-scm.com) and install.

### 3. Clone the Repository

```bash
git clone https://github.com/Bitsec-AI/subnet
cd subnet
```

### 4. Set Up Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 5. Install Dependencies

```bash
pip install -r requirements.txt
```

## Verification

To verify your installation:

```bash
python3 -c "import bittensor; print('Bittensor installed successfully')"
```

## Next Steps

Now that you have everything installed:

- **Validators**: Proceed to [Validator Introduction](../validators/introduction.md)
- **Miners**: Proceed to [Miner Introduction](../miners/introduction.md)

## Troubleshooting

### Common Issues

**Python version conflicts**: Use `python3` explicitly instead of `python`.

**Permission errors**: Use virtual environments to avoid system-wide installations.

**Network issues**: Ensure your firewall allows the necessary connections.

For more help, see our [troubleshooting guide](../subnet/troubleshooting.md).