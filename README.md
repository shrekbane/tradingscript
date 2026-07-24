# 📊 GradeRunner — Trend + Momentum + Volatility + HTF Bias

<p align="center">
  <img src="./graderunner.png" alt="GradeRunner" width="320">
</p>

A Pine Script v5 indicator for TradingView that grades every setup on a 0–10 confluence score, then auto-plots an entry, stop loss, and three take-profit levels the moment a graded signal fires. Built for traders who want a structured, rules-based way to combine trend, momentum, higher-timeframe bias, and volatility into a single readable signal — instead of eyeballing five indicators separately.

Signals only fire on a **confirmed (closed) bar** — there's no repainting of the core score or signal trigger.

> This is an independent, clean-room implementation — not a copy of any private/invite-only script. Built from scratch in Pine Script v5.

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
| `graderunner.png` | Brand artwork, shown at the top of this file |

---

## ⚙️ How it works

Every bar, the script scores the **bull** side and the **bear** side independently across five factors, and whichever side is ahead becomes the active direction:

- 🔹 **Local trend** (2 pts) — fast EMA vs. slow EMA on your chart's timeframe
- 🔹 **Higher-timeframe bias** (2 pts) — same EMA relationship, computed on a separate higher timeframe you choose
- 🔹 **Momentum** (up to 2 pts) — RSI, scaled by distance from 50 (a borderline RSI near 50 barely moves the score)
- 🔹 **MACD histogram** (2 pts) — confirms momentum direction independently of RSI
- 🔹 **Trend strength** (2 pts) — ADX/DMI, but only awarded if ADX > 20 (i.e., a real trend actually exists)

The winning side's total becomes the **score**, and the score maps to a **grade**:

| Score | Grade |
|---|---|
| ≥ 9 | A+ |
| ≥ 7 | A |
| ≥ 5 | B |
| < 5 | C |

You set a **minimum grade** in settings — signals graded below that are simply not shown.

**Table note:** the info table's **Score** row reflects this combined direction (all 5 factors). The **Trend** row, however, shows the **higher-timeframe bias specifically** — a standalone "bigger picture" read, independent of the local chart's tactical indicators (RSI, MACD, ADX). This is intentional: Score tells you how strongly everything currently agrees, while Trend tells you what the bigger picture looks like on its own, even if the local score is fighting it.

**Higher-timeframe agreement is a hard requirement, not just a scored factor.** On top of contributing points to the score above, a signal is only allowed to fire in the direction the higher timeframe already agrees with — at every grade, including A+. This is deliberate: a sharp local bounce candle can score high enough to hit A/A+ on its own while still fighting the bigger-picture trend, and this gate exists specifically to block that "buy/sell the bounce against the trend" trade type. A genuinely good counter-trend reversal getting blocked here is an accepted trade-off — the goal is discipline against chasing bounces, not maximizing signal count or raw win rate.

**MACD agreement is a hard requirement too.** A signal is only allowed to fire in the direction MACD Histogram already agrees with — again, at every grade. Found by comparing factor diagnostics across several real B-grade setups: every one had MACD fully opposed (contributing nothing to the winning side) while Local Trend, HTF Bias, and ADX/DMI were all maxed in agreement — the bigger picture lined up, but the most immediate momentum read hadn't actually turned yet. Every A+/A example checked already had MACD agreeing naturally, so this isn't expected to cost much at the top end.

**Explaining a past signal's grade.** Hover any bar in TradingView's Data Window and you'll see the net point contribution (bull minus bear, on a −2 to +2 scale) for each of the five factors individually, plus the overall score and direction — invisible on the chart itself, only visible in the Data Window. Useful for scrubbing back to a specific signal and seeing exactly which factors carried its grade (e.g. a B mostly built on HTF Bias + RSI, with Local Trend and ADX/DMI contributing nothing) instead of only seeing the final badge.

---

## 🎯 Core Signal (optional breakout gate)

By default, a signal fires the moment the confluence score flips direction and clears your grade filter. That's fast, but it means the *trigger* is purely indicator-based — the score can flip without price actually doing anything abrupt.

Turn on **Core Signal** and the script adds one more requirement: price must **close beyond the recent N-bar high/low** (lookback is adjustable) in the signal's direction before it's allowed to fire. Practically:

- **Off** (default): score does the grading *and* the triggering. Faster, more signals, more noise.
- **On**: score grades quality, Core Signal decides timing. Slower, fewer signals, each one backed by an actual breakout.

There's no universally "correct" setting here — it's a real trade-off between speed and confirmation, so it's worth testing both on your instrument and timeframe.

---

## 🧱 Support/Resistance flip confirmation (on by default)

