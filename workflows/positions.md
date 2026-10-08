# Positions workflow

Use this workflow when the user wants to see their SuperVault positions, pending withdrawals, or claimable rewards.

Read [positions.md](../references/positions.md) and [rewards.md](../references/rewards.md) before you start.

Run [preflight.md](preflight.md) first if it hasn't passed in this session.

## Flow

1. Get positions.
2. Check pending withdrawals.
3. Check rewards.
4. Report.

## Get positions

1. Get the wallet address with `mm wallet address --toon`.
2. Read the balances API and keep entries with a `vault_id`. See [positions.md](../references/positions.md#balances-api).
3. Match each entry against the vaults API to get the vault's chain, APY split, and reward tokens. Mark entries that aren't SuperVaults, such as indexed Morpho vaults, as external positions.

## Check pending withdrawals

For each SuperVault position with `status` set to `pending`, read the redemption state onchain. See [onchain-reads.md](../references/onchain-reads.md#redemption-state).

- Pending: show the shares waiting and when to check back.
- Fulfilled: tell the user they can claim now, and offer the claim step in [withdraw.md](withdraw.md#claim-the-assets).

## Check rewards

1. Read claimable rewards. See [rewards.md](../references/rewards.md#check-claimable-rewards).
2. Convert `amount` to human units with each token's `decimals`.

## Claim rewards

Run this only when the user asks to claim.

1. Compare the claimable `value_usd` with the gas cost on the reward chain. If the rewards are worth less than gas, tell the user and stop.
2. Build the `CLAIM` request. See [rewards.md](../references/rewards.md#claim).
3. Follow [signing.md](../references/signing.md). Confirm the recipient is the user's wallet.
4. After confirmation, read the reward token balance to confirm receipt.

## Report

Show a table with one row per position:

| Column | Source |
| --- | --- |
| Vault and chain | Vaults API |
| Shares and underlying amount | Balances API or onchain reads |
| USD value | `value_usd` |
| APY split | `apy_snapshot_week` and `reward_rate` |
| Status | Active, pending withdrawal, or ready to claim |

Then show total value, claimable rewards per token with USD value, and any action the user can take next.
