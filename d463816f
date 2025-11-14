# Anero Blockchain: Native Currency Configuration Guide

## Overview

This guide walks you through configuring your forked Ethereum blockchain to use **Anero** as the native currency symbol instead of ETH. Since you have already set up your boot node, Node 1, Node 2, Lighthouse beacon, and Lighthouse validators, this guide focuses on the configuration and display aspects of the Anero native currency.

---

## Part 1: Network Configuration (Backend Setup)

### Step 1: Verify Your Genesis Configuration

Your `genesis.json` file should already contain the core network parameters. Ensure it includes:

```json
{
  "config": {
    "chainId": YOUR_CHAIN_ID,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "londonBlock": 0
  },
  "difficulty": "0x400",
  "gasLimit": "0x8000000",
  "alloc": {
    "0x1234567890123456789012345678901234567890": {
      "balance": "0x200000000000000000000000000000000000000000000000000000000000000"
    }
  }
}
```

**Key Points:**
- The `chainId` is a unique identifier for your blockchain (ensure it's not already in use by checking chainlist.org)
- The `gasLimit` and other parameters control network behavior
- The `alloc` section pre-funds specific accounts during genesis initialization

### Step 2: Update Network ID

When starting your Geth nodes, use a consistent `--networkid` flag. This should match your `chainId` from the genesis file:

```bash
# For Node 1
geth --datadir data/node1 \
  --networkid 123454321 \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8545 \
  --http.api eth,web3,net,personal,web3 \
  --ws \
  --ws.addr 0.0.0.0 \
  --ws.port 8546 \
  --ws.api eth,web3,net,personal
```

The native currency of this blockchain is inherently tied to this network configuration. All transactions use this native currency for gas fees.

---

## Part 2: MetaMask Configuration (Display Setup)

### Step 1: Add Anero Network to MetaMask

Since you mentioned adding the blockchain to MetaMask without Node 1 and Node 2 wallets, here's how to properly configure it:

1. **Open MetaMask** and click the **Network selector** (top-left)
2. Click **Add network manually** (if your network isn't listed)
3. Fill in the following details:

| Field | Value | Example |
|-------|-------|---------|
| **Network Name** | Anero Network | Anero Network |
| **New RPC URL** | Your RPC endpoint | http://localhost:8545 |
| **Chain ID** | Your chain ID | 123454321 |
| **Currency Symbol** | Anero | ANERO |
| **Block Explorer URL** (optional) | Your block explorer | http://localhost:4000 |

4. Click **Save**

### Step 2: Verify the Native Currency Display

After adding the network:

1. Switch to the **Anero Network** in MetaMask
2. Check your account balance
3. The currency symbol should now display as **ANERO** instead of ETH
4. Gas fees should also display in ANERO

---

## Part 3: Adding Wallets to MetaMask (If Needed)

If you want to import Node 1 and Node 2 wallet accounts into MetaMask:

### Option A: Import Using Private Key

1. In MetaMask, click the **Account icon** (top-right)
2. Select **Import Account**
3. Enter the private key from your Node 1 or Node 2 account
4. Click **Import**

### Option B: Using Account Mnemonic

If you have the mnemonic phrase for Node 1 or Node 2:

1. Click the **Account icon** → **Settings** → **Security & Privacy**
2. Follow the recovery process using your mnemonic phrase

---

## Part 4: Testing the Native Currency Setup

### Test 1: Check Network Connection

```bash
# Attach to Node 1's console
geth attach node1/geth.ipc

# Run these commands
> net.version          # Should return your network ID
> eth.chainId()        # Should return your chain ID
> eth.blockNumber      # Should show current block number
> web3.eth.getBalance("0xYourAddress")  # Check balance in Wei
```

### Test 2: Send a Transaction

1. In MetaMask, ensure you're on the **Anero Network**
2. Send ANERO to another address on your network
3. Verify the transaction:
   - Gas is calculated in ANERO
   - Transaction confirmation uses ANERO
   - Block explorer displays ANERO as the currency

### Test 3: Verify in Block Explorer

If you have a block explorer set up (like Blockscout):

1. Access your block explorer
2. The native currency should display as **ANERO**
3. All transactions and balances should show in ANERO

---

## Part 5: Optional - Block Explorer Configuration

If using Blockscout or another block explorer, configure it to display the native currency:

### Blockscout Configuration

In your Blockscout environment file (`.env` or configuration file), ensure:

```
COIN=ANERO
COIN_NAME=Anero
```

This ensures the block explorer displays your native currency correctly.

---

## Part 6: Multi-Node Setup Synchronization

Since you have Node 1 and Node 2, ensure all nodes use:

1. **Same genesis.json** - All nodes must initialize with the same genesis file
2. **Same network ID** - Use the same `--networkid` when starting each node
3. **Same chain ID** - The `chainId` in genesis.json must be identical across all nodes

**Verify synchronization:**

```bash
# On Node 1
geth attach node1/geth.ipc
> eth.blockNumber

# On Node 2
geth attach node2/geth.ipc
> eth.blockNumber

# Both should have the same block number or very close
```

---

## Part 7: Lighthouse Validator Configuration

Your Lighthouse beacon and validators should already be configured. The native currency (Anero) is used for:

1. **Stake deposits** - Validators deposit ANERO to participate
2. **Rewards** - Validators earn ANERO for proposing and attesting blocks
3. **Penalties** - Validators lose ANERO for misbehavior (slashing)

All these transactions occur on your configured network with the ANERO native currency.

---

## Part 8: Troubleshooting

### Issue: MetaMask Shows ETH Instead of ANERO

**Solution:**
1. Disconnect from the network in MetaMask
2. Remove the network from MetaMask
3. Re-add the network with the correct **Currency Symbol: ANERO**
4. Switch to the network again

### Issue: Incorrect Chain ID in MetaMask

**Solution:**
1. Verify your `genesis.json` chainId
2. Edit the network in MetaMask
3. Ensure the **Chain ID** field matches exactly
4. Save and refresh MetaMask

### Issue: Nodes Not Syncing

**Solution:**
1. Ensure all nodes have the same `genesis.json`
2. Check that the `--networkid` is identical on all nodes
3. Verify enode connections between nodes
4. Check firewall rules for peer-to-peer communication

---

## Summary

You've successfully configured Anero as the native currency for your forked Ethereum blockchain:

✅ **Backend:** Network configured with chainId and network ID in genesis.json
✅ **Frontend:** MetaMask displays ANERO as the native currency symbol
✅ **Transactions:** All gas fees and transactions use ANERO
✅ **Validators:** Lighthouse validators operate using ANERO for staking and rewards
✅ **Multi-node:** All nodes (boot, Node 1, Node 2) synchronized with ANERO configuration

The Anero Blockchain is now ready for development and testing with its native ANERO currency fully integrated across all components.

---

## Next Steps

1. **Deploy Smart Contracts** - Use Remix IDE or Hardhat to deploy contracts on Anero
2. **Create DApps** - Develop applications that interact with your Anero network
3. **Monitor Validators** - Use Lighthouse dashboards to monitor validator performance
4. **Add More Nodes** - Scale your network by adding additional validator nodes
5. **Implement Token Standards** - Create ERC-20 or other tokens on top of the Anero blockchain
