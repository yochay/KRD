# Install the KLAS Reports extension (Claude Desktop)

Desktop-only members run KLAS reports as a Claude Desktop extension (`.mcpb`).

## Prerequisites (one-time, done by Yochay)

1. Yochay adds your Gmail to project.
2. Yochay shares Viewer to data.

Without both, sign-in fails.

## Install

1. Get `klas-report-mcp.mcpb` from Yochay (Phase 4c will host it on KRD Releases).
2. Open Claude Desktop → **Settings → Extensions**.
3. Drag `klas-report-mcp.mcpb` onto the window (or double-click the file).
4. Confirm install, then **fully restart** Claude Desktop.

## First run — Google sign-in

1. Ask Claude for a report (e.g. *"run a KLAS member report for Yochay"*).
2. If you're not signed in, run the **`auth_login`** tool — a browser opens.
3. Expect a **"Google hasn't verified this app"** warning → **Advanced → Continue** as a Test User.
4. Retry the report once sign-in completes. The token caches; later runs are silent.

## Tools

- `member_report` (name) · `deal_report` (name) · `tax_report` (optional name)
- `klas_context` — ad-hoc Q&A, no file
- `auth_status` · `auth_login`

## Where reports land

```
~/Documents/KLAS Reports/
```

(Override with the `KLAS_OUTPUT_DIR` environment variable.)

## Trouble?

- **Tools missing after install** → fully quit and reopen Claude Desktop.
- **Sign-in fails / access denied** → confirm Yochay added your Gmail as a Test User *and* shared the Sheet.
- Anything else → email Yochay.
