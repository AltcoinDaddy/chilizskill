---
name: chiliz-addresses
description: "Verified contract addresses on Chiliz Chain Mainnet (88888) and Spicy Testnet (88882): system contracts for staking, governance and chain config, Multicall, stablecoins USDC USDT AUSD0 BRZ, LayerZero endpoints, Fan Token staking, and Pepper. Use whenever code needs a contract address on Chiliz Chain, to avoid hallucinated addresses."
---

# Chiliz Chain Contract Addresses

**A wrong address means lost funds.** Never emit a Chiliz address from memory. Use this file, then verify onchain.

---

## System Contracts

Chiliz Chain inherits BSC's pattern of system contracts at fixed low addresses. **These are identical on Mainnet (88888) and Spicy Testnet (88882).**

| Contract | Address |
|---|---|
| Staking | `0x0000000000000000000000000000000000001000` |
| Slashing Indicator | `0x0000000000000000000000000000000000001001` |
| SystemReward | `0x0000000000000000000000000000000000001002` |
| StakingPool (user/dApp-facing wrapper) | `0x0000000000000000000000000000000000007001` |
| Governance | `0x0000000000000000000000000000000000007002` |
| ChainConfig | `0x0000000000000000000000000000000000007003` |
| RuntimeUpgrade | `0x0000000000000000000000000000000000007004` |
| DeployerProxy | `0x0000000000000000000000000000000000007005` |
| Tokenomics | `0x0000000000000000000000000000000000007006` |

For CHZ staking from a dApp, target **StakingPool** (`0x...7001`), not the raw Staking contract.

---

## Multicall: Read This One Carefully

| Contract | Address | Availability |
|---|---|---|
| Multicall | `0x0E6a1Df694c4be9BFFC4D76f2B936bB1A1df7fAC` | **Mainnet only** |

Two traps here:

1. **This is not the canonical Multicall3 address.** `0xcA11bde05977b3631167028862bE2a173976CA11` is deployed on most EVM chains and models reach for it reflexively. On Chiliz, use the address above.
2. **It does not exist on Spicy Testnet.** Any library that auto-configures Multicall by chain ID will fail on Spicy. In viem/wagmi, either disable batching for Spicy or supply the Mainnet-only config conditionally.

```typescript
// viem chain config: do not blanket-apply a multicall3 address
const chiliz = defineChain({
  id: 88888,
  name: 'Chiliz Chain',
  nativeCurrency: { name: 'Chiliz', symbol: 'CHZ', decimals: 18 },
  rpcUrls: { default: { http: ['https://rpc.ankr.com/chiliz'] } },
  blockExplorers: { default: { name: 'Chiliscan', url: 'https://chiliscan.com' } },
});
```

---

## Stablecoins

All stablecoins on Chiliz Chain are **bridged**, not natively issued. There is no Circle-native USDC here.

| Token | Address (Mainnet) | Bridge |
|---|---|---|
| USDC | `0xa37936F56249965d407E39347528a1A91eB1cbef` | Chainport |
| USDT | `0x37C57a89812a0D492AeEd7691F1610CA0a8f74A1` | Chainport |
| AUSD0 (Agora) | `0x00000000eFE302BEAA2b3e6e1b18d08D69a9012a` | Stargate |
| BRZ (Brazilian Real) | `0xE9185Ee218cae427aF7B9764A011bb89FeA761B4` | Transfero |

**Check `decimals()` on these too.** Bridged USDC and USDT commonly carry 6 decimals from their origin chain, but bridge wrappers do not always preserve the original value. Read it, do not assume 6 and do not assume 18.

BRZ is pegged to the Brazilian Real, not the dollar. Chiliz has heavy Brazilian club presence, so BRZ shows up more than you would expect. Do not treat it as a USD stablecoin in any pricing math.

---

## LayerZero (Omnichain Fan Tokens)

**Chiliz Chain Mainnet: Endpoint ID (`eid`) `30409`**

