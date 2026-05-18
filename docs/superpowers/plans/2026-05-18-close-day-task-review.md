# Close Day Task Review — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Gate the Close Day flow behind a sequential review of all incomplete tasks, and add auto-complete labels to the closing flow prompts that fire them.

**Architecture:** The existing `closingModal` is reused. A new Phase 1 runs before `closingStep` counting begins — it uses its own index (`closingTaskIdx`) to walk through incomplete tasks. Phase 2 is the trimmed existing flow (steps 1→3, step 2 "open notes" removed, renumbered). `renderClosingStep` is restructured to dispatch to either phase based on whether `closingTaskIdx < closingTasks.length`.

**Tech Stack:** Vanilla JS/HTML/CSS, single file `index.html`. No build step.

---

## File Map

| Area | What changes |
|------|-------------|
| `closingDraft` global | Remove `hasOpenNotes` field |
| `closingStep` / new `closingTaskIdx` globals | Add `closingTaskIdx` and `closingTasks` array |
| `openClosingFlow` | Collect incomplete base tasks, initialise Phase 1 vars |
| `renderClosingStep` | Restructure: dispatch Phase 1 (task review) or Phase 2 (closing steps) |
| `nextClosingStep` | Remove step 2 (open notes) logic, renumber steps 1→2 |
| `prevClosingStep` | Aware of Phase 1 boundary — no back from Phase 1 |
| `submitClosingDay` | Remove `autoComplete('close_note', 100)` |
| CSS | Add `.closing-task-review` styles for Phase 1 task card |

---

## Task 1: Remove "open notes" step, renumber Phase 2, remove close_note auto-complete

**Files:**
- Modify: `index.html` — `closingDraft`, `nextClosingStep`, `renderClosingStep`, `submitClosingDay`

This task cleans up the existing flow before Phase 1 is inserted. After this task the closing flow has 3 steps: readiness (1), highlight (2), summary (3).

- [ ] **Step 1: Remove `hasOpenNotes` from `closingDraft`**

Find (around line 1474):
```javascript
let closingDraft = { readiness: 75, hasOpenNotes: 'yes', highlight: '' };
```
Replace with:
```javascript
let closingDraft = { readiness: 75, highlight: '' };
```

- [ ] **Step 2: Remove `autoComplete('close_note', 100)` from `submitClosingDay`**

Find `submitClosingDay` (around line 1557). Remove:
```javascript
autoComplete('close_note', 100);
```

The function should become:
```javascript
function submitClosingDay() {
  const currentDayIdx = getTodayIndex();
  viewDayIdx = currentDayIdx;
  state.todayPhase = 'done';
  state.todayPhaseDayIdx = currentDayIdx;
  saveState();
  closeFlowModal('closingModal');
  submitDay();
}
```

- [ ] **Step 3: Rewrite `renderClosingStep` Phase 2 steps (1→2, remove step 2 "open notes")**

Find `renderClosingStep` (around line 1488). Replace the entire function with this version that has 3 steps (readiness, highlight, summary) and adds the sublabel to step 1:

