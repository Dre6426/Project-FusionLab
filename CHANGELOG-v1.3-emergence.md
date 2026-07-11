# Power Fusion Lab — v1.3 Emergence Pass

Claude Fable 5 · 2026-07-11
Scope: **two transformative systems** — The Nemesis (Slot A) and Ancestral Echoes (Slot B). Not refinement, not polish: both recombine ≥3 existing systems and change decisions the player already makes.
File: `index.html` (8,869 → 9,638 lines, still single-file, zero dependencies)

---

## Slot A decision — picked A1 (Nemesis) over A2 (instability stakes)

Honest read of A2: it compresses to one decision modal plus a catalyst sink. That is a
v1.1-scale addition — exactly the "new consumable, new screen" bar this pass was told to
clear. A1 is the system where the game's centerpiece emergent mechanic (mutation) turns
out to have been feeding something — the "the game has more going on underneath" moment.

The clincher: **A1 absorbs A2's goal as a side effect.** The Nemesis's siphon incursions
target the roster's most *unstable* power (instability ≥ 40) and skip a stable lab
entirely — so the CRITICAL badge gets real teeth without shipping a thin standalone
layer. Both flagged expansions are addressed; only one system was built, at full depth.

## Slot B pitch (as required: candidates, tradeoffs, pick)

1. **Ancestral Echoes** — fusion history becomes a live resource. Touches: fusion/tags,
   lineage/evolution, arena (all 3 formats), bonds/encounters, quality-adjacent decisions.
   Risks: ancestry walks on every profile build (mitigated: tiny graphs, computed per
   fight not per frame). Scope: moderate — no save-shape change at all, since ancestry
   derives from `parentIds` the game has recorded since v1.0. Retroactive aha: existing
   saves' old lineage instantly matters.
2. **Cascade fusions** — one fusion triggers sympathetic reactions across the roster.
   Cut: this is ecosystem bleed but bigger — the "v1.1 again" trap named in the brief —
   and untargeted roster-wide side effects read as chaos, not depth. Verification and
   balance surface would have eaten the narrative budget.
3. **Duo awakening** — intensity-5 + completed-arc pairs unlock a paired signature.
   Cut: bonds already pay off through arcs, chemistry, and world events; this adds more
   content in a well-paid direction, and its only output surface is combat.

**Built #1.** The two systems mirror deliberately: the Nemesis is your fusion history
coming back hostile; Echoes are it coming back loyal. The Nemesis has `parentIds:null` —
it is the only fighting thing in the lab with no ancestors and therefore no echoes, and
one of its duel intros says so.

---

## System 1 — THE NEMESIS (`🌑` HUD button, appears on emergence)

**Emergence.** When the lab's mutant vocabulary reaches critical mass
(`NEMESIS_MUTANT_THRESHOLD = 3` mutant tags), the newest mutant tag "does not stabilize":
a full-screen reveal (`maybeNemesisEmergence`, `.nx-overlay`) introduces a persistent
antagonist named from that tag (`_capName(seed.name)` + a title from `NEMESIS_TITLES`,
10 entries). It consumes the entire existing vocabulary as its body. Checked after every
fusion (2.6 s deferred, modal-guarded with the crisis-style retry pattern).

**It grows.** `nemesisConsumeMutants` — every mutant tag born while it's active
(fusion annihilations, spontaneous mutations) is copied into `nemesis.consumed`, with a
taunt toast (`toast-nemesis`, 5-template feed pool). Mutation now has a cost; the
Mutagen Vial becomes double-edged.

**Siphon incursions** (`rollNemesisIncursion`, fired at 0.12 after fusions,
cooldown 4). Targets the highest-instability fused power **≥ 40 only — a stable lab is
invisible to it**. Two choices: *Ward the Breach* (burns a Stabilizer Core if owned,
else −1 to the target's strongest stat; it gets nothing) or *Let It Feed* (it eats one
side of the target's loudest opposing tag pair → instability genuinely drops, +1 CTL,
but the word joins the Nemesis and it grows). Local peace, global escalation.

