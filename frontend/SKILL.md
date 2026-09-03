---
name: chiliz-frontend
description: "Frontend and UX rules for Chiliz Chain dApps: Socios.com Wallet integration via Reown, wallet connection, network switching for chain 88888 and 88882, gas price handling in the browser, Fan Token decimals in the UI, USD values from Pyth, approve/execute flows, and a pre-publish checklist. Use when building or reviewing the UI of a Chiliz dApp."
---

# Chiliz dApp Frontend

The mistakes here are different from Ethereum because the users are different. Your user is a football fan who downloaded Socios, not a DeFi native.

---

## Wallet Connection: Socios First

**The single biggest frontend mistake on Chiliz is supporting only MetaMask.**

Your users hold Fan Tokens in the **Socios.com Wallet**. If your connect modal does not offer it, most of your addressable audience cannot use your app.

Wire it through **Reown AppKit** (formerly WalletConnect):

```typescript
import { createAppKit } from '@reown/appkit';

const chiliz = {
  id: 88888,
  name: 'Chiliz Chain',
  nativeCurrency: { name: 'Chiliz', symbol: 'CHZ', decimals: 18 },
  rpcUrls: { default: { http: ['https://rpc.ankr.com/chiliz'] } },
  blockExplorers: { default: { name: 'Chiliscan', url: 'https://chiliscan.com' } },
};

const spicy = {
  id: 88882,
  name: 'Chiliz Spicy Testnet',
  nativeCurrency: { name: 'Chiliz', symbol: 'CHZ', decimals: 18 },
  rpcUrls: { default: { http: ['https://spicy-rpc.chiliz.com/'] } },
  blockExplorers: { default: { name: 'Spicy Chiliscan', url: 'https://testnet.chiliscan.com' } },
  testnet: true,
};
```

Also consider **Web3Auth**, **Magic**, or **Privy** for social login. Fans should not need to understand seed phrases to vote on a jersey design.

Do not configure `multicall3` for chain 88882. Multicall on Chiliz is Mainnet only, at a non-standard address. Libraries that auto-apply the canonical `0xcA11bde...` will fail silently or loudly. See [addresses/SKILL.md](../addresses/SKILL.md).

---

## Gas in the Browser

Wallets will happily submit a transaction priced below the network floor, and it will sit pending forever while the user stares at a spinner.

```typescript
// Fetch real fee data. Never ship an Ethereum-shaped constant.
const fee = await publicClient.estimateFeesPerGas();

// Chiliz floor: 2,501 gwei base, 1 gwei priority.
const FLOOR = 2501n * 10n ** 9n;
const maxFeePerGas = fee.maxFeePerGas && fee.maxFeePerGas > FLOOR
  ? fee.maxFeePerGas
  : FLOOR;
```

Then handle the stuck case in the UI. If a transaction has not confirmed after a reasonable window, tell the user what is happening and offer a retry with higher gas. Do not leave a spinner running indefinitely.

---

## Fan Token Decimals in the UI

Read `decimals()` per token, cache it per address, and format from that value. Never hardcode.

Since the 2026 migration Fan Tokens are 18 decimals, which means **fractional balances are now possible**. If your UI was written against the old 0-decimal world it will show `1` where the user holds `1.4`, or round away real value. Format with sensible precision, typically 2 to 4 decimal places for display, while keeping full precision in the underlying math.

```typescript
const decimals = await readContract({ address: token, abi: erc20Abi, functionName: 'decimals' });
const display = formatUnits(rawBalance, decimals);
```

---

## Show USD Values

Sports fans do not price things in CHZ. Show a fiat value next to every token amount, sourced from **Pyth** (420+ feeds including $CHZ, $PSG, $BAR, $CITY).

Never derive the displayed price from a FanX pool spot read. Thin pools mean a bad tick shows your user a wildly wrong number.

Some of your audience is Brazilian, where BRZ is pegged to the Real, not the dollar. If you localize currency, do it properly rather than labelling everything USD.

---

## Transaction UX Rules

**Every onchain button gets its own loading state.** A single shared `isLoading` disables the whole page and hides which action is actually running.

**Approve and execute are separate steps.** Show them as a sequence, one active at a time:

1. Switch Network (only if the user is on the wrong chain)
2. Approve (only if allowance is insufficient)
3. Execute

Check the existing allowance before showing an approve button. Making a user approve something they already approved is the most common wasted click in dApps, and on Chiliz it is a wasted transaction fee too.

**Guard against double-fire.** Disable the button on the first click, not after the transaction resolves. Approve buttons that fire twice are a classic bug in AI-generated frontends.

**Link out to the explorer** with every transaction hash so the user can see what happened: `https://chiliscan.com/tx/<hash>`.

**Show addresses properly.** Truncate with a copy button and a link out. Never render a raw 42-character string in the middle of a sentence.

---

## Network Switching

Detect the wrong chain and offer a one-click switch. Do not just error out.

```typescript
if (chainId !== 88888) {
  // Prompt switch, and if the chain is unknown to the wallet, add it
  await walletClient.addChain({ chain: chiliz });
  await walletClient.switchChain({ id: 88888 });
}
```

Make it obvious in the UI whether the user is on Mainnet or Spicy. Shipping a testnet-pointed frontend to production is common and embarrassing.

---

## Reading History

You cannot loop through blocks in the browser. If your app shows transfer history, holder lists, or activity feeds, you need an indexer: The Graph, Goldsky, or Envio. Moralis and Tatum offer REST APIs for wallet balances, transaction history, and NFT metadata if you want to skip running a subgraph.

---

## Pre-Publish Checklist

- [ ] Socios.com Wallet available in the connect modal, not MetaMask only
- [ ] Contract addresses point at **Mainnet**, not leftover Spicy addresses
- [ ] Chain ID 88888 configured; Spicy clearly separated
- [ ] `decimals()` read at runtime for every token displayed
- [ ] Fractional Fan Token balances render correctly
- [ ] USD values shown, sourced from Pyth
- [ ] Gas floor of 2,501 gwei respected; stuck-transaction state handled in the UI
- [ ] Every onchain button has its own loader and disabled state
- [ ] Allowance checked before showing approve; approve cannot double-fire
- [ ] Transaction hashes link to Chiliscan
- [ ] No template or starter-kit branding left anywhere
- [ ] Mobile tested. Fans are on phones, not desktops.
- [ ] No API keys or private keys in client-side code or the repo

---

## One Framing Rule

Write copy for fans, not for traders. "Support your club" lands better than "stake your assets to earn yield." The people using this app care about the team, and the crypto is the mechanism, not the point.

Sources: [Integrate Socios.com Wallet](https://docs.chiliz.com/develop/advanced/integrate-socios.com-wallet-in-your-dapp.md), [Reown](https://docs.chiliz.com/community/chiliz-chain-ecosystem/developer-tools/reown.md), [Get the $USD price of Fan Tokens](https://docs.chiliz.com/develop/advanced/get-the-usdusd-price-of-fan-tokens.md), [Connect using RPC](https://docs.chiliz.com/develop/basics/connect-to-chiliz-chain/connect-using-rpc.md)
