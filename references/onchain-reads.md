# Onchain reads

Read token balances, share balances, and redemption state directly from the contracts with `eth_call`. Use the RPC for the vault's chain. See [chains.md](chains.md).

```bash
curl -s -X POST <rpc> -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_call","params":[{"to":"<vault>","data":"<calldata>"},"latest"]}'
```

The result is a hex `uint256`. Convert it to a decimal raw amount, then to human units with the share or asset decimals.

## Calldata

`<acct>` is the account address without `0x`, lowercased, and left-padded with zeros to 64 hex characters.

| Read | Function | Calldata | Units |
| --- | --- | --- | --- |
| Token balance | `balanceOf(acct)` on the token contract | `0x70a08231<acct>` | Token decimals |
| Share balance | `balanceOf(acct)` on the vault | `0x70a08231<acct>` | Shares |
| Pending redemption | `pendingRedeemRequest(0, acct)` | `0xf5a23d8d` + 64 zeros + `<acct>` | Shares |
| Claimable redemption | `claimableRedeemRequest(0, acct)` | `0xeaed1d07` + 64 zeros + `<acct>` | Shares |
| Claimable assets | `maxWithdraw(acct)` | `0xce96cb77<acct>` | Deposit token |

SuperVaults always use request ID `0`.

## Token balance

Read ERC-20 balances, such as the deposit token, onchain instead of with `mm wallet balance`. The `mm` balance list omits tokens without a price feed, including tokenized stocks like `TSLAc` and `NVDAc`.

```bash
curl -s -X POST <rpc> -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_call","params":[{"to":"<token>","data":"0x70a08231<acct>"},"latest"]}'
```

For example, a result of `0x...404318` is `4211480` raw, which is `0.0421148` of an 8-decimal token.

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
