# Power Fusion Lab — v1.4 Expansion Pass

Claude Fable 5 · 2026-07-11
Scope: **content & feel** — breadth and texture inside existing systems. No new mechanics.
File: `index.html` (9,638 → 9,801 lines + ~57KB of content, still single-file, zero dependencies)

---

## Prioritization (stated before building, per brief)

**Heavy: Content 1 (base powers) + Content 2 (narrative variety). Medium: Content 3
(endgame). Light and targeted: Feel 3 (pacing) + Feel 2 (juice). Minimal by finding:
Feel 1 (tone).** The weighting came from measurement, not preference — details under
each area. The deliberate unevenness: tone audit found the existing voice strong at
every layer sampled, so rewriting it would have been sprawl; the base-power audit
found a third of the tag vocabulary orphaned, so that's where the depth went.

---

## Content 1 — Base powers & tags (HEAVY)

**Audit findings (all reproducible against the shipped data):**
- 23 of 84 `TAG_DEFS` appeared on **no base power** — a third of the tag vocabulary
  unreachable except through awakening/crisis randomness.
- **17 of 37 named reactions were tag-uncompletable from any base pair** — dead
  content. Only 39/190 base pairs fired any reaction.
- Stat-archetype spread: CTL/RNG/VRS specialists plentiful; no agile POW bruiser, no
  dedicated field-control DUR/CTL defender, no low-CTL wildcard.

**Shipped: 3 new base powers, full-depth treatment (20 → 23):**
- **🧲 Magnetokinesis** (`magnet`, Elemental, CTL9/DUR9 field controller) — tags
  `magnetism, attraction, charge, ordered, inertia, current`
- **🐺 Primal Instinct** (`beast`, Physical, POW9/RNG7 predator) — tags
  `feral, instinct, adaptation, primal, reflex`
- **🌪 Chaoskinesis** (`chaos`, Void, VRS10/CTL3 wildcard) — tags
  `chaotic, turbulent, flux, corruption, dormant, entropy`

**Plus 63 new pair fusions (190 → 253)**, every one with name + desc + reason at the
existing register (`PAIRS['magnet+pyro']` = Ferrofluid Inferno, `'beast+sono'` =
Echolocation, `'chaos+prob'` = House Edge…). No thin tier: verified by execution that
every new base has a full entry with all 22 partners.

**Dormant reactions lit up — 10 of the 17 now tag-completable from base pairs:**
Electromagnetic Dominion (magnet+electro), Unstoppable Object (magnet+speed),
Primal Scream (beast+strength), Apex Evolution (beast+bio/heal/chloro),
Phantom Instinct (beast+tele/photo/…), Frozen Storm (chaos+cryo),
Phase Collapse (chaos+astral), Absolute Decay (chaos+pyro/sono),
The Sleeper Wakes (chaos+pyro), Chaos Phase (chaos+shadow — **caveat:** that pair's
union also satisfies Void Heart, which wins reaction priority; Chaos Phase fires only
on descendant pools that carry `chaotic+intangibility` without `void+entropy`).
Reaction-firing base pairs: **39 → 56 of 253** (verified by execution).
Chaoskinesis is also deliberately mutation-prone (`chaotic↔ordered`,
`corruption↔restoration`, `dormant↔volatile` oppositions) — it feeds the
mutation→Nemesis loop by design.

Renamed during build: my `electro+magnet` pair was originally "Unified Field" — the
build's collision assert caught that `electro+gravity` already owns that name; it
shipped as **Maxwell Engine**. Integration is data-driven throughout: new bases flow
into `cloneBasePowers`, gauntlet challenger synthesis, hero enlistment
(`detectFamily` maps them to wave/bio/temporal-adjacent families with existing
backstory pools), `MUTANT_FRAGS` already covered all 17 new-base tags.
Counts updated: header stats, tutorial, credits (23 bases · 253 pairs).

**Considered and cut:** a 4th base (Solarkinesis: plasma/stellar/saturated) — its
unlock yield was 1 reaction, and carrying `plasma+stellar` together would have
solo-completed Stellar Core (a guaranteed legendary on every fusion). Not worth the
21 extra pair entries this pass; noted for a future one.

## Content 2 — Narrative variety (HEAVY)

**Audit method:** for each encounter type, aggregate selection weight ÷ pool size
("pressure"); for combat, per-fight line frequency. Worst offenders: revelation
(pool 3, pressure 3.52), philosophical (3.03), training (2.94), territorial (2.91);
`attack` fires several times per fight from a pool of 6; gauntlet intro/advance/defeat
were 3/3/3 and a 10-wave run displays each ~10×; MUTANT_DESC 4 on every mutation.