**Confrontation.** HUD `🌑 Nemesis` → dossier (phase, banishment progress, its record,
stolen-vocabulary chips, sighting log) → `⚔ CONFRONT` → champion picker → duel via the
real combat engine (`buildNemesisPower`: stats `5+phase` +1 POW/VRS, +1 per 4 consumed
words capped +2; phase 3 gets ×1.15 HP; techniques generated from its stolen tags).
- **Win:** it retreats and escalates (`phase = 1+defeats`), pays a **Paradox Sliver**.
  Three defeats banish it for good (long-form banishment pool, 2 Slivers, milestone).
- **Lose:** it steals a random owned catalyst, or (if stores are empty) **writes one of
  its mutant words into your champion's power** (replaces a non-legendary tag), or
  drains −1 strongest stat as a fallback. Then it can't be found for 3 fusions
  (`lockedUntil = fuseCount + 3`).

**Gauntlet invasion.** Once per run, wave 4+, 15% (`runGauntletWave`): the wave *is* the
Nemesis (bespoke 3-template intro pool). Beating it there counts as a full confrontation
win — same `applyNemesisDefeat`/`applyNemesisVictory` consequence paths.

**Paradox Sliver** — 5th catalyst type (`CATALYST_DEFS.paradox`): next fusion is seeded
with one of the lab's existing mutant tags, no annihilation required. Refunded if the
lab has no mutant vocabulary (same refund philosophy as the Mutagen/named-reaction rule).

## System 2 — ANCESTRAL ECHOES (no button — it's everywhere lineage is)

**Bloodline Resonance.** `findSharedFusedAncestor(sel)` in `fuse()`: if two parents
share a **fused** ancestor (gen > 0 — sharing a base power is trivial with only 20
bases, so common blood must be *discovered* blood), resonance gets +12%, a
`.bloodline-banner` names the ancestor in the result modal (3-template pool), the child
records `bloodlineAncestor`, and the Preview modal announces it before you commit.
Changes fusion-partner choice: related powers are now mechanically drawn to each other.

**Echo Invocation.** `computeEchoProfile(power)` — gen 2+ with ≥ 3 recorded ancestors
(recursive `parentIds` walk **plus `evolvedFrom` absorptions** — draining a source now
deepens the drinker's echo pool). Chance `0.22 + 0.07×(ancestors−3)`, capped 0.45.
Once per fight, when pressed below 35% reserves, the power spends its turn invoking an
ancestor by name: heals 15% of max, fights the rest at +25% base damage (5-template
pool, `combat-round.echo` styling). Works in duels, team relays, the Gauntlet, and
Nemesis confrontations — gauntlet ghosts and the Nemesis (`parentIds:null`) never
qualify. Sets `power.echoInvoked` (persists in saves) for the milestone.

**Lineage surfacing.** `triggerEncounter`: hero pairs whose powers share a fused
ancestor get the ancestor acknowledged in the encounter narrative 45% of the time
(4-template pool, `enc.lineage` recorded). The info modal gains an **Ancestral Echo**
section (ancestor count, invocation odds, has-invoked mark) so the system is legible.

## Cross-system integration

- **5 new milestones** (16 → 20): Blood Calls to Blood (→ Harmonic), The Line Remembers,
  The Third Thing, Hold the Line (→ Lens), Reclaimed Vocabulary.
- Nemesis wards consume Stabilizer Cores; defeats feed the catalyst economy with
  Paradox Slivers; Slivers feed back into the mutation loop that feeds the Nemesis.
- Tutorial Step 13 covers both systems; credits and title updated to v1.3 · Emergence.
- `nemesis-root` added to the crisis modal-guard and to the emergence/incursion guards;
  Escape closes nemesis screens **only** when they're informational (`.nx-dismissible` —
  dossier/picker/duel result), never decisions (incursions, emergence reveal).

## Narrative budget

