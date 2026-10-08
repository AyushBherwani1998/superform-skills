# Signing

Sign and broadcast every bundler execution with the MetaMask Agent Wallet. Follow these steps for each execution, in array order, one at a time.

## Steps

1. Check the target. `to` must be the expected contract for the execution type. See the expected executions tables in [bundler.md](bundler.md). If `to` is anything else, stop and tell the user.
2. Decode the call data:

   ```bash
   mm decode --payload <call_data> --toon
   ```

   Confirm the function and parameters match the request:

   - `approve`: the spender is the vault, and the amount equals the requested amount. Flag unlimited approvals.
   - `deposit`: the receiver is the user's wallet.
   - `requestRedeem` and `withdraw`: the controller, owner, and receiver are the user's wallet.
   - `setOperator`: skip unless the user explicitly agreed. See [bundler.md](bundler.md#redeem-request).

3. Simulate redemption and claim calls with `eth_call`. See [onchain-reads.md](onchain-reads.md#simulate-before-sending).
4. Show the user the step, contract, decoded action, amount, and chain, and get approval. If the user already approved the full plan, continue without asking again, but stop if anything differs from what they approved.
5. Convert `value` to hex. The bundler returns a decimal string, and `mm` expects 0x-prefixed hex. `"0"` becomes `"0x0"`.
6. Send:

   ```bash
   mm wallet send-transaction --chain-id <chain> \
     --payload '{"to":"<to>","value":"0x0","data":"<call_data>"}' \
     --intent "<Approve 0.0288 TSLAc for superTSLA>" --wait --toon
   ```

7. Wait for confirmation before the next execution. Check the receipt with `eth_getTransactionReceipt` on the chain's RPC. A `status` of `0x1` means success.

## MFA

If the output contains `AWAITING_MFA`, the job is waiting for the user to approve it. Show the instruction, give the polling ID, and don't resend. The user can track it with `mm wallet requests watch <polling-id>`.

## Never

- Batch executions or send them in parallel.
- Sign call data the bundler didn't return for the request you just made.
- Send to a contract you haven't checked.
