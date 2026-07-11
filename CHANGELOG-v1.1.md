# Power Fusion Lab — v1.1 Refinement Pass

Single-shot deep-build refinement by Claude Fable 5 · 2026-07-10
Scope: bug pass · balance audit · 3 new systems · UI polish · narrative review
File: `index.html` (8,200 → 8,769 lines, still single-file, zero dependencies)

---

## Architecture Decision

**Kept single-file.** Reasoning: vanilla ES modules break on `file://` (CORS), so a
real split requires either a local server or a build step — both violate the
no-tooling constraint and the one-file distribution story. Multi-`<script>` splitting
keeps `file://` working but sacrifices the single-file identity for near-zero
structural gain (still one global scope). Doing a structural migration in the same
diff as new systems + bug fixes is also the highest-regression-risk combination
before a HACCS audit.

**Split threshold:** ~12k lines or a second contributor. When that day comes, split
data vs. logic vs. render (`data.js` / `game.js` / `render.js`) with a trivial concat
script that still emits one distributable file.

---

## New Systems (3, interlocking)

### 1. Fusion Catalysts (`🧿 Cat` in HUD)
Earned consumables that modify the **next fusion**. Arm one in the HUD panel, fuse,
it's consumed. Inventory capped at 3 per type.

| Catalyst | Effect | Source |
|---|---|---|
| 🧿 Stabilizer Core | Tag cap raised 8 → 10, nothing truncated | Resolving any crisis |
| 🧪 Mutagen Vial | Opposing tag pair guaranteed to annihilate into a mutant | Every 3rd Gauntlet wave |
| 🔷 Harmonic Prism | +15% resonance (can push into Harmonic +1 all stats) | Completing a story arc |
| 🔍 Quality Lens | +12 quality score | Convergence events, milestones |

Mutagen is **refunded** if a named reaction pre-empts the mutation roll (reactions
take precedence over mutations by existing design — the player shouldn't pay for an
effect they never received). Preview modal reflects the armed catalyst.

### 2. Arena Gauntlet (`🔥 Gauntlet` format in Arena)
One hero vs endless waves of synthesized challengers built from random base-power
pairs (they reuse the 190 canonical pair names). Stats scale +1 per 2 waves; past
wave 6, HP/damage scale a further 4%/wave. Damage carries between waves with only
35% recovery, so **every run ends**. Best streak recorded per hero (shown on hero
profile and picker). Mutagen Vial every 3rd wave cleared.

### 3. Lab Milestones (`🏆 Milestones` under ☰ More)
16 persistent progression goals (first mutation, S-grade fusion, S+ tier, intensity-5
bond, 10-wave gauntlet streak…). Seven pay out catalysts, closing the reward loop:
play the systems → earn catalysts → fuse stronger → push deeper. Checked centrally on
every autosave. Loading a pre-v1.1 save marks already-met milestones **silently** —
no retroactive toast shower or reward burst.

**Flagged out of scope (recommend separate passes):** Villain/Nemesis system and
team Missions/Expeditions. Both are the right *next* additions, but each needs a
large bespoke narrative pool to meet the 530-template writing bar — bolting them on
with thin templates would degrade the game's strongest asset.

---

## Bug Fixes

1. **`startNewGame` partial reset (the big one).** Title-screen "New Game" did not
   reset mutant tags, `mutantCount`, `fuseCount`, discovered convergences, session
   chronicle, or any of the 10 convergence rule-flags (`DOUBLE_AWAKENING`,
   `AWAKENING_EP_THRESHOLD=55`, `BLEED_CHANCE=0.18`, injected Tectonic Rage reaction,
   etc.). A new game silently inherited the previous run's buffs. Both reset paths
   now route through one `_resetStateToDefaults()`.