The script tracks the most recent confirmed swing high and swing low (pivot-based, lookback adjustable via **Pivot Lookback**). Once price closes through one of those levels, it "flips" polarity:

- Old **resistance** broken → becomes a candidate **support** (**RBS** — resistance-becomes-support — look for buys)
- Old **support** broken → becomes a candidate **resistance** (**SBR** — support-becomes-resistance — look for sells)

A **reject** is a later bar that wicks back into that flipped zone but still closes back away from it — price testing the level and holding, rather than reclaiming it.

With **Require S/R Flip Reject to Confirm Entry** on (default), a qualifying signal doesn't fire the instant the confluence score/HTF/MACD/Core Signal conditions line up — its direction and grade are locked in as "pending," and the actual signal (alert, entry, badge, SL/TP) only fires once a matching reject shows up within the **Confirmation Window** (default 10 bars). If no reject shows up in time, that candidate is dropped silently — no signal, no alert — and a fresh attempt starts checking again the next bar as long as the underlying conditions still hold.

This is a *delayed* confirmation, not a same-bar requirement like HTF Bias or MACD: the confluence read and the price-action reject don't need to land on the same bar, only within the window. Entry price is still the close of whichever bar the signal actually fires on (the confirmation bar, not the original candidate bar) — never the zone price itself, since by the time a reject is confirmed, price has already moved away from that level.

Turn on **Plot S/R Zone Lines on Chart** (default on) to see the tracked zones directly — dotted circles that switch color once a zone flips polarity.

This only tracks a single zone in each direction at a time (the most recent pivot), not a running history of every past level — a deliberate simplification, not a full support/resistance mapping tool.

---

## 📈 Entry, stop loss & 3 take-profits

Once a signal fires, the script plots:

- 🔵 **Entry** — the close of the signal bar (solid blue line)
- 🟥 **Stop loss** — ATR-based distance from entry (red line)
- **TP1 / TP2 / TP3** — three ATR-based targets, dashed and drawn in a neutral color that auto-adapts to your chart theme (default 2x / 3x / 4x ATR against a 1x ATR stop, giving roughly 2:1, 3:1, 4:1 risk:reward)

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

### 🔐 Breakeven after TP1

The moment TP1 hits, the stop loss automatically moves to your entry price — the trade can no longer turn into a net loss from that point on. This isn't a reminder you have to act on; the script's own SL tracking moves with it, so the drawn SL line on the chart shifts to entry and any future alert checks against the new breakeven level, not the original stop.

Because of this, if price comes back and touches that level afterward, it's not really a "stop loss" anymore — the label and alert message both say **"BE hit"** / **"Breakeven hit"** instead of "SL hit," so you're not caught off guard thinking the trade lost when it actually just closed flat after already banking TP1.

---

## 🔔 Alerts

Six alert conditions are built in, each pickable individually in TradingView's "Create Alert" dialog:

- ✅ Buy Signal
- ✅ Sell Signal
- ✅ Buy or Sell Signal (combined)
- 🎯 TP1 / TP2 / TP3 Hit (each fires once, the first time price touches that level)
- 🛑 Stop Loss Hit

There's also a dynamic, multi-line `alert()` message (selectable as "Any alert() function call") that spells out the entry, TP1, TP2, and SL levels on their own lines — so the exact numbers to act on are readable straight from the notification, without needing to cross-check the chart. This matters in practice: if the indicator is later removed and re-added (or the chart otherwise reloads), Pine recalculates history against each bar's *settled* close, which can drift a pip or two from the *live* close that was actually used when the alert fired — so the chart's redrawn entry/SL/TP lines aren't guaranteed to exactly match what a past alert said. The alert message itself is the authoritative record of what actually fired.

The dynamic TP1 Hit message also confirms the stop moving to breakeven, and — if price later comes back to that level — the dynamic Stop Loss Hit message says "Breakeven hit" instead of "Stop Loss hit," so you're not misled into thinking a trade lost when it actually already banked TP1 first. The *named* alert conditions (picked individually in Create Alert, rather than "Any alert() function call") keep their generic TradingView-placeholder wording regardless — this richer, context-aware phrasing only shows up in the dynamic message.

### 👀 What to look for — example alert previews

Here's what will actually land in your alert feed / notifications once a condition fires (example prices shown, yours will reflect the live market):

> **GradeRunner: Buy Signal**
> BUY signal (A ★★) on XAUUSD
> Entry: 4020.975
> TP1: 4029.545
> TP2: 4033.830
> SL: 4016.690

> **GradeRunner: Sell Signal**
> SELL signal (B ★) on XAUUSD
> Entry: 4016.690
> TP1: 4008.120
> TP2: 4003.835
> SL: 4020.975

> **GradeRunner: TP1 Hit**
> XAUUSD: TP1 hit @ 4029.545

