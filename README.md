# Drosera-Network Trap Deployment Guide

This guide helps you set up and contribute to the **Drosera Testnet** in a simplified and structured way, with a focus on **clarity**, **step-by-step instructions**, and **structure explanation**.



## Project Overview
The Drosera network is a decentralized honeypot-style security system for Ethereum.

You will:
- Install dependencies and CLI tools
- Deploy a Trap (vulnerable contract)
- Register and run an Operator node
- Connect your trap and monitor it from the dashboard



## System Requirements
Make sure your system meets the following minimum specs:

| Resource  | Minimum |
| --------- | ------- |
| CPU Cores | 2       |
| RAM       | 4 GB    |
| Disk      | 20 GB   |
| OS        | Ubuntu Linux (recommended) |



## 1. Install Dependencies
Install all necessary development tools, networking utilities, and libraries.

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip -y
```

### Docker Installation
Drosera uses Docker for isolated component execution.

```bash
# Remove old versions
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repo
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Test
sudo docker run hello-world
```



## 2. Configure Environment Tools
Set up Drosera CLI, Foundry, and Bun.

### Drosera CLI
Install and update Drosera CLI.
```bash
curl -L https://app.drosera.io/install | bash
source /root/.bashrc
droseraup
```

### Foundry CLI
Toolkit for compiling and deploying Ethereum smart contracts.
```bash
curl -L https://foundry.paradigm.xyz | bash
source /root/.bashrc
foundryup
```

### Bun
Used for JavaScript dependency management within the trap template.
```bash
curl -fsSL https://bun.sh/install | bash
source /root/.bashrc
```



## 3. Deploy Contract & Trap
We’ll create the trap project and deploy it to Holesky testnet.

### Create Project Directory
```bash
mkdir my-drosera-trap
cd my-drosera-trap
```

### Git Identity (replace placeholders)
```bash
git config --global user.email "Github_Email"
git config --global user.name "Github_Username"
```

### Initialize Trap Template
```bash
forge init -t drosera-network/trap-foundry-template
```

### Install Dependencies & Compile
```bash
bun install
forge build
```

### Deploy Trap
Replace `xxx` with your wallet's private key (ensure it has Holesky ETH).
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```
Type `ofc` to confirm deployment.



## 4. Dashboard Interaction
Monitor and manage your Trap via the Drosera dashboard:
- Go to [https://app.drosera.io](https://app.drosera.io)
- Connect your wallet
- View "Traps Owned"
- Click **Send Bloom Boost** to deposit Holesky ETH



## 5. Dry Run Simulation
Simulate Trap readiness locally.
```bash
drosera dryrun
```



##  6. Whitelist Operator
Restrict the trap to interact only with your chosen operator.

### Edit `drosera.toml`
```bash
cd ~/my-drosera-trap
nano drosera.toml
```
Add at the bottom:
```toml
private_trap = true
whitelist = ["Operator_Address"]
```
Replace `Operator_Address` with your public wallet address.

### Reapply Configuration
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```



## 7. Install Operator CLI
Download and set up the Operator binary.
```bash
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.16.2/drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz

# Extract



tar -xvf drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz
./drosera-operator --version

# Move to path
sudo cp drosera-operator /usr/bin/
drosera-operator --version
```



## 8. Create Operator systemd
Configure the node to auto-start using systemd.

Replace:
- `PV_KEY` with your private key
- `VPS_IP` with your VPS IP or `localhost`

```bash
sudo tee /etc/systemd/system/drosera.service > /dev/null <<EOF
[Unit]
Description=drosera node service
After=network-online.target

[Service]
User=$USER
Restart=always
RestartSec=15
LimitNOFILE=65535
ExecStart=$(which drosera-operator) node \
  --db-file-path $HOME/.drosera.db \
  --network-p2p-port 31313 \
  --server-port 31314 \
  --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com \
  --eth-backup-rpc-url https://1rpc.io/holesky \
  --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 \
  --eth-private-key PV_KEY \
  --listen-address 0.0.0.0 \
  --network-external-p2p-address VPS_IP \
  --disable-dnr-confirmation true

[Install]
WantedBy=multi-user.target
EOF
```



## 9. Open Firewall Ports
Allow communication ports.
```bash
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw enable
sudo ufw allow 31313/tcp
sudo ufw allow 31314/tcp
```



## 10. Start & Monitor Operator
Enable systemd and monitor logs.
```bash
sudo systemctl daemon-reload
sudo systemctl enable drosera
sudo systemctl start drosera

# Live logs
journalctl -u drosera.service -f
```
Ignore: `WARN drosera_services::network::service: Failed to gossip message: InsufficientPeers`



## 11. Final Dashboard Steps
- Go to Drosera App Dashboard
- Click **Opt-in** to connect your Trap and Operator
- You should now see **green blocks** showing successful operation



## 12. Project Structure
Here's how your project should look:
```
my-drosera-trap/
├── foundry.toml           # Foundry config
├── drosera.toml           # Trap config
├── script/                # Deployment scripts
├── src/                   # Solidity contracts
├── test/                  # Unit tests
├── out/                   # Build outputs
├── lib/                   # Dependencies
```



## 🌐 Join the Community
- Need help? Ask in the [Drosera Discord](https://discord.com/invite/drosera)
- Follow development on GitHub

Happy trapping 🪤

