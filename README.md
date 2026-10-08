# Superform SuperVaults skill

An agent skill for working with [Superform](https://superform.xyz) SuperVaults through the [MetaMask Agent Wallet](https://www.npmjs.com/package/@metamask/agent-wallet).

The skill uses Superform's public APIs, which need no API key. It fetches vault data and unsigned transactions with `curl`, and signs and broadcasts only through the `mm` CLI. It ships no scripts.

## What you can do

- Browse vaults with APY split into strategy yield and rewards, TVL, availability, and timelock.
- Deposit into a vault, including swapping or bridging into the deposit token first.
- Withdraw with ERC-7540 async redemption: request, wait for the keeper, then claim.
- Check positions, pending withdrawals, and claimable Merkl rewards.
- Claim rewards when they're worth more than gas.

Every transaction is decoded with `mm decode`, checked against the expected contract, and simulated where possible before it's sent.

## Supported chains

| Chain | Chain ID | Status |
| --- | --- | --- |
| Base | `8453` | Supported |
| Ethereum | `1` | Supported |
| Robinhood Chain | `4663` | Supported |
| BNB Chain | `56` | Supported, no listed vaults |
| Flare | `14` | Not supported by the agent wallet |

## Prerequisites

- Node.js 22.18 or later.
- `curl` and `jq`.
- An agent that supports skills, such as Claude Code.

You don't need to set up the agent wallet first. On the first wallet action, the skill runs a preflight that:

1. Checks for the [MetaMask Agent Wallet CLI](https://www.npmjs.com/package/@metamask/agent-wallet) and offers to install it.
2. Checks for the `metamask-agent-wallet` skill and offers to install it with `npx skills add metaMask/agent-skills`.
3. Runs `mm doctor`, then walks you through login and `mm init` if needed.

It asks before installing anything.

## Install

Install with the [skills CLI](https://www.npmjs.com/package/skills):

```bash
npx skills add AyushBherwani1998/superform-skills
```

## Structure

```text
SKILL.md                 Index, routing table, and safety rules
references/
  vaults.md              Vault discovery, fields, and APY
  chains.md              Supported chains, RPCs, and explorers
  bundler.md             Transaction builder requests and expected executions
  onchain-reads.md       Share balance and redemption state reads
  positions.md           Balances API
  rewards.md             Merkl rewards and claims
  signing.md             Verify, decode, simulate, and send
  errors.md              Bundler, onchain, and wallet errors
workflows/
  preflight.md           Install checks, login, and initialization
  deposit.md             Deposit into a vault
  withdraw.md            Request a redemption and claim it
  positions.md           Positions, pending withdrawals, and rewards
```

## Example prompts

- "List Superform vaults on Base with APY and TVL."
- "Deposit 10 USDC worth of TSLAc into superTSLA."
- "Show my Superform positions."
- "Withdraw my superTSLA position."
- "Is my superTSLA withdrawal ready to claim?"
- "Claim my Superform rewards."


## References

- [Superform documentation](https://docs.superform.xyz/llms.txt)
- [Superform deployment addresses](https://docs.superform.xyz/deployment-addresses)
- [MetaMask Agent Wallet skill](https://github.com/MetaMask/agent-skills)
