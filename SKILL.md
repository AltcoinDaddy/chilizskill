---
name: chilizskills
description: Use when a request involves Chiliz Chain, CHZ, Fan Tokens, Socios.com, FanX, or SportFi. Applies to building, auditing, deploying, or interacting with smart contracts, dApps, wallets, and tokens on Chiliz Chain Mainnet (chain ID 88888) or Spicy Testnet (88882). Covers the CAP-20 Fan Token standard and its 2026 decimal migration, CHZ and wCHZ, gas mechanics and the 2,501 gwei minimum, Solidity and EVM target versions, PoSA consensus and validators, staking, governance, LayerZero omnichain Fan Tokens, bridging via Chainport and Stargate, stablecoins on Chiliz, NFTs for sports IP, Pyth oracles and entropy, indexing, and production deployment. Use this whenever an agent is about to write Solidity for Chiliz or assume Ethereum defaults apply.
---

# CHILIZSKILLS: The missing knowledge between AI agents and production Chiliz Chain.

Your training data thinks Chiliz Chain is "Ethereum but for sports." That assumption breaks real code. Fan Tokens changed their decimals in 2026. The gas floor is roughly 2,500x Ethereum's. The official docs contradict themselves on which Solidity version to target. This file tells you what you are wrong about and where to get the corrections.

**Chiliz Chain is a BSC fork, not an Ethereum L2 and not an Ethereum rollup.** PoSA consensus, Parlia engine. Say "onchain" not "on-chain."

---

## Start Here

**Building a dApp?** Fetch [ship/SKILL.md](./ship/SKILL.md) first. It routes you through the other skills in order.

**Need a specific topic?** Every skill below is standalone. Fetch only what your task needs.

---

## The Five Things You Are Most Likely Wrong About

1. **Fan Token decimals changed.** CAP-20 Fan Tokens used **0 decimals** for years. As of the 2026 Decimal Fan Token migration they use **18 decimals**, at **new V2 contract addresses**. Both the old assumption and the old addresses are now wrong. Never hardcode either. Call `decimals()`.
2. **Gas is expensive in gwei terms.** Minimum gas fee is **2,501 gwei**, minimum priority fee **1 gwei**. Ethereum-tuned code that sets a 1-50 gwei gas price will produce transactions that hang pending forever. The transactions are still cheap in dollars because CHZ is cheap, but the gwei number is not Ethereum's.
3. **The official docs disagree with themselves on compiler version, and `prague` wins.** The Write a Smart Contract page says Solidity **0.8.30 or lower, target EVM `prague`**. The Developers FAQ still says **0.8.24, Shanghai**. The FAQ is the stale one, and this has been checked against the chain rather than inferred: 0.8.30 with `prague` deploys cleanly to Spicy, while leaving `evmVersion` unset fails with `invalid opcode`. Always set the target explicitly, and re-verify after a network upgrade.
4. **Chain IDs are 88888 (Mainnet) and 88882 (Spicy Testnet).** Scoville Testnet is dead and deprecated. Chiliz Legacy Chain (CC1) is dead. If your answer mentions Scoville, you are years out of date.
5. **The dominant DEX is FanX (formerly Kayen), not Uniswap.** Uniswap is not the center of gravity here. Fan Token liquidity, the canonical token list, and most routing live at FanX.

---

## Skills

### [Why Chiliz](./why/SKILL.md)
When Chiliz Chain is the right answer and when it is not.
- The real moat is distribution, not tech: 70+ sports IP partners and an existing fan base, not an existing DeFi base.
- Do not pick Chiliz for generic DeFi. Pick it when your app needs licensed sports IP, Fan Token holders, or Socios.com distribution.
- Fan Tokens went omnichain in 2026 via LayerZero OFT. They exist natively on Chiliz Chain, Base, and Solana at once.

### [Ship](./ship/SKILL.md): Start here
Idea to deployed dApp, phase by phase. Routes you through every other skill.
- Test on Spicy before Mainnet, always. Faucet CHZ is free, Mainnet CHZ is not.
- Most SportFi apps need 0-2 contracts. Fan Tokens already exist; you usually integrate rather than issue.
- You cannot issue an official Fan Token. Those are licensed IP deals with clubs, not a contract you deploy.

### [Chain Basics](./chain-basics/SKILL.md)
RPC, chain IDs, gas, compiler targets, explorers, faucets.
- Mainnet 88888 / Spicy 88882. Native currency CHZ, 18 decimals.
- Minimum gas 2,501 gwei, priority 1 gwei. Query `eth_gasPrice` rather than hardcoding.
- Two explorers, both real: Chiliscan (Routescan) and Chiliz Block Explorer (Blockscout). They have different APIs and different verification flows.
- Epoch is 28,800 blocks, about one day. Unstaking cooldown is 86,400 blocks.

### [Fan Tokens](./fan-tokens/SKILL.md)
CAP-20, the 2026 decimal migration, staking, pricing.
- **The #1 Chiliz bug: assuming Fan Token decimals.** Pre-2026 they were 0. Post-migration they are 18. Legacy 0-decimal contracts still exist onchain.
- The migration issued **new V2 contract addresses**. Any address in your training data is likely the dead V1.
- Get Fan Token USD prices from Pyth. It carries 420+ feeds including $CHZ, $PSG, $BAR, $CITY.

