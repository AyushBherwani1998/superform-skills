# Bundler

The Superform bundler builds unsigned transactions for deposits, redemptions, and reward claims. It's public and needs no key.

Endpoint:

```bash
curl -s -X POST https://bundler.superform.xyz/hooks/suggest/non-smart-account \
  -H 'Content-Type: application/json' \
  -d '<body>'
```

The response has an ordered `executions` array. Each execution has `type`, `to`, `value`, and `call_data`. Send them in array order. See [signing.md](signing.md).

The bundler checks onchain balances before it builds. If the wallet lacks the source token or shares, it returns an error instead of executions. See [errors.md](errors.md).

## Common fields

| Field | Value |
| --- | --- |
| `account` | The agent wallet address |
| `action.slippage` | Basis points. Use `50` for 0.5%. Use `0` for reward claims |
| `chain_id` | The vault's chain ID as a number |
| `vault_type` | Always `"SUPERVAULT"` |
| `amount` | Raw amount in the token's smallest unit, as a string |

## Deposit

```json
{"account":"<acct>","action":{"type":"DEPOSIT","slippage":50,
 "sources":[{"address":"<asset>","chain_id":<chain>,"amount":"<raw_assets>"}],
 "targets":[{"address":"<vault>","chain_id":<chain>,"vault_type":"SUPERVAULT","amount_proportion":1}],
 "route_type":"OUTPUT","outputs":[]}}
```

Expected executions:

| Index | Type | Call | Target |
| --- | --- | --- | --- |
| 0 | `DEPOSIT` | `approve(vault, raw_assets)` | Deposit token |
| 1 | `DEPOSIT` | `deposit(raw_assets, acct)` | Vault |

## Redeem request

```json
{"account":"<acct>","action":{"type":"REDEEM","slippage":50,"sources":[],
 "targets":[{"address":"<vault>","chain_id":<chain>,"vault_type":"SUPERVAULT","amount":"<raw_shares>"}],
 "outputs":[{"address":"<asset>","chain_id":<chain>,"amount_proportion":1}],
 "route_type":"OUTPUT"}}
```

Expected executions:

| Index | Type | Call | Target | Action |
| --- | --- | --- | --- | --- |
| 0 | `SET_ERC7540_OPERATOR` | `setOperator(<operator>, true)` | Vault | Skip by default |
| 1 | `APPROVE` | `approve(vault, raw_shares)` | Vault | Send |
| 2 | `REDEEM` | `requestRedeem(raw_shares, acct, acct)` | Vault | Simulate, then send |

The `setOperator` step lets Superform's keeper claim on the user's behalf. It isn't required. Send it only if the operator address appears on the [Superform deployment addresses](https://docs.superform.xyz/deployment-addresses) page and the user explicitly agrees. Without it, the user claims manually after fulfillment.

## Redeem claim

Use the same body as the redeem request with `"type":"REDEEM_CLAIM"`. Set the target `amount` to the result of `claimableRedeemRequest(0, acct)`. See [onchain-reads.md](onchain-reads.md).

Expected execution:

| Index | Type | Call | Target |
| --- | --- | --- | --- |
| 0 | `REDEEM_CLAIM` | `withdraw(assets, acct, acct)` | Vault |

## Reward claim

See [rewards.md](rewards.md).
