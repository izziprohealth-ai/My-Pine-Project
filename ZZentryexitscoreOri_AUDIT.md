# ZZ.EntryExit(score) Ori Audit Notes

## Overall status
The script is feature-rich and mostly well organized by modules. I did not find an obvious compile blocker, but there are several cleanup items and a few logic-risk items worth addressing.

## Potential debris (declared but currently unused)
These symbols are declared but appear to have no later use in the file:

- `fcpoBase` (`"FCPO"`) at line 190
- `d_closePos` at line 266
- `isNewPivot` at line 775
- `sectorName` at line 2383
- `mb_majorBullText` at line 2914
- `weeklyMaruLine` at line 3038
- `monthlyMaruLine` at line 3074
- `yearlyMaruLine` at line 3110
- table row constants `ROW_HEADER`, `ROW_CONSENSUS`, `ROW_MOMENTUM`, `ROW_EZP_SCORE`, `ROW_EZP_RATING` at lines 3254–3258
- `intradayTrendCol` and `marubozuCol` at lines 3297–3298
- US session volume inputs `vmU` and `minVU` are declared at lines 2269–2270 but do not drive a `volSpikeU` condition (Bursa has this, US currently does not)

## Logic-risk items (recommended fix)
1. **Re-entry condition is assigned twice back-to-back**, and the second assignment overrides the first.
   - First assignment: line 2893
   - Immediate overwrite: line 2897
   - Why it matters: future edits to line 2893 may appear active but will not affect runtime behavior.

2. **`zzV2PullbackPct` input is currently not consumed by ZZ Major V2 exit logic**.
   - Declared at line 677.
   - Why it matters: users may think they are tuning pullback sensitivity, but the input is effectively inert.

3. **Potential symbol-case mismatch in hardcoded list**.
   - `"maybulk"` (lowercase) at line 50 while most tickers are uppercase.
   - Why it matters: if exact-case matching is used, this entry can silently fail.

## Improvement ideas
1. Convert stale hardcoded status lists into reusable helper checks (PN17/GN3/Shariah flags), with optional “last updated” display in UI.
2. Centralize market session definitions into one helper function to reduce drift between modules.
3. Run a strict cleanup pass:
   - remove unused declarations,
   - keep only one active formula per signal state,
   - move legacy alternatives to a changelog/comment block.
4. Wire `zzV2PullbackPct` into exit calculation (or remove the input) so UI always maps to behavior.
5. Add an optional diagnostics mode (single row/table): market type, active session, selected branch, and key gating booleans for faster troubleshooting.

## Suggested next step order
1. Fix the double-assignment of `e2soReentryState`.
2. Wire/retire `zzV2PullbackPct`.
3. Remove unused declarations in one cleanup commit.
4. Add diagnostics mode for easier long-term maintenance.