```javascript
function renderClosingStep() {
  const content = $('closingStepContent');
  const actions = $('closingActions');
  if (!content || !actions) return;

  if (closingStep === 1) {
    content.innerHTML = `
      <div class="flow-question">
        <div class="flow-question-label">كيف كانت جاهزية الموقع؟</div>
        <div class="flow-sublabel">سيُضاف كجولة إغلاق اليوم</div>
        <div class="flow-slider-row">
          <input id="closingReadinessSlider" type="range" min="0" max="100" value="${closingDraft.readiness}">
          <span class="flow-slider-value" id="closingReadinessValue">${closingDraft.readiness}%</span>
        </div>
      </div>`;
    actions.innerHTML = `<button class="flow-secondary" onclick="closeFlowModal('closingModal')">إلغاء</button><button class="flow-primary" onclick="nextClosingStep()">التالي</button>`;
    const slider = $('closingReadinessSlider');
    slider.oninput = () => {
      closingDraft.readiness = normalizePct(slider.value);
      $('closingReadinessValue').textContent = closingDraft.readiness + '%';
    };
  } else if (closingStep === 2) {
    content.innerHTML = `
      <div class="flow-question">
        <div class="flow-question-label">أهم شيء اليوم</div>
        <textarea class="flow-textarea" id="closingHighlightInput" maxlength="160" placeholder="وش صار اليوم؟">${escapeHtml(closingDraft.highlight)}</textarea>
      </div>`;
    actions.innerHTML = `<button class="flow-secondary" onclick="prevClosingStep()">رجوع</button><button class="flow-primary" onclick="nextClosingStep()">عرض الملخص</button>`;
  } else {
    const checks = (state.todayChecks || {})[getTodayIndex()] || {};
    const { xp, extraXP } = calcDayXP(checks, getPillars(getTodayIndex()), getNAMap(getTodayIndex()));
    const multiplier = getFollowupMultiplier(getSavedFollowupPoints() + extraXP);
    const levelXP = Math.round(xp * multiplier);
    content.innerHTML = `<div class="closing-summary">نقاط اليوم<strong>${levelXP} XP${extraXP ? ' + ' + extraXP + ' متابعة' : ''}</strong></div>`;
    actions.innerHTML = `<button class="flow-secondary" onclick="prevClosingStep()">رجوع</button><button class="flow-primary" onclick="submitClosingDay()">أغلق اليوم</button>`;
  }
}
```

- [ ] **Step 4: Rewrite `nextClosingStep` without step 2 open-notes logic**

Find `nextClosingStep` (around line 1540). Replace:

```javascript
function nextClosingStep() {
  const currentDayIdx = getTodayIndex();
  if (closingStep === 1) {
    addZoneRound(currentDayIdx, 'إغلاق اليوم', closingDraft.readiness, '', 'أخرى');
  } else if (closingStep === 2) {
    closingDraft.highlight = ($('closingHighlightInput')?.value || '').trim();
    const drafts = ensureObject(state, 'evidenceDrafts');
    ensureArray(drafts, currentDayIdx).push({ id: makeId('ev_'), category: 'أثر واضح', text: closingDraft.highlight || 'أهم شيء اليوم', timestamp: new Date().toISOString() });
    saveState();
  }
  closingStep++;
  renderClosingStep();
  updateAllUI();
}
```

- [ ] **Step 5: Add `.flow-sublabel` CSS**

In the `<style>` block, after the `.flow-question-label` rule (search for `.flow-question-label`), add:

```css
.flow-sublabel { font-size: 12px; color: var(--text-dim); margin-top: 2px; margin-bottom: 8px; }
```

- [ ] **Step 6: Verify**

