# 📊 GradeRunner — Trend + Momentum + Volatility + HTF Bias

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

---

## 🎯 Core Signal (optional breakout gate)

By default, a signal fires the moment the confluence score flips direction and clears your grade filter. That's fast, but it means the *trigger* is purely indicator-based — the score can flip without price actually doing anything abrupt.

Turn on **Core Signal** and the script adds one more requirement: price must **close beyond the recent N-bar high/low** (lookback is adjustable) in the signal's direction before it's allowed to fire. Practically:

- **Off** (default): score does the grading *and* the triggering. Faster, more signals, more noise.
- **On**: score grades quality, Core Signal decides timing. Slower, fewer signals, each one backed by an actual breakout.

There's no universally "correct" setting here — it's a real trade-off between speed and confirmation, so it's worth testing both on your instrument and timeframe.

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

---

## 🔔 Alerts

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

## 🛠️ Key settings at a glance

Settings that only matter when a related toggle is on (Core Signal's lookback, the Low/High volatility multipliers, the Alert Hours session/timezone, and "Max Setups to Keep") automatically grey out in the panel until you enable that toggle — so the settings list stays relevant to what you've actually turned on, instead of showing every option all the time.

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

**Alert Hours**
- Only alert during selected hours (toggle)
- Trading hours session (with day-of-week picker)
- Timezone (dropdown)

**Display**
- Show info table, signal markers, EMA lines
- Show previous setups + how many to keep

---

## 💡 Tips

- **Pair your chart timeframe with the HTF input.** As a rule of thumb, keep the higher-timeframe input roughly 6–12x your chart's timeframe. A 1H bias filter on a 1m chart is too slow to be meaningful — try 15–30min HTF instead. On a 5–15min chart, the 60min default is a reasonable starting point.
- **Start with the minimum grade at "B"** and tighten to "A" or "A+" if you want fewer, higher-conviction signals. Loosening to "C" will show everything, including weak/mixed setups.
- **If signals feel too reactive, try Core Signal.** It trades signal frequency for confirmation — good for choppier instruments, less useful on trending ones where you want to catch the move early.
- **Watch the Volatility row.** A signal firing during "High" volatility has meaningfully wider stops — factor that into position sizing.
- **Test before you trust it.** Run it on your instrument and timeframe with alerts on for a while before using it to size real trades.

---

## ⚠️ Things to look out for

- This indicator does **not** predict the future and does **not** guarantee profitable trades. Past signal behavior is not indicative of future results.
- On very low timeframes (1m), spread and slippage can meaningfully eat into tighter stops, especially in "Low volatility" mode where distances shrink.
- The higher-timeframe bias uses `barmerge.lookahead_off` and only updates on confirmed HTF bars — expected non-repainting behavior, but it does mean the HTF read can lag behind fast intraday moves.
- This is a **technical analysis tool**, not financial advice. Always apply your own risk management and trade at your own discretion.

---

**Disclaimer:** The information and publications provided by this script are not intended to be, and do not constitute, financial, investment, trading, or other advice or recommendations of any kind. Trading involves risk, and you are solely responsible for your own trading decisions.
---

## 🎯 Core Signal (optional breakout gate)

By default, a signal fires the moment the confluence score flips direction and clears your grade filter. That's fast, but it means the *trigger* is purely indicator-based — the score can flip without price actually doing anything abrupt.

Turn on **Core Signal** and the script adds one more requirement: price must **close beyond the recent N-bar high/low** (lookback is adjustable) in the signal's direction before it's allowed to fire. Practically:

- **Off** (default): score does the grading *and* the triggering. Faster, more signals, more noise.
- **On**: score grades quality, Core Signal decides timing. Slower, fewer signals, each one backed by an actual breakout.

There's no universally "correct" setting here — it's a real trade-off between speed and confirmation, so it's worth testing both on your instrument and timeframe.

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

---

## 🔔 Alerts

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
> BUY signal (A ★★) on XAUUSD @ 4020.975

> **GradeRunner: Sell Signal**
> SELL signal (B ★) on XAUUSD @ 4016.690

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

## 🛠️ Key settings at a glance

Settings that only matter when a related toggle is on (Core Signal's lookback, the Low/High volatility multipliers, the Alert Hours session/timezone, and "Max Setups to Keep") automatically grey out in the panel until you enable that toggle — so the settings list stays relevant to what you've actually turned on, instead of showing every option all the time.

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

**Alert Hours**
- Only alert during selected hours (toggle)
- Trading hours session (with day-of-week picker)
- Timezone (dropdown)

**Display**
- Show info table, signal markers, EMA lines
- Show previous setups + how many to keep

---

## 💡 Tips

- **Pair your chart timeframe with the HTF input.** As a rule of thumb, keep the higher-timeframe input roughly 6–12x your chart's timeframe. A 1H bias filter on a 1m chart is too slow to be meaningful — try 15–30min HTF instead. On a 5–15min chart, the 60min default is a reasonable starting point.
- **Start with the minimum grade at "B"** and tighten to "A" or "A+" if you want fewer, higher-conviction signals. Loosening to "C" will show everything, including weak/mixed setups.
- **If signals feel too reactive, try Core Signal.** It trades signal frequency for confirmation — good for choppier instruments, less useful on trending ones where you want to catch the move early.
- **Watch the Volatility row.** A signal firing during "High" volatility has meaningfully wider stops — factor that into position sizing.
- **Test before you trust it.** Run it on your instrument and timeframe with alerts on for a while before using it to size real trades.

---

## ⚠️ Things to look out for

- This indicator does **not** predict the future and does **not** guarantee profitable trades. Past signal behavior is not indicative of future results.
- On very low timeframes (1m), spread and slippage can meaningfully eat into tighter stops, especially in "Low volatility" mode where distances shrink.
- The higher-timeframe bias uses `barmerge.lookahead_off` and only updates on confirmed HTF bars — expected non-repainting behavior, but it does mean the HTF read can lag behind fast intraday moves.
- This is a **technical analysis tool**, not financial advice. Always apply your own risk management and trade at your own discretion.

---

**Disclaimer:** The information and publications provided by this script are not intended to be, and do not constitute, financial, investment, trading, or other advice or recommendations of any kind. Trading involves risk, and you are solely responsible for your own trading decisions.
