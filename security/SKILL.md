---
name: chiliz-security
description: "Security patterns and vulnerabilities specific to Chiliz Chain smart contracts: Fan Token decimals bugs, gas floor liveness failures, thin liquidity oracle manipulation, PoSA trust assumptions, plus the standard EVM pre-deploy checklist. Use before deploying any contract to Chiliz Chain or when auditing a Chiliz contract."
---

# Chiliz Chain Security

The generic EVM checklist still applies. This skill covers what is *different* about Chiliz, then the standard list.

---

## Chiliz-Specific Vulnerabilities

### 1. Decimals confusion (the #1 Chiliz bug)

Fan Tokens moved from 0 decimals to 18 decimals in the 2026 migration. Legacy 0-decimal contracts still exist onchain.

```solidity
// VULNERABLE: assumes 18
uint256 reward = userStake * rewardRate / 1e18;

// VULNERABLE: assumes 0, pre-2026 style
uint256 reward = userStake * rewardRate;

// SAFE
uint8 d = IERC20Metadata(token).decimals();
uint256 scale = 10 ** d;
uint256 reward = userStake * rewardRate / scale;
```

Worse, a contract that supports **multiple** Fan Tokens with a single shared scaling constant is broken by construction if any of them differ. Store decimals per token at registration time and validate it:

```solidity
mapping(address => uint8) public tokenDecimals;

function addToken(address token) external onlyOwner {
    uint8 d = IERC20Metadata(token).decimals();
    require(d <= 18, "unsupported decimals");
    tokenDecimals[token] = d;
}
```

Note that with 0-decimal legacy tokens, integer division truncates aggressively. A reward calculation that yields a fraction of a token silently rounds to zero, so users stake and earn nothing. Test that path explicitly.

### 2. Underpriced gas is a liveness bug

Minimum gas fee on Chiliz is **2,501 gwei**, minimum priority fee **1 gwei**. Any hardcoded Ethereum-scale gas price makes transactions hang pending indefinitely.

This is a security issue, not just UX, when your protocol depends on timely execution: liquidations, auction settlement, oracle updates, and time-locked withdrawals all fail open or fail closed in ways an attacker can plan around. If your design assumes "someone will submit this transaction in time," verify their client is not using an Ethereum gas default.

### 3. Thin liquidity makes spot-price oracles cheap to manipulate

Fan Token pools are small compared to major DEX pools. Flash-loan-free manipulation is realistic with modest capital.

- **Never** use a FanX or any DEX spot price as an oracle input.
- Use **Pyth** price feeds for Fan Token and CHZ USD values.
- If you must use pool data, use a long TWAP and cap per-update movement.

Also consider the sports-specific angle: Fan Token prices move sharply on match results, transfers, and trophy events. A liquidation threshold that is safe on a Tuesday may be crossed by fifty percent of your users simultaneously on a Saturday evening. Model correlated, event-driven volatility, not just random walks.

### 4. PoSA trust assumptions

Chiliz Chain runs Proof of Staked Authority with a small validator set (10M CHZ minimum stake, plus validator set approval, governed by a two-thirds validator quorum).

Be honest about what this means:
- Censorship resistance is materially weaker than Ethereum L1. Do not claim otherwise in documentation.
- Reorg and liveness assumptions differ from Ethereum. The chain has had outages, documented publicly, including a failed Dragon8 hard fork in May 2024 that required a rollback.
- Governance can upgrade system contracts via `RuntimeUpgrade`. Your contract's environment is more mutable than on Ethereum.

For anything holding significant value, state these assumptions explicitly and give users an exit path.

### 5. Bridged asset risk

Every stablecoin on Chiliz is bridged (Chainport, Stargate, LayerZero). Your "USDC" is a bridge IOU, not Circle-issued USDC. A bridge failure depegs it to zero regardless of what happens to real USDC.

