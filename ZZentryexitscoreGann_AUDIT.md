# ZZ.EntryExit(score) Audit Notes

## Overall status
The script is feature-rich and structurally organized by modules. I did not find a critical compile-time blocker, but there are several **cleanup items** and a few **logic-risk items** that are worth fixing before they become maintenance issues.

## Potential debris (declared but currently unused)
These symbols are declared but appear to have no later use in the file:

- `fcpoBase` (`"FCPO"`) at line 194
- `d_closePos` at line 270
- `isNewPivot` at line 779
- `sectorName` at line 2395
- `mb_majorBullText` at line 2942
- `weeklyMaruLine` at line 3066
- `monthlyMaruLine` at line 3102
- `yearlyMaruLine` at line 3138
- table row constants `ROW_HEADER`, `ROW_CONSENSUS`, `ROW_MOMENTUM`, `ROW_EZP_SCORE`, `ROW_EZP_RATING` at lines 3282–3286
- `intradayTrendCol` and `marubozuCol` at lines 3325–3326
- US session volume inputs `vmU` and `minVU` are declared at lines 2281–2282 but do not drive a `volSpikeU` condition (Bursa has this, US currently does not)

## Logic-risk items (recommended fix)
1. **Re-entry condition is assigned twice back-to-back**, and the second assignment overrides the first.
   - First assignment: line 2921
   - Immediate overwrite: line 2925
   - Why it matters: future edits to line 2921 may appear “active” but will have no effect.

2. **`zzV2PullbackPct` input is currently not consumed by ZZ Major V2 exit logic**.
   - Declared at line 681.
   - Why it matters: user thinks they are tuning pullback sensitivity, but the input is effectively inert.

3. **Potential symbol-case mismatch in hardcoded list**.
   - `"maybulk"` (lowercase) at line 50 while most tickers are uppercase.
   - Why it matters: if comparison uses exact case, this entry can silently fail.

## Improvement ideas
1. **Convert stale hardcoded status lists into reusable helper functions and table-based checks** (PN17/GN3/Shariah flags), with optional “last updated” timestamp in UI.
2. **Centralize market session definitions** into one function returning session windows by market to reduce drift between modules.
3. **Enable strict cleanup pass**:
   - remove unused declarations,
   - keep only one active formula per signal state,
   - preserve older formulas as comments in a changelog section instead of inline duplicates.
4. **Wire `zzV2PullbackPct` into exit calculation** (or remove the input) so UI always maps to behavior.
5. **Add optional diagnostics mode** (single table row): current market type, active session, selected branch, and top gating booleans for easier troubleshooting.

## Suggested next step order
1. Fix double-assignment of `e2soReentryState`.
2. Wire/retire `zzV2PullbackPct`.
3. Remove unused symbols in one cleanup commit.
4. Add diagnostics mode for easier long-term maintenance.
