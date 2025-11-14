# Complete Anero Blockchain Fresh Start Guide

This guide walks through restarting your entire Anero PoS blockchain from scratch, reusing your premine wallets while reinitializing the chain state, Lighthouse configs, validators, and nodes.

---

## Phase 1: Preparation and Cleanup

### 1.1 Stop All Running Processes

```bash
# Kill all running services
pkill -9 -f geth
pkill -9 -f lighthouse
pkill -9 -f bootnode
pkill -9 -f "python3 -m http.server"

sleep 2
```

### 1.2 Clean Blockchain State (Keep Keystore)

```bash
cd ~/anero-blockchain

# DELETE these directories to reset chain state
rm -rf node1/geth
rm -rf node2/geth
rm -rf beacon_data
rm -rf my_custom_chain
rm -rf anero_config
rm -rf anero_mainnet
rm -rf lighthouse_config

# KEEP these (your premine accounts)
# node1/keystore/ and node2/keystore/ should still exist

# Verify keystores are still there
ls -la node1/keystore/
ls -la node2/keystore/

echo "Cleanup complete - keystores preserved"
```

### 1.3 Verify Premine Wallets Still Exist

```bash
cd ~/anero-blockchain

# Extract addresses to verify they're still accessible
python3 << 'PYEOF'
import json
import os

node1_keystore = 'node1/keystore'
if os.path.exists(node1_keystore):
    keyfiles = [f for f in os.listdir(node1_keystore) if f.startswith('UTC')]
    print(f"✓ Node1 keystores found: {len(keyfiles)} files")
    for kf in keyfiles[:2]:
        with open(os.path.join(node1_keystore, kf), 'r') as f:
            data = json.load(f)
            address = data.get('address', 'unknown')
            print(f"  - Address: 0x{address}")
else:
    print("✗ Node1 keystore not found!")

node2_keystore = 'node2/keystore'
if os.path.exists(node2_keystore):
    keyfiles = [f for f in os.listdir(node2_keystore) if f.startswith('UTC')]
    print(f"✓ Node2 keystores found: {len(keyfiles)} files")
else:
    print("✗ Node2 keystore not found!")
PYEOF
```

---

## Phase 2: Initialize Execution Layer (Geth)

### 2.1 Generate Fresh Geth Genesis

```bash
cd ~/anero-blockchain

# Remove old genesis.json
rm -f config/genesis.json

# Create new genesis.json with premine
python3 << 'PYEOF'
import json
import time

# Extract premine address from your keystore
premine_address = "0x<YOUR_PREMINE_ADDRESS>"  # Replace with actual address

# Example: if your keystore shows 0x1234...5678, use that
# You can find it in node1/keystore/<UTC-file> under the "address" field

genesis = {
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
        "terminalTotalDifficultyPassed": True,
        "shanghaiTime": 0,
        "cancunTime": 0,
        "pragueTime": 0,
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
        premine_address: {
            "balance": "0x52b7d2dcc80cd2e4000000"  # 100,000 ETH in wei
        }
    },
    "number": "0x0",
    "gasUsed": "0x0",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp": str(int(time.time()))
}

with open('config/genesis.json', 'w') as f:
    json.dump(genesis, f, indent=2)

print("✓ Genesis created with premine allocation")
print(f"  Premine address: {premine_address}")
print(f"  Amount: 100,000 ETH")
PYEOF
```

**Important:** Replace `0x<YOUR_PREMINE_ADDRESS>` with your actual premine address from the keystore.

### 2.2 Initialize Geth Node 1

```bash
cd ~/anero-blockchain

# Initialize node1 with new genesis
geth --datadir node1 init config/genesis.json

# You should see: "Successfully wrote genesis state"
```

### 2.3 Initialize Geth Node 2

```bash
cd ~/anero-blockchain

# Initialize node2 with same genesis
geth --datadir node2 init config/genesis.json
```

---

## Phase 3: Setup JWT Secret

### 3.1 Generate or Recreate JWT

```bash
cd ~/anero-blockchain

# Generate new JWT (or reuse existing one if you want consistency)
openssl rand -hex 32 > jwt.hex

# Verify
cat jwt.hex
echo ""  # Add newline for clarity
```

---

## Phase 4: Lighthouse Configuration

### 4.1 Create Lighthouse Config Directory

```bash
cd ~/anero-blockchain

mkdir -p lighthouse_config
```

### 4.2 Create config.yaml for Lighthouse

