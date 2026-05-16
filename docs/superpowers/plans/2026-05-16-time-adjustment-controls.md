# Time Adjustment Controls Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add ▲/▼ buttons to the right side of the running timer that add/subtract 30 seconds from both `remainingSeconds` and `totalSeconds`, with milestone recalculation.

**Architecture:** All changes are in `index.html` — CSS rules, one new HTML block inside `#screen-timer`, three new DOM refs, and one new `adjustTime(delta)` function wired to two button listeners. Two small modifications to existing functions (`handleTimerEnd`, `btnRestart` listener) to hide/show the panel.

**Tech Stack:** Vanilla JS, Web Audio API, no build tools — open `index.html` in browser to verify.

---

### Task 1: HTML + CSS — Add the adjust panel

**Files:**
- Modify: `index.html` (CSS block ~line 165, HTML block ~line 279)

- [ ] **Step 1: Add `position: relative` to `#screen-timer` CSS rule**

Find this existing rule (~line 118):
```css
#screen-timer {
  gap: 0;
}
```
Replace with:
```css
#screen-timer {
  gap: 0;
  position: relative;
}
```
This is required so that `position: absolute` on `#time-adjust` positions relative to the timer screen, not the viewport.

- [ ] **Step 2: Add `#time-adjust` CSS rules**

Add these rules immediately after the `#btn-pause` rule (~line 193, just before `</style>`):
```css
/* ── Time Adjust Panel ── */
#time-adjust {
  position: absolute;
  right: 1.5rem;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  z-index: 10;
}

#time-adjust button {
  background: rgba(26, 26, 46, 0.85);
  border: 2px solid #2a2a45;
  border-radius: 10px;
  color: #ccc;
  font-size: 1.8rem;
  width: 56px;
  height: 56px;
  line-height: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

#time-adjust button:hover { border-color: #4ade80; color: #fff; }

#time-adjust .adjust-label {
  color: #666;
  font-family: monospace;
  font-size: 0.7rem;
  letter-spacing: 0.05em;
}
```

- [ ] **Step 3: Add the HTML panel inside `#screen-timer`**

Find this comment block (~line 279):
```html
    <!-- Bottom bar -->
    <div id="bottom-bar">
```
Insert the panel **immediately before** it:
```html
    <!-- Time adjust controls -->
    <div id="time-adjust">
      <button id="btn-add-time">▲</button>
      <span class="adjust-label">±0:30</span>
      <button id="btn-sub-time">▼</button>
    </div>

    <!-- Bottom bar -->
    <div id="bottom-bar">
```

- [ ] **Step 4: Verify visually in browser**

Open `index.html` in Chrome. Start a timer. Confirm:
- ▲ and ▼ buttons appear vertically centered on the right side of the screen
- The `±0:30` label appears between them
- Buttons are ~56×56px and easy to tap
- Buttons do nothing yet (no JS wired)

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add time-adjust panel HTML and CSS"
```

---

### Task 2: JS — `adjustTime()` function and event wiring

**Files:**
- Modify: `index.html` (script block)

- [ ] **Step 1: Add DOM refs for the new buttons**

Find the DOM refs block (~line 309). After this line:
```js
const blockDivs      = blocksContainer.querySelectorAll('.block');
```
Add:
```js
const btnAddTime     = $('btn-add-time');
const btnSubTime     = $('btn-sub-time');
const timeAdjust     = $('time-adjust');
```

- [ ] **Step 2: Add the `adjustTime(delta)` function**

Find the `trigger1MinFlash` function (~line 560). Add `adjustTime` immediately before it:
```js
function adjustTime(delta) {
  const newRemaining = Math.max(30, state.remainingSeconds + delta);
  const actualDelta = newRemaining - state.remainingSeconds;
  state.remainingSeconds = newRemaining;
  state.totalSeconds = Math.max(30, state.totalSeconds + actualDelta);
  if (state.remainingSeconds > 60) state.milestoneTriggered = false;
  if (state.remainingSeconds > 15) state.milestone15Triggered = false;
  updateTimerDisplay();
  checkMilestones();
}
```

`actualDelta` handles clamping: if `remainingSeconds` was 45 and delta is −30, `newRemaining` clamps to 30 and `actualDelta` is −15, so `totalSeconds` also shrinks by only 15.

- [ ] **Step 3: Wire up button click listeners**

Find the `btnToggleView` listener (~line 515). Add these two listeners immediately after its closing `});`:
```js
btnAddTime.addEventListener('click', () => adjustTime(30));
btnSubTime.addEventListener('click', () => adjustTime(-30));
```

- [ ] **Step 4: Hide the panel when timer ends**

Find `handleTimerEnd` (~line 473). After `updateTimerDisplay();` and before the closing `}`, add:
```js
timeAdjust.classList.add('hidden');
```

So the full function becomes:
```js
function handleTimerEnd() {
  state.running = false;
  document.body.classList.remove('flash-1min', 'flash-15sec');
  void document.body.offsetWidth;
  document.body.classList.add('flash-end');
  playSiren();
  updateTimerDisplay();
  btnPause.textContent = '⏹ Done';
  timeAdjust.classList.add('hidden');
}
```

- [ ] **Step 5: Show the panel again on restart**

Find the `btnRestart` listener (~line 490). After `btnPause.textContent = '⏸ Pause';`, add:
```js
timeAdjust.classList.remove('hidden');
```

So that section becomes:
```js
btnPause.textContent = '⏸ Pause';
timeAdjust.classList.remove('hidden');
updateTimerDisplay();
```

- [ ] **Step 6: Verify behavior in browser**

Open `index.html`. Run through these checks:

**Add time:**
- Start a 5-minute timer, press ▲ — remaining and total both increase by 0:30, ring/blocks stay at same visual percentage

**Subtract time:**
- Press ▼ repeatedly near 0:31 — it stops at 0:30 and won't go lower

**Milestone reset:**
- Start a 3-minute timer, let it count to ~0:55 (1-min flash fires), press ▲ three times to get back to ~1:25 remaining — then let it count down again and confirm the 1-min flash fires a second time

**End state:**
- Let timer reach 0:00 — ▲/▼ buttons should disappear

**Restart:**
- After timer ends, press Restart — ▲/▼ buttons reappear

**Paused:**
- Pause the timer, press ▲ and ▼ — display updates immediately even while paused

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: implement time adjustment controls"
```
