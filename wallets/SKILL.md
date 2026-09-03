---
name: chiliz-wallets
description: Use when connecting a Chiliz dApp to Socios Wallet or another EVM wallet, switching between Chiliz Mainnet and Spicy, handling approvals and transactions, or designing fan-friendly wallet UX.
---

# Wallets and UX on Chiliz

Chiliz users are often football fans using the Socios.com Wallet, not only MetaMask or a browser extension. Make wallet connection approachable and keep the crypto mechanics visible only when they matter.

## Connection

- Support Socios Wallet through Reown AppKit (formerly WalletConnect); do not ship a MetaMask-only connect flow.
- Consider social-login providers when the product audience should not manage seed phrases directly.
- Show the connected address in truncated form with copy and explorer links.
- Never request a signature or transaction before explaining what it does and which network it uses.

## Networks

- Chiliz Chain Mainnet: chain ID `88888`, native currency CHZ.
- Chiliz Spicy Testnet: chain ID `88882`, native currency testCHZ.
- Detect the active chain and offer a one-click add/switch flow. Make Mainnet and Spicy visually unambiguous.
- Do not configure the canonical Multicall3 address on Spicy; Chiliz Multicall is Mainnet-only at the address in `addresses/SKILL.md`.

## Transaction flow

Present a transaction as a sequence:

1. Connect wallet.
2. Switch network if needed.
3. Approve only when the current allowance is insufficient.
4. Execute the action.
5. Show pending, confirmed, reverted, rejected, and delayed states with an explorer link.

Every onchain button needs its own loading and disabled state. Disable on the first click to prevent double submission. Do not leave a spinner running indefinitely.

## Chiliz-specific details

- Fetch fee data and respect the current gas floor; never hardcode an Ethereum-shaped gas price.
- Read Fan Token `decimals()` at runtime and format fractional V2 balances correctly.
- Show fiat values next to Fan Token amounts when useful, sourced from Pyth rather than a thin DEX spot price.
- Explain that bridged stablecoins are bridge-issued representations, not automatically native or fungible with one another.
- Keep wallet recovery and account-abstraction choices separate from the ordinary connect flow.

## Verify

```bash
cast chain-id --rpc-url https://rpc.ankr.com/chiliz
cast chain-id --rpc-url https://spicy-rpc.chiliz.com/
cast call <TOKEN> "decimals()(uint8)" --rpc-url https://rpc.ankr.com/chiliz
```

Sources: [Integrate Socios.com Wallet](https://docs.chiliz.com/develop/advanced/integrate-socios-wallet-in-your-dapp.md), [Reown on Chiliz](https://docs.chiliz.com/community/chiliz-chain-ecosystem/developer-tools/reown.md), [Chiliz RPC configuration](https://docs.chiliz.com/develop/basics/connect-to-chiliz-chain/connect-using-rpc.md), [Frontend rules](../frontend/SKILL.md)