```bash
cat > ~/anero-blockchain/lighthouse_config/config.yaml << 'EOF'
# Anero Testnet Config
PRESET_BASE: "mainnet"
CONFIG_NAME: "anero_testnet"

# Fork versions (Deneb/Dencun is latest)
GENESIS_FORK_VERSION: 0x00000000
ALTAIR_FORK_VERSION: 0x01000000
BELLATRIX_FORK_VERSION: 0x02000000
CAPELLA_FORK_VERSION: 0x03000000
DENEB_FORK_VERSION: 0x04000000

# Fork epochs (all at 0 for testnet - no hard forks)
GENESIS_EPOCH: 0
ALTAIR_FORK_EPOCH: 0
BELLATRIX_FORK_EPOCH: 0
CAPELLA_FORK_EPOCH: 0
DENEB_FORK_EPOCH: 0

# Genesis requirements
MIN_GENESIS_ACTIVE_VALIDATOR_COUNT: 2
MIN_GENESIS_TIME: 0
GENESIS_DELAY: 0

# Timing
SECONDS_PER_SLOT: 12
SLOTS_PER_EPOCH: 32
SECONDS_PER_ETH1_BLOCK: 14

# Validator settings
MIN_VALIDATOR_WITHDRAWABILITY_DELAY: 256
SHARD_COMMITTEE_PERIOD: 256
ETH1_FOLLOW_DISTANCE: 2048

# Deposit and balance
MIN_DEPOSIT_AMOUNT: 1000000000
MAX_EFFECTIVE_BALANCE: 32000000000
EFFECTIVE_BALANCE_INCREMENT: 1000000000
EJECTION_BALANCE: 16000000000

# Churn limits
MIN_PER_EPOCH_CHURN_LIMIT: 4
CHURN_LIMIT_QUOTIENT: 65536

# Deposit contract (zero address for testnet)
DEPOSIT_CONTRACT_ADDRESS: 0x0000000000000000000000000000000000000000
DEPOSIT_CHAIN_ID: 1889
DEPOSIT_NETWORK_ID: 1889
DEPOSIT_CONTRACT_BLOCK: 0

# BLS withdrawal prefix
BLS_WITHDRAWAL_PREFIX: 0x00

# Penalty and reward quotients
INACTIVITY_SCORE_BIAS: 4
INACTIVITY_SCORE_RECOVERY_RATE: 16
PROPOSER_SCORE_BOOST: 40

# Committee settings
MAX_COMMITTEES_PER_SLOT: 64
TARGET_COMMITTEE_SIZE: 128
MAX_VALIDATORS_PER_COMMITTEE: 2048
SHUFFLE_ROUND_COUNT: 90

# Hysteresis
HYSTERESIS_QUOTIENT: 4
HYSTERESIS_DOWNWARD_MULTIPLIER: 1
HYSTERESIS_UPWARD_MULTIPLIER: 1

# Sync committee
MIN_SYNC_COMMITTEE_SIZE: 512
MAX_SYNC_COMMITTEE_SIZE: 2048
SYNC_COMMITTEE_SUBNETS: 4
EPOCHS_PER_SYNC_COMMITTEE_PERIOD: 256

# Slashing
MIN_SLASHING_PENALTY_QUOTIENT: 128
WHISTLEBLOWER_REWARD_QUOTIENT: 512
PROPOSER_REWARD_QUOTIENT: 8
INACTIVITY_PENALTY_QUOTIENT: 33554432

# Attestation
MIN_ATTESTATION_INCLUSION_DELAY: 1

# Historical roots
SLOTS_PER_HISTORICAL_ROOT: 8192
MIN_SEED_LOOKAHEAD: 1
MAX_SEED_LOOKAHEAD: 4
MIN_EPOCHS_TO_INACTIVITY_PENALTY: 4
EPOCHS_PER_HISTORICAL_VECTOR: 65536
EPOCHS_PER_SLASHINGS_VECTOR: 8192
HISTORICAL_ROOTS_LIMIT: 16777216
VALIDATOR_REGISTRY_LIMIT: 1099511627776

# Network gossip limits
MAX_REQUEST_BODY_SIZE: 1052672
GOSSIP_MAX_SIZE: 1052672
MAX_CHUNK_SIZE: 1052672
MAX_OBJECT_SIZE: 4294967296

# Domain separators
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

echo "✓ Lighthouse config.yaml created"
```

### 4.3 Create mnemonics.yaml for Validators

```bash
cat > ~/anero-blockchain/lighthouse_config/mnemonics.yaml << 'EOF'
- mnemonic: "casual pill wear project analyst accuse aspect oak trumpet analyst grid taste door observe produce input globe uncle avoid urge wisdom three carbon holiday"
  count: 2
EOF

echo "✓ Mnemonics file created (2 validators)"
```

