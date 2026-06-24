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

## DONE (2)

1. **Third-place → R32 slot assignment.** ✅ Implemented (24 Jun). `THIRD_FIXTURES`
   encodes each of the 8 third-place fixtures' fixed 5-group eligibility pool;
   `thirdAllocation()` assigns the best-ranked available third to each fixture in
   match-number order (FIFA Annex C method), with backtracking as a safety net.
   Verified to produce a complete valid bijection for all C(12,8)=495 group
   combinations. `resolveSlot()` fills `{t:[…]}` slots once all 12 groups finish;
   seeded teams are marked ◇ in the bracket.
   - **VERIFY ON 28 JUN:** once FIFA publishes the actual R32 draw, confirm our
     allocator output matches the official third-place matchups. The greedy/
     match-order method is the widely-documented one, but cross-check against the
     official bracket and correct `thirdAllocation()` if any combination diverges.

## TODO — CHECK IN ON **Sun 28 Jun 2026** (R32 starts; group stage ends Sat 27 Jun)

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
