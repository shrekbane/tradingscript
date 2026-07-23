# Changelog

All notable changes to GradeRunner are logged here, newest first. Entries are written as short, commit-message-style summaries.

## 2026-07-23

- `chore(rename)`: rename the project from the working title "Confluence Scalper" to **GradeRunner** — script file renamed `Confluence_Scalper.pine` → `GradeRunner.pine`, `indicator()` title/shorttitle updated, README updated throughout. No functional changes.
- `refactor(table)`: trim info table from 9 rows to 6 — drop the title/branding row, Core Signal row, and Alerts row (all static settings echoes, not live data). Underlying Core Signal gate and Alert Hours gating logic are unaffected; only their table display was removed.
- `feat(table)`: change Trend row to reflect higher-timeframe bias specifically (`htfBull`) instead of the combined 5-factor confluence direction, so it's a standalone "bigger picture" read independent of Score.
- `style(table)`: restyle info table to a minimalist, theme-adaptive design — flat single-tone panel (no zebra striping), thin adaptive border, muted label / full-contrast value text using `chart.fg_color`, removed loud dark header and orange divider row.
- `style(lines)`: recolor TP1/TP2/TP3 lines and labels from dark green to near-black to better match the reference screenshot's dashed-line style.
- `feat(alerts)`: add "Alert Hours" setting — optional restriction so alerts only fire during a selected trading session (TradingView native session/day picker) and timezone (dropdown of common cities/regions instead of free text). Gates alerts only; score, badge, table, and drawn levels keep updating regardless.
- `feat(risk)`: wire Volatility into real SL/TP scaling — High/Low volatility multiplies the ATR-based stop/target distances (defaults 1.5x / 0.75x), toggleable.
- `feat(signal)`: wire Core Signal into a real breakout-confirmation gate — requires price to close beyond the recent N-bar high/low in the signal direction before firing, on top of the confluence grade filter. Previously a cosmetic label with no effect.
- `fix(signal)`: `lastDir` now only advances when a signal actually fires (not on every raw score flip), so Core Signal keeps checking for breakout confirmation across subsequent bars instead of getting stuck.
- `feat(history)`: add "Show Previous Setups" toggle + max count. TP/SL lines, labels, and hit tags always clear down to the current setup; only the entry-direction badge persists across past setups.
- `feat(alerts)`: add TP1/TP2/TP3 Hit and Stop Loss Hit detection, each firing once with its own `alertcondition()` and dynamic `alert()` message.
- `feat(alerts)`: add named Buy Signal / Sell Signal / Buy or Sell Signal `alertcondition()`s alongside the existing dynamic `alert()` call.
- `fix(badge)`: badge now spells out BUY/SELL explicitly (e.g. `▲ BUY B ★`) instead of just an arrow + grade letter — the grade letter alone (e.g. "B") was being misread as a buy/sell indicator.
- `feat(display)`: add toggle to hide the fast/slow EMA plot lines from the chart while keeping them fully computed and feeding the trend score.
- `style`: initial visual pass to mirror reference screenshot — dashed TP1–3 lines, solid entry/SL lines, signal badge, and info table layout.
- `feat`: initial build (working title "Confluence Scalper", later renamed GradeRunner). 5-factor confluence score (local trend, HTF bias, RSI, MACD, ADX/DMI) graded A+/A/B/C, auto-plotted entry/SL/TP1–3 (ATR-based), non-repainting signal trigger on confirmed bar close.
- `docs`: add `README.md` (GitHub-oriented, with install steps) and standalone TradingView description draft.

---

*Format: newest entries at the top. Feel free to edit/reword before using as actual commit messages — these are written from the session history, not from real diffs.*
