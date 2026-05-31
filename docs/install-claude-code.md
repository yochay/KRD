# Install the KLAS Reports plugin (Claude Code)

The `/klas-report` skill ships as a Claude Code plugin.

## Prerequisites (one-time, done by Yochay)

1. Yochay adds your Gmail to project.
2. Yochay shares Viewer to data.

Without both, OAuth login will fail.

## Install

In Claude Code, run:

```
/plugin marketplace add yochay/KRD
/plugin install klas-report@KRD
```

## First run — Google sign-in

The first report triggers a one-time Google OAuth login:

1. A browser window opens.
2. You will see a **"Google hasn't verified this app"** warning. This is expected — the app runs in Testing mode.

   ![Google hasn't verified this app](./img/oauth-unverified-warning.png) <!-- screenshot placeholder -->

3. Click **Advanced** → **Continue** (you're an approved Test User).
4. Grant read-only access. The token caches locally; later runs are silent.

## Use it

```
/klas-report member "name"
/klas-report deal "name"
/klas-report tax
/klas-report context     # ad-hoc Q&A, no file
```

## Fewer prompts

Claude Code asks you to approve each report run (a security feature). The first time you run a report, Claude will offer to add a one-time permission rule so it stops asking — scoped to the KLAS tool only, nothing else. Say **yes** once and you won't be prompted again. (Say no and nothing changes.)

## Where reports land

Generated `.xlsx` files save to:

```
~/Documents/KLAS Reports/
```

(Override with the `KLAS_OUTPUT_DIR` environment variable.)

## Updates

Plugin updates arrive via `/plugin marketplace update`. No version bump or reinstall needed — the latest plugin content is pulled on update.

## Trouble?

- **"Not authenticated"** → run `node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js auth login` and complete the browser flow.
- **OAuth fails / access denied** → confirm Yochay added your Gmail as a Test User *and* shared the Sheet.
- Anything else → email Yochay. (Bug reports go to the private source repo; members don't file issues directly.)