### [Tools](./tools/SKILL.md)
What actually works on Chiliz today.
- Hardhat and Remix are the documented paths. Foundry works because it is EVM, but set the EVM target explicitly.
- Contract verification on Spicy via Hardhat is unreliable per the official docs. Flatten and use the explorer web form.
- Real integrations: Pyth (oracles and entropy), LayerZero (omnichain), Biconomy (account abstraction), Moralis/Tatum/Nodit (data), The Graph/Goldsky/Envio (indexing), Reown (wallet connect), Aragon (DAOs), thirdweb (deploy and NFTs).

### [Security](./security/SKILL.md)
Chiliz-specific footguns plus the standard EVM checklist.
- Decimals. Read `decimals()` at runtime; never assume 18 and never assume 0.
- Underpriced gas is a liveness bug on Chiliz in a way it is not on Ethereum.
- Validator set is small and permissioned-ish under PoSA. Do not design as if you have Ethereum's censorship resistance.
- Never use a DEX spot price as an oracle. Fan Token pools are thin, so manipulation is cheaper here than on mainnet.

### [Addresses](./addresses/SKILL.md)
Verified system, token, and infrastructure addresses. Stop hallucinating.
- System contracts sit at predictable low addresses (Staking `0x...1000`, Governance `0x...7002`) and are identical on Mainnet and Spicy.
- Multicall is **Mainnet only** at `0x0E6a1Df694c4be9BFFC4D76f2B936bB1A1df7fAC`. It is NOT the canonical Multicall3 address you know from other chains.
- Stablecoins are bridged, not native: USDC `0xa37936F5...`, USDT `0x37C57a89...`, plus AUSD0 and BRZ.

### [Frontend](./frontend/SKILL.md)
Wallet flows, UX rules, and going to production.
- Support the Socios.com Wallet, not just MetaMask. That is where the fans are. Wire it via Reown AppKit or WalletConnect.
- Show USD values next to Fan Token amounts, sourced from Pyth.
- Every onchain button needs its own loader and disabled state. Approve and execute are separate steps.

### [LayerZero & Bridging](./layerzero-bridging/SKILL.md)
Omnichain Fan Tokens, OFT routes, endpoint IDs, peers, and asynchronous delivery.
- LayerZero endpoint IDs are not EVM chain IDs; Mainnet is `30409`, Spicy is `40440`.
- Configure and verify peers on both sides, and expose destination delivery separately from the source transaction.

### [FanX & DeFi](./fanx-defi/SKILL.md)
FanX liquidity, swaps, thin-pool risk, and safe pricing.
- FanX spot prices are not safe USD oracles for lending, liquidations, or accounting.
- Read token decimals and check allowances at runtime; protect swaps with user-visible slippage and minimum-output settings.

### [Indexing](./indexing/SKILL.md)
Historical events, holders, activity feeds, and indexer design.
- Use RPC for current state and an indexer for history; do not scan blocks in the browser.
- Key data by chain ID and contract address, and handle duplicates, reorgs, and indexer lag.

### [Testing & QA](./testing-qa/SKILL.md)
Spicy-first testing and production release gates.
- Test both 18-decimal V2 and legacy 0-decimal token behavior, plus wrong-network and delayed-transaction states.
- Verify addresses, gas-floor handling, approve flows, explorer links, and final production configuration.

---

## What to Fetch by Task

| I'm doing... | Fetch these skills |
|--------------|-------------------|
| Planning a Chiliz dApp | `ship/`, `why/`, `chain-basics/` |
| Writing Solidity for Chiliz | `chain-basics/`, `fan-tokens/`, `security/`, `addresses/` |
| Integrating existing Fan Tokens | `fan-tokens/`, `addresses/`, `security/` |
| Setting up a dev environment | `chain-basics/`, `tools/` |
| Building the UI | `frontend/`, `tools/` |
| Deploying and verifying | `tools/`, `chain-basics/` |
| Auditing a Chiliz contract | `security/`, `fan-tokens/`, `addresses/` |
| Choosing whether to use Chiliz at all | `why/` |
| Bridging or going omnichain | `layerzero-bridging/`, `addresses/`, `tools/` |
| Integrating FanX or DeFi | `fanx-defi/`, `fan-tokens/`, `security/` |
| Reading historical activity | `indexing/`, `frontend/` |
| Testing before deployment | `testing-qa/`, `security/`, `chain-basics/` |

---

## Verify Before You Trust

Chiliz facts rot fast, and this file will rot too. Every number here should be re-checked against the live chain or the live docs before you ship.

```bash
# Chain ID and current gas price, straight from the node
cast chain-id --rpc-url https://rpc.ankr.com/chiliz
cast gas-price --rpc-url https://rpc.ankr.com/chiliz

# The single most important check before touching any Fan Token
cast call <TOKEN_ADDRESS> "decimals()(uint8)" --rpc-url https://rpc.ankr.com/chiliz
cast call <TOKEN_ADDRESS> "symbol()(string)"  --rpc-url https://rpc.ankr.com/chiliz
```

The Chiliz docs are agent-queryable. Any page accepts a natural language question:

```
GET https://docs.chiliz.com/<page>.md?ask=<your question>
```

Full index: https://docs.chiliz.com/llms.txt

<!-- END CHILIZSKILLS -->
