# 📊 GradeRunner — Trend + Momentum + Volatility + HTF Bias

<p align="center">
  <img src="graderunner.png" alt="GradeRunner" width="360">
</p>

A Pine Script v6 indicator for TradingView that grades every setup on a 0–10 confluence score, then auto-plots an entry, stop loss, and three take-profit levels the moment a graded signal fires. Built for traders who want a structured, rules-based way to combine trend, momentum, higher-timeframe bias, and volatility into a single readable signal — instead of eyeballing five indicators separately.

Signals only fire on a **confirmed (closed) bar** — there's no repainting of the core score or signal trigger.

> This is an independent, clean-room implementation — not a copy of any private/invite-only script. Built from scratch in Pine Script v6.

## 📥 Installation

This isn't published on TradingView's public script library, so you'll add it manually via the Pine Editor:

1. Open [`GradeRunner.pine`](./GradeRunner.pine) in this repo and copy the full contents (use the "Raw" button on GitHub, then select all + copy).
2. In TradingView, open any chart and click the **Pine Editor** tab at the bottom of the screen.
3. Click **Open** → **New blank script**, select all the placeholder code, and delete it.
4. Paste in the copied script.
5. Click **Save**, name it, then click **Add to Chart**.
6. Open the indicator's settings (⚙️ gear icon) to configure it for your instrument and timeframe — see [Key settings](#-key-settings-at-a-glance) below.

## 📁 Repository contents

| File | Description |
|---|---|
| `GradeRunner.pine` | The indicator source code |
| `README.md` | This file |
| `graderunner.png` | Brand artwork, shown above and used as the site favicon |

---

## ⚙️ How it works

Every bar, the script scores the **bull** side and the **bear** side independently across five factors, and whichever side is ahead becomes the active direction:

- 🔹 **Local trend** (2 pts) — Fast EMA vs. Slow EMA (default 9/21)
- 🔹 **Higher-timeframe bias** (2 pts) — Fast EMA vs. Slow EMA, computed on a separate higher timeframe you choose
- 🔹 **Momentum** (up to 2 pts) — RSI, scaled by distance from 50 (a borderline RSI near 50 barely moves the score)
- 🔹 **MACD histogram** (up to 2 pts) — confirms momentum direction independently of RSI, scaled by the histogram's size relative to ATR (a bare zero-line cross barely moves the score; a clearly expanding histogram earns close to the full 2)
- 🔹 **Trend strength** (up to 2 pts) — ADX/DMI, only awarded at all if ADX > 15 (i.e., a real trend actually exists), then scaled by how far above 15 it sits (a bare 15.1 barely counts; ADX 35+ earns close to the full 2)

The winning side's total becomes the **score** (out of 10), and the score maps to a **grade**:

| Score | Grade |
|---|---|
| ≥ 9.0 | A+ |
| ≥ 7.0 | A |
| ≥ 5.0 | B |
| < 5.0 | C |

You set a **minimum grade** in settings — signals graded below that are simply not shown.

**Every signal, regardless of grade, must also pass always-on requirements that sit outside the score entirely:** the immediate chart-timeframe trend (Fast EMA vs. Slow EMA) must agree with the signal's direction, ADX/DMI must show a genuinely strong trend (ADX > 15) somewhere in the market, price must show a genuine breakout beyond recent price action (see [Core Signal](#-core-signal-always-on-breakout-requirement) below), and volatility must be within a normal range — not an abnormal spike or a dead session (see [Volatility regime gate](#-volatility-regime-gate) below). Local Trend and ADX/DMI already contribute points to the score above, but on their own those points don't guarantee much: RSI + MACD + ADX/DMI can total 6 of the 10 points with zero help from either trend factor, which is enough to clear the default "B" threshold (5.0) purely on momentum — even while the chart is trending the opposite way. These checks close that gap, so a B-grade signal can no longer fire against the immediate trend, in a flat/choppy market with no real trend strength, without decisive price action behind it, or during volatility conditions that make the whole read less trustworthy, no matter how the rest of the score adds up. None of these four is a toggle — they apply unconditionally to every signal, regardless of grade.