- Do not treat bridged stables as risk-free units of account.
- Do not assume 6 decimals. Read `decimals()`.
- If you accept several bridged versions of the same asset, they are **not fungible** with each other. Never allow them into a shared pool at a 1:1 assumption.

### 6. Fan Tokens are IP, and IP has off-chain rules

Fan Tokens exist because of commercial licensing agreements with clubs. This creates non-technical risk your code cannot see: a club can end a partnership, and a token's real-world utility can be withdrawn even though the contract keeps working perfectly.

Do not build a protocol whose solvency depends on a specific Fan Token retaining value or utility indefinitely.

---

## Standard EVM Checklist (still mandatory)

### Reentrancy
Follow **Checks-Effects-Interactions**. Update state before external calls. Add `ReentrancyGuard` on anything moving funds. Remember that any Fan Token transfer is an external call to a contract you do not control.

### Access control
Every privileged function needs a modifier. `Ownable` for single admin, `AccessControl` for roles. Audit for functions that are `public` but should be `internal`, and initializers that can be called twice.

### ERC-20 return values
Use OpenZeppelin's `SafeERC20`. Some tokens do not return a bool from `transfer()`. Never trust a bare `token.transfer(...)` return.

### Input validation
Reject `address(0)`. Reject zero amounts. Validate array lengths match. Validate that a token address actually has code before treating it as a token.

### Upgradeability
If using proxies, prefer UUPS. Never reorder or remove storage variables. Remember `RuntimeUpgrade` exists at the system level and is separate from your own proxy.

### Randomness
`block.timestamp`, `blockhash`, and `block.difficulty` are **not random** and validators can influence them. Under PoSA with a small validator set this is a stronger objection than on Ethereum. Use **Pyth Entropy**, which is available on Chiliz, for NFT mints, games, and prize draws.

### Gas efficiency
Custom errors instead of long revert strings. Pack storage variables into 32-byte slots. Mark functions `external` where they are never called internally. Avoid unbounded loops, especially over Fan Token holder lists, which can be large.

---

## Pre-Deploy Checklist

- [ ] `decimals()` read at runtime everywhere, never hardcoded
- [ ] Tested against both an 18-decimal and a 0-decimal token
- [ ] Gas configuration respects the 2,501 gwei floor on every client and script
- [ ] No DEX spot price used as an oracle; Pyth used for USD values
- [ ] No `block.timestamp` or `blockhash` randomness; Pyth Entropy if randomness is needed
- [ ] `SafeERC20` used for all token transfers
- [ ] Checks-Effects-Interactions followed; `ReentrancyGuard` where funds move
- [ ] All privileged functions gated; ownership transferred to a Safe multisig, not an EOA
- [ ] Deployed and exercised on **Spicy Testnet** first with real user flows
- [ ] Contract verified on Chiliscan or Chiliz Block Explorer
- [ ] No private keys or API keys in the repo, git history, or deploy scripts
- [ ] Trust assumptions (PoSA validator set, bridged assets, IP licensing) documented for users

---

## Production Ownership

Use **Safe multisig** at `safe.chiliz.com` (deployed by Protofire) for any contract holding user funds or admin rights. A single EOA holding upgrade authority is not a production setup.

Chiliz has published audits by Certik and Halborn covering governance contracts, the Fan Token contract, the Chiliz Bridge, and Tokenomics. Those cover *chain* contracts, not yours. See [Security Audits](https://docs.chiliz.com/learn/about-chiliz-chain/security-audits.md).

Sources: [Write a Smart Contract](https://docs.chiliz.com/develop/basics/write-a-smart-contract.md), [Developers FAQ](https://docs.chiliz.com/develop/developers-faq.md), [Generate random numbers onchain](https://docs.chiliz.com/develop/advanced/generate-random-numbers-on-chain.md), [About Validators](https://docs.chiliz.com/learn/about-validators.md), [Outage reports](https://docs.chiliz.com/chiliz-chain-changelog/outage-reports.md)
