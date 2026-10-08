# Rewards

SuperVault rewards are Merkl campaigns. They accrue automatically while the wallet holds vault shares. No staking is needed, but the user has to claim them.

## Check claimable rewards

```bash
curl -s "https://persephone.superform.xyz/v1/rewards/external/claimable/<acct>"
```

| Field | Use |
| --- | --- |
| `total_earned_value_usd` | Lifetime rewards in USD |
| `rewards[].chain_id` | Chain to claim on |
| `rewards[].extra_data` | Merkl proof. Pass it to the claim request |
| `rewards[].tokens[].symbol`, `address`, `decimals` | Reward token |
| `rewards[].tokens[].amount` | Claimable now, in raw units. Resets to `0` after a claim |
| `rewards[].tokens[].provider_claim_amount` | Cumulative earned, in raw units. Use this in the claim request |
| `rewards[].tokens[].value_usd` | USD value of the claimable amount |

Merkl updates every few hours, so new deposits show rewards after a delay.

## Claim

Build one request per `rewards[]` entry, with up to seven tokens per request:

```json
{"account":"<acct>","action":{"type":"CLAIM","slippage":0,"sources":[],
 "outputs":[{"address":"<token>","chain_id":<chain>,"amount":"<provider_claim_amount>"}],
 "targets":[{"address":"<acct>","chain_id":<chain>,"tags":["merkl"],"extra_data":"<extra_data>"}],
 "route_type":"OUTPUT"}}
```

- `outputs[].amount` is the cumulative `provider_claim_amount`, not the claimable delta.
- `targets[].tags` must be `["merkl"]`.
- The execution targets the Merkl distributor contract. Decode it and confirm the recipient is the user's wallet before you sign.

## When to claim

Compare `value_usd` with the gas cost on the reward chain. If the rewards are worth less than the gas, tell the user and don't claim.

Rewards paid in `sUP` are Superform's staked token. Their USD value depends on the token price.