> **GradeRunner: TP2 Hit**
> XAUUSD: TP2 hit @ 4033.830

> **GradeRunner: TP3 Hit**
> XAUUSD: TP3 hit @ 4038.115

> **GradeRunner: Stop Loss Hit**
> XAUUSD: Stop Loss hit @ 4016.690

TP3 is intentionally left out of the Buy/Sell alert message — by the time price reaches it, you should already be watching the chart.

If you select the named conditions (Buy Signal, Sell Signal, TP1/TP2/TP3 Hit, Stop Loss Hit) in Create Alert, the message uses TradingView's own placeholders instead (e.g. "XAUUSD: TP1 hit at 4025.26"). Pick "Any alert() function call" if you want the richer version above with grade and star rating included.

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

## 📊 Grade win-rate stats

A small second panel (bottom-right, on by default via **Show Grade Win-Rate Stats**) tracks, per grade (A+/A/B/C), how many setups reached TP1 before hitting SL vs. the reverse. It's pure measurement — it doesn't feed back into the score, grading, or signal logic in any way, it just gives you a running scoreboard to sanity-check whether the grading is actually working the way it's supposed to (e.g. are A+ setups really winning more often than C setups?).

Each trade's outcome is tallied against the grade it entered at, not whatever the grade has drifted to by the time it resolves. A row reads as `wins/total (win%)`, or `—` until a grade has at least one closed setup.

---

## 🛠️ Key settings at a glance

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
- Core Signal toggle + lookback
- Volatility scaling toggle + Low/High multipliers
- Lock new signals until TP1 or SL hit (one-trade-at-a-time toggle)

**S/R Confirmation**
- Pivot Lookback (bars each side)
- Plot S/R Zone Lines on Chart
- Require S/R Flip Reject to Confirm Entry (the delayed confirmation gate)
- Confirmation Window (bars)

**Alert Hours**
- Only alert during selected hours (toggle)
- Trading hours session (with day-of-week picker)
- Timezone (dropdown)

**Display**
- Show info table, signal markers, EMA lines
- Show previous setups + how many to keep
- Show Grade Win-Rate Stats panel
- Table Text Size (Tiny / Small / Normal / Large) — applies to both tables

---

## 💡 Tips

- **Pair your chart timeframe with the HTF input.** As a rule of thumb, keep the higher-timeframe input roughly 6–12x your chart's timeframe. A 1H bias filter on a 1m chart is too slow to be meaningful — try 15–30min HTF instead. On a 5–15min chart, the 60min default is a reasonable starting point.
- **Start with the minimum grade at "B"** and tighten to "A" or "A+" if you want fewer, higher-conviction signals. Loosening to "C" will show everything, including weak/mixed setups.
- **If signals feel too reactive, try Core Signal.** It trades signal frequency for confirmation — good for choppier instruments, less useful on trending ones where you want to catch the move early.
- **Watch the Volatility row.** A signal firing during "High" volatility has meaningfully wider stops — factor that into position sizing.
- **Test before you trust it.** Run it on your instrument and timeframe with alerts on for a while before using it to size real trades.
- **Give the Grade Win-Rate Stats panel a real sample size before trusting it.** A handful of closed setups per grade can look great or terrible by chance — wait until each grade has a reasonable number of trades before drawing conclusions.
- **On a phone or small screen, drop Table Text Size to Tiny or Small.** The default size is tuned for a desktop chart and can crowd a small display — this setting shrinks both tables at once.
- **If signals feel too rare with the S/R gate on, widen the Confirmation Window.** A confluence read that never gets a reject within the window is dropped entirely rather than firing late — widening the window gives price more room to test the zone before that candidate expires.

---

## ⚠️ Things to look out for

- This indicator does **not** predict the future and does **not** guarantee profitable trades. Past signal behavior is not indicative of future results.
- With the S/R flip confirmation gate on, a strong confluence read can go completely unfired if price never rejects the tracked zone within the Confirmation Window — this is by design (no reject, no trade), but it does mean you'll see fewer signals than the raw confluence score alone would produce.
- On very low timeframes (1m), spread and slippage can meaningfully eat into tighter stops, especially in "Low volatility" mode where distances shrink.
- The higher-timeframe bias uses `barmerge.lookahead_off` and only updates on confirmed HTF bars — expected non-repainting behavior, but it does mean the HTF read can lag behind fast intraday moves.
- This is a **technical analysis tool**, not financial advice. Always apply your own risk management and trade at your own discretion.

---

**Disclaimer:** The information and publications provided by this script are not intended to be, and do not constitute, financial, investment, trading, or other advice or recommendations of any kind. Trading involves risk, and you are solely responsible for your own trading decisions.