**The ADX "strong trend" threshold was loosened from 20 to 15** to increase how often setups qualify — 20 was blocking moderately-trending markets outright regardless of how strong every other factor scored. 15 still screens out genuinely flat/choppy conditions, just with a bit more room. This isn't user-adjustable yet (it's a plain value in the script, not an input) — let me know if you'd like it exposed as a setting.

**The higher-timeframe bias has a lighter version of this same requirement, applied only to B-grade signals:** A and A+ signals already have very high overall confluence and are allowed to bypass it. Requiring HTF agreement unconditionally (for every grade) was tried first and rolled back — the HTF EMA cross is slow to update relative to the immediate trend and momentum, so it blocked genuinely good reversals without meaningfully filtering out bad ones. Restricting it to B-grade signals keeps the check where it likely matters most (marginal setups) without blocking the strongest ones on a lagging HTF snapshot.

**Table note:** the info table's **Score** row reflects this combined direction (all 5 factors). The **Trend** row, however, shows the **higher-timeframe bias specifically** — a standalone "bigger picture" read, independent of the local chart's tactical indicators (RSI, MACD, ADX). This is intentional: Score tells you how strongly everything currently agrees, while Trend tells you what the bigger picture looks like on its own, even if the local score is fighting it.

---

## 🎯 Core Signal (always-on breakout requirement)

The confluence score alone can flip direction purely because a lagging indicator crossed, with no real price action behind it. Core Signal closes that gap: every signal, any grade, additionally requires price to **close beyond the recent N-bar high/low** (**Core Signal Lookback (bars)**, default 5) in the signal's direction — not the score, not the other filters, but the actual price action itself confirming the move.

On top of that, the breakout has to be decisive, not just barely there. **Core Signal Breakout Margin (x ATR)** (default 0.1x) requires the close to clear that high/low by a real margin, scaled to the instrument/timeframe via ATR — a close that ekes past the level by a single tick doesn't count as confirmed.

This isn't a toggle — it applies unconditionally to every signal, same as the Local Trend and ADX/DMI hard requirements described earlier. Score grades quality; Core Signal decides timing and demands genuine price action behind it.

---

## 🌪️ Volatility regime gate

Abnormal volatility makes any signal less trustworthy than usual, regardless of how the rest of the score adds up — a news spike or gap can produce wild, unreliable readings, and a dead/illiquid session can produce breakouts that don't hold. This gate blocks signals outright at either extreme, using the same ATR-vs-its-own-average ratio already computed for [volatility-scaled stops](#-volatility-scaled-stops-on-by-default) below.

- **Extreme High Volatility Ratio** (default 2.5x) — signals are blocked when current ATR is this many times its own recent average or higher.
- **Extreme Low Volatility Ratio** (default 0.35x) — signals are blocked when current ATR is this fraction of its own recent average or lower.

These thresholds are deliberately much wider than the Low/Normal/High classification used for SL/TP sizing — that one's tuned to catch "somewhat more volatile than usual" for scaling purposes; this one's meant to catch genuinely abnormal conditions, not everyday fluctuation. Like the other checks in this section, this isn't a toggle — it applies unconditionally to every signal.

---

## 📈 Entry, stop loss & 3 take-profits

Once a signal fires, the script plots:

- 🔵 **Entry** — the close of the signal bar (solid blue line)
- 🟥 **Stop loss** — ATR-based distance from entry (red line)
- **TP1 / TP2 / TP3** — three ATR-based targets, dashed and drawn in a neutral color that auto-adapts to your chart theme (default ~1x / 2x / 3x ATR, giving roughly 1:1, 1:2, 1:3 risk:reward)

All four ATR multiples are adjustable in settings. All colors on the chart (entry/SL/TP lines, the signal badge, EMA lines, and the info table) are chosen to hold contrast on both light and dark TradingView themes — nothing is tuned for one theme only.

### 🌡️ Volatility-scaled stops (on by default)