**Note:** Replace the mnemonic with your actual validator mnemonic if different.

### 4.4 Create deploy_block.txt

```bash
echo "0" > ~/anero-blockchain/lighthouse_config/deploy_block.txt

echo "✓ deploy_block.txt created"
```

---

## Phase 5: Generate Genesis State (Beacon Chain)

### 5.1 Generate genesis.ssz using eth2-testnet-genesis

```bash
cd ~/eth2-testnet-genesis

# Clean previous output
rm -f genesis.ssz genesis.json

# Generate new genesis state
./eth2-testnet-genesis capella \
  --config ~/anero-blockchain/lighthouse_config/config.yaml \
  --eth1-config ~/anero-blockchain/config/genesis.json \
  --mnemonics ~/anero-blockchain/lighthouse_config/mnemonics.yaml

# Verify file size (should be 2-3 MB, not kilobytes)
ls -lh genesis.ssz

# Copy to lighthouse config
cp genesis.ssz ~/anero-blockchain/lighthouse_config/genesis.ssz

echo "✓ Genesis SSZ generated and copied"
```

---

## Phase 6: Start Services (5 Terminals)

### Terminal 1: Bootnode (Optional)

```bash
cd ~/anero-blockchain

# Generate bootnode key
bootnode -genkey boot.key

# Start bootnode
bootnode -nodekey boot.key -addr :30301 -verbosity 9
```

**Output:** Note the `enode://...` address for peer discovery.

### Terminal 2: Geth Node 1 (Execution Layer 1)

```bash
cd ~/anero-blockchain

geth --datadir node1 \
  --port 30306 \
  --http --http.addr 0.0.0.0 --http.port 8545 \
  --http.corsdomain "*" --http.api eth,net,web3,admin,debug \
  --networkid 1889 \
  --authrpc.addr 0.0.0.0 --authrpc.port 8551 \
  --authrpc.vhosts "*" \
  --authrpc.jwtsecret jwt.hex console
```

**Wait 5 seconds for Node 1 to start...**

### Terminal 3: Geth Node 2 (Execution Layer 2 - Optional)

```bash
cd ~/anero-blockchain

geth --datadir node2 \
  --port 30307 \
  --http --http.addr 0.0.0.0 --http.port 8546 \
  --http.corsdomain "*" --http.api eth,net,web3,admin,debug \
  --networkid 1889 \
  --authrpc.addr 0.0.0.0 --authrpc.port 8552 \
  --authrpc.vhosts "*" \
  --authrpc.jwtsecret jwt.hex console
```

**Wait 5 seconds for Node 2 to start...**

### Terminal 4: Lighthouse Beacon Node (Consensus Layer)

```bash
cd ~/anero-blockchain

lighthouse beacon_node \
  --testnet-dir lighthouse_config \
  --datadir beacon_data \
  --http --http-address 0.0.0.0 --http-port 5052 \
  --execution-endpoint http://127.0.0.1:8551 \
  --execution-jwt-secret-key $(cat jwt.hex) \
  --enr-address 3.140.21.248 \
  --enr-udp-port 9000 \
  --enr-tcp-port 9000 \
  --allow-insecure-genesis-sync \
  --disable-deposit-contract-sync
```

**Expected output:** Genesis state loads, slot 0 appears, beacon chain syncing.

**Wait 10 seconds for beacon to sync...**

### Terminal 5: Lighthouse Validator Client

```bash
cd ~/anero-blockchain

lighthouse validator_client \
  --testnet-dir lighthouse_config \
  --datadir beacon_data \
  --beacon-nodes http://127.0.0.1:5052
```

**Expected output:** Validators found and connected to beacon node.

---

## Phase 7: Verify Blockchain Is Running

### 7.1 Check Geth Block Height (Terminal 6)

```bash
cd ~/anero-blockchain

# Query Geth via HTTP
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Expected response: {"jsonrpc":"2.0","result":"0x1","id":1}
# The "0x1" means block 1 has been created

# Run this repeatedly to see block height increase
watch -n 5 'curl -s -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  -d "{\"jsonrpc\":\"2.0\",\"method\":\"eth_blockNumber\",\"params\":[],\"id\":1}" | jq -r ".result" | xargs printf "Block height: %s\n"'
```

### 7.2 Check Lighthouse Beacon Node Health (Terminal 6)