Open the file in a browser. Trigger Close Day. Confirm:
- Step 1 shows readiness slider with sublabel "سيُضاف كجولة إغلاق اليوم"
- Step 2 shows highlight textarea (no open-notes step)
- Step 3 shows XP summary with "أغلق اليوم" button
- `close_note` task is NOT auto-completed when the day closes

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "refactor: remove open-notes step, add readiness sublabel, remove close_note auto-complete"
```

---

## Task 2: Add Phase 1 — Incomplete Task Review

**Files:**
- Modify: `index.html` — globals, `openClosingFlow`, `renderClosingStep`, `renderTaskReviewStep` (new), CSS

This is the main task. Phase 1 runs before `closingStep` counting starts. It uses two new globals: `closingTasks` (array of incomplete base task objects) and `closingTaskIdx` (current position in that array).

The flow:
- `openClosingFlow` collects incomplete tasks into `closingTasks`, sets `closingTaskIdx = 0`
- `renderClosingStep` checks: if `closingTaskIdx < closingTasks.length` → render task review step; else → render Phase 2 step (`closingStep`)
- `nextClosingTaskStep(action)` handles the three actions and advances `closingTaskIdx`; when `closingTaskIdx >= closingTasks.length`, Phase 1 is done and Phase 2 begins (`closingStep = 1`, `renderClosingStep()`)

- [ ] **Step 1: Add `closingTasks` and `closingTaskIdx` globals**

Find the existing globals (around line 1473):
```javascript
let closingStep = 1;
let closingDraft = { readiness: 75, highlight: '' };
```
Add below them:
```javascript
let closingTasks = [];
let closingTaskIdx = 0;
let closingNAInput = '';
```

- [ ] **Step 2: Update `openClosingFlow` to collect incomplete tasks**

Find `openClosingFlow` (around line 1476). Replace:

```javascript
function openClosingFlow() {
  if (isDaySubmitted(getTodayIndex())) return;
  state.todayPhase = 'closing';
  state.todayPhaseDayIdx = getTodayIndex();
  saveState();
  closingStep = 1;
  closingDraft = { readiness: calcReadiness(getZoneRoundsForDay(getTodayIndex())) || 75, highlight: '' };

  // Collect incomplete base tasks for Phase 1
  const dayIdx = getTodayIndex();
  const checks = (state.todayChecks || {})[dayIdx] || {};
  const naMap = getNAMap(dayIdx);
  closingTasks = getPillars(dayIdx)
    .flatMap(p => p.tasks.filter(t => t._scope === 'base'))
    .filter(t => !naMap[t.id] && getCheckValue(checks, t.id) !== 100);
  closingTaskIdx = 0;
  closingNAInput = '';

  renderClosingStep();
  openFlowModal('closingModal');
  updateHomeUI();
}
```

- [ ] **Step 3: Update `renderClosingStep` to dispatch Phase 1 or Phase 2**

Find `renderClosingStep` (updated in Task 1). Add Phase 1 dispatch at the top of the function, before the `if (closingStep === 1)` check:

```javascript
function renderClosingStep() {
  const content = $('closingStepContent');
  const actions = $('closingActions');
  if (!content || !actions) return;

  // Phase 1: incomplete task review
  if (closingTaskIdx < closingTasks.length) {
    renderTaskReviewStep(content, actions);
    return;
  }

  // Phase 2: closing steps
  if (closingStep === 1) {
    // ... (existing step 1 code unchanged)
  } else if (closingStep === 2) {
    // ... (existing step 2 code unchanged)
  } else {
    // ... (existing summary code unchanged)
  }
}
```

- [ ] **Step 4: Add `renderTaskReviewStep` function**

Add this function immediately before `renderClosingStep`:

```javascript
function renderTaskReviewStep(content, actions) {
  const task = closingTasks[closingTaskIdx];
  const total = closingTasks.length;
  const current = closingTaskIdx + 1;

  content.innerHTML = `
    <div class="closing-task-review">
      <div class="closing-task-progress">مهام اليوم · ${current} من ${total}</div>
      <div class="closing-task-text">${escapeHtml(task.text)}</div>
      <div class="closing-task-btns" id="closingTaskBtns">
        <button class="flow-choice closing-task-btn-complete" onclick="advanceTaskReview('complete')">اكتملت</button>
        <button class="flow-choice closing-task-btn-na" onclick="advanceTaskReview('na')">لا ينطبق</button>
        <button class="flow-choice closing-task-btn-skip" onclick="advanceTaskReview('skip')">لم تكتمل</button>
      </div>
      <div class="closing-task-na-row" id="closingTaskNARow" style="display:none">
        <input type="text" id="closingTaskNAReason" maxlength="80" placeholder="السبب (مطلوب)"
          class="flow-textarea" style="height:auto;padding:8px 10px;font-size:14px;margin-top:8px;">
        <button class="flow-primary" style="margin-top:8px;width:100%" onclick="confirmTaskNA()">تأكيد</button>
      </div>
    </div>`;
  actions.innerHTML = '';
}

function advanceTaskReview(action) {
  const task = closingTasks[closingTaskIdx];
  if (action === 'complete') {
    autoComplete(task.id, 100);
    closingTaskIdx++;
    renderClosingStep();
    updateAllUI();
  } else if (action === 'na') {
    const naRow = $('closingTaskNARow');
    if (naRow) naRow.style.display = 'block';
    setTimeout(() => $('closingTaskNAReason')?.focus(), 0);
  } else if (action === 'skip') {
    // task stays at 0, no state change
    closingTaskIdx++;
    renderClosingStep();
  }
}