**Shipped, +46 pool templates, all at the house register** (verified counts by
execution): revelation 3→7, training 5→8, philosophical 5→8, territorial 3→6,
rescue 3→6, crisis 5→7, downtime 5→7, clash 5→7, first_contact 3→5, betrayal 3→5
(+26); combat attack 6→10, crit 3→5, sig 3→5, finisher 3→5, heal 4→6 (+12);
gauntlet 3/3/3→5/5/5 (+6); MUTANT_DESC 4→6 (+2). All new templates are callable
long-form functions (asserted >120 chars each, executed).

## Content 3 — Endgame/postgame (MEDIUM)

**Decision: successive nemeses deferred to a systems pass**, as v1.3 recommended — a
second emergence needs its own incursion types and threshold curve or it's a re-skin.
This pass ships lighter-touch recognition that the ending happened:

- **World Event #13 — 🕯 The Quiet** (12 → 13): fires through the normal world-event
  pipeline when the Nemesis is banished (a `checkWorldEvents` call added at the
  banishment site, 1.8s deferred). Permanent effect: **mutation chance +5%**
  (`window._worldReclaimed`, applied in `applyTagMutations`, capped 0.65) — the
  reclaimed vocabulary answers the lab again. Persists via the existing
  `worldEventLog` flag-scan on load; cleared on reset. Verified by execution
  end-to-end (banish → event fires → flag set → save/load roundtrip → reset clears).
- **Veteran encounter epilogues** (`VETERAN_ENC_LINES`, 4 templates): when both
  heroes' powers are Gen 3+, 30% chance — late rosters finally read as late.
- **Post-banishment epilogues** (`BANISHED_ENC_LINES`, 4 templates): 20% chance,
  reference the banished Nemesis **by name**. Verified by execution.
- Appendix hygiene: encounters append **at most one** of lineage/veteran/banished
  (priority in that order) so late-game narratives don't stack three postscripts.
- The 3 new bases are themselves endgame content: fresh discovery space + 10 dormant
  reactions for players who have seen all 190 original pairs.

## Feel 1 — Atmosphere/tone (MINIMAL, by audit)

Read a cross-section before deciding: base descs, pair descs/reasons, reaction
banners, encounter narratives, Nemesis/Echo pools. **Finding: the voice is already
strong and consistent at every layer, including the v1.0 copy** (base descs are not
the weak point I expected). No rewrites shipped — introducing edits there risked
regression-by-taste. All ~180 new prose pieces this pass (54 pool templates, 126
pair desc/reasons, 3 base descs, 1 world event) were written to the strongest
existing register instead.

## Feel 2 — Juice (LIGHT, targeted)

Audit: post-v1.2, the flattest remaining moment was **arena combat logs dumping the
entire fight at once** — `.combat-round`/`.combat-hp` had no entrance animation.
Shipped: staggered playback — all four combat renderers (duel, team, gauntlet,
Nemesis) now emit per-line `animation-delay` (90ms/line, capped 2200ms) over a new
`fadeUp` rule, so fights *play out* top to bottom. Also: `.enc-result` and
`.arena-result` gained entrances, `.arena-winner` gets the house `cscale` overshoot
pop (matching the gauntlet verdict from v1.2). Verified by execution that rendered
duel HTML carries capped delays; visual feel is inferred (see below).

## Feel 3 — Pacing (LIGHT, one knob)

**Audit:** a simulated 40-fusion session showed the real shape — dense first ~25
fusions (reactions + one-shot convergences), then a nearly silent tail once
convergences exhaust and dupes block reactions, with the flat 5% fuse-crisis roll as
the only heartbeat.

**Shipped: quiet-streak crisis escalator.** `_quietFuses` counts fusions that produce
no reaction, mutation, spontaneous mutation, or convergence; the fuse-path crisis
roll becomes `min(0.25, 0.05 + 0.02×streak)`. The streak resets on any notable
fusion **and** whenever a crisis actually fires (`showCrisisPrompt`), so dense
early-game sessions are untouched. Transient variable — no save impact.
**Measured by execution** (40 runs × 40 quiet-biased fusions, both builds):
average crises **1.68 → 2.92** per session — roughly one extra beat per dead
stretch, nothing added to already-eventful play. Knobs: base 0.05, step 0.02,
cap 0.25, all on one line.

---

## Save compatibility

`SAVE_VERSION` stays **1**; zero new save fields. New bases regenerate from
`BASE_POWERS` on every load (base powers were never serialized), so **existing saves
gain the 3 new bases automatically**. `the_quiet` persists through the existing
`worldEventLog` array + flag scan. `_quietFuses` is transient. Verified: full v1.3
save/load/reset assertion suite passes unchanged on the v1.4 build.