```bash
# Query beacon node HTTP API
curl http://127.0.0.1:5052/eth/v1/node/health

# Expected response: 200 OK (synced)

# Check sync status
curl http://127.0.0.1:5052/eth/v1/node/syncing

# Check peer count
curl http://127.0.0.1:5052/eth/v1/node/peers | jq '.data | length'

# Check validator status
curl http://127.0.0.1:5052/eth/v1/beacon/states/head/validators | jq '.data | length'
```

### 7.3 Monitor Slot Progression

```bash
# Watch slots increase (new slot every 12 seconds)
watch -n 1 'curl -s http://127.0.0.1:5052/eth/v1/beacon/head | jq ".data.header.message.slot"'
```

### 7.4 Check Validator Attestations (Terminal 6)

```bash
# View active validators
curl http://127.0.0.1:5052/eth/v1/beacon/states/head/validators | jq '.data[] | {index, pubkey, status}'

# Should show validators with status "active_ongoing"
```

---

## Phase 8: Continuous Monitoring

### 8.1 Setup Monitoring Script

```bash
cat > ~/anero-blockchain/monitor.sh << 'MONEOF'
#!/bin/bash

while true; do
  echo "=== Block Height ==="
  curl -s -X POST http://127.0.0.1:8545 \
    -H "Content-Type: application/json" \
    -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq '.result'

  echo "=== Current Slot ==="
  curl -s http://127.0.0.1:5052/eth/v1/beacon/head | jq '.data.header.message.slot'

  echo "=== Peers Connected ==="
  curl -s http://127.0.0.1:5052/eth/v1/node/peers | jq '.data | length'

  echo "=== Active Validators ==="
  curl -s http://127.0.0.1:5052/eth/v1/beacon/states/head/validators | jq '[.data[] | select(.status == "active_ongoing")] | length'

  echo ""
  sleep 30
done
MONEOF

chmod +x ~/anero-blockchain/monitor.sh

# Run monitoring
~/anero-blockchain/monitor.sh
```

---

## Phase 9: Expected Behavior

### 9.1 First 2-3 Minutes

- Geth nodes initialize
- Lighthouse beacon syncs genesis
- Validators load from keystore
- Block 0 (genesis) appears

### 9.2 Continuous Operation

- **Block height** increases every 12 seconds (1 slot)
- **Slot number** increments: 0 → 1 → 2 → ...
- **Validators** submit attestations
- **Peers** discover each other (if running multiple nodes)

### 9.3 Target Metrics

- Block height: Incrementing ✓
- Slots: 0, 1, 2, 3, ... (every 12 seconds) ✓
- Active validators: 2 ✓
- Peer count: 1+ (if multi-node) ✓

---

## Phase 10: Troubleshooting

### Issue: "Built-in genesis state SSZ bytes are invalid"

**Solution:** Ensure lighthouse_config is a unique directory name not recognized by Lighthouse as built-in.

```bash
grep "CONFIG_NAME:" ~/anero-blockchain/lighthouse_config/config.yaml
# Should NOT be "anero" or "mainnet" - use "anero_testnet"
```

### Issue: Block height not increasing

**Cause:** Geth not processing blocks.

**Fix:**
```bash
# Check Geth logs for errors
tail -50 ~/anero-blockchain/node1/geth.log

# Verify execution endpoint connectivity
curl http://127.0.0.1:8551 -X POST -d '{}' -H "Content-Type: application/json"
```

### Issue: Validators not attesting

**Cause:** Beacon node not synced or validators not loaded.

**Fix:**
```bash
# Check beacon sync status
curl http://127.0.0.1:5052/eth/v1/node/syncing

# Check validator logs
cat ~/anero-blockchain/beacon_data/validator/logs/validator.log | tail -20
```

### Issue: JWT authentication error

**Cause:** JWT secret mismatch between Geth and Lighthouse.

**Fix:**
```bash
# Verify JWT is same everywhere
cat ~/anero-blockchain/jwt.hex

# Restart both services with same jwt.hex file
```

---

## Summary: Complete Restart Checklist

- [ ] Stopped all processes
- [ ] Cleaned blockchain state (kept keystores)
- [ ] Verified premine wallets exist
- [ ] Initialized Geth with new genesis
- [ ] Generated JWT secret
- [ ] Created Lighthouse config directory
- [ ] Created config.yaml with unique CONFIG_NAME
- [ ] Generated genesis.ssz with eth2-testnet-genesis
- [ ] Started bootnode
- [ ] Started Geth Node 1
- [ ] Started Geth Node 2
- [ ] Started Lighthouse Beacon
- [ ] Started Lighthouse Validators
- [ ] Verified block height increasing
- [ ] Verified validators attesting
- [ ] Setup monitoring script

Your Anero blockchain is now running fresh from genesis with your premine accounts intact!
