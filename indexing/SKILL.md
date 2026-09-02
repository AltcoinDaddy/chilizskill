---
name: chiliz-indexing
description: Use when reading Chiliz transfers, holders, events, transaction history, or activity feeds for a dApp, wallet, or Fan Token product.
---

# Indexing Chiliz Data

Do not scan the chain from a browser or loop through every block for user-facing history. RPC calls are useful for current state; an indexer is needed for historical queries and feeds.

## Choose the data path

- Use direct RPC reads for current balances, allowances, metadata, and contract state.
- Use event indexing for transfers, votes, stakes, swaps, and application activity.
- Use The Graph, Goldsky, or Envio when you need queryable historical entities and filtering.
- Use a hosted wallet-data API such as Moralis, Tatum, or Nodit only after checking coverage, freshness, limits, and chain support.

## Indexer design

- Key entities by chain ID and contract address; symbol and name are not stable identifiers.
- Store block number, block hash, transaction hash, log index, and timestamp for every event.
- Handle reorgs and provider retries idempotently. A `(txHash, logIndex)` natural key prevents duplicate events.
- Decode Fan Token amounts with the token's runtime decimals; never apply one global decimal setting.
- Paginate all history and expose an indexer lag or partial-data state in the UI.

## Browser and API rules

- Do not expose provider secrets in client-side code.
- Cache immutable metadata, but refresh balances and allowance state before a transaction.
- Link activity items to the correct explorer for Mainnet or Spicy.
- Clearly label indexed data as pending or stale when the head has not caught up.

## Verify

```bash
cast block-number --rpc-url https://rpc.ankr.com/chiliz
cast logs --rpc-url https://rpc.ankr.com/chiliz --from-block <START> --to-block <END>
```

Sources: [Chiliz RPC documentation](https://docs.chiliz.com/develop/basics/connect-to-chiliz-chain/connect-using-rpc.md), [The Graph](https://thegraph.com/docs/en/), [Goldsky](https://docs.goldsky.com/), [Envio](https://docs.envio.dev/)