function confirmTaskNA() {
  const reason = ($('closingTaskNAReason')?.value || '').trim();
  if (!reason) { $('closingTaskNAReason')?.focus(); return; }
  const task = closingTasks[closingTaskIdx];
  setTaskNA(task.id, reason);
  closingTaskIdx++;
  renderClosingStep();
  updateAllUI();
}
```

- [ ] **Step 5: Add CSS for task review step**

In the `<style>` block, after the `.flow-sublabel` rule added in Task 1, add:

```css
.closing-task-review { display: flex; flex-direction: column; gap: 12px; }
.closing-task-progress { font-size: 12px; color: var(--gold-dim); font-family: 'Cinzel Decorative', cursive; }
.closing-task-text { font-size: 16px; color: var(--text); line-height: 1.5; font-family: 'Amiri', serif; }
.closing-task-btns { display: flex; gap: 8px; }
.closing-task-btns .flow-choice { flex: 1; padding: 10px 6px; font-size: 14px; text-align: center; }
.closing-task-btn-complete.active, .closing-task-btn-complete:hover { background: rgba(42,122,106,0.2); border-color: var(--teal); color: var(--teal-light); }
.closing-task-btn-na.active, .closing-task-btn-na:hover { background: rgba(200,151,42,0.15); border-color: var(--gold-dim); color: var(--gold); }
.closing-task-btn-skip.active, .closing-task-btn-skip:hover { background: rgba(122,32,32,0.1); border-color: var(--red); color: var(--red-light); }
```

- [ ] **Step 6: Verify**

Open browser. Go home screen. Leave some tasks incomplete. Trigger Close Day.
Confirm:
1. Phase 1 appears first — task text shown, progress header "١ من X"
2. Tap "اكتملت" → task advances, next task shown (or Phase 2 if last)
3. Tap "لا ينطبق" → reason input appears inline → confirm → advances
4. Tap "لم تكتمل" → advances with no state change, task stays at 0
5. After all tasks handled → Phase 2 begins with readiness slider (sublabel visible)
6. If no incomplete tasks → Phase 1 skipped entirely, straight to readiness slider
7. Day submits correctly from summary step

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: Close Day Phase 1 — sequential incomplete task review before closing steps"
```

---

## Self-Review

**Spec coverage:**
- ✅ Gate: can't close without acting on all incomplete tasks — Phase 1 collects and walks through them
- ✅ Three actions: complete (full XP), لا ينطبق (reason required, setTaskNA), لم تكتمل (0 XP, acknowledged)
- ✅ Sequential one-by-one — `closingTaskIdx` advances per task
- ✅ "لا ينطبق" inline reason input — shown/hidden within same step
- ✅ Progress label "X من Y"
- ✅ No back navigation in Phase 1 — `prevClosingStep` only fires in Phase 2
- ✅ Phase 2 readiness step has sublabel "سيُضاف كجولة إغلاق اليوم"
- ✅ Open-notes step removed
- ✅ `close_note` auto-complete removed from `submitClosingDay`
- ✅ Phase 1 skipped when no incomplete tasks

**Watch: `setTaskNA` uses `viewDayIdx`** — in the closing flow context, `viewDayIdx` should equal `getTodayIndex()` because `openClosingFlow` only fires for today. This is safe as long as no one calls `openClosingFlow` while viewing a past day. The `isDaySubmitted(getTodayIndex())` guard already prevents that.

**Watch: `prevClosingStep`** — currently just decrements `closingStep`. After Task 1, Phase 2 starts at step 1. If the user is at Phase 2 step 1 and hits back, `prevClosingStep` decrements to step 0. Add a guard so `closingStep` cannot go below 1:

In `prevClosingStep`:
```javascript
function prevClosingStep() {
  if (closingStep > 1) closingStep--;
  renderClosingStep();
}
```
This is already the existing code — no change needed. Step 1 back button says "إلغاء" (cancel), not "رجوع", so the user can't go back past step 1 anyway. ✅
