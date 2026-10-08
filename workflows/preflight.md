# Preflight workflow

Run this workflow once per session, before the first vault lookup that leads to a transaction and before any `mm` command. Don't move on to another workflow until every check passes.

Read-only vault browsing with the Persephone API doesn't need the agent wallet. If the user only wants to list vaults, you can skip preflight until they ask to deposit, withdraw, or check positions.

## Flow

1. Check Node.js.
2. Check the `mm` CLI.
3. Check the agent wallet skill.
4. Check login and initialization.
5. Confirm the wallet.

## Check Node.js

```bash
node --version
```

The `mm` CLI needs Node.js 22.18 or later. If Node.js is missing or older, tell the user to install or upgrade it, for example with `nvm install 22`, and stop until they confirm.

## Check the mm CLI

```bash
command -v mm && mm --version
```

If `mm` isn't found, ask the user whether to install the CLI. Install only after they agree:

```bash
npm install -g @metamask/agent-wallet@latest
```

Run `mm --version` again to confirm. If the command still isn't found, the npm global `bin` directory probably isn't on `PATH`. Show the output of `npm prefix -g` and ask the user to add its `bin` directory to `PATH`.

If `mm` is installed, compare its version with the latest release:

```bash
npm view @metamask/agent-wallet version
```

If the installed version is behind, tell the user and offer to update with the same install command. Don't block on it.

## Check the agent wallet skill

This skill depends on the `metamask-agent-wallet` skill for `mm` command syntax, safety rules, and the login flow. Check whether it's installed:

```bash
ls ~/.claude/skills/metamask-agent-wallet/SKILL.md ~/.agents/skills/metamask-agent-wallet/SKILL.md 2>/dev/null
```

If neither path exists and the skill isn't in your available skills, ask the user whether to install it. Install only after they agree:

```bash
npx skills add metaMask/agent-skills
```

After installing, load the `metamask-agent-wallet` skill. If your agent only picks up new skills on restart, tell the user to restart the session and run the request again.

## Check login and initialization

```bash
mm doctor --toon
```

Read `authenticated`, `initialized`, and `hints` from the output.

| Result | Action |
| --- | --- |
| `authenticated: false` | Run the login flow below |
| `authenticated: true`, `initialized: false` | Run the initialization step below |
| Both `true` | Go to [confirm the wallet](#confirm-the-wallet) |

Show any `hints` to the user. If `doctor` reports a skill and CLI version mismatch, warn once and continue.

### Log in

Follow the `metamask-agent-wallet` login workflow. In short:

1. Ask the user which method they want: browser sign-in with Google or email, or MetaMask Mobile QR. QR needs an interactive terminal, so use browser sign-in in a headless session.
2. Start browser sign-in:

   ```bash
   mm login browser --no-wait
   ```

3. Show the sign-in URL and wait for the user to finish in the browser.
4. Complete sign-in with the token the user receives:

   ```bash
   mm login --token "<token>"
   ```

5. Run `mm doctor --toon` again and confirm `authenticated: true`.

### Initialize

If `initialized` is `false`, ask the user to choose:

- Wallet mode: `server-wallet` is recommended, because MetaMask hosts the keys. `byok` uses the user's own mnemonic.
- Trading mode: `guard` enforces policies and asks for MFA on violations. `beast` skips policy checks.

```bash
mm init --wallet server-wallet --mode guard
```

For `byok`, never pass the mnemonic or password inline. Tell the user to set `MM_MNEMONIC` and, if needed, `MM_PASSWORD` as environment variables first. See the `metamask-agent-wallet` onboarding workflow.

Run `mm doctor --toon` again and confirm both `authenticated` and `initialized` are `true`.

## Confirm the wallet

```bash
mm wallet address --toon
```

Use this address as `<acct>` in every Superform request. Tell the user which address you're using, then continue with the workflow they asked for.

## Stop conditions

Stop and wait for the user when:

- They decline to install the CLI or the agent wallet skill.
- Login or initialization fails. Show the error verbatim.
- `mm doctor` still reports `authenticated: false` or `initialized: false` after the steps above.
