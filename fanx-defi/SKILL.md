---
name: chiliz-fanx-defi
description: Use when integrating FanX liquidity, swaps, pools, or DeFi products on Chiliz Chain, especially when pricing Fan Tokens or designing around thin liquidity.
---

# FanX and DeFi on Chiliz

FanX is the main Fan Token liquidity venue on Chiliz. Use it for venue-specific integration questions, but do not make a pool spot price the security boundary for lending, liquidations, minting, or accounting.

## Integration rules

- Source the current FanX router, factory, token, and pool addresses from official documentation or the live deployment; never rely on memory.
- Read token `decimals()` at runtime. Fan Token V2 contracts use 18 decimals, while legacy contracts can still exist with 0 decimals.
- Use `SafeERC20`, check allowances before showing approval UI, and separate approve and swap actions.
- Set slippage and minimum-output protections from the user's quote, not from a hardcoded percentage hidden in the contract.
- Display the route, token symbols, amounts, fees, and deadline before signing.

## Price and liquidity risk

Fan Token pools are thin and sports events can move many tokens at once. A small trade can move the pool price materially.

- Never use a FanX spot price as a USD oracle.
- Use Pyth for supported Fan Token and CHZ USD feeds; define what happens when a feed is stale or unavailable.
- If pool data is unavoidable, use a long TWAP, cap update movement, and model event-driven correlated volatility.
- Do not assume two bridged versions of a stablecoin are 1:1 fungible.

## Verify

```bash
cast call <TOKEN> "symbol()(string)" --rpc-url https://rpc.ankr.com/chiliz
cast call <TOKEN> "decimals()(uint8)" --rpc-url https://rpc.ankr.com/chiliz
cast code <ROUTER> --rpc-url https://rpc.ankr.com/chiliz
```

Sources: [Chiliz Fan Tokens](https://docs.chiliz.com/learn/about-fan-tokens.md), [FanX documentation](https://fanx.gitbook.io/fanx-docs), [Chiliz Fan Token pricing](https://docs.chiliz.com/develop/advanced/get-the-usdusd-price-of-fan-tokens.md), [Security](../security/SKILL.md)
