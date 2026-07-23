# Changelog

A commit backlog for GradeRunner — one line per change, newest at the top, grouped by date. Each line is a `type(scope): summary` in the same style as a real commit message, just written after the fact from session history rather than from actual diffs.

## 2026-07-24

- `feat(table)`: add "Table Text Size" setting (Tiny/Small/Normal/Large) so both the info table and grade stats table can shrink for phone/small-screen use.
- `feat(brand)`: re-add `index.html`'s favicon/apple-touch-icon wiring to the existing `favicon.png`.
- `feat(brand)`: re-add `graderunner.png` brand artwork to the top of `README.md` and its repo contents table.
- `feat(pages)`: re-add public Telegram channel links to `index.html` (nav button, hero CTA, Alerts-section callout).
- `feat(stats)`: re-add the Grade Win-Rate Stats panel (per-grade TP1-before-SL tracking) after the full revert below.
- `revert(major)`: revert `GradeRunner.pine`, `README.md`, and `index.html` to their exact state at commit `6db787b`, undoing every feature/gate built on top of it this session.

## 2026-07-23

- `feat(grading)`: lower the ADX "strong trend" hard-gate threshold from 20 to 15, with the scoring anchor moved to match.
- `revert(grading)`: remove the VWAP distance confluence factor entirely; score back to 0-10.
- `revert(grading)`: remove the unconditional MACD momentum gate entirely.
- `feat(grading)`: add an unconditional MACD momentum gate, prompted by a faked-out A-grade signal with near-flat MACD.
- `fix(grading)`: restrict the HTF Bias hard gate to B-grade signals only; A/A+ bypass it.
- `feat(grading)`: make HTF Bias a hard gate, same posture as Local Trend/ADX/Core Signal.
- `feat(diagnostics)`: add per-factor score diagnostics (net point contribution per factor) to TradingView's Data Window.
- `feat(grading)`: add VWAP distance as a 6th confluence factor, rescaling the score to 0-12.
- `revert(grading)`: remove the EMA ribbon + volume confirmation feature set entirely; score back to 0-10.
- `feat(grading)`: add EMA ribbon (Guppy-style) trend refinement and a volume confirmation factor, rescaling the score to 0-12.
- `fix(display)`: fix "Table Text Size" incorrectly greying out when only the Grade Win-Rate Stats table was on.
- `feat(stats)`: add the "Grade Win-Rate Stats" panel — per-grade TP1-vs-SL tracking, pure measurement.
- `feat(grading)`: scale MACD histogram and ADX/DMI strength by magnitude instead of a flat 2 points on threshold cross.
- `feat(grading)`: add a volatility regime gate blocking signals at extreme ATR-ratio readings (news spikes or dead markets).
- `feat(grading)`: make Core Signal (breakout confirmation) an always-on requirement instead of a toggle.
- `revert(signal)`: remove "Wait for Pullback Before Entry" and "Only Trade With HTF Trend" entirely.
- `feat(grading)`: add `localTrendOk`/`adxStrong` hard requirements, closing a gap that let momentum-only signals fire against the trend.
- `revert(signal)`: remove the "Avoid Entries Near Support/Resistance" feature entirely.
- `fix(signal)`: fix an out-of-bounds runtime error in the support/resistance filter on empty arrays.
- `feat(signal)`: add "Avoid Entries Near Support/Resistance" toggle, tracking swing pivots.
- `docs(pages)`: fix `index.html`'s feature grid missing two already-shipped features (confirmed-close SL, pullback entry).
- `feat(signal)`: add "Only Trade With HTF Trend" toggle with a neutral-zone check.
- `feat(table)`: add "Table Text Size" setting for the info table (original addition).
- `chore(defaults)`: change "Wait for Pullback Before Entry" default from off to on.
- `fix(signal)`: fix Minimum Grade filter not being re-checked at pullback resumption.
- `feat(signal)`: add "Wait for Pullback Before Entry" toggle with armed/pending state machine.
- `feat(risk)`: add "Require Confirmed Close for Stop Loss" toggle to avoid wick-through stopouts.
- `feat(brand)`: add `graderunner.png` brand artwork and generate `favicon.png` (original addition).
- `style(pages)`: shorten hero's Telegram CTA to "Get the signals →".
- `feat(pages)`: add public Telegram channel links to `index.html` (original addition).
- `chore(repo)`: add `.gitattributes` mapping `*.pine` to Python's syntax highlighter.
- `docs`: note that all six alert conditions require at least TradingView's Essential plan.
- `chore(cleanup)`: remove `TradingView_Description.txt`.
- `fix(pages)`: fix `index.html`'s alert previews and stale "Pine Script v5" references.
- `refactor(history)`: replace 10 parallel arrays with a single `Setup` user-defined type.
- `refactor(pine)`: routine cleanup — combined HTF `request.security()` calls, `_` placeholders, `barstate.islast` table gating, helper functions for trimming/hit-alerts.
- `feat(alerts)`: change the Buy/Sell `alert()` message to a multi-line Entry/TP1/TP2/SL format.
- `docs`: add `TradingView_Description.txt` for the Publish/Edit Script description box.
- `docs(readme)`: add a warning about clearing the default alert Message box in TradingView.
- `chore(pine)`: upgrade `//@version=5` to `//@version=6`, add `active=` to dependent inputs.
- `feat(signal)`: add one-trade-at-a-time gating via "Lock New Signals Until TP1 or SL Hit".
- `feat(pages)`: add `index.html` landing page for GitHub Pages.
- `style(colors)`: introduce a shared light/dark-theme-safe color palette.
- `chore(rename)`: rename the project from "Confluence Scalper" to GradeRunner.
- `refactor(table)`: trim info table from 9 rows to 6.
- `feat(table)`: change Trend row to reflect HTF bias specifically, independent of Score.
- `style(table)`: restyle info table to a minimalist, theme-adaptive design.
- `style(lines)`: recolor TP1/TP2/TP3 lines/labels from dark green to near-black.
- `feat(alerts)`: add "Alert Hours" setting restricting alerts to a session + timezone.
- `feat(risk)`: wire Volatility into real SL/TP scaling (High/Low multipliers).
- `feat(signal)`: wire Core Signal into a real breakout-confirmation gate.
- `fix(signal)`: fix `lastDir` only advancing when a signal actually fires, not on every score flip.
- `feat(history)`: add "Show Previous Setups" toggle + max count.
- `feat(alerts)`: add TP1/TP2/TP3 Hit and Stop Loss Hit alert conditions.
- `feat(alerts)`: add named Buy Signal/Sell Signal/Buy-or-Sell Signal alert conditions.
- `fix(badge)`: badge now spells out BUY/SELL explicitly instead of arrow + grade letter alone.
- `feat(display)`: add toggle to hide the EMA plot lines while still feeding the trend score.
- `style`: initial visual pass to mirror reference screenshot (dashed TPs, solid entry/SL, badge, table).
- `feat`: initial build — 5-factor confluence score graded A+/A/B/C, auto-plotted entry/SL/TP1-3, non-repainting signal trigger.
- `docs`: add `README.md` and a standalone TradingView description draft.
