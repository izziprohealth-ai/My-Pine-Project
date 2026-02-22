# My-Pine-Project

`ZZ.EntryExit(score)` is a multi-engine Pine Script indicator that combines market structure, trend/momentum filters, session-aware scoring, and visual trade signals (Entry / Re-entry / Exit) for Bursa, US, FCPO, and Crypto symbols.

## What the indicator does

- Draws ZigZag-style wave structure (major/minor/subwave variants).
- Computes an intraday “ZZ Score” using session-aware AM/PM trend progression.
- Runs several signal engines in parallel (ZZ Core, ZZ Major V2, ZZ Minor, E2So, E2Su, IBO, BTST_PRO).
- Adds market-specific overlays and risk/context labels (e.g., PN17/GN3 lists, Shariah/non-Shariah symbol lists, timeframe risk tag).
- Shows signal markers and optional scoreboard/table UI with configurable theme and layout.

## Core architecture

1. **Market + symbol context**
   - Detects market type using `syminfo` (Bursa/US/FCPO/Crypto).
   - Maintains curated symbol registries (e.g., non-Shariah, doubtful, halal crypto, US Shariah master, PN17/GN3 distress lists).

2. **Session-aware trend scoring**
   - Session windows are market-aware (split AM/PM for Bursa, regular US session, full-day crypto).
   - Tracks session OHLC and computes percentage-based trend grades (`A+` to `F`) from open-to-close progression and range position.
   - Keeps AM/PM snapshots and combines them with PM-biased weighting for recovery-friendly intraday grading.

3. **Signal engines (toggle-based)**
   - **ZZ Core** for medium/long-term entry/re-entry/exit flow.
   - **ZZ Major V2** with WMA-vs-BB trigger and pullback-from-peak exit control.
   - **ZZ Minor** legacy short-term engine.
   - **E2So / E2Su / IBO** momentum- and EMA-based engines.
   - **BTST_PRO** (Buy Today Sell Tomorrow) with session structure + next-day gap confirmation logic.

4. **Visual + alert layer**
   - Extensive `plotshape`, bar/background coloring, and optional line overlays.
   - Master switch and per-engine toggles prevent unwanted clutter.
   - Theme system supports dark/light visual palettes and session-theme variants.

## How to read it in practice

- Start with the **master controls** and enable only one or two engines first.
- Use **ZZ Score / trend grade** as directional context (quality filter), not as standalone entry trigger.
- Treat **Entry/Re-entry/Exit** labels as engine-specific events; different engines intentionally fire at different phases.
- For Bursa/US intraday workflow, BTST and AM/PM logic are session-sensitive; timeframe and timezone matter.

## Notes

- This script is feature-rich and intentionally modular; many inputs are independent toggles.
- It is best used as a decision-support overlay, not a guaranteed signal system.
