# Time Adjustment Controls — Design Spec

**Date:** 2026-05-16  
**Status:** Approved

## Overview

Add ▲/▼ buttons to the right side of the running timer screen so the user can add or subtract time in 30-second increments without stopping the timer.

## Layout

- **Position:** Absolutely positioned inside `#timer-screen`, vertically centered on the right edge (`position: absolute; right: 1.5rem; top: 50%; transform: translateY(-50%)`)
- **Structure:** ▲ button on top, `±0:30` label in the middle, ▼ button below — stacked vertically
- **Size:** Minimum 56×56px tap targets (TV + touch friendly)
- **Style:** Semi-transparent dark background, border-radius matching existing bottom-bar buttons

## Behavior

### `adjustTime(delta)` — called with `+30` or `−30`

1. `newRemaining = Math.max(30, state.remainingSeconds + delta)`
2. `actualDelta = newRemaining − state.remainingSeconds` — may differ from `delta` when clamped at the floor
3. `state.remainingSeconds = newRemaining`
4. `state.totalSeconds += actualDelta` — total shifts by the same amount as remaining
5. Reset milestone flags if adjustment moved back out of their windows:
   - `state.remainingSeconds > 60` → `state.milestoneTriggered = false`
   - `state.remainingSeconds > 15` → `state.milestone15Triggered = false`
6. Call `updateTimerDisplay()` — re-renders ring/block progress and color immediately
7. Call `checkMilestones()` — so subtracting into a milestone zone while paused triggers the flash right away

### Constraints

- **Minimum:** 30 seconds — the ▼ button cannot reduce `remainingSeconds` below 30
- **Works while:** running and paused
- **End state:** buttons are hidden when the timer has ended (siren playing, frozen at 0:00)

### Milestone recalculation

Both milestone flags are tied to absolute thresholds (≤60 sec, ≤15 sec), not percentages. Adding time can push remaining back above a threshold, resetting the flag so the warning fires again naturally when the timer counts back down through it.

### Color/progress recalculation

`totalSeconds` shifts with `remainingSeconds` by `actualDelta`, preserving the current percentage position. The ring arc, block fill, and color band all stay visually consistent — no jump in the progress indicator.
