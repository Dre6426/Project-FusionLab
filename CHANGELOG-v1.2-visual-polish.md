# Power Fusion Lab — v1.2 Visual Polish Pass

Claude Fable 5 · 2026-07-10
Scope: visual/motion/layout only. No new mechanics, no balance changes, no state or save-shape changes.
File: `index.html` (8,769 → 8,869 lines, still single-file, zero dependencies)

## The core finding

The three v1.1 systems were built with **zero dedicated CSS** — every visual decision inline in their render functions. Every established system (reactions, mutations, ecosystem bleed, awakening, codex, arcs) has a proper class family with the house signatures: 3px left accent bar, 1px top hairline gradient, 9px/.22em uppercase labels, Rajdhani names, Bebas display, glow-shadows keyed to the accent color. This pass gives Catalysts, Milestones, and Gauntlet the same treatment — extending existing vocabulary, never inventing new. All new CSS lives in one labeled block (`v1.2 VISUAL POLISH`, just before the mobile media query).

## What changed, where to look

### 1. Catalyst HUD panel (`🧿 Cat` in HUD)
**Before:** flat inline-styled rows — border color swap for armed, no hover, no motion.
**After:** each catalyst is a `.cat-entry` with a left accent bar in its own color (brightens on hover, like codex entries). Names are now Rajdhani 13px/700 (matching card names, was generic DM Sans). Owned entries lift on hover; the **armed** entry breathes with a new `catArmed` glow-pulse (box-shadow in the catalyst's color — same language as `fpulse` on the fuse button), its icon gets a drop-shadow glow, and the ARMED badge pulses via the existing `awakenPulse`. The HUD button itself also breathes while something is armed. Panel width now clamps to viewport on small screens.
**Verify:** earn any catalyst (resolve a crisis), open 🧿 Cat, hover entries, arm one, watch the panel entry + HUD button pulse.

### 2. Catalyst-consumed banner (fusion result modal)
**Before:** one-off inline-styled box.
**After:** `.catalyst-banner` — a proper member of the reaction/mutation/ecosystem banner family: top hairline gradient in catalyst color, 9px/.22em uppercase label, glowing icon. Sits visually identical in weight to its siblings.
**Verify:** arm a catalyst, fuse, check the result modal between the resonance row and reaction banner.

### 3. Milestones modal (☰ More → 🏆 Milestones)
**Before:** plain codex list, inline `✓ UNLOCKED` text, no sense of progression.
**After:** header gains a progress bar filled with the title gradient (purple→pink→gold) plus a Bebas gold counter with glow ("7 / 16"). Entries stagger in with the existing `fadeUp` (35ms cascade — locked entries land at .55 opacity via a `fadeUpDim` variant so the fill-mode doesn't fight the dimming). Unlocked entries get a gold background tint + gold accent bar; the unlock marker is now a proper badge pill. Footer counts remaining ("· 9 remaining" / "· lab complete").
**Verify:** open Milestones with a mid-progress save — watch the cascade, the bar width, gold vs. dimmed entries.

### 4. Milestone unlock moment
**Before:** identical to a save toast.
**After:** `toast-milestone` variant — gold border/text, outer glow, and a `toastPop` entrance using the house overshoot curve `cubic-bezier(.34,1.56,.64,1)`. Copy upgraded to "🏆 Milestone unlocked: …". Catalyst payouts get a matching cyan `toast-catalyst` variant. (I kept this a toast rather than an interrupting modal — unlocks fire mid-action via autosave, often during fusion resolution; a modal would fight the result modal. The reward-room feel lives in the Milestones modal itself.)
**Verify:** trigger any milestone (first fusion on a fresh game) and watch the toast.

### 5. Gauntlet result screen
**Before:** reused the duel's green/blue `arena-result` box — end-of-run read like any casual match.
**After:** dedicated `.gnt-result` verdict block in the gauntlet's flame identity. Winner/challenger name renders as a Bebas 38px gradient title (white→orange→gold on clear; white→red→orange on defeat) entering with the `cscale` overshoot pop (same as convergence text). Below it, a header-style stat strip (Bebas numbers with flame glow): on wave clear — waves cleared / reserves next wave / waves to next catalyst; on defeat — streak / best, in red. Defeat with a record shows a pulsing gold "★ NEW PERSONAL BEST" badge. The next-wave button is now `.gnt-next-btn`: Bebas, flame gradient, breathing via the existing `fpulse` keyframe with `--fuse-glow` overridden to orange — the "one more wave" pull now looks like the fuse button's "ready" state.
**Verify:** run a gauntlet — clear a wave (title pop + stat strip + pulsing WAVE N button), then lose (red gradient, streak/best numbers, NEW BEST badge if applicable).

### 6. Gauntlet champion picker
**Before:** borrowed relationship cards; best-streak buried as gray text.
**After:** `.gnt-pick` cards — power-colored top bar, flame-tinted hover lift, Rajdhani hero names, "🔥 Best: N waves" right-aligned (or "Unproven" for first-timers).

### 7. Toast queue pacing
**Before:** strict 2.2s per toast; a fusion burst (milestone + catalyst + save) serialized into ~7s of bottom-screen traffic.
**After:** adaptive — a lone toast holds 1.9s (unchanged feel); when more are queued, each holds 1.35s+0.3s fade. A 4-toast burst now clears in ~6.6→~5.0s and *feels* deliberate rather than backed up. Dupe-fusion toast folded into the same `.save-toast` family (purple `toast-dupe` variant, gains the exit fade it previously lacked).
**Verify:** fuse something that triggers a milestone + catalyst payout; toasts should feel briskly sequential.

### 8. Small consistency touches
Title screen version string → "v1.2 · Visual Polish". Mobile media query covers new elements (`.gnt-title` 30px, tighter stat gap, catalyst panel viewport clamp).

## Visual changes with functional side effects — re-test these

1. **`showSaveToast(msg, cls)` signature** — queue now stores `{msg, cls}` objects. All existing callers pass strings only (verified); behavior identical for them. Re-test: any toast still displays text correctly.
2. **Toast timing** — hold time is now 1350ms when queue is non-empty (was always 2200ms total). Nothing reads toast timing elsewhere (verified: crisis rolls use their own timers). Re-test: dense toast bursts.
3. **`renderGauntletResult(..., isNewBest)`** — new trailing param (default `false`); `runGauntletWave` computes it *before* writing `hero.gauntletBest` (write itself unchanged: same condition, same value). Re-test: best streak still records correctly on defeat, including first-ever runs (streak 0 = no badge, no record).
4. **`updateCatalystBtn`** — now toggles a `cat-armed` class and a `--cat-rgb` custom property on the HUD button. Pure presentation, but it runs on every render tick — verify no console errors with 0 catalysts and on a fresh game.
5. **Escape/backdrop close for Milestones** — unchanged code paths, but the modal markup gained a header child (progress bar). `animateClose` targets `.codex-modal`/`.codex-backdrop` — still matches. Re-test: Esc closes Milestones with exit animation.
6. **`showDupeToast`** — now uses `.save-toast` classes instead of inline cssText, and gains a fade-out. Independent of the queue (unchanged). Re-test: fusing a duplicate pair.

Save compatibility: untouched. No reads/writes to the save shape changed; `SAVE_VERSION` still 1.

## HACCS manual-verification script (suggested order)

1. Fresh game → first fusion → gold milestone toast pops (overshoot entrance).
2. Resolve a crisis → cyan catalyst toast → open 🧿 Cat → hover, arm, watch the double pulse (entry + HUD button).
3. Fuse with catalyst armed → catalyst banner in result modal matches reaction-banner styling.
4. ☰ → Milestones → staggered cascade, gradient progress bar, gold unlocked entries.
5. Arena → Gauntlet → picker cards → clear a wave (gradient title pop, stat strip, pulsing WAVE 2 button) → lose (red verdict, streak/best, NEW BEST badge).
6. Mobile width (<700px) → gauntlet title shrinks, catalyst panel stays inside viewport.

## Housekeeping note (not code)

`index.html` v1.1 + v1.2 changes are **uncommitted** in git (working tree only — v1.1 was never committed). Recommend committing from your machine: `git add index.html CHANGELOG-v1.1.md CHANGELOG-v1.2-visual-polish.md && git commit`. I deliberately did not run git from this session — the session's sandbox mirror of the folder was byte-capped/truncated and staging from it could have committed a corrupt file.
