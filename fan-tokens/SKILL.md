---
name: chiliz-fan-tokens
description: CAP-20 Fan Token standard on Chiliz Chain, the 2026 migration from 0 decimals to 18 decimals, V2 contract addresses, Fan Token staking, wCHZ, and pricing Fan Tokens in USD via Pyth. Use whenever code reads, transfers, stakes, prices, or displays a Fan Token, or whenever token decimals are involved on Chiliz Chain.
---

# Fan Tokens and CAP-20

This is the skill that prevents the most expensive Chiliz mistake. Read it before any code touches a Fan Token balance.

---

## The Decimals Trap

CAP-20 (Chiliz Advancement Proposal 20) is code-identical to ERC-20. The difference was never the interface. It was the decimals.

| Era | Decimals | Meaning |
|---|---|---|
| Launch through 2025 | **0** | `1 PSG = 1 unit`. No fractional ownership. |
| After the 2026 Decimal Migration | **18** | `1 PSG = 10^18 units`. Standard ERC-20 behavior. |

The 2026 migration completed across Q3 2025 to Q2 2026. Per Chiliz docs, as of Q2 2026 all Fan Tokens have been fully migrated to their decimal contracts.

**This creates two failure modes, in opposite directions:**

1. **Model trained pre-2026** writes `amount` as a whole number, or divides by 1. Against an 18-decimal V2 token this under-transfers by a factor of 10^18. A user asking to send 100 PSG sends 0.0000000000000001 PSG.
2. **Model assuming standard ERC-20** writes `parseUnits(amount, 18)`. Against a surviving legacy 0-decimal contract this over-transfers by 10^18, which simply reverts on insufficient balance, or worse succeeds against a contract that trusted the input.

**The rule, without exception:**

```solidity
// NEVER
uint256 amount = userAmount * 1e18;   // wrong for legacy tokens
uint256 amount = userAmount;          // wrong for V2 tokens

// ALWAYS
uint8 d = IERC20Metadata(token).decimals();
uint256 amount = userAmount * (10 ** d);
```

```typescript
// viem / ethers: read decimals at runtime, cache per address, never hardcode
const decimals = await publicClient.readContract({
  address: token, abi: erc20Abi, functionName: 'decimals'
});
const amount = parseUnits(userInput, decimals);
```

Do not write a `FAN_TOKEN_DECIMALS` constant. Do not infer decimals from the ticker. Read the contract.

---

## The Address Trap

The migration did not upgrade tokens in place. **It issued new V2 contracts at new addresses.** Holders migrated balances through `app.fantokens.com`, and centralized exchanges migrated on users' behalf.

Consequence: **every Fan Token address in a pre-2026 training set is a dead or legacy V1 contract.** If you emit a Fan Token address from memory, you are almost certainly emitting the wrong one, and on Chiliz a wrong token address means lost funds or a silently broken integration.

Verified V2 addresses for major tokens (Chiliz Chain Mainnet):

| Token | V2 Contract |
|---|---|
| $PSG (Paris Saint-Germain) | `0xFe1d4A935df7A4A52F835f6104C97AF9D72217f2` |
| $BAR (FC Barcelona) | `0x1589248b4B61ed472cc21CA1F2114d93ab6910D5` |
| $CITY (Manchester City) | `0x7Bd6242D775fAEf1d50B2aA18C2FBF329BDDF295` |
| $JUV (Juventus) | `0xeAf368DAdC22524deF47E8A1C26bFC17AC16E6F5` |
| $ACM (AC Milan) | `0x062F6004FD0BF204D272Ff115E5b84F7A01489D1` |
| $INTER (Inter Milan) | `0x1b3385A26214057bB7e27c173ee2D14201752e73` |
| $ATM (Atlético de Madrid) | `0x7da0eB973D982FFcA095E80437F5e37459a95C67` |
| $AFC (Arsenal) | `0x76088F3eD5dC655De9295D93868ec1EeC654A615` |
| $SPURS (Tottenham) | `0xd699ACD21011c20381E5138A430bb0d7b6E9BC7F` |
| $NAP (Napoli) | `0x90593E9602b38A0D5b63d9f34AC3560798cEE7d4` |
| $GAL (Galatasaray) | `0x770da1e5dDB22f3Ccc2482493BD9B10A7A8A38Ae` |
| $UFC | `0xC9f723625e80a81cBa2CAd3e6871D3bdf2a7ECC7` |
| $ARG (Argentina NT) | `0x4394886B1eec08Fe88681462914702dC99D97Eb7` |
| $POR (Portugal NT) | `0x013F2407c6eF765F1199f8818B805121F269F5b8` |
| $MENGO (Flamengo) | `0xBfF8FaBb04f6494fe393EB7416A698869569A310` |

