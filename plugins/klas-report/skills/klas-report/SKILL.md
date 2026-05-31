---
name: klas-report
description: Generate a Member, Deal, or Tax report (.xlsx) from the live KLAS Google Sheet, or load a pre-computed snapshot for ad-hoc Q&A. Use when the user types /klas-report or asks for a member report, deal report, tax report, KLAS report, or asks any financial question about KLAS Members or Deals. Invoke as `/klas-report member "<name>"`, `/klas-report deal "<name>"`, `/klas-report tax ["<name>"]`, or `/klas-report context`.
---

# klas-report

Generates a styled `.xlsx` financial report for a KLAS Member, Deal, or LLC-wide tax withholding, OR loads a pre-computed snapshot of every Member and Deal so Claude can answer ad-hoc questions without generating a file. Data is read live from the KLAS Google Sheet via OAuth (Phase 2). All computations are sourced from the verified compute layer inside the bundled CLI.

The skill delegates to the bundled CLI at `${CLAUDE_PLUGIN_ROOT}/dist/cli.js`. Run it with `node`. No clone, no `npm install`.

**First run:** after the first successful report or context answer in a session, follow the **Offer to silence per-run approval prompts** section below — once, only if it isn't already set up.

## Modes

| Mode | Invocation | What it does |
| :--- | :--- | :--- |
| Member report | `/klas-report member "<name>"` | Writes `Member_<Name>_<date>.xlsx` |
| Deal report | `/klas-report deal "<name>"` | Writes `Deal_<Name>_<date>.xlsx` |
| Tax report (LLC-wide) | `/klas-report tax` | Writes `Tax_Report_<date>.xlsx` — 4 tabs: Summary, By Member, By Quarter, By Deal |
| Tax report (per member) | `/klas-report tax "<name>"` | Writes `Tax_Report_<Name>_<date>.xlsx` — single sheet for one Non-US member |
| Context (Q&A) | `/klas-report context` | Prints a plain-text snapshot — no file |

**Name matching (member/deal modes):** case-insensitive partial match against `Member.Name` or `Deal.Deal_Name`.

Examples:
- `/klas-report member "Yochay"`
- `/klas-report deal "Ocala"`
- `/klas-report member "kob"` (partial match — finds member real name)
- `/klas-report context` (or whenever the user asks a financial question)

## When to use context mode

Use `context` mode whenever the user asks a financial question about KLAS data — Members, Deals, balances, distributions, tax — and does NOT explicitly ask for a report file. Examples that should trigger `context`:

- "What is Yochay's available cash?"
- "Which members are in the Ocala deal?"
- "How much has Yochay received in distributions?"
- "What's the total capital deployed across all active deals?"

The output is a structured plain-text snapshot with every virtual column pre-computed. Read values from the snapshot — do NOT recalculate formulas. The snapshot is the single source of truth for the session.

## Procedure — member/deal mode

1. Parse the two arguments. If either is missing, tell the user the usage: `/klas-report <member|deal> "<name>"`. Stop.

2. Confirm the user is authenticated. Run `node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js auth status`. If it prints "Not authenticated", tell the user: "Run `node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js auth login` once — a browser will open for Google OAuth. Expect a 'Google hasn't verified this app' warning — click **Advanced** → **Continue** as a Test User. Then try again."

3. Run the CLI:
   ```bash
   node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js <type> "<name>"
   ```
   Use the Bash tool. The script prints the report's absolute path on stdout on success. Data is fetched live from the KLAS Google Sheet using the user's cached OAuth token.

4. Report back to the user:
   - On success: just the file path, one line.
   - On any non-zero exit: pass the script's stderr to the user verbatim (it already lists valid candidates or the usage hint).

## Procedure — tax mode

1. Confirm the user is authenticated (`node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js auth status`). Same prompt as member/deal mode if not.

2. Run the CLI:
   ```bash
   # LLC-wide
   node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js tax

   # Per Non-US member (optional name argument)
   node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js tax "<name>"
   ```

3. The script validates the data before writing. It will throw — and the report will NOT be generated — if:
   - A Tax Payment row has no Deal_Ref or Member_Ref (data bug, not edge case)
   - A Tax Payment references a deal with no Date_Closed
   - The Settings table is missing `Short_Term_Tax_Rate` or `Long_Term_Tax_Rate`
   - The user passes a US member name to per-member mode

   On any of these, pass the error message to the user verbatim — it points to the row or setting that needs fixing.

4. On success: print the absolute path of the generated report.

5. **Tab layout** (LLC-wide mode): Summary (rates, grand totals, per-year and per-member rollups), By Member (every payment with date/deal/term/rate/amount), By Quarter (grouped by calendar quarter, with per-member breakdown — KLAS pays withholding quarterly), By Deal (only Complete deals that triggered tax, with per-member amounts and holding-period classification).

## Procedure — context mode

1. Confirm the user is authenticated (`node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js auth status`). Same prompt as report mode if not.

2. Run the CLI:
   ```bash
   node ${CLAUDE_PLUGIN_ROOT}/dist/cli.js context
   ```

3. The CLI prints a plain-text snapshot to stdout. Sections:
   - **Header:** source file, generation timestamp, top-line totals (member count, deal count, capital deployed, distributions, expenses, tax).
   - **`=== Members ===`** block: one entry per Member with all wallet virtual columns pre-computed.
   - **`=== Deals ===`** block: one entry per Deal with cap table and distribution/tax totals per member.

4. Use the snapshot to answer the user's question directly. Read values — do NOT re-derive formulas. Quote the relevant line if it helps.

5. If the user asks something the snapshot does not cover (e.g. a specific Balance Sheet transaction, a distribution timeline), tell them — do not invent answers.

## Offer to silence per-run approval prompts (first run, opt-in)

Each CLI call goes through the Bash tool, so Claude Code asks the user to approve every run. After the **first successful** run of any mode in a session, offer to make that approval permanent — scoped to this plugin's CLI alone. Do it once, only when not already configured, and only with the user's explicit yes.

1. **Check if already set up.** Read the user's global settings file `~/.claude/settings.json` (Windows: `%USERPROFILE%\.claude\settings.json`). If `permissions.allow` already has an entry containing this plugin's `cli.js` path, it is configured — say nothing and skip the rest.

2. **Ask once.** If not configured, ask the user:
   > "Claude Code asks you to approve each KLAS run. Want me to add a one-time rule so it stops asking? It permits only this KLAS tool (`node .../cli.js` with any report) — nothing else."

   If the user declines, add nothing and do not ask again this session.

3. **On yes, add the rule.** Edit `~/.claude/settings.json` (create the file and/or the `permissions.allow` array if missing; preserve everything already there — append, never overwrite). Add exactly one entry, built from the **same `node <path>` prefix you used to run the CLI**, with a trailing ` *`:
   ```
   Bash(node "<the cli.js path you invoked>" *)
   ```
   Then confirm in one line: "Done — KLAS reports won't prompt you again."

Never widen the scope (no `Bash(node:*)`), never modify any other setting, and never edit settings without an explicit yes.

## What the skill does NOT do

- It does NOT modify the KLAS Google Sheet. Read-only scope.
- It does NOT email or upload the report. Local file only.
- It does NOT run computations directly — always delegates to the bundled CLI so the verified compute layer is the single source of truth.
- It does NOT cache the context snapshot across sessions. Re-run `context` mode to refresh after the Sheet has been edited.
