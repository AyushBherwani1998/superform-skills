# Vaults

Use the Persephone API to discover SuperVaults and read APY, TVL, and deposit tokens. The API is public and needs no key.

Base URL: `https://persephone.superform.xyz/v1`

## List vaults

```bash
curl -s "https://persephone.superform.xyz/v1/supervaults"
```

The response has a `supervaults` array that covers every chain. The `chain_id` query parameter doesn't filter results, so filter on each vault's `chain_id` field yourself.

Exclude vaults where `visibility` is `hidden`. These are test or deprecated deployments.

```bash
curl -s "https://persephone.superform.xyz/v1/supervaults" \
  | jq '[.supervaults[] | select(.visibility != "hidden" and .chain_id == "8453")]'
```

## Find one vault

Filter the list by address. The single-vault endpoint `GET /vaults/<chain_id>_<address>` can return empty `assets`, so don't rely on it for the deposit token.

```bash
curl -s "https://persephone.superform.xyz/v1/supervaults" \
  | jq --arg v "<vault>" '.supervaults[] | select(.address == $v)'
```

## Historical APY and TVL

```bash
curl -s "https://persephone.superform.xyz/v1/vaults/stats/historical/<chain_id>_<vault>?start_ts=<unix>&end_ts=<unix>&granularity=day"
```

`granularity` is `day` or `hour`. The response is a time series of `pps`, `tvl_total`, `apy_snapshot_week`, and `reward_rate`.

## Fields

| Field | Use |
| --- | --- |
| `address`, `chain_id` | Vault contract and chain. `chain_id` is a string |
| `symbol`, `friendly_name` | Display name, such as `superTSLA` |
| `availability` | Deposit only when `available` or `deposit_only`. For `withdraw_only`, `paused`, or `unavailable`, tell the user and stop |
| `visibility` | Skip `hidden` vaults |
| `assets[0]` | Deposit token with `address`, `symbol`, `decimals`, and `price_usd`. Stock vaults take tokenized stocks such as `NVDAc` or `TSLAc`, not USDC |
| `decimals` | Share token decimals. Stock vaults use 8 |
| `stats_basic.apy_snapshot_week` | 7-day strategy yield in percentage points |
| `stats_basic.reward_rate` | Merkl reward APR in percentage points |
| `rewards[]` | Reward tokens and their rates, such as USDC and sUP |
| `stats_basic.tvl_total` | TVL in USD |
| `stats_basic.pps`, `stats_basic.pps_usd` | Price per share in the deposit token and in USD |
| `has_timelock`, `timelock_duration`, `timelock_unit` | Expected wait between a redeem request and claim |
| `deployment_date` | Launch date. Mention when a vault is less than three months old |

Total APY is `apy_snapshot_week` plus `reward_rate`. A value of `-1000000` means no data yet, so treat it as `0`.

## Presenting vaults

Show symbol, chain, deposit token, total APY split into strategy and rewards, TVL, availability, and timelock.

- Point out when most of the APY comes from rewards. Reward campaigns can change or end.
- Flag rewards paid in `sUP` or `UP` as exposed to the Superform token price.
- Mention when TVL is below $500,000 or the vault is new.