| Contract | Address |
|---|---|
| EndpointV2 | `0x6F475642a6e85809B1c36Fa62763669b1b48DD5B` |
| sendUln302 | `0xC39161c743D0307EB9BCc9FEF03eeb9Dc4802de7` |
| receiveUln302 | `0xe1844c5D63a9543023008D332Bd3d2e6f1FE1043` |
| blockedMessageLib | `0xc1ce56b2099ca68720592583c7984cab4b6d7e7a` |
| executor | `0x4208D6E27538189bB48E603D6123A94b8Abe0A0b` |
| Dead DVN | `0x6788f52439ACA6BFF597d3eeC2DC9a44B8FEE842` |

**Spicy Testnet: Endpoint ID (`eid`) `40440`**

| Contract | Address |
|---|---|
| EndpointV2 | `0x3aCAAf60502791D199a5a5F0B173D78229eBFe32` |
| sendUln302 | `0x45841dd1ca50265Da7614fC43A361e526c0e6160` |
| receiveUln302 | `0xd682ECF100f6F4284138AA925348633B0611Ae21` |
| blockedMessageLib | `0xa229b65cc2191bf60bc24efcda3487d7b5c0c9f0` |
| executor | `0x701f3927871EfcEa1235dB722f9E608aE120d243` |

LayerZero `eid` is **not** the EVM chain ID. Confusing `30409` with `88888` is a common failure when wiring OFT peers. Cross-check against https://docs.layerzero.network/v2/deployments/chains/chiliz

---

## Fan Token Staking

| Contract | Address (Mainnet) |
|---|---|
| Proxy | `0x5ff7f9724fd477d9a07dcdb894d0ca7f8fae1501` |
| Implementation | `0xD1bAfa7A246f5685cd22A563FD20fb164fAF0A4c` |

Interact through the Proxy. This is Fan Token staking, unrelated to CHZ validator staking.

---

## Pepper ($PEPPER community token)

| Contract | Address (Mainnet) |
|---|---|
| Pepper Factory | `0xb06709919e0279fC7e01bfBc4Cead2dD99F74Ca8` |
| Pepper Staking | `0x5cA4C88339D89B2547a001003Cca84F62F557A72` |

---

## wCHZ

Spicy Testnet reference contract: `0x678c34581db0a7808d0aC669d7025f1408C9a3C6`

For Mainnet wCHZ, source the address from Chiliz Bridge rather than from memory.

---

## Fan Tokens

Do not list Fan Token addresses from memory. The 2026 decimal migration reissued all of them at new V2 addresses. See [fan-tokens/SKILL.md](../fan-tokens/SKILL.md) for verified majors, and these canonical lists for everything else:

- [Token Contract Addresses](https://docs.chiliz.com/quick-start/token-contract-addresses.md): covers Chiliz Chain, Solana, and Base
- https://fanx.gitbook.io/fanx-docs/contract/tokens-in-fanx: includes wrapped variants and Spicy equivalents

---

## Verify Every Address Before Use

```bash
RPC=https://rpc.ankr.com/chiliz

# 1. Does code exist there at all? Empty output means nothing is deployed.
cast code <ADDR> --rpc-url $RPC | head -c 20

# 2. Is it the token you think it is?
cast call <ADDR> "symbol()(string)"  --rpc-url $RPC
cast call <ADDR> "name()(string)"    --rpc-url $RPC
cast call <ADDR> "decimals()(uint8)" --rpc-url $RPC
```

If `cast code` returns `0x`, you have an EOA or an unused address. Stop and re-source it.

Sources: [Smart Contract Addresses](https://docs.chiliz.com/quick-start/smart-contract-addresses.md), [Stablecoins on Chiliz Chain](https://docs.chiliz.com/learn/about-stablecoins/stablecoins-on-chiliz-chain.md), [Developers FAQ](https://docs.chiliz.com/develop/developers-faq.md)
