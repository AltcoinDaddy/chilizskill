---
name: chiliz-testing-qa
description: Use when testing or reviewing a Chiliz contract or dApp before deployment, including Spicy flows, decimal migration behavior, gas configuration, address checks, and frontend transaction UX.
---

# Testing and QA on Chiliz

Test on Spicy first, then repeat the production-critical checks against Mainnet configuration. A passing generic EVM test suite does not prove a Chiliz integration is safe.

## Required test matrix

- Mainnet chain ID `88888` and Spicy chain ID `88882` are configured separately.
- Test at least one 18-decimal Fan Token and one legacy 0-decimal token wherever token amounts are handled.
- Confirm every configured address has bytecode and that its `symbol()`, `name()`, and `decimals()` match expectations.
- Exercise gas estimation with the Chiliz minimum fee of 2,501 gwei and a 1 gwei priority fee; verify underpriced transactions produce a clear recovery state.
- Test wrong-network detection, network switching, wallet rejection, reverted transactions, delayed confirmation, and retry.
- Test approve-then-execute, already-approved, double-click, insufficient balance, and exact-allowance paths.
- Verify transaction hashes link to the correct Chiliscan or Spicy explorer.

## Contract QA

- Use `SafeERC20`, access control, checks-effects-interactions, and reentrancy protection where funds move.
- Test oracle staleness, price movement caps, and event-driven Fan Token volatility.
- Test proxy upgrades and storage compatibility if upgradeability is used.
- Test randomness with a supported verifiable source; never use timestamps or block hashes as randomness.
- Run unit, fuzz, fork, and invariant tests in proportion to the value and complexity of the system.

## Release gate

Deploy and exercise the real user flow on Spicy, verify source code, review the final addresses and chain config, then use a multisig for production administration. Record the exact commit and deployment configuration.

```bash
cast chain-id --rpc-url https://spicy-rpc.chiliz.com/
cast gas-price --rpc-url https://spicy-rpc.chiliz.com/
cast code <ADDRESS> --rpc-url https://spicy-rpc.chiliz.com/
```

Sources: [Chiliz developer toolbox](https://docs.chiliz.com/quick-start/developer-toolbox), [Spicy faucets](https://docs.chiliz.com/develop/basics/testnet-faucets), [Chiliz frontend checklist](../frontend/SKILL.md), [Chiliz security checklist](../security/SKILL.md)
