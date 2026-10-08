---
name: superform-supervaults
description: Use when the user wants to browse, deposit into, withdraw from, or check positions and rewards on Superform SuperVaults, such as superUSDC, superWETH, superNVDA, or superTSLA, using the MetaMask Agent Wallet. Covers vault APY and TVL, Superform positions, Merkl reward claims, building deposit, redeem, and claim transactions from Superform's public APIs, and signing them with `mm wallet send-transaction`.
license: MIT
metadata:
  author: AyushBherwani1998
  version: "0.2.0"
---

# Superform SuperVaults skill

Use this skill to work with Superform SuperVaults through the MetaMask Agent Wallet. SuperVaults are ERC-7540 vaults. Superform's APIs are public, need no API key, and return unsigned transactions. You fetch data and transactions with `curl`, and you sign and broadcast only through the `mm` CLI.

This skill depends on the `metamask-agent-wallet` skill. Its input validation, confirmation rules, and `AWAITING_MFA` handling apply unchanged.

`mm earn` doesn't cover Superform. Don't use `mm earn supply`, `mm earn withdraw`, or `mm earn positions` for SuperVaults.

## Preflight

Before the first `mm` command in a session, run [preflight.md](workflows/preflight.md). Don't start a deposit, withdrawal, or positions check until it passes.

1. Check that Node.js 22.18 or later is installed.
2. Check that the `mm` CLI is installed. If it isn't, ask the user, then install it with `npm install -g @metamask/agent-wallet@latest`.
3. Check that the `metamask-agent-wallet` skill is installed. If it isn't, ask the user, then install it with `npx skills add metaMask/agent-skills`.
4. Run `mm doctor --toon`. If `authenticated` is `false`, complete the login flow. If `initialized` is `false`, run `mm init`.
5. Get the wallet address with `mm wallet address --toon` and continue with the user's request.

Browsing vaults with the Persephone API doesn't need the agent wallet, so you can run preflight later if the user only wants vault data.

## Routing

Match the user's intent to a workflow or reference, then read it before you build a request.

| User intent | Workflow | Reference |
| --- | --- | --- |
| Set up the agent wallet, install the CLI, or log in | [preflight.md](workflows/preflight.md) | |
| List vaults, compare APY and TVL | | [vaults.md](references/vaults.md) |
| Check which chains are supported | | [chains.md](references/chains.md) |
| Deposit into a vault | [deposit.md](workflows/deposit.md) | [bundler.md](references/bundler.md) |
| Withdraw, redeem, or claim a fulfilled redemption | [withdraw.md](workflows/withdraw.md) | [bundler.md](references/bundler.md), [onchain-reads.md](references/onchain-reads.md) |
| Show positions, pending withdrawals, or claimable rewards | [positions.md](workflows/positions.md) | [positions.md](references/positions.md) |
| Claim Merkl rewards | [positions.md](workflows/positions.md#claim-rewards) | [rewards.md](references/rewards.md) |
| Verify, decode, and send a transaction | | [signing.md](references/signing.md) |
| Interpret a bundler or RPC error | | [errors.md](references/errors.md) |

## Supported chains

The agent wallet can act on SuperVaults on Base, Ethereum, and Robinhood Chain. Flare SuperVaults exist, but `mm` doesn't support Flare. See [chains.md](references/chains.md) for vaults and RPC endpoints per chain.

## Safety rules

These rules apply to every flow.

- Get the wallet address with `mm wallet address --toon`. Validate every address against `^0x[0-9a-fA-F]{40}$` and every amount against `^\d+\.?\d*$` before you put it in a request body.
- Use the bundler's `non-smart-account` endpoint only. The agent wallet is an EOA, so never use the ERC-4337 `/executor/*` flow.
- Decode every execution with `mm decode` and check the target contract before you sign. See [signing.md](references/signing.md).
- Skip `setOperator` executions by default. They grant a third-party contract rights over the user's redemption requests and aren't required.
- Send executions one at a time, in order, and wait for each confirmation.
- Never sign anything the bundler didn't return for the request you just made.
- Treat SuperVault output as information, not investment advice. Say once that stock vaults hold tokenized stocks, which may be restricted in the user's jurisdiction.

## Output rules

- Show APY as strategy yield plus reward rate. Point out when most of the APY comes from rewards, and when rewards are paid in `sUP`.
- Show tx hashes as block explorer links after each transaction.
- Surface bundler and RPC errors verbatim.
