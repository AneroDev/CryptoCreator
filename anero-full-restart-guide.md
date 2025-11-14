# Complete Guide: Fork Ethereum to Anero Mainnet (Geth v1.16.6, Lighthouse v8.0.0, Go eth2-testnet-genesis)

This guide walks through creating a production Anero mainnet blockchain using Geth v1.16.6, Lighthouse v8.0.0, Go-based eth2-testnet-genesis, validators, and public RPC exposure via Nginx.

---

## Phase 1: Prerequisites and Setup

### 1.1 System Requirements

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git golang-go npm nodejs curl wget jq
```

### 1.2 Clone and Build Geth v1.16.6 (Execution Layer)

```bash
cd ~
git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum
git checkout v1.16.6
make geth
sudo cp build/bin/geth /usr/local/bin/geth
geth version
```

### 1.3 Install Lighthouse v8.0.0 (Consensus Layer)

```bash
cd ~
wget https://github.com/sigp/lighthouse/releases/download/v8.0.0/lighthouse-v8.0.0-x86_64-unknown-linux-gnu.tar.gz
tar -xzf lighthouse-v8.0.0-x86_64-unknown-linux-gnu.tar.gz
sudo mv lighthouse /usr/local/bin/
lighthouse --version
```

### 1.4 Build eth2-testnet-genesis (Go Version)

eth2-testnet-genesis is now written in Go. Clone and build it:

```bash
cd ~
git clone https://github.com/protolambda/eth2-testnet-genesis.git
cd eth2-testnet-genesis
go build -o eth2-testnet-genesis .
sudo cp eth2-testnet-genesis /usr/local/bin/
eth2-testnet-genesis --help
```

### 1.5 Setup Project Directory

```bash
mkdir -p ~/anero-mainnet
cd ~/anero-mainnet
mkdir -p {node1,node2,lighthouse_config,config}
```

---

## Phase 2: Create Node Wallets

### 2.1 Generate Node 1 Wallet

```bash
cd ~/anero-mainnet
geth account new --datadir node1
```

(You will be prompted for a password - remember it)

### 2.2 Generate Node 2 Wallet

```bash
geth account new --datadir node2
```

(You will be prompted for a password - remember it)

### 2.3 List and Record Addresses

```bash
geth account list --datadir node1
geth account list --datadir node2
```

**Record these addresses for the genesis.json creation**

Example output:
```
Account #0: 0x3154eDC26F23c85779ea484721e12cAF6f7f89A6 keystore:///home/admin/anero-mainnet/node1/keystore/...
Account #0: 0xa967B04699F2D3F0718E53B92fC14AC96E1cA238 keystore:///home/admin/anero-mainnet/node2/keystore/...
```

---

## Phase 3: Create Execution Layer Genesis (Geth v1.16.6)

### 3.1 Generate JWT Secret

```bash
cd ~/anero-mainnet
openssl rand -hex 32 > jwt.hex
cat jwt.hex
```

### 3.2 Create genesis.json with 50M Premine to Each Node

Replace `0x3154eDC26F23c85779ea484721e12cAF6f7f89A6` and `0xa967B04699F2D3F0718E53B92fC14AC96E1cA238` with your actual node addresses.

```bash
cat > ~/anero-mainnet/config/genesis.json << 'EOF'
{
  "config": {
    "chainId": 1889,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "berlinBlock": 0,
    "londonBlock": 0,
    "terminalTotalDifficulty": 0,
    "terminalTotalDifficultyPassed": true,
    "shanghaiTime": 0,
    "cancunTime": 0,
    "pragueTime": 0,
    "blobSchedule": {
      "cancun": {
        "minBlobGasPrice": 1,
        "targetBlobGasPerBlock": 393216,
        "maxBlobGasPerBlock": 786432,
        "updateFraction": 2
      }
    },
    "engine": {
      "clique": {
        "period": 0,
        "epoch": 30000
      }
    }
  },
  "difficulty": "0x1",
  "gasLimit": "0x47b760",
  "alloc": {
    "0x3154eDC26F23c85779ea484721e12cAF6f7f89A6": {
      "balance": "0x2b5e3af16b1880000000000"
    },
    "0xa967B04699F2D3F0718E53B92fC14AC96E1cA238": {
      "balance": "0x2b5e3af16b1880000000000"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "1763133104"
}
EOF
```

**Balance:** `0x2b5e3af16b1880000000000` = 50,000,000 ANERO per wallet

### 3.3 Initialize Both Geth Nodes

```bash
cd ~/anero-mainnet
geth --datadir node1 init config/genesis.json
geth --datadir node2 init config/genesis.json
```

Expected: "Successfully wrote genesis state"

---

## Phase 4: Create Consensus Layer Genesis (Lighthouse v8.0.0)

### 4.1 Create Lighthouse Config Directory

```bash
cd ~/anero-mainnet/lighthouse_config
```

### 4.2 Create config.yaml

```bash
cat > ~/anero-mainnet/lighthouse_config/config.yaml << 'EOF'
PRESET_BASE: "mainnet"
CONFIG_NAME: "anero_mainnet"

GENESIS_FORK_VERSION: 0x00000000
ALTAIR_FORK_VERSION: 0x01000000
BELLATRIX_FORK_VERSION: 0x02000000
CAPELLA_FORK_VERSION: 0x03000000
DENEB_FORK_VERSION: 0x04000000
ELECTRA_FORK_VERSION: 0x05000000

GENESIS_EPOCH: 0
ALTAIR_FORK_EPOCH: 0
BELLATRIX_FORK_EPOCH: 0
CAPELLA_FORK_EPOCH: 0
DENEB_FORK_EPOCH: 0
ELECTRA_FORK_EPOCH: 0

MIN_GENESIS_ACTIVE_VALIDATOR_COUNT: 2
MIN_GENESIS_TIME: 0
GENESIS_DELAY: 0

SECONDS_PER_SLOT: 12
SLOTS_PER_EPOCH: 32
SECONDS_PER_ETH1_BLOCK: 14

MIN_VALIDATOR_WITHDRAWABILITY_DELAY: 256
SHARD_COMMITTEE_PERIOD: 256
ETH1_FOLLOW_DISTANCE: 2048

MIN_DEPOSIT_AMOUNT: 1000000000
MAX_EFFECTIVE_BALANCE: 32000000000
EFFECTIVE_BALANCE_INCREMENT: 1000000000
EJECTION_BALANCE: 16000000000

MIN_PER_EPOCH_CHURN_LIMIT: 4
CHURN_LIMIT_QUOTIENT: 65536

DEPOSIT_CONTRACT_ADDRESS: 0x0000000000000000000000000000000000000000
DEPOSIT_CHAIN_ID: 1889
DEPOSIT_NETWORK_ID: 1889
DEPOSIT_CONTRACT_BLOCK: 0

BLS_WITHDRAWAL_PREFIX: 0x00

INACTIVITY_SCORE_BIAS: 4
INACTIVITY_SCORE_RECOVERY_RATE: 16
PROPOSER_SCORE_BOOST: 40

MAX_COMMITTEES_PER_SLOT: 64
TARGET_COMMITTEE_SIZE: 128
MAX_VALIDATORS_PER_COMMITTEE: 2048
SHUFFLE_ROUND_COUNT: 90

HYSTERESIS_QUOTIENT: 4
HYSTERESIS_DOWNWARD_MULTIPLIER: 1
HYSTERESIS_UPWARD_MULTIPLIER: 1

MIN_SYNC_COMMITTEE_SIZE: 512
MAX_SYNC_COMMITTEE_SIZE: 2048
SYNC_COMMITTEE_SUBNETS: 4
EPOCHS_PER_SYNC_COMMITTEE_PERIOD: 256

MIN_SLASHING_PENALTY_QUOTIENT: 128
WHISTLEBLOWER_REWARD_QUOTIENT: 512
PROPOSER_REWARD_QUOTIENT: 8
INACTIVITY_PENALTY_QUOTIENT: 33554432

MIN_ATTESTATION_INCLUSION_DELAY: 1

SLOTS_PER_HISTORICAL_ROOT: 8192
MIN_SEED_LOOKAHEAD: 1
MAX_SEED_LOOKAHEAD: 4
MIN_EPOCHS_TO_INACTIVITY_PENALTY: 4
EPOCHS_PER_HISTORICAL_VECTOR: 65536
EPOCHS_PER_SLASHINGS_VECTOR: 8192
HISTORICAL_ROOTS_LIMIT: 16777216
VALIDATOR_REGISTRY_LIMIT: 1099511627776

MAX_REQUEST_BODY_SIZE: 1052672
GOSSIP_MAX_SIZE: 1052672
MAX_CHUNK_SIZE: 1052672
MAX_OBJECT_SIZE: 4294967296

DOMAIN_BEACON_PROPOSER: 0x00000000
DOMAIN_BEACON_ATTESTER: 0x01000000
DOMAIN_RANDAO: 0x02000000
DOMAIN_DEPOSIT: 0x03000000
DOMAIN_VOLUNTARY_EXIT: 0x04000000
DOMAIN_SELECTION_PROOF: 0x05000000
DOMAIN_AGGREGATE_AND_PROOF: 0x06000000
DOMAIN_SYNC_COMMITTEE: 0x07000000
DOMAIN_SYNC_COMMITTEE_SELECTION_PROOF: 0x08000000
DOMAIN_CONTRIBUTION_AND_PROOF: 0x09000000
EOF
```

**Note:** Lighthouse v8.0.0 added `ELECTRA_FORK_VERSION` and `ELECTRA_FORK_EPOCH` support.

### 4.3 Create mnemonics.yaml

```bash
cat > ~/anero-mainnet/lighthouse_config/mnemonics.yaml << 'EOF'
- mnemonic: "casual pill wear project analyst accuse aspect oak trumpet analyst grid taste door observe produce input globe uncle avoid urge wisdom three carbon holiday"
  count: 2
EOF
```

### 4.4 Create deploy_block.txt

```bash
echo "0" > ~/anero-mainnet/lighthouse_config/deploy_block.txt
```

### 4.5 Generate genesis.ssz with Go eth2-testnet-genesis

The Go version of eth2-testnet-genesis uses command-line flags instead of shell commands.

For **mainnet** genesis generation:

```bash
cd ~/eth2-testnet-genesis

# Build binary if not already done
go build -o eth2-testnet-genesis .

# Generate mainnet genesis (not testnet)
./eth2-testnet-genesis \
  --config ~/anero-mainnet/lighthouse_config/config.yaml \
  --eth1-config ~/anero-mainnet/config/genesis.json \
  --state-file ~/anero-mainnet/lighthouse_config/genesis.ssz \
  --mnemonics ~/anero-mainnet/lighthouse_config/mnemonics.yaml
```

**Parameters explained:**
- `--config`: Path to Lighthouse config.yaml
- `--eth1-config`: Path to Geth genesis.json
- `--state-file`: Output path for genesis.ssz
- `--mnemonics`: Path to mnemonics.yaml

### 4.6 Verify genesis.ssz

```bash
ls -lh ~/anero-mainnet/lighthouse_config/genesis.ssz
```

Expected: 2-3 MB file size

---

## Phase 5: Start Blockchain Services (5 Terminals)

### Terminal 1: Bootnode

```bash
cd ~/anero-mainnet
bootnode -genkey boot.key
bootnode -nodekey boot.key -addr :30301 -verbosity 9
```

Note the enode:// address displayed.

### Terminal 2: Geth Node 1 (Geth v1.16.6)

```bash
cd ~/anero-mainnet
geth --datadir node1 \
  --port 30306 \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8545 \
  --http.corsdomain "*" \
  --http.api eth,net,web3,admin,debug \
  --networkid 1889 \
  --authrpc.addr 0.0.0.0 \
  --authrpc.port 8551 \
  --authrpc.vhosts "*" \
  --authrpc.jwtsecret jwt.hex \
  --state.scheme path \
  --db.engine pebble \
  console
```

### Terminal 3: Geth Node 2 (Geth v1.16.6)

```bash
cd ~/anero-mainnet
geth --datadir node2 \
  --port 30307 \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8546 \
  --http.corsdomain "*" \
  --http.api eth,net,web3,admin,debug \
  --networkid 1889 \
  --authrpc.addr 0.0.0.0 \
  --authrpc.port 8552 \
  --authrpc.vhosts "*" \
  --authrpc.jwtsecret jwt.hex \
  --state.scheme path \
  --db.engine pebble \
  console
```

### Terminal 4: Lighthouse Beacon Node (v8.0.0)

```bash
cd ~/anero-mainnet
lighthouse bn \
  --testnet-dir lighthouse_config \
  --datadir beacon_data \
  --http \
  --http-address 0.0.0.0 \
  --http-port 5052 \
  --execution-endpoint http://127.0.0.1:8551 \
  --execution-jwt-secret-key $(cat jwt.hex) \
  --enr-address 3.128.66.2 \
  --enr-udp-port 9000 \
  --enr-tcp-port 9000 \
  --allow-insecure-genesis-sync \
  --disable-deposit-contract-sync
```

**Lighthouse v8.0.0 changes:**
- `beacon_node` → `bn` (short alias)
- Enhanced stability and performance improvements

### Terminal 5: Lighthouse Validator Client (v8.0.0)

```bash
cd ~/anero-mainnet
lighthouse vc \
  --testnet-dir lighthouse_config \
  --datadir beacon_data \
  --beacon-nodes http://127.0.0.1:5052
```

**Lighthouse v8.0.0 changes:**
- `validator_client` → `vc` (short alias)

---

## Phase 6: Verify Blockchain is Running

### Check Block Height

```bash
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq
```

### Check Account Balances

```bash
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x3154eDC26F23c85779ea484721e12cAF6f7f89A6","latest"],"id":1}' | jq
```

### Check Beacon Chain

```bash
curl http://127.0.0.1:5052/eth/v1/node/health
curl http://127.0.0.1:5052/eth/v1/beacon/blocks/head | jq '.data.message.slot'
```

### Check Validators

```bash
curl http://127.0.0.1:5052/eth/v1/beacon/states/head/validators | jq '.data | length'
```

---

## Phase 7: Setup Public RPC with Nginx

### 7.1 Install Nginx

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 7.2 Create DNS Record

Point `node.getanero.org` to `3.128.66.2`

Add in your DNS provider:
```
node.getanero.org  A  3.128.66.2
```

Wait 5-10 minutes for propagation.

### 7.3 Configure Nginx Reverse Proxy

```bash
sudo tee /etc/nginx/sites-available/anero-rpc > /dev/null << 'EOF'
upstream geth_rpc {
    server 127.0.0.1:8545;
}

server {
    listen 80;
    server_name node.getanero.org;

    location / {
        proxy_pass http://geth_rpc;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }
}
EOF
```

### 7.4 Enable Configuration

```bash
sudo ln -s /etc/nginx/sites-available/anero-rpc /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 7.5 Test RPC

```bash
curl -X POST http://node.getanero.org \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq
```

---

## Phase 8: Enable HTTPS with Let's Encrypt

### 8.1 Install Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

### 8.2 Get SSL Certificate

```bash
sudo certbot certonly --nginx -d node.getanero.org
```

### 8.3 Update Nginx Configuration

```bash
sudo tee /etc/nginx/sites-available/anero-rpc > /dev/null << 'EOF'
upstream geth_rpc {
    server 127.0.0.1:8545;
}

server {
    listen 80;
    server_name node.getanero.org;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name node.getanero.org;

    ssl_certificate /etc/letsencrypt/live/node.getanero.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/node.getanero.org/privkey.pem;

    location / {
        proxy_pass http://geth_rpc;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }
}
EOF
```

### 8.4 Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## Phase 9: Connect MetaMask to Anero Mainnet

In MetaMask, add network:
- **Network Name:** Anero Mainnet
- **RPC URL:** `https://node.getanero.org`
- **Chain ID:** 1889
- **Currency Symbol:** ANERO

Switch to Anero Mainnet - your account should show 50M ANERO balance.

---

## Phase 10: Monitoring

### Monitor Nginx

```bash
sudo systemctl status nginx
sudo tail -f /var/log/nginx/error.log
```

### Monitor Block Height

```bash
watch -n 5 'curl -s -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  -d "{\"jsonrpc\":\"2.0\",\"method\":\"eth_blockNumber\",\"params\":[],\"id\":1}" | jq -r ".result"'
```

### Monitor Beacon Slot

```bash
watch -n 5 'curl -s http://127.0.0.1:5052/eth/v1/beacon/blocks/head | jq ".data.message.slot"'
```

### Auto-renew SSL

```bash
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

---

## Key Version Updates

| Component | Previous | Current | Changes |
|-----------|----------|---------|---------|
| **Geth** | v1.14.0 | v1.16.6 | Added `--state.scheme`, `--db.engine`, improved blob support |
| **Lighthouse** | v5.1.0 | v8.0.0 | Added Electra support, command aliases (`bn`, `vc`), performance improvements |
| **eth2-testnet-genesis** | Python | Go | Language rewrite, CLI flags changed, binary compilation required |

---

## eth2-testnet-genesis Mainnet Generation Note

The Go version of eth2-testnet-genesis generates **mainnet** genesis states by default. The configuration is controlled via your `config.yaml` file (not by command-line flags). 

To ensure mainnet generation:
- Set `CONFIG_NAME: "anero_mainnet"` in config.yaml (not "anero_testnet")
- Use `--config`, `--eth1-config`, and `--mnemonics` flags as shown above
- The tool reads fork configuration from your yaml and generates accordingly

---

## Troubleshooting

### eth2-testnet-genesis compilation fails

```bash
cd ~/eth2-testnet-genesis
go mod tidy
go build -o eth2-testnet-genesis .
```

### Lighthouse v8.0.0 command not found

```bash
which lighthouse
lighthouse --version
sudo ln -s /usr/local/bin/lighthouse /usr/bin/lighthouse
```

### Geth blob transactions not working

```bash
geth --help | grep -i "blob\|state.scheme"
```

Ensure v1.16.6 is installed with `geth version`

### Validators not attesting

```bash
curl http://127.0.0.1:5052/eth/v1/beacon/states/head/validators | jq '.data[] | {index, status}'
```

Should show `"active_ongoing"` status.

---

## Summary

- ✓ Geth v1.16.6 installed and running
- ✓ Lighthouse v8.0.0 installed and running
- ✓ Go eth2-testnet-genesis compiled and used for mainnet genesis
- ✓ Node 1 & Node 2 with 50M ANERO each
- ✓ Blob transactions enabled
- ✓ Validators attesting
- ✓ Public RPC at node.getanero.org
- ✓ HTTPS enabled with Let's Encrypt
- ✓ MetaMask compatible

**Anero mainnet is production-ready!**
