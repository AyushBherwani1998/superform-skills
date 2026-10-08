# Withdraw workflow

Use this workflow when the user wants to withdraw from a SuperVault. SuperVaults use ERC-7540 async redemption, so a withdrawal takes two transactions with a wait in between.

Read [bundler.md](../references/bundler.md), [onchain-reads.md](../references/onchain-reads.md), and [signing.md](../references/signing.md) before you start.

Run [preflight.md](preflight.md) first if it hasn't passed in this session.

## Flow

1. Check the position.
2. Request the redemption.
3. Wait for fulfillment.
4. Claim the assets.
5. Report.

## Check the position

1. Read `balanceOf`, `pendingRedeemRequest`, and `maxWithdraw` onchain. See [onchain-reads.md](../references/onchain-reads.md).
2. Pick the path:

   | State | Path |
   | --- | --- |
   | Shares held, nothing pending | Request the redemption |
   | Redemption pending | Tell the user it's waiting for the keeper and stop |
   | Redemption fulfilled | Go to [claim the assets](#claim-the-assets) |
   | No shares and nothing pending | Tell the user there's nothing to withdraw |

3. For a partial withdrawal, convert the requested amount to raw shares with the vault's `decimals`. For a full withdrawal, use the raw `balanceOf` result.

## Request the redemption

1. Send the `REDEEM` request to the bundler. See [bundler.md](../references/bundler.md#redeem-request).
2. Review the executions:

   - Skip `SET_ERC7540_OPERATOR` unless the user explicitly agrees and the operator is on Superform's published deployment list.
   - The `APPROVE` spender must be the vault itself, for the exact share amount.
   - `REDEEM` must call `requestRedeem(raw_shares, acct, acct)`.

3. Send the approval and wait for confirmation.
4. Simulate `requestRedeem` with `eth_call` and `from` set to the wallet. If it fails, stop.
5. Send `requestRedeem` and wait for confirmation.
6. Read the state again. `balanceOf` drops by the redeemed shares and `pendingRedeemRequest` shows them.

## Wait for fulfillment

The keeper usually fulfills requests within the vault's timelock, about one hour for most vaults and seven days for sUP staking.

Tell the user when to check back. Don't poll in a loop for the full wait. When the user asks, read `maxWithdraw`. If it's still `0`, the request is still pending.

## Claim the assets

1. Read `claimableRedeemRequest(0, acct)` and `maxWithdraw(acct)`. Both must be above `0`.
2. Send the `REDEEM_CLAIM` request with the target `amount` set to the claimable shares. See [bundler.md](../references/bundler.md#redeem-claim).
3. Decode the execution. It must call `withdraw(assets, acct, acct)` on the vault with the user's wallet as receiver and controller.
4. Simulate it with `eth_call`, then send it and wait for confirmation.
5. Confirm that `maxWithdraw` is now `0` and the deposit token balance went up.

## Report

Show:

- Tx hashes as explorer links for the request and the claim
- The deposit token amount received
- Any rewards still claimable. See [positions.md](positions.md#claim-rewards)
- Next options, such as swapping the deposit token back to a stablecoin with the `metamask-agent-wallet` swap workflow
