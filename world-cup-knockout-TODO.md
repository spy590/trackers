# World Cup tracker — Knockout screen: plan & check-in

Tracks the build-out of the **Knockout** tab in `world-cup-tracker-2026.html`.
Before this work the tab rendered a static grid of "TBD" slots and did nothing.

## Done (this change)

- Added `KO_BRACKET` — the official 2026 bracket structure locked at the Final
  Draw (all 31 matches, R32 #73–88 → R16 #89–96 → QF #97–100 → SF #101–102 →
  Final #104), with each slot encoded as a token (`{w}` winner, `{r}` runner-up,
  `{t:[…]}` best-third pool, `{m}` winner-of-match).
- `renderBracket()` now resolves winner/runner-up slots from the **live group
  standings** (`computeStandings`). A slot shows the real team as soon as a group
  has any finished match; a gold `·` marks it *provisional* until the group is
  mathematically clinched (all 6 group matches FT → shown in accent colour).
- Added a live **best third-placed teams** table (`renderThirds`) — ranks all 12
  third-placed sides by Pts → GD → GF and highlights the current top 8 that would
  advance.
- Wired `renderBracket()` into `renderAll()` (30s live poll) and the 60s clock
  fallback so the bracket updates as results come in.

## TODO — CHECK IN AFTER THE GROUP STAGE ENDS (group stage runs to **Sat 27 Jun 2026**; R32 starts **Sun 28 Jun**)

1. **Third-place → R32 slot assignment.** Third-place slots currently render as a
   label (`3rd · A/B/C/D/F`), not a resolved team. FIFA assigns the 8 qualifying
   thirds to specific winners via a fixed lookup table keyed on *which 8 of the 12
   groups* the thirds come from. That table is not yet in the code. Once the group
   stage is complete:
   - Add the FIFA third-place assignment lookup table (12-choose-8 → slot map).
   - Resolve `{t:[…]}` slots in `resolveSlot()` using it + `thirdsRanking()`.

2. **Live knockout results / advancement.** `MATCHES` only contains group-stage
   fixtures, so `{m:NN}` (winner-of-match) slots render as `Winner #NN` and never
   advance. To make R16→Final fill in:
   - Add the KO fixtures to the data (or a parallel `KO_MATCHES`) with their
     `gmt`/venue, and let the ESPN fetch (`applyEvent` / `FIX_BY_PAIR`) attach
     scores the same way it does for the group stage.
   - Resolve `{m:NN}` to the winner of match NN (handle extra time / penalties —
     ESPN exposes shootout detail; group-stage code currently skips shootout
     scoring plays, so KO winner logic needs its own path).

3. **Verify the bracket template against the live draw** once real teams populate
   it — sanity-check that group winners/runners-up land in the venues/dates FIFA
   actually scheduled (a few R32 venues are still unset in `KO_BRACKET`).

## Sources for the bracket structure

- FIFA official knockout schedule & bracket (canadamexicousa2026)
- ESPN / NBC Sports 2026 World Cup bracket pages
- Cross-checked match-by-match pairings #73–104 (see commit discussion)
