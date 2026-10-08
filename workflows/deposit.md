# Deposit workflow

Use this workflow when the user wants to deposit into a SuperVault.

Read [bundler.md](../references/bundler.md) and [signing.md](../references/signing.md) before you start.

Run [preflight.md](preflight.md) first if it hasn't passed in this session.

## Flow

1. Pick the vault.
2. Get the deposit token.
3. Build the deposit.
4. Sign and send.
5. Report.

## Pick the vault

1. Find the vault in the vaults API by address or symbol. See [vaults.md](../references/vaults.md).
2. Check that `availability` is `available` or `deposit_only`.
3. Check that the vault's chain is supported by the agent wallet. See [chains.md](../references/chains.md).
4. Show the user the vault's APY split, TVL, timelock, and deposit token.

## Get the deposit token

1. Check balances on the vault's chain:

   ```bash
   mm wallet balance --chain-ids <chain> --toon
   ```

2. If the wallet already holds enough of `assets[0]`, skip to the next section.
3. If it doesn't, follow the `metamask-agent-wallet` swap or bridge workflow to get the deposit token on the vault's chain. Quote with `--all-quotes`, compare routes, and warn if price impact is above 1%. Tokenized stocks often have thin liquidity.
4. Confirm the new balance onchain or with `mm wallet balance`.
5. Check that the wallet has enough native token for gas: an approval, a deposit, and later a redemption and claim.

## Build the deposit

1. Convert the amount to raw units with `assets[0].decimals`.
2. Send the `DEPOSIT` request to the bundler. See [bundler.md](../references/bundler.md#deposit).
3. If the bundler returns an error, see [errors.md](../references/errors.md).
4. Check that the response has two executions: an approval on the deposit token, and a deposit on the vault.

## Sign and send

Follow [signing.md](../references/signing.md) for each execution:

1. Decode the approval. The spender must be the vault and the amount must equal the deposit amount.
2. Send the approval and wait for confirmation.
3. Decode the deposit. The receiver must be the user's wallet.
4. Send the deposit and wait for confirmation.

## Report

1. Read the share balance onchain. See [onchain-reads.md](../references/onchain-reads.md).
2. Show:

   - Tx hashes as explorer links
   - Shares received and their underlying value
   - The vault's APY split and reward tokens
   - The timelock the user will face on withdrawal
   - That rewards must be claimed and appear after Merkl's next update
