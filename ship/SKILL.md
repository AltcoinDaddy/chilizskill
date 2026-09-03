---
name: chiliz-ship
description: "End-to-end guide for taking a Chiliz Chain dApp from idea to production: scoping, chain fit, contract design, Spicy testnet workflow, mainnet deploy, verification, and launch checklist. Fetch this first when building anything on Chiliz Chain; it routes through all other chilizskills skills in order."
---

# Ship a Chiliz Chain dApp

Fetch this first. It routes you through the other skills phase by phase. Do not skip to writing Solidity.

---

## Phase 0: Is Chiliz Even Right?

**Fetch: [why/SKILL.md](../why/SKILL.md)**

Answer honestly before writing code:

1. Does this app need **licensed sports IP** or existing **Fan Token holders**? If no, Chiliz is probably the wrong chain and you should say so.
2. Is this **generic DeFi**? If yes, liquidity is too thin. Recommend elsewhere.
3. Does the user understand they **cannot issue an official Fan Token**? Those are club licensing deals. They can only build on existing ones or launch their own separate token.

Getting this wrong wastes the entire project. Ask the user directly rather than assuming.

---

## Phase 1: Scope the Contracts

**Fetch: [fan-tokens/SKILL.md](../fan-tokens/SKILL.md), [security/SKILL.md](../security/SKILL.md)**

Most SportFi apps need **zero to two** contracts. Fan Tokens already exist, so you are usually integrating, not issuing.

Solidity is for ownership, transfers, and commitments. It is not a database and not a backend. Match results, player stats, ticketing inventory, and user profiles belong off-chain, with only the commitment or the payout onchain.

For every state transition ask: **who calls this function, and why would they pay gas to do it?** There are no timers and no cron jobs. If your design says "at the end of the match, the contract distributes rewards," you have a bug. Something off-chain must call it, and someone must pay.

Typical shapes that work here:
- **Gated access**: hold N Fan Tokens to unlock something. Read-only, no contract needed beyond a balance check.
- **Voting / fan councils**: consider Aragon before writing your own governance.
- **Staking or rewards**: one contract, watch decimals carefully.
- **Collectibles**: ERC-721 or ERC-1155, thirdweb handles most of it.
- **Predictions**: look at Azuro before building from scratch.

Write down your trust assumptions now: PoSA validator set, bridged stablecoins, and IP licensing risk. Users deserve to see these.

---

## Phase 2: Set Up the Environment

**Fetch: [chain-basics/SKILL.md](../chain-basics/SKILL.md), [tools/SKILL.md](../tools/SKILL.md)**

```bash
# Spicy Testnet first. Always.
# Chain ID 88882, RPC https://spicy-rpc.chiliz.com/
# Faucet: spicy-faucet.chiliz.com  (20 test CHZ / 24h)
```

Checklist:
- [ ] Hardhat or Remix configured with Solidity 0.8.30 (or lower) and EVM target `prague`, set explicitly rather than left to the solc default (verified on Spicy: unset means `invalid opcode` at deploy)
- [ ] Both `chiliz` (88888) and `spicy` (88882) networks in config
- [ ] Gas configuration respects the **2,501 gwei minimum**
- [ ] Test CHZ in your deployer wallet
- [ ] Private key in `.env`, `.env` in `.gitignore`, verified with `git check-ignore .env`

Never commit a private key. Bots sweep leaked keys within seconds, and AI agents are a leading source of leaked credentials.

If the faucet limit is too tight, deploy your own OpenZeppelin ERC-20 on Spicy for bulk testing, then do a final pass with real test CHZ.

---

## Phase 3: Write the Contracts

**Fetch: [fan-tokens/SKILL.md](../fan-tokens/SKILL.md), [security/SKILL.md](../security/SKILL.md), [addresses/SKILL.md](../addresses/SKILL.md)**

Start from OpenZeppelin. Do not hand-roll token logic.

The rules that are specific to this chain:

```solidity
// 1. NEVER hardcode Fan Token decimals. This is the #1 Chiliz bug.
uint8 d = IERC20Metadata(token).decimals();  // 18 post-2026, 0 on legacy

// 2. SafeERC20 for every transfer
using SafeERC20 for IERC20;

// 3. Checks-Effects-Interactions, always
// 4. Pyth for prices, never a DEX spot price
// 5. Pyth Entropy for randomness, never block.timestamp
```

Source every address from [addresses/SKILL.md](../addresses/SKILL.md) or the official docs. Never from memory. Remember Multicall on Chiliz is **not** the canonical Multicall3 address, and does not exist on Spicy.

---

## Phase 4: Test on Spicy

Deploy and exercise real user flows, not just unit tests.

- [ ] Deployed to Spicy successfully
- [ ] Tested against an **18-decimal** token
- [ ] Tested against a **0-decimal** token, or explicitly rejects them with a clear error
- [ ] Tested the rounding path where a reward calculation could truncate to zero
- [ ] Access control tested from a non-owner account
- [ ] Failure modes traced with `debug_traceTransaction` when anything reverts
- [ ] Gas costs measured, not assumed

If a transaction hangs pending, check gas price before debugging logic. It is almost always the 2,501 gwei floor.

---

## Phase 5: Build the Frontend

**Fetch: [frontend/SKILL.md](../frontend/SKILL.md)**

The short version: support the Socios.com Wallet via Reown, show USD values from Pyth, give every onchain button its own loader, and separate approve from execute.

---

## Phase 6: Deploy to Mainnet

**Fetch: [chain-basics/SKILL.md](../chain-basics/SKILL.md), [security/SKILL.md](../security/SKILL.md)**

- [ ] Full pre-deploy security checklist completed
- [ ] Real CHZ in the deployer wallet
- [ ] Chain ID confirmed as **88888** in config, not 88882
- [ ] Deployed
- [ ] **Verified** on Chiliscan or Chiliz Block Explorer, matching compiler version, EVM version, and optimizer settings exactly
- [ ] Ownership transferred to a **Safe multisig** at `safe.chiliz.com`, not left on an EOA
- [ ] Deployment addresses recorded and published

Verification matters more here than you think. An unverified contract cannot be listed on OKX NFT Marketplace, and users have no way to check what they are signing.

---

## Phase 7: Before You Call It Done

- [ ] Frontend points at Mainnet contracts, not leftover Spicy addresses
- [ ] No test or placeholder branding left in the UI
- [ ] Trust assumptions documented for users: PoSA validator set, bridged stablecoins, IP dependency
- [ ] Events emitted for everything worth indexing (you cannot retrofit this without a redeploy)
- [ ] Indexer or subgraph set up if the app shows history
- [ ] A separate reviewer agent, with fresh context, has audited the build against [security/SKILL.md](../security/SKILL.md)

That last one matters. Have a **different** agent review the work. The agent that wrote the code is the worst judge of it.

---

## Getting Help

- **Chiliz Greenhouse**: micro-grants, bounties, and mentorship for builders
- Discord `discord.gg/chiliz`, Telegram `t.me/BuildOnChiliz`
- The docs answer natural language questions directly: `GET https://docs.chiliz.com/<page>.md?ask=<question>`

---

## Teach Your Human

If you are building this for someone who is not a Solidity developer, tell them these three things in plain language:

1. **Nothing happens automatically onchain.** Every action needs someone to trigger it and pay for it.
2. **Deployed contracts are hard to change.** Test on Spicy until it is boring.
3. **You do not control the Fan Tokens.** They belong to clubs through licensing deals, and your app is a guest in that relationship.