## Verification — executed vs. inferred

**Verified by execution** — two suites against the shipped script, plus `node --check`:
- **Regression: the complete v1.3 harness, 63/63 passing on the v1.4 build**
  (echoes, Nemesis lifecycle, siphon, paradox, save/load, milestones — untouched).
- **New v1.4 suite, 53/53 passing**, covering: 23 bases / 253 pairs / full partner
  coverage / no solo-completed reactions; the 9 direct reaction unlocks by exact name
  (+ Void Heart priority on chaos+shadow); reaction-firing pairs 39→56; new-base
  fusion→reaction→legendary→enlist→backstory pipeline end-to-end; every pool count
  listed in Content 2; all new templates callable and long-form; The Quiet end-to-end
  incl. persistence; veteran + banished epilogues (forced rolls, name check);
  quiet-streak accumulation and reset on notable fusion; staggered `animation-delay`
  in rendered duel HTML with the 2200ms cap; CSS rules present.
- **Pacing measured**: 1.68 → 2.92 avg crises per 40 quiet-biased fusions (40 runs each).

**Inferred, not executed (verify manually in a browser):**
- Visual feel of combat playback pacing (90ms/line — knob in the four renderers),
  verdict entrances, and the new-base card colors (teal/ember/lime) in the grid.
- The Quiet's reveal timing when banishment happens inside the Gauntlet result screen
  (deferral logic is shared and tested, exact overlay layering is not).
- New pair copy in situ — 63 entries read in context of the result modal.
- Escalator *feel* — whether ~3 beats per quiet session is right; knobs are inline.

## Manual verification script (suggested order)

1. Continue an existing save → grid shows 23 bases (🧲 🐺 🌪 present), header reads
   23 / 253.
2. Fuse magnet+electro → **Maxwell Engine** + ⚗ Electromagnetic Dominion reaction
   banner. Fuse beast+strength → Primal Scream. Fuse chaos+cryo → Frozen Storm.
3. Fuse chaos+intel (opposing `chaotic↔ordered`) a few times → mutation rate visibly
   healthy; check the two new MUTANT_DESC variants appear.
4. Run any duel → combat log lines fade in sequentially; winner name pops.
5. Run 8–10 dupe/quiet fusions in a row → a crisis should interrupt within the
   streak (escalator working).
6. Trigger encounters on a pair of Gen 3+ heroes → veteran epilogue paragraph within
   a few rolls; with a banished Nemesis, the by-name epilogue.
7. Banish the Nemesis (or load a save mid-fight) → 🕯 The Quiet world-event reveal;
   check ☰ Events log lists 13.
8. Enlist a hero on a chaos-descended power → backstory generates normally.
9. New Game → tutorial says 23 base powers; no flags leak.

## In scope but risky (watch during audit)

- **Reaction priority interactions.** 63 new pairs widen the tag unions reaching
  `checkReaction`; existing reactions can now fire on new pairs (by design — e.g.
  beast+bio hits Apex Evolution) but priority collisions like Void Heart > Chaos
  Phase are inherent to the sort order. If a specific reaction should win, the fix
  is REACTIONS ordering, not tags.
- **Escalator + Nemesis incursions both key off fusions** — a long quiet streak in a
  Nemesis-active, unstable lab can produce crisis→incursion back-to-back (guards
  serialize them; pacing may feel busy in exactly that state; drop cap to 0.20 if so).
- **Chaoskinesis instability**: chaos-descended powers reach high instability fast,
  which makes them preferred siphon targets. Intentional coupling, but it
  concentrates Nemesis attention on one lineage.

## Out of scope, recommend separate pass

- **Successive nemeses** (systems pass; The Quiet's +5% mutation deliberately makes
  a future re-emergence threshold easy to justify).
- The **7 still-dormant reactions** (World Voice, Absolute Authority, Quantum
  Resonance, Perfect Equilibrium, Eternal Foundation, Stellar Core, Critical
  Overload) — they need `acoustic/pressure/sovereign/microscopic/cyclic/stellar/
  saturated` carriers; a 4th/5th base pass or an awakening-resonance route could
  light them up.
- Sound design (real audio is a dependency question, not a juice tweak).
- Data/logic/render split — ~9.8k lines now, brushing the ~12k threshold; next
  systems pass should probably budget for it.

## Housekeeping

- v1.1–v1.4 remain uncommitted in git (recommend committing from your machine).
- `_write_test.txt` scratch file from the v1.3 session is still in the folder;
  safe to delete.
