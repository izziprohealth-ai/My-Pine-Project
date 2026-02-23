# Pine Script Indicator Audit Report

This report reviews all uploaded indicator files for potential failure points, repaint risk, runtime/performance issues, and trader-facing quality improvements.

## Scope Reviewed
- `GANNlines.pine`
- `MAJORBULLSCORE.pine`
- `OSOB.pine`
- `RRCalculator.pine`
- `SUTC.pine`
- `ZZ.DUAL MACD.pine`
- `ZZ_EntryExit.pine`
- `ZZintradayScanner.pine`

---

## 1) GANNlines.pine
### Findings
- **Low risk / code hygiene:** `src = input(close)` is declared but never used. This does not break runtime, but suggests stale logic and can confuse maintainers.
- **Maintainability risk:** The script uses a very long list of hard-coded `hline()` calls; this is hard to maintain and easy to introduce mistakes when editing line sets.
- **Version debt:** Script is still Pine v4 while most others are v5/v6.

### Suggestions
- Remove unused `src` input.
- Migrate to Pine v6 and generate levels via arrays + loop + `line.new` (if dynamic) or grouped constants (if static).
- Add presets (e.g., conservative/standard/aggressive spacing) so traders can switch quickly.

---

## 2) ZZ.DUAL MACD.pine
### Findings
- **Medium risk / repaint behavior:** Weekly MACD is requested directly from higher timeframe (`request.security(..., "W", ta.macd(...))`) without explicit handling of confirmed weekly bar behavior. On intraday charts, users may see values move during an unfinished weekly candle.
- **Low risk / dead code:** `weeklyClose` is fetched but never used.

### Suggestions
- Add an option: `Use confirmed weekly values only` and implement with confirmed-bar logic.
- Remove unused `weeklyClose` variable.
- Add alert conditions for weekly cross confirmation to make it trader-ready.

---

## 3) MAJORBULLSCORE.pine
### Findings
- **Medium risk / unstable ratio:** `volX = volume / ta.sma(volume, 20)` can become `na` or unstable on early bars / sparse symbols.
- **Medium risk / HTF consistency:** Scanner `request.security` calls do not explicitly pass gap/lookahead parameters. Defaults are usually OK, but explicit settings improve consistency and avoid future regression when script evolves.
- **High risk / chart clutter + object pressure:** When universe is empty, `label.new(...)` is created without guard and can spawn repeatedly.

### Suggestions
- Guard `volX` with denominator fallback (`math.max(ta.sma(volume, 20), 1)` or `nz(...)`).
- Make `request.security` parameters explicit (`barmerge.gaps_off`, `barmerge.lookahead_off`) for all scanner calls.
- Replace repeated empty-universe label creation with a single `var label` object updated/deleted safely.
- Consider caching heavy calculations and only refreshing scan table on `barstate.islast`.

---

## 4) OSOB.pine
### Findings
- **Low risk / state behavior:** PVI is stateful (`var float pvi`), which is correct conceptually, but no session/reset mode exists; intraday traders may want day-reset behavior.
- **Low risk / normalization artifact:** OBV normalization uses fixed denominator floor (`1`) which can distort normalized output on very low-activity symbols.

### Suggestions
- Add optional reset mode: `Continuous` vs `Session Reset` for PVI/flow components.
- Use symbol-aware floor (`syminfo.mintick` style for price domains or dynamic epsilon for volume domains).
- Add one-line status output (e.g., `FLOW: A/B/C`) for easier trader interpretation.

---

## 5) RRCalculator.pine
### Findings
- **High risk / nested `request.security` architecture:** `f_scan()` contains an internal daily `request.security(...)`, and `f_scan()` is then called again through external `request.security(sym, tf, f_scan())` in the scanner loop. This pattern is expensive and can become fragile in future refactors.
- **High risk / invalid symbol handling:** Main scan call does not set `ignore_invalid_symbol=true`, so one bad symbol may break scan behavior.
- **Technical debt signal:** Inline comment indicates intended cleanup was not completed.

### Suggestions
- Refactor `f_scan()` to be **pure** (no internal `request.security`). Pull all higher-timeframe values outside and pass them in.
- Add `ignore_invalid_symbol = true` to scanner `request.security` calls.
- Split this script into modules: (1) trade calculator, (2) symbol scanner, (3) table renderer.
- Add a strict “safe mode” input to disable non-essential table rows for performance on low-end devices.

---

## 6) SUTC.pine
### Findings
- **Low risk / UX confusion:** Two different groups share the same title text (`"🎛️ Overlay Engine"`), which can confuse settings layout.
- **Low risk / visual scaling:** `plot(useDCBB ? closeC / 2 : na, ...)` halves curve value; this may look visually detached from price for some symbols and may confuse users.

### Suggestions
- Rename the second group to a unique heading (e.g., `"🎛️ S.UTC Signal Toggles"`).
- Make DCBB curve scaling configurable (`Auto`, `1x`, `0.5x`) with tooltip.
- Add concise mode descriptions (“trend-follow”, “early reversal filter”, etc.) for professional usability.

---

## 7) ZZ_EntryExit.pine
### Findings
- **Medium risk / object count pressure:** Multiple `label.new(...)` calls across events can create many objects over long history and may hit platform limits or reduce responsiveness.
- **Medium risk / performance:** Frequent `array.includes` checks on large hard-coded symbol lists run each bar and can be optimized.

### Suggestions
- Convert repetitive event labels to controlled lifecycle labels (`var label` + update/delete), or gate historical plotting.
- Cache symbol classification results once (`barstate.isfirst`) where possible.
- Consider a “lightweight mode” toggle for intraday charts to disable non-critical overlays.

---

## 8) ZZintradayScanner.pine
### Findings
- **Medium risk / repaint-consistency risk:** Scanner and chart-level `request.security` calls use defaults without explicit lookahead/gaps configuration.
- **Low risk / throughput limit mismatch:** UI text says manual mode supports up to 15 symbols, but current manual input set only defines 5 symbols and scan cap is 5.
- **Low risk / repeated expensive calls:** `chartESS` is requested inside rendering block; this can be computed once alongside other chart metrics for clarity/performance.

### Suggestions
- Standardize all `request.security` calls with explicit gap/lookahead policy.
- Align UX text with actual capability (or extend manual inputs to intended count).
- Build a compact scoring struct/function so chart row and scanner rows share the same calculated snapshot.

---

## Cross-Indicator Professional Upgrade Plan (Recommended)
1. **Repaint policy standardization**
   - Add a global input for HTF behavior (`Live HTF` vs `Confirmed HTF`) and apply consistently.
2. **Security call hygiene**
   - Ensure every scanner `request.security` has explicit `gaps/lookahead` and `ignore_invalid_symbol` where applicable.
3. **Performance tiers**
   - Add `Full / Balanced / Lightweight` mode in heavy scripts (`ZZ_EntryExit`, `RRCalculator`, scanners).
4. **Table/label lifecycle management**
   - Use persistent objects (`var`) with update/delete instead of frequent creation.
5. **Codebase consistency**
   - Migrate remaining v4/v5 scripts to v6, harmonize naming, and remove dead code/unused inputs.

---

## Priority Fix Order
1. **RRCalculator** nested security + invalid symbol handling.
2. **MAJORBULLSCORE** empty-universe label spam + safer volume ratio.
3. **ZZ.DUAL MACD / ZZintradayScanner** explicit HTF confirmation policy.
4. **ZZ_EntryExit** label/object/performance controls.
5. **SUTC / GANNlines / OSOB** quality and maintainability refinements.
