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

## DONE (2) — 28 Jun

2. **Live knockout results / advancement.** ✅ Implemented ESPN-driven (no hardcoded
   KO schedule). `buildKoLive()` captures every non-group `fifa.world` event as a KO
   match (teams, time, venue, score, `winner`, `shootoutScore`); `buildKoResults()`
   maps each to its `KO_BRACKET` number and advances winners up the tree to a
   fix-point; `resolveSlot()` fills `{m:NN}` from `KO_RESULT`. KO games now show on
   the **Schedule** (round badge, AET/Pens, pen score) and the **bracket** shows live/
   final scores + a ✓ on the advancing side. `ALL_DAYS` extended through Jul 19.
   Validated structurally: advancement simulated R32→Final (all 31 resolve), and a
   group/KO rematch can't corrupt a group result (date guard in `applyEvent`).
   - **VERIFY LIVE (today):** confirm against real R32 games that ESPN's `winner` /
     `shootoutScore` field names are correct and KO games map to the right bracket
     slots. Watch the live page as today's games go final.

   **ESPN `fifa.world` knockout format notes (researched 24 Jun — VERIFY against
   the first real R32 payload before trusting):**
   - Same envelope as group stage: `events[] → competitions[0] → competitors[]`,
     `status.type.state` = `pre`/`in`/`post`. `fetchDay` + team matching reuse as-is.
   - **Winner ≠ score comparison.** `competitors[].score` is the regulation/ET
     score (a shootout shows as e.g. `1–1`), so comparing `hs`/`as` calls it a
     draw. Read the per-competitor **`winner: true`** boolean as the authoritative
     advancement signal (correct for 90', AET, and penalties).
   - **`competitors[].shootoutScore`** holds the penalty tally (e.g. 4 vs 2) — capture
     it to show "1–1 (4–2 pens)".
   - `status.type.description`/`detail` carry "After Extra Time" / "After Penalties";
     `displayClock` runs to 90–120'. `matchState` (in→live, post→ft) still holds;
     add an "AET/Pens" badge variant instead of plain "FT".

3. **Verify the bracket template against the live draw** once real teams populate
   it — sanity-check that group winners/runners-up land in the venues/dates FIFA
   actually scheduled (a few R32 venues are still unset in `KO_BRACKET`).

## Sources for the bracket structure

- FIFA official knockout schedule & bracket (canadamexicousa2026)
- ESPN / NBC Sports 2026 World Cup bracket pages
- Cross-checked match-by-match pairings #73–104 (see commit discussion)
