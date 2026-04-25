# Quip Network Node Setup Guide

Complete guide for deploying and running a Quip Network validator node on **TESTNET**.

## Overview

Quip Network is a decentralized infrastructure network currently in **TESTNET** phase. Running a testnet node helps secure the network and may qualify for future mainnet rewards.

⚠️ **Important**: Quip Network is currently in TESTNET. Mainnet launch date TBA.

## Network Status

| Phase | Status | Date |
|-------|--------|------|
| Testnet | ✅ Active | Live |
| Mainnet | ⏳ Planned | TBA |

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

### Method 1: Docker (Recommended for Testnet)

```bash
# Install Docker
sudo apt update
sudo apt install -y docker.io docker-compose

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Pull Quip testnet image
docker pull quipnetwork/quip-node:testnet

# Create data directory
mkdir -p ~/.quip/data

# Run testnet node
docker run -d \
  --name quip-node \
  --restart unless-stopped \
  -p 30303:30303 \
  -p 30303:30303/udp \
  -v ~/.quip/data:/data \
  quipnetwork/quip-node:testnet \
  --data-dir /data \
  --network testnet
```

### Method 2: Binary Installation

```bash
# Download latest testnet release
wget https://github.com/quipnetwork/quip-node/releases/latest/download/quip-node-linux-amd64.tar.gz

# Extract
tar -xzf quip-node-linux-amd64.tar.gz
sudo mv quip-node /usr/local/bin/

# Create data directory
mkdir -p ~/.quip/data

# Initialize node for testnet
quip-node init --data-dir ~/.quip/data --network testnet

# Start testnet node
quip-node start --data-dir ~/.quip/data --network testnet
```

### Method 3: Build from Source

```bash
# Install dependencies
sudo apt update
sudo apt install -y build-essential cmake git libssl-dev

# Clone repository
git clone https://github.com/quipnetwork/quip-node.git
cd quip-node

# Build for testnet
mkdir build && cd build
cmake -DNETWORK=testnet ..
make -j$(nproc)

# Install
sudo make install
```

## Configuration

### Testnet Configuration

```bash
# Create config file
mkdir -p ~/.quip
cat > ~/.quip/config.toml << 'EOF'
[network]
id = "testnet"
bootnodes = [
  "/dns4/bootnode1.testnet.quip.network/tcp/30303/p2p/...",
  "/dns4/bootnode2.testnet.quip.network/tcp/30303/p2p/..."
]

[node]
data_dir = "/home/$USER/.quip/data"
name = "your-testnet-node"
max_peers = 50

[rpc]
enabled = true
port = 8545
cors_origins = ["*"]

# Testnet faucet for test tokens
[faucet]
enabled = true
url = "https://faucet.testnet.quip.network"
EOF
```

### Getting Testnet Tokens

```bash
# Request testnet tokens from faucet
quip-node faucet request --address YOUR_ADDRESS

# Or visit faucet website
curl -X POST https://faucet.testnet.quip.network \
  -d '{"address": "YOUR_ETH_ADDRESS"}'
```

## Systemd Service Setup

```bash
sudo tee /etc/systemd/system/quip-node.service > /dev/null << 'EOF'
[Unit]
Description=Quip Network Testnet Node
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME
ExecStart=/usr/local/bin/quip-node start --config $HOME/.quip/config.toml --network testnet
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

### Check Testnet Node Status

```bash
# Node status
quip-node status --network testnet

# Peer count
quip-node peers count

# Block height
quip-node chain height

# Sync status
quip-node sync status

# Testnet specific info
quip-node network info
```

### Logs

```bash
# View logs
sudo journalctl -u quip-node -f

# Docker logs
docker logs -f quip-node
```

## Testnet to Mainnet Migration

When mainnet launches:

1. **Backup testnet data**:
   ```bash
   cp -r ~/.quip/data ~/.quip/data-testnet-backup
   ```

2. **Update to mainnet**:
   ```bash
   # Pull mainnet image
   docker pull quipnetwork/quip-node:mainnet
   
   # Or update binary
   quip-node update --network mainnet
   ```

3. **Update config**:
   ```toml
   [network]
   id = "mainnet"
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

### Issue: Cannot connect to testnet

**Symptoms**: Node stuck at "connecting to network"

**Solutions**:
1. Verify testnet bootnodes:
   ```bash
   quip-node config get network.bootnodes
   ```

2. Check testnet status:
   ```bash
   curl https://status.testnet.quip.network
   ```

3. Reset and resync:
   ```bash
   quip-node reset --network testnet
   quip-node start --network testnet
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
   telnet bootnode1.testnet.quip.network 30303
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

### Issue: Faucet not working

**Symptoms**: Cannot get testnet tokens

**Solutions**:
1. Check faucet status page
2. Try alternative faucet:
   ```bash
   quip-node faucet request --provider alternate
   ```
3. Join Discord for manual faucet request

### Issue: Low testnet rewards

**Symptoms**: Not earning testnet points

**Solutions**:
1. Verify node is fully synced
2. Check uptime requirements
3. Ensure proper peer connections
4. Check testnet reward criteria

## FAQ

**Q: Is Quip mainnet live?**
A: No, Quip is currently in testnet phase. Mainnet launch date is TBA.

**Q: Do testnet rewards transfer to mainnet?**
A: Testnet participation may qualify for mainnet incentives, but not guaranteed.

**Q: How long does testnet sync take?**
A: Initial sync takes 2-6 hours depending on network and hardware.

**Q: Can I run multiple testnet nodes?**
A: Yes, but each needs unique resources and configuration.

**Q: What happens to my node after mainnet launch?**
A: You can migrate your node to mainnet or continue running on testnet.

**Q: How do I update the node?**
A: ```bash
quip-node stop
quip-node update --network testnet
quip-node start --network testnet
```

## Resources

- [Official Docs](https://docs.quip.network)
- [Testnet Explorer](https://explorer.testnet.quip.network)
- [Discord Community](https://discord.gg/quip)
- [GitHub](https://github.com/quipnetwork)
- [Testnet Faucet](https://faucet.testnet.quip.network)

## License

MIT
