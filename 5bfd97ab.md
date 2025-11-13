# Complete Guide to Forking Dero HE Blockchain on Linux Mint

## Table of Contents

1. [Prerequisites and System Setup](#prerequisites-and-system-setup)
2. [Understanding Dero HE](#understanding-dero-he)
3. [Cloning the Dero HE Repository](#cloning-the-dero-he-repository)
4. [Configuring Your Blockchain Fork](#configuring-your-blockchain-fork)
5. [Setting Up a Premine](#setting-up-a-premine)
6. [Building Your Forked Blockchain](#building-your-forked-blockchain)
7. [Running Your Blockchain Network](#running-your-blockchain-network)
8. [Creating and Claiming Premine](#creating-and-claiming-premine)
9. [Troubleshooting and Next Steps](#troubleshooting-and-next-steps)

---

## Prerequisites and System Setup

### Step 1: Update Your Linux Mint System

Before beginning the forking process, ensure your system is completely up to date with the latest security patches and packages.

Open your terminal and run the following commands:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential git wget curl -y
```

The `build-essential` package provides essential compilation tools, while `git`, `wget`, and `curl` will help you download and manage the source code.

### Step 2: Install Go (Golang)

Dero HE is written entirely in Go, so you must install Golang. Linux Mint recommends Go version 1.17 or higher for Dero HE development.

#### Method 1: Install from APT Repository (Recommended for Linux Mint)

```bash
sudo apt install golang-go -y
```

After installation, verify the version:

```bash
go version
```

If the version is older than 1.17, use Method 2 below.

#### Method 2: Install Latest Go from Official Repository

Download the latest Go binary:

```bash
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
```

Set up Go paths by editing your shell profile:

```bash
echo "export GOPATH=\$HOME/go" >> ~/.bashrc
echo "export PATH=\$PATH:/usr/local/go/bin:\$GOPATH/bin" >> ~/.bashrc
source ~/.bashrc
```

Verify installation:

```bash
go version
```

### Step 3: Set Up Go Workspace

Create the necessary Go workspace directories:

```bash
mkdir -p ~/go/{bin,pkg,src}
```

Verify the workspace:

```bash
ls -la ~/go/
```

---

## Understanding Dero HE

### What is Dero HE?

Dero HE (Homomorphic Encryption) is a privacy-focused blockchain written in Go that features:

- **Homomorphic Encryption**: Allows operations on encrypted data without decryption
- **Account-Based Model**: Unlike UTXO-based blockchains
- **16-18 Second Block Time**: Fast transaction confirmation
- **AstroBWT Mining Algorithm**: CPU-friendly, ASIC-resistant proof-of-work
- **Fully Encrypted Transactions**: Privacy-preserving smart contracts
- **21 Million Total Supply**: Maximum cap similar to Bitcoin

### Key Components You'll Modify

When forking Dero HE, you'll primarily modify:

1. **Chain Parameters**: Block time, difficulty adjustment, network ports
2. **Genesis Block Configuration**: Initial state and premine allocation
3. **Coin Parameters**: Supply limits, atomic units, block rewards
4. **Network Identification**: Chain ID and network ports
5. **Mining Parameters**: Difficulty algorithm and mining rewards

---

## Cloning the Dero HE Repository

### Step 1: Clone the Official Dero HE Repository

Navigate to your Go source directory and clone the repository:

```bash
cd ~/go/src
git clone https://github.com/deroproject/derohe.git
cd derohe
```

### Step 2: Explore the Repository Structure

Understanding the directory structure is crucial:

```bash
ls -la
```

Key directories:

- `cmd/` - Command-line utilities and main executables
- `cmd/derod/` - The daemon/node software
- `cmd/dero-wallet-cli/` - CLI wallet
- `cmd/explorer/` - Block explorer
- `cryptography/` - Cryptographic implementations
- `blockchain/` - Core blockchain logic
- `p2p/` - Peer-to-peer networking
- `config/` - Configuration files (if present)

### Step 3: Review the Current Branch

Check which branch you're on:

```bash
git branch -a
git log --oneline | head -20
```

---

## Configuring Your Blockchain Fork

### Step 1: Create Your Fork Directory

Create a new directory for your forked blockchain:

```bash
mkdir -p ~/blockchains/myblockchain
cd ~/blockchains/myblockchain
```

Copy the Dero HE source code to your fork location:

```bash
cp -r ~/go/src/derohe/* .
```

### Step 2: Rename the Blockchain

Throughout the codebase, you'll need to rename references from "DERO" to your blockchain name. Let's call it "MYCOIN" for this example.

#### Update Main Configuration Files

First, identify files that contain blockchain-specific parameters:

```bash
grep -r "DERO" . --include="*.go" | head -20
```

### Step 3: Modify Key Source Files

#### File 1: Main blockchain parameters (typically in `blockchain/` or root config)

Create or modify a configuration file with your blockchain parameters:

```bash
cat > blockchain_config.txt << 'EOF'
# MyBlockchain Configuration

# Basic Chain Parameters
CHAIN_NAME=MyBlockchain
CHAIN_SYMBOL=MBC
CHAIN_ATOMIC_UNITS=5
MAX_SUPPLY=21000000
GENESIS_TIMESTAMP=1700000000

# Network Parameters
MAINNET_P2P_PORT=11111
MAINNET_RPC_PORT=11112
MAINNET_WALLET_RPC_PORT=11113
TESTNET_P2P_PORT=41111
TESTNET_RPC_PORT=41112
TESTNET_WALLET_RPC_PORT=41113

# Mining Parameters
BLOCK_TIME_TARGET=16  # seconds
DIFFICULTY_BLOCKS_COUNT=720
DIFFICULTY_CUT=60
PRIORITY_CUT=60
INITIAL_DIFFICULTY=100000

# Supply Parameters
PREMINE_AMOUNT=1000000
BLOCK_REWARD=1.5
HALVING_INTERVAL=262800  # ~4 years at 16s blocks
EOF
```

#### File 2: Modify version and branding strings

Locate the main application files and update version strings:

```bash
# Find files with version info
find . -name "*.go" -type f -exec grep -l "version" {} \; | head -10
```

Create a script to help with replacements:

```bash
# Create a replacement script
cat > rename_blockchain.sh << 'EOF'
#!/bin/bash

# Replace DERO references with MYCOIN
find . -name "*.go" -type f -exec sed -i 's/DERO/MYCOIN/g' {} \;
find . -name "*.go" -type f -exec sed -i 's/dero/mycoin/g' {} \;

# Replace specific version identifiers
find . -name "*.go" -type f -exec sed -i 's/Homomorphic Encryption/MyBlockchain Private Network/g' {} \;

echo "Blockchain naming replacement completed!"
EOF

chmod +x rename_blockchain.sh
./rename_blockchain.sh
```

### Step 4: Configure Network Parameters

Locate the network configuration files (typically in `p2p/` or `cmd/derod/`):

```bash
find . -name "*.go" -type f -exec grep -l "10101" {} \;
```

Edit the files containing port definitions:

```bash
# Example modification for P2P port
# Change from: const DefaultMainnetPort = 10101
# To your custom port
sed -i 's/const DefaultMainnetPort = 10101/const DefaultMainnetPort = 11111/g' $(find . -name "*.go")
```

### Step 5: Update Genesis Block Configuration

The genesis block is the foundation of your blockchain. Modify it with:

```bash
# Find genesis block configuration
find . -name "*.go" -type f -exec grep -l "genesis" {} \;
```

In the genesis block configuration file, modify:

- **Genesis Timestamp**: Unix timestamp for your blockchain launch
- **Initial Difficulty**: Starting mining difficulty
- **Premine Accounts**: Addresses receiving premined coins
- **Supply Limits**: Total coin supply

Example genesis configuration in Go (pseudocode):

```go
type GenesisConfig struct {
    GenesisTimestamp uint64
    InitialDifficulty uint64
    InitialSupply uint64
    MaxSupply uint64
    PremineAmount uint64
    BlockReward float64
    BlockTimeTarget uint64
    HalvingInterval uint64
}

var MyBlockchainGenesis = GenesisConfig{
    GenesisTimestamp: 1700000000,
    InitialDifficulty: 100000,
    InitialSupply: 0,
    MaxSupply: 21000000,
    PremineAmount: 1000000,
    BlockReward: 1.5,
    BlockTimeTarget: 16,
    HalvingInterval: 262800,
}
```

---

## Setting Up a Premine

### Understanding Premine

A premine is a portion of cryptocurrency tokens created before official network launch. These tokens are allocated to addresses of your choice for:

- Development team funding
- Early investor allocation
- Community rewards
- Network bootstrap reserves

### Step 1: Generate Premine Addresses

First, create addresses for premine allocation. You'll do this after compiling your wallet:

```bash
# This will be done after building, but plan your premine structure now
PREMINE_TOTAL=1000000
TEAM_SHARE=500000        # 50%
INVESTORS_SHARE=300000   # 30%
COMMUNITY_SHARE=200000   # 20%
```

### Step 2: Modify Genesis Block for Premine

Locate the genesis block initialization code:

```bash
find . -name "*.go" -type f -exec grep -l "genesis.*block" {} \; | head -5
```

In the genesis block creation function, add premine allocation logic:

```go
// Example premine allocation structure
type PremineAllocation struct {
    Address string
    Amount uint64
    Purpose string
}

var PremineAllocations = []PremineAllocation{
    {
        Address: "YOUR_TEAM_ADDRESS_HERE",
        Amount: 500000 * 10^5,  // 500,000 coins (using atomic units)
        Purpose: "Development Team",
    },
    {
        Address: "YOUR_INVESTOR_ADDRESS_HERE",
        Amount: 300000 * 10^5,  // 300,000 coins
        Purpose: "Early Investors",
    },
    {
        Address: "YOUR_COMMUNITY_ADDRESS_HERE",
        Amount: 200000 * 10^5,  // 200,000 coins
        Purpose: "Community Rewards",
    },
}
```

### Step 3: Configure Premine Distribution

Create a configuration file for your premine:

```bash
cat > premine_config.json << 'EOF'
{
  "premine_total": 1000000,
  "atomic_units": 5,
  "allocations": [
    {
      "address": "YOUR_TEAM_ADDRESS",
      "amount": 500000,
      "percentage": 50,
      "description": "Development Team"
    },
    {
      "address": "YOUR_INVESTOR_ADDRESS",
      "amount": 300000,
      "percentage": 30,
      "description": "Early Investors"
    },
    {
      "address": "YOUR_COMMUNITY_ADDRESS",
      "amount": 200000,
      "percentage": 20,
      "description": "Community Fund"
    }
  ],
  "genesis_block": {
    "timestamp": 1700000000,
    "difficulty": 100000,
    "miner": "PREMINE_MINER_ADDRESS"
  }
}
EOF
```

---

## Building Your Forked Blockchain

### Step 1: Initialize Go Modules

Ensure your Go module dependencies are properly configured:

```bash
cd ~/blockchains/myblockchain
go mod tidy
go mod download
```

### Step 2: Build the Daemon

Build the daemon executable for your forked blockchain:

```bash
go build -o myblockchain-daemon ./cmd/derod/
```

Verify the build completed successfully:

```bash
ls -la myblockchain-daemon
./myblockchain-daemon --version
```

### Step 3: Build the CLI Wallet

Build the command-line wallet for managing coins:

```bash
go build -o myblockchain-wallet-cli ./cmd/dero-wallet-cli/
```

Verify:

```bash
ls -la myblockchain-wallet-cli
./myblockchain-wallet-cli --version
```

### Step 4: Build the Explorer (Optional)

Build a personal blockchain explorer:

```bash
go build -o myblockchain-explorer ./cmd/explorer/
```

### Step 5: Create Start Scripts

Create convenient startup scripts:

```bash
# Create daemon startup script
cat > start-daemon.sh << 'EOF'
#!/bin/bash
./myblockchain-daemon --data-dir ~/.myblockchain/data
EOF

# Create wallet startup script
cat > start-wallet.sh << 'EOF'
#!/bin/bash
./myblockchain-wallet-cli --daemon-address 127.0.0.1:11112
EOF

chmod +x start-daemon.sh start-wallet.sh
```

---

## Running Your Blockchain Network

### Step 1: Create Data Directories

Create the directories where blockchain data will be stored:

```bash
mkdir -p ~/.myblockchain/data
mkdir -p ~/.myblockchain/logs
```

### Step 2: Launch Your Daemon (Node)

Start your blockchain daemon as the first node:

```bash
./myblockchain-daemon --data-dir ~/.myblockchain/data --log-level 1
```

**Output to expect:**

```
MyBlockchain Daemon Starting...
P2P Server listening on: 127.0.0.1:11111
RPC Server listening on: 127.0.0.1:11112
Syncing genesis block...
Block height: 0
```

The daemon will:
- Create the genesis block
- Initialize the blockchain database
- Listen for peer connections
- Wait for wallet connections

**Running in Background:**

To run the daemon in the background, use:

```bash
nohup ./myblockchain-daemon --data-dir ~/.myblockchain/data > ~/.myblockchain/logs/daemon.log 2>&1 &
```

Or use a terminal multiplexer like `tmux`:

```bash
tmux new-session -d -s myblockchain "./myblockchain-daemon --data-dir ~/.myblockchain/data"
```

### Step 3: Connect a Wallet

In a new terminal window, start the CLI wallet:

```bash
cd ~/blockchains/myblockchain
./myblockchain-wallet-cli
```

The wallet will prompt you with a menu:

```
MyBlockchain CLI Wallet v1.0
================================
1. Create a new wallet
2. Open existing wallet
3. Exit
Select option:
```

### Step 4: Create Your First Wallet

Select option "1" to create a new wallet:

```
Enter wallet name: my-first-wallet
Enter password: ****
Confirm password: ****
```

The wallet will generate:
- **Private Keys**: Keep these completely secret
- **Public Address**: Your wallet address for receiving coins
- **Seed Phrase**: 25-word recovery phrase - back this up securely

**Important**: Save your seed phrase in multiple secure locations!

---

## Creating and Claiming Premine

### Step 1: Generate Premine Addresses

Before launching your network publicly, generate addresses that will receive premine:

```bash
# Using the wallet, create multiple addresses for different purposes
./myblockchain-wallet-cli
```

In the wallet CLI:

```
1. Create new address
Address label: Team Premine
(Save the generated address)

2. Create new address
Address label: Investor Premine
(Save the generated address)

3. Create new address
Address label: Community Premine
(Save the generated address)
```

### Step 2: Modify Premine in Source Code

Update your premine configuration with the generated addresses:

Edit the file containing genesis block initialization and update:

```go
// Update with your actual generated addresses
var PremineAllocations = []PremineAllocation{
    {
        Address: "MYBLOCKCHAINxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",  // Your team address
        Amount: 500000 * 100000,  // 500,000 coins in atomic units
        Purpose: "Development Team",
    },
    // Add other allocations...
}
```

### Step 3: Rebuild with Premine Configuration

After updating the premine addresses, rebuild the daemon:

```bash
cd ~/blockchains/myblockchain
go build -o myblockchain-daemon ./cmd/derod/
```

### Step 4: Launch Fresh Network with Premine

Remove the old blockchain data to start fresh with your premine:

```bash
# WARNING: This deletes all blockchain data!
rm -rf ~/.myblockchain/data/*
```

Launch the daemon again:

```bash
./myblockchain-daemon --data-dir ~/.myblockchain/data
```

### Step 5: Claim Premine in Wallet

Open your wallet:

```bash
./myblockchain-wallet-cli
```

Connect to your daemon:

```
Daemon address: 127.0.0.1:11112
Connecting...
```

View your balance:

```
1. Check balance
```

You should see your premine allocation appearing in the wallet as blocks are generated.

### Step 6: Verify Premine Blocks

Check that the premine was properly created:

```bash
# In daemon CLI (if available), check genesis block
print_block 0
print_bc
```

### Step 7: Transfer Premine Funds

Once you've verified the premine exists, transfer funds as needed:

In the wallet CLI:

```
1. Send funds
To address: [destination address]
Amount: [amount in MBC]
Description: [optional]
```

The transaction will appear on the network immediately.

---

## Troubleshooting and Next Steps

### Common Issues and Solutions

#### Issue 1: "Cannot find Go" or "go: command not found"

**Solution:**

```bash
# Verify Go is installed
which go
go version

# If not found, reinstall Go and add to PATH
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
```

#### Issue 2: Build Fails with Module Errors

**Solution:**

```bash
# Clean and download dependencies
cd ~/blockchains/myblockchain
go clean -modcache
go mod download
go mod tidy
go build -o myblockchain-daemon ./cmd/derod/
```

#### Issue 3: Daemon Won't Start - Port Already in Use

**Solution:**

```bash
# Check what's using the port
lsof -i :11111  # P2P port
lsof -i :11112  # RPC port

# Kill the process (if needed)
kill -9 <PID>

# Or change the port in your code and rebuild
```

#### Issue 4: Wallet Can't Connect to Daemon

**Solution:**

```bash
# Verify daemon is running
ps aux | grep myblockchain-daemon

# Check if ports are listening
netstat -tlnp | grep 11112

# Verify daemon RPC port is correct in wallet config
# Usually: 127.0.0.1:11112
```

#### Issue 5: Premine Not Appearing in Wallet

**Solution:**

1. Verify addresses in code match wallet addresses
2. Rebuild daemon with correct addresses
3. Delete blockchain data and restart fresh
4. Check block generation - mine a few blocks if needed

```bash
# In wallet, check mining (if enabled)
1. Start mining
```

### Next Steps: Launching Your Network

Once you've successfully created and tested your forked blockchain locally:

#### Step 1: Network Bootstrap

- Distribute your daemon binary to other network participants
- Provide peers with your daemon startup instructions
- Document your network parameters

#### Step 2: Add Bootstrap Nodes

Modify your daemon to include bootstrap peers:

```go
// In peer-to-peer configuration
BootstrapPeers = []string{
    "your-node-1.com:11111",
    "your-node-2.com:11111",
    "your-node-3.com:11111",
}
```

#### Step 3: Create Public RPC Endpoints

Allow external connections to your RPC:

```bash
# Modify daemon to listen on all interfaces
./myblockchain-daemon --listen 0.0.0.0:11112
```

#### Step 4: Deploy Block Explorer

Build and deploy the explorer:

```bash
./myblockchain-explorer --listen 0.0.0.0:8080
```

Users can then access it at `http://your-server:8080`

#### Step 5: Mining Pool Setup (Optional)

Configure mining pools for distributed mining:

- Document AstroBWT mining algorithm
- Provide pool software to miners
- Set pool fee structure

#### Step 6: Official Website and Documentation

Create:

- Official website with blockchain info
- Block explorer link
- Wallet download links
- Mining guide
- Community Discord or forums

#### Step 7: Exchange Listings

Once your network is stable:

- Contact exchanges for listing
- Provide technical documentation
- Set up community support channels

### Security Considerations

1. **Private Key Security**: Never share private keys
2. **Seed Phrase Backup**: Store securely offline
3. **Network Security**: Use firewalls and SSH keys only for node access
4. **Regular Backups**: Backup your wallet files and blockchain data
5. **Code Audits**: Have your modified code reviewed before mainnet launch
6. **Testnet First**: Always run on testnet before mainnet

### Performance Optimization

For production use:

```bash
# Use dedicated hardware
# Minimum: 4GB RAM, 20GB SSD, 2-4 CPU cores
# Recommended: 8GB RAM, 100GB SSD, 4+ CPU cores

# Run daemon with optimized settings
./myblockchain-daemon \
  --data-dir ~/.myblockchain/data \
  --log-level 0 \
  --max-peers 128 \
  --sync-mode full
```

### Monitoring Your Network

Create monitoring scripts:

```bash
cat > monitor-blockchain.sh << 'EOF'
#!/bin/bash
while true; do
    echo "=== Blockchain Status ==="
    echo "Block Height: $(curl -s http://127.0.0.1:11112 | grep height)"
    echo "Connected Peers: $(ps aux | grep myblockchain-daemon)"
    echo "Memory Usage: $(ps aux | grep myblockchain-daemon | awk '{print $6}')"
    sleep 30
done
EOF

chmod +x monitor-blockchain.sh
```

---

## Conclusion

You have successfully forked the Dero HE blockchain and created your own custom blockchain with premine allocation! 

The steps covered:

1. ✅ Set up Linux Mint environment
2. ✅ Installed Go and prerequisites
3. ✅ Cloned and configured Dero HE source code
4. ✅ Modified network and blockchain parameters
5. ✅ Configured premine allocation
6. ✅ Built custom daemon and wallet
7. ✅ Launched and tested your blockchain
8. ✅ Created and claimed premine funds

Your blockchain is now operational on your local network. Continue with the next steps to prepare for wider deployment if desired.

For more information, visit:

- **Dero Official**: https://dero.io
- **Dero GitHub**: https://github.com/deroproject/derohe
- **Go Documentation**: https://golang.org/doc

Good luck with your blockchain project!
