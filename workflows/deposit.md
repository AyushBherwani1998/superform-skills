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

Always check the agent wallet's balance before you swap or bridge. Don't swap or bridge if the wallet already holds enough of the deposit token on the vault's chain.

1. Read the deposit token balance onchain with `balanceOf(acct)` on `assets[0].address`, using the vault chain's RPC. See [onchain-reads.md](../references/onchain-reads.md#token-balance). Don't rely on `mm wallet balance` for the deposit token. It omits tokens it can't price, such as tokenized stocks like `TSLAc`, so a wallet that holds them can look empty.
2. Convert the result to human units with `assets[0].decimals` and compare it with the deposit amount.
3. If the balance is greater than or equal to the deposit amount, don't swap or bridge. Tell the user you're using the existing balance and skip to [Build the deposit](#build-the-deposit).
4. If the balance is lower than the deposit amount, tell the user the balance and the shortfall. Then ask whether to deposit only the existing balance, or to swap or bridge just the shortfall. Never swap or bridge the full amount when the wallet already holds part of it.
5. If the user chooses to swap or bridge, follow the `metamask-agent-wallet` swap or bridge workflow to get the shortfall on the vault's chain. Quote with `--all-quotes`, compare routes, and warn if price impact is above 1%. Tokenized stocks often have thin liquidity.
6. After a swap or bridge, read the deposit token balance onchain again to confirm it.
7. Check that the wallet has enough native token for gas: an approval, a deposit, and later a redemption and claim. Use `mm wallet balance --chain-ids <chain> --toon` for the native token.

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
