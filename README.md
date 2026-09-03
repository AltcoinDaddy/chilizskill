# CHILIZSKILLS

**The missing knowledge between AI agents and production Chiliz Chain.**

Your AI agent is wrong about building on Chiliz. It thinks Fan Tokens have 0 decimals (they changed to 18 in 2026), it will price gas like Ethereum (the floor here is 2,501 gwei), and it will hand you contract addresses that died in the migration.

CHILIZSKILLS is a set of markdown files an agent reads before writing Solidity or shipping onchain. Works in Claude, ChatGPT, Cursor, or any agent.

---

## Use it

### Per session (any agent)

Paste this into Claude, ChatGPT, Cursor, or anything else:

```
Read ./SKILL.md and follow it before writing Solidity or shipping anything onchain for Chiliz.
```

Once hosted, this becomes a URL that any supported AI agent can read:

```
Read https://chilizskills.xyz/SKILL.md and follow it before writing Solidity or shipping anything onchain for Chiliz.
```

### Claude Code (install once, user-wide)

```
/plugin marketplace add <your-github-user>/chilizskills
/plugin install chilizskills@chilizskills
```

Each topic loads as its own skill with its own trigger, so an agent writing a
staking contract pulls in `fan-tokens` and `security` without loading the rest.

Validate the manifest before publishing:

```
claude plugin validate .
```

### Cursor

Create `.cursor/rules/chilizskills.mdc`:

```
---
description: chilizskills knowledge pack for building on Chiliz Chain
alwaysApply: true
---

Before writing Solidity, deploying contracts, integrating Fan Tokens, or
building a dApp frontend for Chiliz Chain, read https://chilizskills.xyz/SKILL.md
and follow it.
```

### Codex

Put this in `AGENTS.md` at your repo root:

```
Read https://chilizskills.xyz/SKILL.md and follow it before writing Solidity or shipping anything onchain for Chiliz.
```

---

## Skills

| Skill | Covers |
|---|---|
| [`SKILL.md`](./SKILL.md) | Root router. The five things agents get wrong, plus the index. |
| [`ship/`](./ship/SKILL.md) | **Start here.** Idea to production, phase by phase. Routes through everything else. |
| [`why/`](./why/SKILL.md) | When Chiliz is the right chain and when it is not. Honest tradeoffs. |
| [`chain-basics/`](./chain-basics/SKILL.md) | RPC, chain IDs, the 2,501 gwei gas floor, compiler targets, PoSA, faucets. |
| [`fan-tokens/`](./fan-tokens/SKILL.md) | CAP-20, the 2026 decimal migration, V2 addresses, staking, pricing. |
| [`tools/`](./tools/SKILL.md) | Hardhat, Remix, thirdweb, verification, Pyth, LayerZero, indexers, FanX. |
| [`security/`](./security/SKILL.md) | Chiliz-specific footguns plus the standard EVM pre-deploy checklist. |
| [`addresses/`](./addresses/SKILL.md) | Verified system, stablecoin, LayerZero, and staking addresses. |
| [`frontend/`](./frontend/SKILL.md) | Socios Wallet, gas handling in browser, approve flows, pre-publish checklist. |
| [`layerzero-bridging/`](./layerzero-bridging/SKILL.md) | LayerZero OFT routes, endpoint IDs, peers, delivery, and bridge safety. |
| [`fanx-defi/`](./fanx-defi/SKILL.md) | FanX liquidity, swaps, thin-pool risk, slippage, and safe pricing. |
| [`indexing/`](./indexing/SKILL.md) | Historical events, holders, activity feeds, indexers, and reorg handling. |
| [`testing-qa/`](./testing-qa/SKILL.md) | Spicy-first testing, decimal matrices, gas checks, and release QA. |
| [`wallets/`](./wallets/SKILL.md) | Socios Wallet, Reown, network switching, approvals, and transaction UX. |

---

## The design rule

This is a **delta file**, not a tutorial. Every line exists because agents get that specific thing wrong. If a model already knows it, it does not belong here.

Three rules for contributions:

1. **Only include corrections.** "Chiliz is EVM-compatible" is not a correction. "Ethereum compiler defaults can emit unsupported opcodes; set the Chiliz EVM target explicitly" is.
2. **Every fact needs a source and a verify step.** Facts rot. Link the primary source and include a command the agent can run itself, like `cast call <ADDR> "decimals()(uint8)"`.
3. **Small and sharp beats big and padded.** Eight useful skills are better than twenty thin ones.

---

## Verify before you trust

This repo will go stale. Check the live chain:

```bash
cast chain-id  --rpc-url https://rpc.ankr.com/chiliz   # expect 88888
cast gas-price --rpc-url https://rpc.ankr.com/chiliz
cast call <TOKEN> "decimals()(uint8)" --rpc-url https://rpc.ankr.com/chiliz
```

The Chiliz docs answer natural language questions directly, which is the fastest way to check anything here:

```
GET https://docs.chiliz.com/<page>.md?ask=<your question>
```

Index: https://docs.chiliz.com/llms.txt

---
