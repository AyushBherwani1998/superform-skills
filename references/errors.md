# Errors

Bundler errors are deterministic. An identical retry returns the same error, so fix the request first. Surface the `message` field to the user verbatim.

## Bundler errors

| Message contains | Cause | Fix |
| --- | --- | --- |
| `insufficient balance` | The wallet doesn't hold enough of the source token on that chain | Get the deposit token first. See [deposit.md](../workflows/deposit.md) |
| `vault share balance not found` | The wallet holds no shares for a `REDEEM` | Check the share balance onchain |
| `target amount ... is not set` | The target has no `amount` or `amount_proportion` | Add `"amount_proportion": 1` for deposits or `"amount"` for redemptions |
| `failed to validate source amount` | `sources[].amount` is missing | Add the raw amount |
| `HOOK_NOT_FOUND` | A stale hook ID was used | Call `/hooks/suggest/non-smart-account` again. Never hardcode hook IDs |

## Onchain and wallet errors

| Symptom | Cause | Fix |
| --- | --- | --- |
| `eth_call` returns an `error` during simulation | The call would revert | Don't send. Recheck amounts and redemption state |
| Receipt `status` is `0x0` | The transaction reverted | Show the explorer link and stop |
| `maxWithdraw` stays `0` after the timelock | The keeper hasn't fulfilled the request yet | Check again later. `pendingRedeemRequest` should still be above `0` |
| Chain not in `mm chains list` | The agent wallet doesn't support the chain | Tell the user and stop. See [chains.md](chains.md) |

For `mm` errors, follow the `metamask-agent-wallet` skill's error reference.