For anything not listed, and for the Spicy Testnet equivalents, use the canonical lists rather than guessing:
- Chiliz docs: [Token Contract Addresses](https://docs.chiliz.com/quick-start/token-contract-addresses.md)
- FanX token list: https://fanx.gitbook.io/fanx-docs/contract/tokens-in-fanx
- Full migration status table: [2026 Migration to Decimal Fan Tokens](https://docs.chiliz.com/learn/about-fan-tokens/2026-migration-to-decimal-fan-tokens.md)

Always confirm before use:

```bash
cast call <ADDR> "symbol()(string)"   --rpc-url https://rpc.ankr.com/chiliz
cast call <ADDR> "decimals()(uint8)"  --rpc-url https://rpc.ankr.com/chiliz
```

---

## You Cannot Mint an Official Fan Token

Fan Tokens are licensed IP. A club signs a commercial deal with Chiliz and the token is issued through that relationship. There is no permissionless "deploy a Fan Token" path.

What you **can** do:
- Deploy your own ERC-20 or CAP-20 style token for your own project.
- Build apps that read, stake, gate on, or reward existing Fan Tokens.
- Launch liquidity for your own token on FanX.

If a user asks you to "create a Fan Token for my club," clarify this early. Building the wrong thing here wastes the whole project.

---

## Fan Token Staking

Official staking contracts on Chiliz Chain Mainnet:

| Contract | Address |
|---|---|
| Proxy | `0x5ff7f9724fd477d9a07dcdb894d0ca7f8fae1501` |
| Implementation | `0xD1bAfa7A246f5685cd22A563FD20fb164fAF0A4c` |

Holders lock Fan Tokens and accrue reward points daily from a pool. Note this is **Fan Token staking**, which is distinct from **CHZ staking** to validators. Do not conflate them in explanations or code. CHZ staking uses the system contracts at `0x...1000` and `0x...7001`.

---

## wCHZ (Wrapped CHZ)

CHZ is the native gas token, like ETH on Ethereum. It is **not** an ERC-20 and has no contract address on its own chain. When a contract needs an ERC-20 interface for CHZ, use wCHZ.

- Wrap/unwrap UI: Chiliz Bridge, and `spicy-bridge.chiliz.com/wrap` on testnet.
- Spicy wCHZ reference contract: `0x678c34581db0a7808d0aC669d7025f1408C9a3C6`
- CHZ has **18 decimals**. This never changed and is unrelated to the Fan Token migration.

If an agent tells a user "the CHZ contract address is 0x...", it is either giving the Ethereum-side bridged CHZ ERC-20, or hallucinating. Be precise about which chain and which representation.

---

## Pricing Fan Tokens in USD

Use **Pyth**. It carries 420+ feeds on Chiliz including $CHZ and Fan Token tickers such as $PSG, $BAR, $CITY, $ACM.

Do **not** derive a display price from a FanX pool spot price. Fan Token pools are thin, prices move on match results, and a spot read is trivially manipulable. Spot price is acceptable for routing a swap. It is not acceptable as an oracle input to anything that moves value.

See [Get the $USD price of Fan Tokens](https://docs.chiliz.com/develop/advanced/get-the-usdusd-price-of-fan-tokens.md).

---

## Omnichain Fan Tokens

Since Q1 2026, Fan Tokens exist natively on **Chiliz Chain, Base, and Solana simultaneously**, connected through LayerZero's OFT standard. This is lock-and-mint style native issuance, not a wrapped-asset bridge.

Practical implications:
- The "canonical" Fan Token address is chain-dependent. Always qualify which chain you mean.
- Liquidity is fragmented across three chains. Do not assume Chiliz Chain holds all of it.
- LayerZero EID for Chiliz Mainnet is `30409`, Spicy is `40440`.

---

## Checklist Before Shipping Fan Token Code

- [ ] `decimals()` read at runtime, never hardcoded
- [ ] Token addresses sourced from the docs or FanX list, verified with `symbol()`
- [ ] Legacy 0-decimal path handled or explicitly rejected with a clear error
- [ ] USD prices from Pyth, not from pool spot
- [ ] Fan Token staking not confused with CHZ validator staking
- [ ] UI displays fractional amounts correctly now that 18 decimals allow them

Sources: [About Fan Tokens](https://docs.chiliz.com/learn/about-fan-tokens.md), [2026 Migration to Decimal Fan Tokens](https://docs.chiliz.com/learn/about-fan-tokens/2026-migration-to-decimal-fan-tokens.md), [CAP-20 glossary](https://docs.chiliz.com/learn/glossary/cap-20.md), [Write a Smart Contract](https://docs.chiliz.com/develop/basics/write-a-smart-contract.md), [Smart Contract Addresses](https://docs.chiliz.com/quick-start/smart-contract-addresses.md)
