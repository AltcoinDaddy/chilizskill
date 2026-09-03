---
name: chiliz-tools
description: "Development tooling that actually works on Chiliz Chain: Hardhat, Remix, Foundry, thirdweb, contract verification on Chiliscan and Blockscout, Pyth oracles and entropy, LayerZero OFT, Biconomy account abstraction, Moralis Tatum Nodit data APIs, The Graph Goldsky Envio indexing, Reown wallet connection, Aragon DAOs. Use when choosing or configuring tooling for a Chiliz Chain project."
---

# Chiliz Chain Tooling

What is actually integrated, versus what you assume works because the chain is EVM-compatible.

---

## Development Frameworks

| Tool | Status |
|---|---|
| **Hardhat** | Documented, first-class. The default recommendation. |
| **Remix** | Documented, first-class. Fastest path for a single contract. |
| **thirdweb** | Documented, supports both 88888 and 88882. Dashboard and CLI deploys, strong for NFTs. |
| **Foundry** | Works (EVM is EVM), but not in the official docs. Set `evm_version` explicitly or you will hit unsupported opcodes. |

Configure the compiler target carefully, and always set it explicitly. The current docs specify Solidity 0.8.30 or lower with EVM target `prague`. Leaving `evmVersion` unset is the trap: solc picks its own newest default, emits an opcode the chain does not implement, and the deployment can revert with `invalid opcode`.

---

## Contract Verification (read before you deploy)

There are **two** explorers and they are not interchangeable.

| Explorer | Backend | URL |
|---|---|---|
| Chiliscan | Routescan | https://chiliscan.com |
| Chiliz Block Explorer | Blockscout | https://scan.chiliz.com |

Each has its own API, its own API key, and its own Hardhat verify configuration. A key for one does not work on the other.

**Known limitation:** per the official Developers FAQ, there is currently no reliable direct method to verify contracts on **Spicy Testnet** via Hardhat. The documented workaround is to flatten your contract and verify through the explorer web form.

When verifying, match your build exactly: compiler version, EVM version, and optimizer settings. A mismatch on EVM target is the most common verification failure on Chiliz because the default in most tooling is not what Chiliz expects.

```bash
# Flatten for manual verification
npx hardhat flatten contracts/MyContract.sol > flat.sol
# Then strip duplicate SPDX and pragma lines before pasting
```

---

## Oracles and Randomness

**Pyth** is the oracle on Chiliz.
- 420+ price feeds including $CHZ and Fan Token tickers ($PSG, $BAR, $CITY, $ACM).
- **Pyth Entropy** provides secure onchain randomness. Use it for NFT mints, game mechanics, and prize draws. Do not use `block.timestamp` or `blockhash`.

**Blocknative** Gas Network provides gas estimation, `chainid=88888`.

---

## Cross-Chain

**LayerZero** is the omnichain layer. Fan Tokens went omnichain via the OFT standard in Q1 2026, existing natively on Chiliz Chain, Base, and Solana.
- Chiliz Mainnet `eid` **30409**, Spicy `eid` **40440**. These are LayerZero endpoint IDs, not EVM chain IDs.
- OFT Adapter pattern for existing tokens, Native OFT for new ones.
- Documented guides exist for Base to/from Chiliz and Solana to/from Chiliz.

**Hyperlane** is also integrated for interchain messaging and bridging.

**Bridges:** Chainport (ERC-20 CHZ and stablecoins from Ethereum/Polygon), Chiliz Bridge (native CHZ from Ethereum, plus omnichain via LayerZero for Base and Solana), Stargate (80+ chains via LayerZero OFT).

---

## Account Abstraction and Wallets

**Biconomy**: account abstraction on Chiliz. Gasless transactions, batched transactions, Smart Accounts. Verify current chain support in Biconomy's own docs before committing.

**Reown** (formerly WalletConnect): AppKit and WalletKit. This is how you get the **Socios.com Wallet** into your connect modal. On a SportFi app that matters more than MetaMask support.

