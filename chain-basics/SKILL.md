---
name: chiliz-chain-basics
description: Chiliz Chain network parameters, RPC endpoints, chain IDs, gas mechanics, Solidity compiler and EVM targets, block explorers, faucets, epochs, and consensus. Use before configuring any dev environment, wallet, RPC client, or hardhat/foundry config for Chiliz Chain or Spicy Testnet.
---

# Chiliz Chain Basics

Everything an agent needs to configure a working environment, plus the defaults that silently break when copied from Ethereum.

---

## Network Parameters

| | Chiliz Chain Mainnet | Spicy Testnet |
|---|---|---|
| Chain ID | **88888** | **88882** |
| Currency | CHZ (18 decimals) | CHZ (18 decimals) |
| RPC (recommended) | `https://rpc.ankr.com/chiliz` | `https://spicy-rpc.chiliz.com/` |
| RPC (alt) | `https://chiliz-rpc.publicnode.com`<br>`https://chiliz-mainnet.gateway.tatum.io` | `https://chiliz-spicy.publicnode.com/`<br>`https://chiliz-testnet.gateway.tatum.io` |
| WebSocket | via provider | `wss://spicy-rpc-ws.chiliz.com/` |
| Explorer (recommended) | `https://chiliscan.com` | `https://testnet.chiliscan.com/` |
| Explorer (alt) | `https://scan.chiliz.com` | `https://spicy-explorer.chiliz.com/` |
| LayerZero EID | 30409 | 40440 |

Public RPCs are rate limited. Space out requests when testing, or use a provider key from Ankr, Tatum, or Nodit.

**Scoville Testnet is deprecated and dead.** It was replaced by Spicy in 2023. Never target it.
**Chiliz Legacy Chain (CC1) is dead.** All Fan Tokens migrated to Chiliz Chain (CC2) in September 2023.

---

## Gas: The Number That Breaks Ethereum Code

```
Minimum gas fee:      2,501 gwei
Minimum priority fee: 1 gwei
```

This is the single most common cause of a Chiliz transaction sitting in `pending` forever. Ethereum-derived code and AI-generated snippets routinely set `maxFeePerGas` to something like `20 gwei`, which is roughly 125x below the Chiliz floor. The transaction is accepted by the mempool and then never mined.

Do not hardcode the gas price either. Query it:

```bash
curl -s https://rpc.ankr.com/chiliz \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_gasPrice","params":[]}'
```

```javascript
// ethers v6
const fee = await provider.getFeeData();
// Use fee.gasPrice / fee.maxFeePerGas as the floor, then add headroom.
// Do NOT substitute an Ethereum-style constant.
```

Alternative sources: Chiliscan (Routescan) `avg-gas-price`, Chiliz Block Explorer `/api/v2/stats`, or Blocknative's Gas Network with `chainid=88888`. Explorer stats endpoints have been observed reporting values inconsistent with the documented 2,501 gwei floor, so treat `eth_gasPrice` from the node as the source of truth and the 2,501 gwei figure as a safety floor.

High traffic is event-driven here, not DeFi-driven. Expect congestion during live matches and Fan Token drops, not during liquidation cascades.

---

## Solidity and EVM Target

The official docs currently disagree with themselves. Both of these are live pages:

| Source | Says |
|---|---|
| [Write a Smart Contract](https://docs.chiliz.com/develop/basics/write-a-smart-contract.md) | Solidity **0.8.30 or lower**, EVM target **`prague`** |
| [Developers FAQ](https://docs.chiliz.com/develop/developers-faq.md) | Solidity **0.8.24**, EVM **Shanghai** |

The FAQ is the stale page. Prefer the Write a Smart Contract guidance, but **verify by deploying to Spicy first**. Never set `evmVersion` to `cancun`, `default`, or leave it unset while assuming Ethereum parity, because an unsupported opcode fails at deploy time or, worse, at runtime.

```javascript
// hardhat.config.js
module.exports = {
  solidity: {
    version: "0.8.30",
    settings: { optimizer: { enabled: true, runs: 200 }, evmVersion: "prague" }
  },
  networks: {
    chiliz: { url: "https://rpc.ankr.com/chiliz", chainId: 88888 },
    spicy:  { url: "https://spicy-rpc.chiliz.com/", chainId: 88882 }
  }
};
```

```toml
# foundry.toml: Foundry is not the documented path but works, EVM is EVM.
[profile.default]
solc_version = "0.8.30"
evm_version  = "prague"
[rpc_endpoints]
chiliz = "https://rpc.ankr.com/chiliz"
spicy  = "https://spicy-rpc.chiliz.com/"
```

If a deploy reverts with no reason on Spicy but compiles clean, drop the EVM target one fork and retry before debugging your logic.

---

## Consensus and Timing

- **PoSA (Proof of Staked Authority), Parlia engine.** Chiliz Chain is a BSC fork, not an Ethereum client and not a rollup. Ankr helped build it.
- **Epoch = 28,800 blocks, roughly 1 day.** Validator elections, reward distribution, and unstaking cooldowns all key off epochs.
- **Unstaking cooldown = 86,400 blocks**, roughly 3 days.
- **Validator minimum stake = 10,000,000 CHZ**, plus validator set approval. The set is small and governed.
- **Snake8 hard fork (October 2025)** replaced equal block distribution with randomized block producer selection weighted by staked CHZ.
- **Dragon8 hard fork (2024)** brought Tokenomics 2.0: dynamic inflation starting at 8.80% decaying to a 1.88% floor, EIP-1559 fee burning, and a 65/25/10 split of rewards across validators+delegators / Ecosystem & Operational / Community Vault.
- Governance uses a **Compound Alpha** model with a two-thirds quorum, voted by validators.

The practical consequence: **do not design as though you have Ethereum's censorship resistance.** A small permissioned-leaning validator set is a real trust assumption. State it honestly in any architecture review.

---

## Testnet CHZ

- **Spicy Faucet** at `spicy-faucet.chiliz.com`: 20 test CHZ per 24h.
- **Tatum Faucet**: 10 test CHZ per 24h, requires a Tatum Dashboard account.

If you need more volume than the faucets allow, deploy your own OpenZeppelin ERC-20 on Spicy and test against that, then do a final pass with real test CHZ before Mainnet.

---

## Debugging a Failed Transaction

The explorers often will not show you the revert reason. Trace it directly:

```bash
curl -s https://rpc.ankr.com/chiliz \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"debug_traceTransaction",
       "params":["<TX_HASH>",{"tracer":"callTracer"}]}'
```

Before you trace, rule out the two boring causes: gas below 2,501 gwei, and wrong chain ID.

---

## Verify

```bash
cast chain-id   --rpc-url https://rpc.ankr.com/chiliz   # expect 88888
cast gas-price  --rpc-url https://rpc.ankr.com/chiliz
cast block-number --rpc-url https://rpc.ankr.com/chiliz
```

Sources: [Connect using RPC](https://docs.chiliz.com/develop/basics/connect-to-chiliz-chain/connect-using-rpc.md), [Developers FAQ](https://docs.chiliz.com/develop/developers-faq.md), [Estimate gas fees](https://docs.chiliz.com/develop/advanced/estimate-gas-fees.md), [Write a Smart Contract](https://docs.chiliz.com/develop/basics/write-a-smart-contract.md), [Staking FAQ](https://docs.chiliz.com/learn/about-staking/chiliz-staking-faq.md), [Snake8](https://docs.chiliz.com/chiliz-chain-changelog/governance-proposals-and-decisions/october-2025-snake8-hard-fork.md)
