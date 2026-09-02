---
name: chiliz-layerzero-bridging
description: Use when building or reviewing omnichain Fan Token transfers, LayerZero OFT integrations, peer configuration, or bridging between Chiliz Chain, Base, and Solana.
---

# LayerZero and Bridging on Chiliz

Use this skill for omnichain Fan Token work. Do not treat LayerZero endpoint IDs as EVM chain IDs, and do not describe bridged representations as interchangeable without checking their contracts.

## Network identifiers

- Chiliz Chain EVM chain ID: `88888`.
- Spicy Testnet EVM chain ID: `88882`.
- Chiliz Mainnet LayerZero endpoint ID: `30409`.
- Spicy Testnet LayerZero endpoint ID: `40440`.

The LayerZero `eid` is not the EVM `chainId`. Store them as separate typed configuration values and test both directions of every route.

## Before wiring a route

1. Source EndpointV2, message library, executor, and DVN addresses from the current deployment documentation or `addresses/SKILL.md`.
2. Confirm the token contract is the intended V2 contract and read `decimals()` and `symbol()` onchain.
3. Configure and verify peers on both source and destination contracts; a one-sided peer is not a working route.
4. Quote native and token fees from the LayerZero endpoint. Never hardcode an Ethereum gas assumption.
5. Test failed messages, retries, and destination-side receipt before showing a transfer as complete.

## Safety rules

- Do not invent peer addresses, endpoint IDs, DVNs, or message-library addresses.
- Treat bridge delivery as asynchronous. Keep the source transaction hash and expose destination status separately.
- Validate recipient, amount, source chain, destination chain, and minimum received values before sending.
- Do not assume a bridged stablecoin is native or fungible with another wrapper.
- Use a multisig for peer and route administration; document how users exit if a route is paused.

## Verify

```bash
cast chain-id --rpc-url https://rpc.ankr.com/chiliz
cast call <TOKEN> "decimals()(uint8)" --rpc-url https://rpc.ankr.com/chiliz
cast code <ENDPOINT_OR_PEER> --rpc-url https://rpc.ankr.com/chiliz
```

Sources: [LayerZero Chiliz deployments](https://docs.layerzero.network/v2/deployments/chains/chiliz), [Chiliz contract addresses](../addresses/SKILL.md), [Chiliz Fan Tokens](https://docs.chiliz.com/learn/about-fan-tokens.md)