The table's **Volatility** reading (Low / Normal / High, based on current ATR vs. its own average) doesn't just sit there for reference — it actively scales your SL/TP distances:

- **High volatility** → stops and targets widen automatically (default 1.5x), so noise doesn't stop you out early
- **Low volatility** → stops and targets tighten (default 0.75x), so targets aren't unrealistically far for a quiet market
- **Normal** → no adjustment (1.0x)

You can disable this and use fixed ATR multiples instead if you prefer consistency over adaptiveness.

### 🔒 One trade at a time (on by default)

By default, the confluence score keeps evaluating every bar regardless of whether a setup is already active — this setting decides what happens if a new qualifying signal shows up before the current one has resolved.

- **Lock New Signals Until TP1 or SL Hit** (on by default): while a setup is active and hasn't yet hit TP1 or SL, new signals are simply ignored — the current trade is left to run toward TP2/TP3 uninterrupted. Once it's hit TP1 (first profit locked in) or SL (stopped out), the script becomes eligible to open a new setup again, and a new qualifying signal will override the current one.
- **Off**: any qualifying signal immediately overrides the current setup, regardless of how far it's progressed — the original behavior.

There's a deliberate one-bar lag built in: eligibility to override starts the bar *after* TP1/SL is actually touched, not the same bar, so this stays non-repainting.

### 🛡️ Confirmed-close stop loss (on by default)

A sharp pullback right after entry can wick through the SL intrabar and reverse immediately after — stopping the trade out on noise rather than a genuine breakdown.

- **Require Confirmed Close for Stop Loss** (on by default): the SL only counts as hit once a bar actually *closes* beyond it, instead of the instant a wick touches it. Cuts down on fake-outs from brief pullbacks.
- **Off**: the original behavior — SL triggers the instant price wicks through it intrabar, no confirmation wait.

Trade-off either way: with it on, a genuine breakdown confirms one bar later, so the realized loss can run a little past the displayed SL price. TP1/TP2/TP3 are unaffected by this setting — profit-taking still triggers the instant price touches a target, since eagerly locking in gains carries the opposite risk from eagerly cutting losses.

---

## 🔔 Alerts

> 📋 **Plan requirement:** all six conditions below — including "Any alert() function call" — are TradingView **technical alerts** (alerts driven by an indicator/script), which require at least the **Essential** plan. TradingView's free plan only supports basic price alerts and doesn't include any technical alerts, so these won't be available until you upgrade.

Six alert conditions are built in, each pickable individually in TradingView's "Create Alert" dialog:

- ✅ Buy Signal
- ✅ Sell Signal
- ✅ Buy or Sell Signal (combined)
- 🎯 TP1 / TP2 / TP3 Hit (each fires once, the first time price touches that level)
- 🛑 Stop Loss Hit

There's also a dynamic `alert()` message (selectable as "Any alert() function call") that includes the ticker, grade, star rating, and exact price — useful if you're piping alerts into a bot or webhook.

### 👀 What to look for — example alert previews

Here's what will actually land in your alert feed / notifications once a condition fires (example prices shown, yours will reflect the live market):

> **GradeRunner: Buy Signal**
> BUY signal (A ★★) on XAUUSD
> Entry: 4020.975
> TP1: 4024.660
> TP2: 4028.345
> SL: 4017.290

> **GradeRunner: Sell Signal**
> SELL signal (B ★) on XAUUSD
> Entry: 4016.690
> TP1: 4012.405
> TP2: 4008.120
> SL: 4020.975

TP3 is deliberately left out of this message — by the time price gets that far, you should already be watching the chart to manage it, so it's kept off the initial entry alert to stay short.

> **GradeRunner: TP1 Hit**
> XAUUSD: TP1 hit @ 4025.260

> **GradeRunner: TP2 Hit**
> XAUUSD: TP2 hit @ 4029.545

> **GradeRunner: TP3 Hit**
> XAUUSD: TP3 hit @ 4033.829

> **GradeRunner: Stop Loss Hit**
> XAUUSD: Stop Loss hit @ 4016.690

