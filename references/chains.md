# Chains

SuperVaults are deployed on several chains. The agent wallet can only act on chains that `mm chains list` returns.

## Support matrix

| Chain | Chain ID | Listed vaults | Agent wallet | RPC for onchain reads |
| --- | --- | --- | --- | --- |
| Base | `8453` | superUSDC, superWETH, superCBBTC, stock vaults such as superNVDA and superTSLA, sUP staking | Supported | `https://mainnet.base.org` |
| Ethereum | `1` | superUSDC, superWETH, superWBTC | Supported | `https://ethereum-rpc.publicnode.com` |
| Robinhood Chain | `4663` | superUSDG | Supported | `https://rpc.mainnet.chain.robinhood.com` |
| BNB Chain | `56` | None listed | Supported | `https://bsc-dataseed.bnbchain.org` |
| Flare | `14` | bizFXRP, bizUSDT0 | Not supported | Not applicable |

Vault lists change. Always read the current list from the vaults API instead of relying on this table. See [vaults.md](vaults.md).

## Rules

- Before you build a transaction, confirm the vault's `chain_id` appears in `mm chains list`. If it doesn't, tell the user the agent wallet can't act on that chain and stop.
- Use the vault's `chain_id` for every `chain_id` field in bundler requests and for `--chain-id` in `mm wallet send-transaction`.
- Use the RPC for the vault's chain for onchain reads. See [onchain-reads.md](onchain-reads.md).
- Gas is paid in the chain's native token. Check the balance with `mm wallet balance --chain-ids <chain-id>` before you start a flow.

## Block explorers

| Chain | Explorer |
| --- | --- |
| Base | `https://basescan.org/tx/<hash>` |
| Ethereum | `https://etherscan.io/tx/<hash>` |
| Robinhood Chain | `https://robinhoodchain.blockscout.com/tx/<hash>` |
| BNB Chain | `https://bscscan.com/tx/<hash>` |
