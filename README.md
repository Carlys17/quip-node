# Quip Network Node Setup Guide

Complete guide for deploying and running a Quip Network validator node.

## Overview

Quip Network is a decentralized infrastructure network providing distributed computing services. Running a node helps secure the network while earning rewards.

## System Requirements

### Minimum Requirements
- **CPU**: 4 cores
- **RAM**: 8 GB
- **Storage**: 100 GB SSD
- **Network**: 10 Mbps stable connection
- **OS**: Ubuntu 20.04/22.04 LTS

### Recommended
- **CPU**: 8+ cores
- **RAM**: 16 GB
- **Storage**: 200 GB NVMe SSD
- **Network**: 100 Mbps+ with static IP

## Installation

### Method 1: Docker (Recommended)

```bash
# Install Docker
sudo apt update
sudo apt install -y docker.io docker-compose

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Pull Quip image
docker pull quipnetwork/quip-node:latest

# Create data directory
mkdir -p ~/.quip/data

# Run node
docker run -d \
  --name quip-node \
  --restart unless-stopped \
  -p 30303:30303 \
  -p 30303:30303/udp \
  -v ~/.quip/data:/data \
  quipnetwork/quip-node:latest \
  --data-dir /data \
  --network mainnet
```

### Method 2: Binary Installation

```bash
# Download latest release
wget https://github.com/quipnetwork/quip-node/releases/latest/download/quip-node-linux-amd64.tar.gz

# Extract
tar -xzf quip-node-linux-amd64.tar.gz
sudo mv quip-node /usr/local/bin/

# Create data directory
mkdir -p ~/.quip/data

# Initialize node
quip-node init --data-dir ~/.quip/data

# Start node
quip-node start --data-dir ~/.quip/data --network mainnet
```

### Method 3: Build from Source

```bash
# Install dependencies
sudo apt update
sudo apt install -y build-essential cmake git libssl-dev

# Clone repository
git clone https://github.com/quipnetwork/quip-node.git
cd quip-node

# Build
mkdir build && cd build
cmake ..
make -j$(nproc)

# Install
sudo make install
```

## Configuration

### Basic Configuration

```bash
# Create config file
mkdir -p ~/.quip
cat > ~/.quip/config.toml << 'EOF'
[network]
id = "mainnet"
bootnodes = [
  "/dns4/bootnode1.quip.network/tcp/30303/p2p/...",
  "/dns4/bootnode2.quip.network/tcp/30303/p2p/..."
]

[node]
data_dir = "/home/$USER/.quip/data"
name = "your-node-name"
max_peers = 50

[rpc]
enabled = true
port = 8545
cors_origins = ["*"]

[rewards]
address = "YOUR_ETHEREUM_ADDRESS"
EOF
```

### Advanced Configuration

```toml
# ~/.quip/config.toml
[network]
id = "mainnet"
bootnodes = [...]

[node]
data_dir = "/data/quip"
name = "carly-quip-node"
max_peers = 100
min_peers = 10

[rpc]
enabled = true
port = 8545
host = "127.0.0.1"
cors_origins = ["http://localhost:3000"]

[metrics]
enabled = true
port = 9090

[rewards]
address = "0x..."
auto_claim = true

[performance]
cache_size = 4096
db_threads = 4
```

## Systemd Service Setup

```bash
sudo tee /etc/systemd/system/quip-node.service > /dev/null << 'EOF'
[Unit]
Description=Quip Network Node
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME
ExecStart=/usr/local/bin/quip-node start --config $HOME/.quip/config.toml
Restart=always
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable quip-node
sudo systemctl start quip-node
```

## Monitoring

### Check Node Status

```bash
# Node status
quip-node status

# Peer count
quip-node peers count

# Block height
quip-node chain height

# Sync status
quip-node sync status
```

### Logs

```bash
# View logs
sudo journalctl -u quip-node -f

# Docker logs
docker logs -f quip-node
```

## Troubleshooting

### Issue: Node won't start

**Symptoms**: Service fails immediately

**Solutions**:
1. Check config syntax:
   ```bash
   quip-node validate-config --config ~/.quip/config.toml
   ```

2. Check permissions:
   ```bash
   ls -la ~/.quip/data
   sudo chown -R $USER:$USER ~/.quip
   ```

3. Check port conflicts:
   ```bash
   netstat -tlnp | grep 30303
   ```

### Issue: Sync stuck

**Symptoms**: Block height not increasing

**Solutions**:
1. Restart node:
   ```bash
   sudo systemctl restart quip-node
   ```

2. Reset database (WARNING: loses local data):
   ```bash
   sudo systemctl stop quip-node
   rm -rf ~/.quip/data/*
   sudo systemctl start quip-node
   ```

3. Check network connectivity:
   ```bash
   telnet bootnode1.quip.network 30303
   ```

### Issue: Low peer count

**Symptoms**: Fewer than 10 peers

**Solutions**:
1. Check firewall:
   ```bash
   sudo ufw allow 30303/tcp
   sudo ufw allow 30303/udp
   ```

2. Add manual peers:
   ```bash
   quip-node peers add /ip4/x.x.x.x/tcp/30303/p2p/...
   ```

3. Check NAT/port forwarding

### Issue: No rewards

**Symptoms**: Node running but no rewards

**Solutions**:
1. Verify address in config
2. Check minimum uptime requirement
3. Verify node is fully synced
4. Check eligibility on dashboard

## FAQ

**Q: How long does sync take?**
A: Initial sync takes 2-6 hours depending on network and hardware.

**Q: Can I run multiple nodes?**
A: Yes, but each needs unique resources and configuration.

**Q: What happens if my node goes offline?**
A: Rewards pause but resume when node comes back online (if within grace period).

**Q: How to backup node?**
A: Backup `~/.quip/data` directory and config file.

## Resources

- [Official Docs](https://docs.quip.network)
- [Explorer](https://explorer.quip.network)
- [Discord](https://discord.gg/quip)
- [GitHub](https://github.com/quipnetwork)

## License

MIT