If you select the named conditions (Buy Signal, Sell Signal, TP1/TP2/TP3 Hit, Stop Loss Hit) in Create Alert, the message uses TradingView's own placeholders instead (e.g. "XAUUSD: TP1 hit at 4025.26"). Pick "Any alert() function call" if you want the richer version above with grade and star rating included.

> ⚠️ **Important: clear the Message box.** When you pick "Any alert() function call," TradingView auto-fills the Message field with a generic description (the indicator's name plus every input value, ending in "Any alert() function call") — that's what shows up in your notifications/webhook if you leave it as-is, instead of the actual message above. Delete everything in the Message box and leave it **blank** so TradingView uses the real message from the script's `alert()` call. This trips people up because the auto-filled text looks like a normal value, not a placeholder.

---

## ⏰ Alert Hours (optional)

By default, alerts can fire any time a signal or TP/SL hit occurs, 24/7. Turn on **Only Alert During Selected Hours** if you only want to be pinged during your own trading hours (e.g. skip overnight or the session you don't trade):

- **Trading Hours (session)** — TradingView's native session picker, including which days of the week are active, not just a time range.
- **Timezone** — a dropdown of common timezones (Exchange default, UTC, New York, London, Tokyo, Sydney, etc.) rather than a free-text field, so you don't need to know your IANA timezone string.

This setting only gates the **alerts** — the confluence score, badge, table, and drawn entry/SL/TP levels keep updating on the chart around the clock either way. You'll never lose visibility on the chart itself; you'll just stop getting notified outside your chosen hours.

---

## 🕰️ Setup history

By default, only the most recent setup is shown on the chart. Turn on **Show Previous Setups** to keep a trail of past signal markers so you can visually audit how the last several setups played out. Everything else — the entry line/label, SL and TP1–3 levels, and the "TP1/TP2/TP3/SL hit" tags — is always cleared down to just the current, live setup the moment a new signal fires. Only the small direction + grade badge (e.g. `▲ BUY B ★`) persists across past setups, keeping history clean instead of piling up old price levels and hit tags.

---

## 📊 Grade win-rate stats (on by default)

Grading logic is only useful if it's actually calibrated correctly — rather than just reasoning about that in the abstract, **Show Grade Win-Rate Stats** adds a small panel (bottom-right of the chart) that tracks it directly: for each grade (A+/A/B/C), how many setups reached TP1 before hitting SL, versus the reverse.

**Win** is defined simply as reaching TP1 before SL — the first real checkpoint every setup has to clear. A trade that later gives back profit and stops out *after* already reaching TP1 still counts as a win; a trade that never gets there before SL hits counts as a loss. The panel shows each grade as `wins/total (win%)`, e.g. `B: 7/10 (70.0%)`, or `—` if that grade hasn't had any setups yet.

This is pure measurement — turning it on or off doesn't change signal behavior, entries, or exits at all. It only reflects trades that have actually occurred live on the chart since the script was added; it isn't a backtest and doesn't reflect anything before that. If a grade's win rate looks off after a reasonable sample size, that's a signal to revisit the scoring or gates above — not something the script corrects for automatically.

Sized by the same **Table Text Size** setting as the info table (see [Display](#-key-settings-at-a-glance) below), so it shrinks along with it on phone screens.

---

## 🔍 Per-factor diagnostics (Data Window)

The badge on the chart only shows a signal's final grade (e.g. "SELL B ★"), not which factors actually drove it — so figuring out why a specific past setup graded the way it did (and, in hindsight, why it won or lost) meant no visibility into the breakdown. To fix that, every factor's net point contribution is plotted with `display=display.data_window` — invisible on the chart, price scale, and legend, but visible the moment you hover any bar and open TradingView's **Data Window** panel (right sidebar).

Seven values are exposed this way, all for the specific bar you're hovering (so you can scrub back to any past signal, not just the latest one):

- **Diag: Local Trend / HTF Bias / RSI Momentum / MACD Histogram / Trend Strength (pts)** — each factor's net contribution (bull points minus bear points), all on the same −2 to +2 scale so they're directly comparable at a glance. A value near 0 means that factor was roughly neutral/undecided on that bar.
- **Diag: Confluence Score** — the total score (out of 10) on that bar.
- **Diag: Direction** — 1 = bullish, −1 = bearish, 0 = no clear direction.

This is pure read-only instrumentation — it doesn't change scoring, gating, or signal behavior in any way, it just exposes numbers that were already being computed internally.

---

## 🛠️ Key settings at a glance

Settings that only matter when a related toggle is on (the Low/High volatility multipliers, the Alert Hours session/timezone, and "Max Setups to Keep") automatically grey out in the panel until you enable that toggle — so the settings list stays relevant to what you've actually turned on, instead of showing every option all the time.

**Trend**
- Fast / Slow EMA lengths
- Higher-timeframe bias — see the timeframe tip below

**Momentum / Strength**
- RSI length, ADX/DMI length, MACD fast/slow/signal

**Volatility**
- ATR length, ATR averaging length (defines what counts as Low/Normal/High)

**Signal / Risk**
- Minimum grade to show a signal (C / B / A / A+)
- SL and TP1–3 ATR multiples
- Core Signal lookback + breakout margin (x ATR) — always on, not a toggle
- Volatility scaling toggle + Low/High multipliers
- Extreme high/low volatility ratio (volatility regime gate) — always on, not a toggle
- Lock new signals until TP1 or SL hit (one-trade-at-a-time toggle)
- Require confirmed close for stop loss (toggle)

**Alert Hours**
- Only alert during selected hours (toggle)
- Trading hours session (with day-of-week picker)
- Timezone (dropdown)

**Display**
- Show info table, signal markers, EMA lines
- Table text size (Tiny / Small / Normal / Large) — applies to both the info table and the grade win-rate stats table; handy for shrinking things down on phone screens
- Show previous setups + how many to keep
- Show grade win-rate stats (toggle)

---

## 💡 Tips

- **Pair your chart timeframe with the HTF input.** As a rule of thumb, keep the higher-timeframe input roughly 6–12x your chart's timeframe. A 1H bias filter on a 1m chart is too slow to be meaningful — try 15–30min HTF instead. On a 5–15min chart, the 60min default is a reasonable starting point.
- **Start with the minimum grade at "B"** and tighten to "A" or "A+" if you want fewer, higher-conviction signals. Loosening to "C" will show everything, including weak/mixed setups.
- **If signals feel too infrequent, loosen Core Signal Lookback or Breakout Margin.** Since Core Signal's breakout requirement now always applies, a shorter lookback or a smaller ATR margin (down to 0) will let signals fire sooner — at the cost of less decisive breakout confirmation.
- **Watch the Volatility row.** A signal firing during "High" volatility has meaningfully wider stops — factor that into position sizing.
- **Test before you trust it.** Run it on your instrument and timeframe with alerts on for a while before using it to size real trades.
- **Watch the Grade Win-Rate Stats panel over time**, not after just a handful of setups. A grade's win rate needs a reasonable sample size before it means much — a couple of trades either way is noise, not signal.
- **On a phone?** Set Table Text Size to "Tiny" or "Small" so the info table doesn't crowd a smaller chart.

---

## ⚠️ Things to look out for

- This indicator does **not** predict the future and does **not** guarantee profitable trades. Past signal behavior is not indicative of future results.
- On very low timeframes (1m), spread and slippage can meaningfully eat into tighter stops, especially in "Low volatility" mode where distances shrink.
- The higher-timeframe bias uses `barmerge.lookahead_off` and only updates on confirmed HTF bars — expected non-repainting behavior, but it does mean the HTF read can lag behind fast intraday moves.
- This is a **technical analysis tool**, not financial advice. Always apply your own risk management and trade at your own discretion.

---

**Disclaimer:** The information and publications provided by this script are not intended to be, and do not constitute, financial, investment, trading, or other advice or recommendations of any kind. Trading involves risk, and you are solely responsible for your own trading decisions.