**58 new bespoke templates**, all long-form house voice, none on thin coverage:
Nemesis 46 (emergence 3, feed 5, siphon intro 3, ward 2+2, feed-result 3, duel intro 4,
retreat 4, banish 3, loss 3, steal 3, corrupt 3, drain 2, gauntlet intro 3, dossier 3)
+ Echoes 12 (invocation 5, bloodline 3, encounter lineage 4). Plus 10 Nemesis titles and
a tutorial slide. Highest-frequency triggers got the deepest pools (feed 5, invocation 5,
retreat 4, duel intro 4); one-shot beats (banishment) got fewer but longer entries.
Credits template count 550+ → 600+.

## Bug/balance notes that emerged

- `renderGauntletResult`'s outcome line now renders `white-space:pre-line` — needed for
  multi-paragraph Nemesis text, harmless for the existing one-liners (cosmetic).
- No pre-existing bugs found in the touched paths this pass; the v1.1 crisis-deferral
  and toast-queue infrastructure absorbed both new event sources without modification.
- Balance intent: Nemesis phase 1 (stats ~7) is beatable by a mid-tier hero; phase 3
  (~9 + 15% HP) wants an echo-capable champion — the two systems meet in the ending.

## Save compatibility

`SAVE_VERSION` stays **1**. Additive only: `save.nemesis` (plain object or null),
`catalysts.paradox` (defaulted to 0 on load/reset/new-game). Echoes add **zero** save
fields — ancestry derives from `parentIds`/`evolvedFrom` already saved; `echoInvoked`
and `bloodlineAncestor` ride on fused-power objects that were already serialized.
A pre-v1.3 save with 3+ mutant tags does **not** silently spawn a Nemesis on load —
emergence is a moment, and it fires on that player's next fusion instead.

---

## Verification — executed vs. inferred

**Verified by execution** (headless harness: DOM-stubbed `vm` context running the actual
shipped `<script>`, **63/63 assertions passing**, plus `node --check` on the full script):

- Ancestry: recursive walk (child of F1 has 4 ancestors); base-only overlap → no
  bloodline; shared fused ancestor detected; `evolvedFrom` counted
- Bloodline fusion end-to-end: banner rendered, `bloodlineAncestor` recorded, milestone
  unlocked, Harmonic Prism paid
- Echo profile gating (gen-1 → null; ghosts → null; Nemesis → null) and the exact chance
  formula `min(0.45, 0.22+0.07×(n−3))`
- Echo invocation in a real simulated fight: fires exactly once, names the ancestor,
  raises base damage, flags `echoInvoked`, unlocks the milestone
- Emergence at exactly 3 mutant tags: named from the newest tag, consumes all 3, reveal
  rendered, milestone unlocked, no double-emergence
- Phase-1 body math (POW 7 / DUR 6, tags = consumed); feeding grows `consumed`
- Confrontation consequences: defeat 1 → phase 2 + Paradox Sliver + milestone; its
  victory → catalyst stolen + 3-fusion lockout; defeat 3 → banished + milestone;
  banished → incursions permanently silent
- Siphon: targets the most unstable power (≥ 40); feeding removes the tag, grows the
  Nemesis, and **measurably lowers the target's instability**; ward burns a Stabilizer
  when owned; a stable lab produces no incursion at all
- Paradox Sliver: seeds a mutant tag into the child, consumed on use, **refunded** when
  the lab has no mutant vocabulary
- Gauntlet invasion at wave 4 applies real defeat/victory consequences
- Save/load roundtrip for `nemesis` + `paradox`; pre-v1.3 save (no nemesis field) loads
  cleanly with `paradox:0`; reset clears both; `SAVE_VERSION === 1`
- Encounter lineage note lands in `enc.narrative` for shared-blood hero pairs
- Regressions: quality grades intact, 20 milestones, 5 catalyst types, crisis guard
  covers `nemesis-root`, stabilizer 10-tag path intact

**Inferred, not executed** (verify manually in a browser):

- All visual rendering: `.nx-overlay` reveal/incursion, dossier layout, vocabulary chips,
  `.bloodline-banner`, `.combat-round.echo` styling, HUD button pulse (DOM was stubbed)
- Escape-key layering with `.nx-dismissible` in practice
- Pacing: emergence deferral (2.6 s + 900 ms retries) vs. result-modal close timing;
  incursion frequency at 0.12/fusion with cooldown 4 — knobs are at the top of the
  Nemesis section if it feels too chatty or too absent
