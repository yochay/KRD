# KRD — KLAS Reports Distribution

Distribution artifacts for the KLAS Reports Claude Code plugin and Claude Desktop MCP server. **Source repository is private.**

This repo hosts only the built artifacts. There is no source code, no documentation about internal logic, and no development happens here.

## Install (Claude Code)

```
/plugin marketplace add yochay/KRD
/plugin install klas-report@KRD
```

Then in any Claude Code session:

```
/klas-report member "<name>"
/klas-report deal "<name>"
/klas-report tax
```

Reports save to `~/Documents/KLAS Reports/`.

## Install (Claude Desktop)

Download the latest `.mcpb` from the [Releases page](https://github.com/yochay/KRD/releases), drag it into Claude Desktop, restart.

## First-run setup

The first command opens a browser for Google OAuth consent (read-only access to the KLAS Google Sheet). Your Gmail must already be:

1. Added as a Test User on the KLAS GCP project
2. Granted Viewer access to the KLAS Google Sheet

Contact the maintainer to be added.

## Issues / bug reports

Issues are disabled on this repo. Report bugs directly to the maintainer.

## License

All rights reserved. Internal use only. See `LICENSE`.