**Web3Auth**: OAuth social login and embedded wallets. Useful when your users are sports fans, not crypto natives.

**Magic** and **Privy**: wallet-as-a-service and embedded wallets.

**Safe**: multisig at `safe.chiliz.com`, deployed by Protofire. Use for production treasuries and admin rights.

**Custody:** Cobo and Fireblocks both support Chiliz for institutional MPC and multisig.

---

## Data and Indexing

You cannot loop through blocks to read history. Use an indexer.

| Tool | Use for |
|---|---|
| **The Graph** | Subgraphs, GraphQL. Documented for tracking Fan Token transfers. |
| **Goldsky** | Hosted subgraphs, Graph-compatible, real-time streaming. |
| **Envio** | Fast indexer. Documented example: a Telegram whale-watcher bot for Fan Token transfers. |
| **Moralis** | Wallet balances, transaction history, NFT metadata via REST. |
| **Tatum** | Unified SDK, RPC access, token/NFT APIs, plus a testnet faucet. |
| **Nodit** | RPC and data APIs. |
| **Chiliz Chain API** | Official REST at `cc-api.chiliz.com`: CHZ circulating supply by block, staking data by address. OpenAPI v3 spec available. |

Design your contracts **event-first**. Emit an event for every state change worth reading. Retrofitting indexing onto a contract that emits nothing is not possible without a redeploy.

---

## NFTs

**thirdweb** for no-code or SDK deploys with built-in IPFS upload. **Rarible** Multichain SDK for minting and sell orders. **viem** or **ethers** for direct minting against your own ERC-721/ERC-1155.

**OKX NFT Marketplace** has native Chiliz Chain support and is the documented listing venue. Contract must be verified.

Host metadata on **IPFS** via Pinata or similar. Do not point `tokenURI` at your own server; that is a rug waiting to happen when the domain lapses.

---

## DEX and Liquidity

**FanX** (formerly Kayen) is the dominant DEX on Chiliz Chain. This is the most commonly wrong assumption an agent makes: it is not Uniswap.

- Canonical Fan Token list and wrapped variants: https://fanx.gitbook.io/fanx-docs/contract/tokens-in-fanx
- FanX contracts: https://fanx.gitbook.io/fanx-docs/contract/contracts
- Trading UI: https://app.fanx.xyz/trade

**Azuro** provides onchain prediction and sports betting infrastructure on Chiliz, which is a natural fit for the chain's use case.

---

## DAOs

**Aragon** is integrated for no-code DAO creation on Chiliz. This is the documented path for Fan Token community governance: fan councils, stadium experience voting, treasury governance.

---

## Ecosystem Support

**Chiliz Greenhouse** offers micro-grants, technical bounties, and mentorship for builders, plus a showcase hub for SportFi projects. Worth mentioning to anyone building seriously.

Community: Discord at `discord.gg/chiliz`, Telegram at `t.me/BuildOnChiliz`.

---

## Querying the Docs Programmatically

The Chiliz docs are built for agents. Every page answers natural language questions:

```
GET https://docs.chiliz.com/<page>.md?ask=<question>&goal=<end goal>
```

Full index: https://docs.chiliz.com/llms.txt
Full corpus: https://docs.chiliz.com/llms-full.txt (expensive, use the index first)

Prefer `.md` URLs for clean structured content.

Sources: [Developer Tools](https://docs.chiliz.com/community/chiliz-chain-ecosystem/developer-tools.md), [Verify a Smart Contract](https://docs.chiliz.com/develop/basics/verify-a-smart-contract.md), [Developers FAQ](https://docs.chiliz.com/develop/developers-faq.md), [Use Omnichain Tokens](https://docs.chiliz.com/develop/advanced/use-omnichain-tokens.md), [Work with NFTs](https://docs.chiliz.com/develop/advanced/work-with-nfts.md)