2. **Crisis overwrite.** `_fireCrisisWhenReady`'s modal guard didn't include
   `crisis-root` or `title-screen-root` — a second crisis could replace an unresolved
   one (losing the player's pending choice), and a deferred crisis could fire over
   the title screen. Both added to the guard list.
3. **Shared base-power references.** `resetGame`/`loadGame` shallow-copied
   `BASE_POWERS` (only `startNewGame` deep-copied). Any future mutation of a base
   power would have corrupted the pristine definitions across resets. All three paths
   now use `cloneBasePowers()` (deep copy of stats + tags).
4. **`drained` type inconsistency.** World events set `p.drained=true` while
   evolution used a counter — "Drained ×true" could render in the evolution modal.
   Normalized to counters everywhere + save migration for old boolean flags.
5. **Hero profile markup.** `.hp-meta-row` was closed with `</span>` instead of
   `</div>`.
6. **Stale technique cache.** Crisis tag swaps (`tag_storm` Embrace path) replaced a
   tag without changing tag-count, so the ability cache served stale techniques.
   All crisis tag mutations now invalidate the cache.
7. **Memory Bleed dead outcome.** "Absorb the Ghost" could charge −1 DUR while
   failing to add a tag (duplicate roll or full tag list). Now picks only from
   tags the power lacks, and skips the DUR cost when there's no room.
8. **Counter-kill had no ending.** A counter that dropped the attacker to 0 produced
   no finisher narrative (fight just stopped). New `counterFinish` pool (duel + team).
9. **Team double-KO favored Team B.** Mutual KO on the final relay always awarded
   Team B (`idxA>=3` checked first). Now breaks the tie on chemistry, then coin flip.
10. **Dead Events counter.** `updateConvergenceLog` targeted a `conv-log-btn` id that
    no longer existed; the dropdown Events button never showed its count. Id restored.
11. **Awakening ignored the 8-tag cap** — could push a power to 9 tags. Stat boost
    still applies; the tag only lands if there's room.
12. **Toast pile-up.** Simultaneous toasts (bond + world event + milestone) rendered
    on top of each other at the same fixed position. Toasts now queue sequentially.
13. **Biased crisis picks.** `resonance_spike` (Let Them Sync) and `fracture_point`
    (Let It Burn) always charged/rewarded `heroA` of the bond. Now a fair coin flip,
    and the result text names who paid.
14. **Raw mutant ids in crisis text.** Tag-storm results printed `mutant_3` instead
    of the mutant's name.

## Balance Changes

1. **Mutant tags were a silent downgrade.** `computeFusedStats` affinity and
   `computeQualityScore` looked up only `TAG_DEFS`/`LEGENDARY_TAGS`, so a mutant tag
   (intensity 2–3) contributed *nothing* to stats and scored as an intensity-1
   unknown in quality. Mutation — the system's centerpiece emergent mechanic —
   mechanically punished the player. Mutants now grant stat affinity and count in
   quality (intensity + a rare-tag bonus of 5/mutant sharing the legendary 0–20 budget).
2. **Infinite evolution drain closed.** A source could be re-drained forever, and the
   arc bonus (+up to 3 stats) re-applied on *every* evolution — full-10s were
   trivially farmable from one source + 3 completed arcs. Sources now survive max
   2 drains, and arc bonuses are **spent** on use (`arcBonusSpent` tracked per power;
   new arcs replenish the pool). UI copy updated to say "unspent arcs · consumed on
   evolution".
3. **Encounter farming damped.** Rescue/revelation spam (+2 intensity rolls) could be
   farmed indefinitely via "Another Encounter". Pairs with ≥6 encounters now clamp
   non-type-shift intensity swings to ±1, with a "familiarity" outcome label.
4. **Arena VRS/utility buff.** VRS was the weakest combat stat (counter capped 25%,
   utility techniques were just weak attacks). Counter cap raised to 30%; utility
   techniques now erode the defender's evade by 4 per use — sustained analysis
   defeats slippery builds.

## UI Polish

- Catalyst HUD panel follows the existing dropdown pattern (no layout change).
- Global **Escape** closes the topmost dismissible overlay — deliberately excludes
  crisis prompts, world-event reveals (decisions), and the tutorial (own handler).
- Milestones entry added to ☰ More; Events button count restored.
- Hero profile shows 🔥 Gauntlet best; gauntlet picker shows per-hero records.
- Tutorial: new Step 12 slide covering the three systems; quick-reference updated.
- Title screen: v1.1 · Refinement; credits updated (and the stale "7,700+ lines /
  186 functions" claim removed rather than left wrong).

## Narrative Template Review

Coverage was thinnest exactly where repetition is most visible: combat micro-lines
(counter had 2 templates, heal 2, evade 3, and counter-kill had **zero**) and the
high-traffic encounter types (weights concentrate rolls on training/clash/downtime/
philosophical/crisis, each with only 3 templates). Added, in the established voice:

- Combat: counter +3, counterFinish +3 (new pool), evade +2, heal +2
- Encounters: training +2, philosophical +2, crisis +2, downtime +2, clash +2
- Gauntlet: intro/advance/defeat pools, 3 each (new)

Net +25 templates, all targeted at observed thin spots — no volume padding.
Bond narratives (5 short/3 deep × 10 types) and family arcs (5 × 10) reviewed:
coverage adequate, left untouched.

## Save Compatibility

`SAVE_VERSION` stays 1 — all new fields are additive with defaults, so existing
saves load cleanly. Legacy `drained:true` booleans migrate to counters on load.

---

## Verification — what was tested vs. inferred

**Verified by execution** (headless harness: DOM-stubbed `vm` context running the
actual shipped `<script>`, 43/43 assertions passing):
- Full script parses (`node --check`) and boots without error
- Fusion pipeline end-to-end incl. quality grade, tag cap, milestone unlock
- Catalyst grant → arm → consume; lens +12 math; stabilizer cap-10; mutagen forced
  annihilation (direct unit call); mutant-tag quality no longer scores below plain tags
- Hero enlistment, bond formation, 8 sequential encounters with intensity staying
  in [1,5]; arena duel records; team 3v3 returns a winner; chemistry computes
- Gauntlet runs waves without error and maintains coherent state
- Evolution drain cap (source excluded after 2 drains, numeric counter)
- Save → mutate → load roundtrip (fused powers, catalysts, milestones)
- `startNewGame` now resets all 7 previously-leaked state groups (asserted individually)
- Awakening respects the 8-tag cap; crisis resolution grants a Stabilizer
- Incidental finding confirming milestone correctness: the harness's first fusion
  (pyro+cryo) fired Thermal Paradox → legendary tag → "Impossible Made Real"
  milestone → lens payout, exactly as designed

**Inferred, not executed** (verify manually in a browser):
- Visual rendering of the catalyst panel, milestone modal, gauntlet result screen,
  and the result-modal catalyst banner (DOM was stubbed — layout/CSS untested)
- Escape-key layering order feels right in practice
- Toast queue pacing (2.2s per toast) during dense event bursts
- Crisis timing interplay at 800ms deferral with the new guard roots
- Gauntlet difficulty curve — math says runs end around waves 4–8 for mid-tier
  heroes; playtest to taste (tuning knobs: 0.35 recovery, 0.04 late-wave scale)

## Manual verification checklist (suggested)

1. Hard-refresh with an existing v1.0 save → Continue → confirm no toast burst,
   milestones panel shows already-met goals unlocked
2. Fuse once → resolve the eventual crisis → confirm Stabilizer toast → arm it →
   fuse two 6+ tag powers → confirm 9–10 tags and the consumed-banner in the result
3. Run a Gauntlet to defeat → check streak on hero profile → confirm Mutagen at wave 3
4. Title → New Game after a long session → fuse pyro+cryo → confirm mutation chance
   feels baseline (no inherited Great Conflict 50% boost)
5. Esc through stacked modals (info over grid, preview over HUD)

## In scope but risky (watch during audit)

- **`autoSave()` now runs `checkMilestones()` first.** Single central hook — clean,
  but any future milestone `check()` that throws would be swallowed by the
  try/catch and silently never unlock. The try/catch is deliberate (a bad check
  must never block saving).
- **Gauntlet ghost powers enter the ability cache** (bounded per session, cleared on
  any fusion via existing global invalidation — but worth knowing they're there).
- **Familiarity clamp** changes long-pair encounter economics; if players report
  bonds feeling "stuck," raise the threshold from 6 to 10 before touching the clamp.

## Out of scope, recommend separate pass

- Villain/Nemesis system; team Missions (narrative-pool-heavy, see above)
- Instability currently has no downside beyond spontaneous mutation (which reads as
  a reward). A "stabilize or risk" decision layer would give the CRITICAL badge
  teeth — pairs naturally with a future catalyst type
- Data/logic/render file split at the ~12k-line threshold
