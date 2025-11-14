# Complete Guide: Fork Ethereum to Anero Mainnet

This guide walks through creating a production Anero mainnet blockchain from scratch, including execution layer (Geth), consensus layer (Lighthouse), validators, and public RPC exposure via Nginx.

---

## Phase 1: Prerequisites and Setup

### 1.1 System Requirements

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y build-essential git golang-go npm nodejs curl wget jq
```

### 1.2 Clone and Build Geth (Execution Layer)

```bash
cd ~
git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum
git checkout v1.14.0
make geth
sudo cp build/bin/geth /usr/local/bin/geth
geth version
```

### 1.3 Install Lighthouse (Consensus Layer)

```bash
# Download pre-built Lighthouse binary or build from source
cd ~
wget https://github.com/sigp/lighthouse/releases/download/v5.1.0/lighthouse-v5.1.0-x86_64-unknown-linux-gnu.tar.gz
tar -xzf lighthouse-v5.1.0-x86_64-unknown-linux-gnu.tar.gz
sudo mv lighthouse /usr/local/bin/
lighthouse --version
```

### 1.4 Setup eth2-testnet-genesis (for generating genesis.ssz)

```bash
cd ~
git clone https://github.com/protolambda/eth2-testnet-genesis.git
cd eth2-testnet-genesis
pip install -r requirements.txt
chmod +x ./eth2-testnet-genesis
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

## Phase 3: Create Execution Layer Genesis (Geth)

### 3.1 Generate JWT Secret

```bash
cd ~/anero-mainnet
openssl rand -hex 32 > jwt.hex
cat jwt.hex
```

### 3.2 Create genesis.json with 50M Premine to Each Node

Replace `0x3154eDC26F23c85779ea484721e12cAF6f7f89A6` and `0xa967B04699F2D3F0718E53B92fC14AC96E1cA238` with your actual node addresses from Step 2.3

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

**Balance Breakdown:** `0x2b5e3af16b1880000000000` = 50,000,000 ANERO (50M * 10^18 wei)

### 3.3 Initialize Both Geth Nodes

```bash
cd ~/anero-mainnet
geth --datadir node1 init config/genesis.json
geth --datadir node2 init config/genesis.json
```

**Expected output:** "Successfully wrote genesis state"

---

## Phase 4: Create Consensus Layer Genesis (Lighthouse)

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

GENESIS_EPOCH: 0
ALTAIR_FORK_EPOCH: 0
BELLATRIX_FORK_EPOCH: 0
CAPELLA_FORK_EPOCH: 0
DENEB_FORK_EPOCH: 0

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

### 4.5 Generate genesis.ssz

```bash
cd ~/eth2-testnet-genesis
./eth2-testnet-genesis capella \
  --config ~/anero-mainnet/lighthouse_config/config.yaml \
  --eth1-config ~/anero-mainnet/config/genesis.json \
  --mnemonics ~/anero-mainnet/lighthouse_config/mnemonics.yaml

cp genesis.ssz ~/anero-mainnet/lighthouse_config/genesis.ssz
ls -lh ~/anero-mainnet/lighthouse_config/genesis.ssz
```

**Expected:** 2-3 MB file size

---

## Phase 5: Start Blockchain Services (5 Terminals)

### Terminal 1: Bootnode

```bash
cd ~/anero-mainnet
bootnode -genkey boot.key
bootnode -nodekey boot.key -addr :30301 -verbosity 9
```

**Note the enode:// address displayed**

### Terminal 2: Geth Node 1 (Execution Layer)

```bash
cd ~/anero-mainnet
geth --datadir node1 \
  --port 30306 \
  --http --http.addr 0.0.0.0 --http.port 8545 \
  --http.corsdomain "*" --http.api eth,net,web3,admin,debug \
  --networkid 1889 \
  --authrpc.addr 0.0.0.0 --authrpc.port 8551 \
  --authrpc.vhosts "*" \
  --authrpc.jwtsecret jwt.hex \
  --gcmode archive \
  console
```

### Terminal 3: Geth Node 2 (Execution Layer)

```bash
cd ~/anero-mainnet
geth --datadir node2 \
  --port 30307 \
  --http --http.addr 0.0.0.0 --http.port 8546 \
  --http.corsdomain "*" --http.api eth,net,web3,admin,debug \
  --networkid 1889 \
  --authrpc.addr 0.0.0.0 --authrpc.port 8552 \
  --authrpc.vhosts "*" \
  --authrpc.jwtsecret jwt.hex \
  --gcmode archive \
  console
```