- Duel difficulty curve per phase — math says phase 1 loses to a 7-stat hero more often
  than not; playtest to taste (knobs: `base=5+phase`, growth cap 2, ×1.15 phase-3 HP)
- Mobile media-query rules for `.nx-title`/`.nx-narrative`/`.nx-stat-num`

## Manual verification script (suggested order)

1. **Continue** with an existing save (3+ mutant tags) → fuse anything → after closing
   the result modal, the emergence reveal should fire within ~3 s → HUD gains 🌑.
2. Open 🌑 → dossier shows born-from tag, stolen vocabulary chips, 0/3 banishment.
3. Fuse an unstable combo (opposing tags, e.g. heat+cold parents) until a siphon
   incursion fires → pick *Let It Feed* → check the target's info card: tag gone,
   +1 CTL, instability badge lower; dossier vocabulary grew by one chip.
4. Trigger a mutation (arm a Mutagen Vial) → crimson feed toast names the new word.
5. 🌑 → CONFRONT → pick a champion → win: retreat text + Paradox Sliver toast + phase 2
   in dossier. Lose deliberately (weak hero): catalyst stolen or tag corrupted, and
   CONFRONT replaced by "fuse 3 more times" lockout text.
6. Arm the Paradox Sliver → fuse → result modal shows the 🌑 banner and a mutant tag in
   the child's pool.
7. Fuse two descendants of the same fused power → ❖ Bloodline banner in the result
   modal (and the ❖ note in Preview beforehand).
8. Take a Gen-2+ hero (3+ ancestors — check the info card's *Ancestral Echo* section)
   into the Gauntlet → when pressed below ~35% HP, watch for the rose-colored echo line;
   the info card then shows "👁 Has invoked."
9. Run encounters between two heroes whose powers share a fused ancestor → within a few
   encounters, the narrative appends the shared-ancestor paragraph.
10. Beat the Nemesis three times total → banishment long-form + 2 Slivers → HUD reads
    🌑 Banished; dossier shows the epitaph; milestones 20/20 path visible.
11. Title → New Game → confirm no 🌑 button, no nemesis, paradox count 0.

## In scope but risky (watch during audit)

- **`fuse()` now schedules three deferred rolls** (emergence 2.6 s, incursion 3.3 s,
  crisis 4 s). All three share the same modal-guard list and defer politely, but the
  interleaving under rapid fusing is timing-dependent — if two decisions ever stack,
  the guard lists are the place to look (`_nxModalOpen`, `_fireCrisisWhenReady`).
- **Nemesis ghost powers enter the ability cache** (same known caveat as gauntlet
  ghosts since v1.1 — bounded per session, cleared on any fusion).
- **Siphon can eat a tag off a hero's own power** (heroes' powers are valid targets).
  Deliberate — instability should threaten what you care about — but if playtests say
  it feels unfair, filter `heroes.some(h=>h.powerId===p.id)` out of the target scan.
- **`applyNemesisVictory` steals from any catalyst slot**, including one currently
  armed (it un-arms if the count hits 0). Tested, but it's the most state-entangled
  consequence path.

## Out of scope, recommend separate pass

- **Successive nemeses** — after banishment the vocabulary keeps growing; a second,
  differently-shaped emergence (higher threshold, new incursion types) is the natural
  sequel. Deliberately not shipped thin.
- **Echo choice** — invocation is automatic; a pre-fight "which ancestor do you carry?"
  pick would deepen it but needs its own UI and balance pass.
- Nemesis-vs-team (3v3) and nemesis-in-story-arcs integration.
- Data/logic/render file split — now ~9.6k lines, still under the ~12k threshold from
  v1.1, and this pass again mixed new systems with existing files; same reasoning holds.

## Housekeeping (not code)

- v1.1–v1.3 `index.html` changes remain **uncommitted** in git (working tree only).
  Recommend committing from your machine as before.
- `_write_test.txt` in the project folder is a session scratch file (write-path test).
  Safe to delete — the session couldn't remove it without delete permission.
