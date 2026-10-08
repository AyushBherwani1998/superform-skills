# Positions

Read a wallet's SuperVault positions from the Persephone balances API, and confirm details onchain when you need exact amounts.

## Balances API

```bash
curl -s "https://persephone.superform.xyz/v1/balances/<acct>"
```

The response has a `balances` array and portfolio totals. The array includes plain token balances, so keep only entries with a non-empty `vault_id`.

```bash
curl -s "https://persephone.superform.xyz/v1/balances/<acct>" \
  | jq '[.balances[] | select(.vault_id != null and .vault_id != "")]'
```

Some entries are non-Superform vaults that Superform indexes, such as Morpho vaults. Treat a position as a SuperVault only if its address appears in the vaults API. See [vaults.md](vaults.md).

## Fields

| Field | Use |
| --- | --- |
| `vault_id` | `<chain_id>_<vault>` |
| `symbol`, `name` | Share token symbol and name |
| `amount`, `decimals` | Raw share balance and share decimals |
| `underlying_amount` | Raw value in the deposit token |
| `value_usd` | USD value |
| `status` | `available` for an active position, `pending` for a redemption in progress |
| `action` | `withdrawal-request` when a redemption is pending |
| `total_vault_share_value_usd` | Total across vault positions |
| `avg_vault_share_apy` | Weighted APY across positions |

The balances API can lag behind onchain state. For withdrawals, read the redemption state onchain. See [onchain-reads.md](onchain-reads.md).

## Underlying value

Underlying value equals shares multiplied by `stats_basic.pps` from the vaults API. Use the deposit token decimals for display.
