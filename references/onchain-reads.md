# Onchain reads

Read share balances and redemption state directly from the vault contract with `eth_call`. Use the RPC for the vault's chain. See [chains.md](chains.md).

```bash
curl -s -X POST <rpc> -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_call","params":[{"to":"<vault>","data":"<calldata>"},"latest"]}'
```

The result is a hex `uint256`. Convert it to a decimal raw amount, then to human units with the share or asset decimals.

## Calldata

`<acct>` is the account address without `0x`, lowercased, and left-padded with zeros to 64 hex characters.

| Read | Function | Calldata | Units |
| --- | --- | --- | --- |
| Share balance | `balanceOf(acct)` | `0x70a08231<acct>` | Shares |
| Pending redemption | `pendingRedeemRequest(0, acct)` | `0xf5a23d8d` + 64 zeros + `<acct>` | Shares |
| Claimable redemption | `claimableRedeemRequest(0, acct)` | `0xeaed1d07` + 64 zeros + `<acct>` | Shares |
| Claimable assets | `maxWithdraw(acct)` | `0xce96cb77<acct>` | Deposit token |

SuperVaults always use request ID `0`.

## Redemption state

| State | Condition | Next step |
| --- | --- | --- |
| None | `pendingRedeemRequest` is `0` and `maxWithdraw` is `0` | Nothing to claim |
| Pending | `pendingRedeemRequest` is above `0` and `maxWithdraw` is `0` | Wait for the keeper, usually about one hour |
| Fulfilled | `maxWithdraw` is above `0` | Build a `REDEEM_CLAIM` |

Don't read `maxWithdraw == 0` as "no position". It stays `0` until the keeper fulfills the request.

## Simulate before sending

Simulate any state-changing call by adding `from` to the `eth_call` params:

```json
{"from":"<acct with 0x>","to":"<vault>","data":"<call_data>"}
```

A result without an `error` field means the call would succeed.