### Terminal 4: Lighthouse Beacon Node (Consensus Layer)

```bash
cd ~/anero-mainnet
lighthouse beacon_node \
  --testnet-dir lighthouse_config \
  --datadir beacon_data \
  --http --http-address 0.0.0.0 --http-port 5052 \
  --execution-endpoint http://127.0.0.1:8551 \
  --execution-jwt-secret-key $(cat jwt.hex) \
  --enr-address 3.128.66.2 \
  --enr-udp-port 9000 \
  --enr-tcp-port 9000 \
  --allow-insecure-genesis-sync \
  --disable-deposit-contract-sync
```

### Terminal 5: Lighthouse Validator Client

```bash
cd ~/anero-mainnet
lighthouse validator_client \
  --testnet-dir lighthouse_config \
  --datadir beacon_data \
  --beacon-nodes http://127.0.0.1:5052
```

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

### Check Beacon Chain Status

```bash
curl http://127.0.0.1:5052/eth/v1/node/health
curl http://127.0.0.1:5052/eth/v1/beacon/head | jq '.data.header.message.slot'
```

### Check Active Validators

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

Point `node.getanero.org` to your public IP: `3.128.66.2`

Add A record in your DNS provider:
```
node.getanero.org  A  3.128.66.2
```

**Wait 5-10 minutes for DNS propagation**

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

### 7.4 Enable Nginx Configuration

```bash
sudo ln -s /etc/nginx/sites-available/anero-rpc /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 7.5 Test RPC Endpoint

```bash
curl -X POST http://node.getanero.org \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq
```

---

## Phase 8: Enable HTTPS (SSL/TLS) with Let's Encrypt

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

### 9.1 In MetaMask:

1. Click **Networks** → **Add Network**
2. Fill in:
   - **Network Name:** Anero Mainnet
   - **RPC URL:** `https://node.getanero.org`
   - **Chain ID:** 1889
   - **Currency Symbol:** ANERO
   - **Block Explorer URL:** (optional, leave blank if not available)

3. Click **Save**

### 9.2 Test Connection

- Switch to Anero Mainnet in MetaMask
- Your premine account should appear with 50M ANERO balance

---

## Phase 10: Monitoring and Maintenance

### 10.1 Monitor Nginx

```bash
sudo systemctl status nginx
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

### 10.2 Check Beacon Chain Health

```bash
watch -n 5 'curl -s http://127.0.0.1:5052/eth/v1/beacon/head | jq ".data.header.message.slot"'
```

### 10.3 Monitor Block Production

```bash
watch -n 5 'curl -s -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  -d "{\"jsonrpc\":\"2.0\",\"method\":\"eth_blockNumber\",\"params\":[],\"id\":1}" | jq -r ".result"'
```

### 10.4 Auto-renew SSL Certificate

```bash
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

---

## Troubleshooting

### Issue: "connection refused" from MetaMask

```bash
curl http://localhost:8545  # Should work locally
curl https://node.getanero.org  # Should work externally
```

Check Nginx is running:
```bash
sudo systemctl status nginx
sudo tail -f /var/log/nginx/error.log
```

### Issue: DNS not resolving

```bash
nslookup node.getanero.org
dig node.getanero.org
```

Wait 10+ minutes after DNS record creation.

### Issue: SSL certificate error

```bash
sudo certbot renew --dry-run
sudo certbot renew --force-renewal
```

### Issue: Validators not attesting

```bash
curl http://127.0.0.1:5052/eth/v1/beacon/states/head/validators | jq '.data[] | {index, status}'
```

Should show `"active_ongoing"` status.

---

## Summary Checklist

- [ ] Geth v1.14.0+ installed
- [ ] Lighthouse installed and running
- [ ] Node 1 and Node 2 wallets created with 50M ANERO each
- [ ] Bootnode started
- [ ] Geth nodes initialized and running
- [ ] Lighthouse beacon synced
- [ ] Lighthouse validators attesting
- [ ] Block height increasing
- [ ] DNS record created for node.getanero.org
- [ ] Nginx configured as reverse proxy
- [ ] SSL certificate obtained from Let's Encrypt
- [ ] MetaMask can connect to https://node.getanero.org
- [ ] Account shows 50M ANERO balance

Your Anero mainnet is now live and publicly accessible!
