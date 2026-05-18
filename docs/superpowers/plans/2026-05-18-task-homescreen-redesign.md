# Task & Home Screen Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Redesign the task list (remove duplicates, convert partials to binary, add N/A mechanic) and build a collapsible home screen task panel so users can complete their full day from either view.

**Architecture:** All changes are inside the single `index.html` file. State lives in `localStorage` under `hajj_rpg_v3`. The N/A mechanic adds a new `todayNA` key to state (parallel to `todayChecks`). XP calc is updated to exclude NA tasks from both numerator and denominator. The home panel re-uses the existing `toggleBinary` and NA functions — no duplicate logic.

**Tech Stack:** Vanilla JS, HTML, CSS — no build step, no framework.

---

## File Map

| File | What changes |
|------|-------------|
| `index.html` — `BASE_PILLARS` const | Remove 5 tasks, update 3 task texts, set all `partial: false` |
| `index.html` — `normalizeLoadedState` + `initState` | Add `todayNA` key |
| `index.html` — `calcDayXP` | Accept and apply `naMap` to exclude NA tasks |
| `index.html` — `submitDay` | Pass `naMap` into `calcDayXP`, snapshot NA state |
| `index.html` — `buildBinaryItem` | Render NA state (muted row + reason text) |
| `index.html` — `buildActionBtns` | Add "لا ينطبق" button |
| `index.html` — NA modal HTML | New modal for N/A reason input |
| `index.html` — NA CSS | `.task-item.na` muted style |
| `index.html` — `submitNoteModal` | Remove `autoComplete('log_note', 100)` |
| `index.html` — `submitQuickCheckModal` | Remove the 6 `autoComplete` calls for facilities tasks |
| `index.html` — `nextClosingStep` (step 3) | Remove `autoComplete('update', 100)` |
| `index.html` — `submitClosingDay` | Add `autoComplete('close_note', 100)` unconditionally |
| `index.html` — `nextClosingStep` (step 2) | Remove conditional `autoComplete('close_note', 100)` |
| `index.html` — home screen HTML | Add collapsible `#homeTasksPanel` section |
| `index.html` — home screen CSS | `.home-tasks-*` styles |
| `index.html` — `renderHomeTasks` (new) | Compact pillar+task renderer for home panel |
| `index.html` — `updateHomeUI` | Call `renderHomeTasks` |

---

## Task 1: Update BASE_PILLARS

**Files:**
- Modify: `index.html` lines 618–638 (`BASE_PILLARS` const)

Remove the 5 tasks that either duplicate zone rounds or were absorbed into redesigned tasks. Update text and `partial` flag on 3 tasks.

- [ ] **Step 1: Replace BASE_PILLARS**

Find the `BASE_PILLARS` const (line 618) and replace it with:

```javascript
const BASE_PILLARS = [
  { id: 'attendance', icon: '🕌', name: 'الحضور والانضباط', tasks: [
    { id: 'checkin',     text: 'حضور الاجتماع الصباحي والاستعداد للعمل', xp: 12, partial: false, core: true },
    { id: 'briefing',   text: 'فهم أولويات اليوم قبل الانشغال بالتفاصيل', xp: 10, partial: false, core: true },
    { id: 'shift_ready', text: 'هل أنهيت يومك بوضوح ذهني وراجعت ما تم وما لم يتم؟', xp: 18, partial: false }
  ]},
  { id: 'facilities', icon: '🚶', name: 'الموقع والجولات', tasks: [
    { id: 'corridor_crowding', text: 'مراجعة وجود ما يستحق الانتباه', xp: 8,  partial: false, readiness: true },
    { id: 'fridge_empty',      text: 'التأكد من عدم ترك احتياج واضح بلا متابعة', xp: 8, partial: false, readiness: true },
    { id: 'cleaning_weak',     text: 'مراجعة ما يحتاج متابعة لاحقة', xp: 8, partial: false, readiness: true }
  ]},
  { id: 'reports', icon: '📋', name: 'التقارير والمتابعة', tasks: [
    { id: 'update',     text: 'إذا واجهت مشكلة — هل حاولت حلها؟ وإن لم تستطع، هل أوصلتها بوضوح مع مقترح لمسؤولك؟', xp: 22, partial: false, core: true, readiness: true },
    { id: 'close_note', text: 'مراجعة المفتوح قبل إغلاق اليوم', xp: 24, partial: false, core: true, readiness: true }
  ]}
];
```

- [ ] **Step 2: Verify in browser**

Open `index.html`. Go to the Tasks page. Confirm the three pillars each show the correct tasks — 3 tasks in الحضور, 3 in الموقع, 2 in التقارير. Confirm no percentage inputs appear anywhere.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "refactor: update BASE_PILLARS — remove duplicates, convert to binary, update task text"
```

---

## Task 2: Add N/A State Infrastructure

**Files:**
- Modify: `index.html` — `normalizeLoadedState`, `initState`, new helper functions, `calcDayXP`, `submitDay`

The N/A mechanic stores per-day per-task dismissals in `state.todayNA[dayIdx][taskId] = { reason }`. Submitted days snapshot NA state into `state.days[dayIdx].naMap`. XP calc excludes NA tasks from both earned and possible XP.

- [ ] **Step 1: Update `normalizeLoadedState`**

In `normalizeLoadedState` (around line 762), find the line:
```javascript
['days', 'todayChecks', 'todayCheckTimestamps', 'zoneRoundsDrafts', 'evidenceDrafts', 'customTasks', 'dayCustomTasks', 'taskOverrides'].forEach(key => {
```
Add `'todayNA'` to the array:
```javascript
['days', 'todayChecks', 'todayCheckTimestamps', 'zoneRoundsDrafts', 'evidenceDrafts', 'customTasks', 'dayCustomTasks', 'taskOverrides', 'todayNA'].forEach(key => {
```

- [ ] **Step 2: Update `initState`**

In `initState` (around line 830), find:
```javascript
state = { startDate, totalDays: parseInt(totalDays), totalXP: 0, streak: 0, perfectDays: 0,
    todayPhase: 'morning', todayPhaseDayIdx: 0,
    days: {}, todayChecks: {}, todayCheckTimestamps: {}, zoneRoundsDrafts: {}, evidenceDrafts: {}, customTasks: {}, dayCustomTasks: {}, hiddenTasks: [], taskOverrides: {} };
```
Add `todayNA: {}`:
```javascript
state = { startDate, totalDays: parseInt(totalDays), totalXP: 0, streak: 0, perfectDays: 0,
    todayPhase: 'morning', todayPhaseDayIdx: 0,
    days: {}, todayChecks: {}, todayCheckTimestamps: {}, zoneRoundsDrafts: {}, evidenceDrafts: {}, customTasks: {}, dayCustomTasks: {}, hiddenTasks: [], taskOverrides: {}, todayNA: {} };
```

- [ ] **Step 3: Add NA helper functions**

Add these four functions immediately after the `autoComplete` function (after line 850):

```javascript
function getNAMap(dayIdx) {
  if (isDaySubmitted(dayIdx)) return state.days[dayIdx]?.naMap || {};
  return (state.todayNA || {})[dayIdx] || {};
}

function isTaskNA(dayIdx, taskId) {
  return !!(getNAMap(dayIdx)[taskId]);
}

function setTaskNA(taskId, reason) {
  const dayIdx = getTodayIndex();
  ensureObject(ensureObject(state, 'todayNA'), dayIdx)[taskId] = { reason };
  saveState();
}

function clearTaskNA(taskId) {
  const dayIdx = getTodayIndex();
  const map = (state.todayNA || {})[dayIdx];
  if (map) { delete map[taskId]; saveState(); }
}
```

- [ ] **Step 4: Update `calcDayXP` to exclude NA tasks**

Find `calcDayXP` (line 939) and replace it entirely:

```javascript
function calcDayXP(checks, pillars, naMap) {
  const na = naMap || {};
  let baseXP = 0, extraXP = 0, total = 0, fullDone = 0, baseTotal = 0, baseFullDone = 0;
  pillars.forEach(p => p.tasks.forEach(t => {
    if (na[t.id]) return;
    total++;
    const v = getCheckValue(checks, t.id);
    const earned = earnedXP(t.xp, v);
    if (t._scope === 'base') {
      baseTotal++;
      baseXP += earned;
      if (v === 100) baseFullDone++;
    } else {
      extraXP += earned;
    }
    if (v === 100) fullDone++;
  }));
  const perfect = baseTotal > 0 && baseFullDone === baseTotal;
  const xp = baseXP + (perfect ? PERFECT_BONUS : 0);
  return { xp, activityXP: xp + extraXP, extraXP, perfect, total, fullDone, baseTotal, baseFullDone };
}
```

- [ ] **Step 5: Pass naMap in all `calcDayXP` call sites**

Search for every call to `calcDayXP` and add the `naMap` argument. There are 4 call sites:

**In `updateQuestUI`** (around line 1520):
```javascript
// old:
const { xp, extraXP, perfect } = calcDayXP(checks, pillars);
// new:
const naMap = isDaySubmitted(viewDayIdx) ? (state.days[viewDayIdx]?.naMap||{}) : getNAMap(viewDayIdx);
const { xp, extraXP, perfect } = calcDayXP(checks, pillars, naMap);
```

**In `updateHomeUI`** (around line 1137):
```javascript
// old:
const liveExtra = liveChecks ? calcDayXP(liveChecks, getPillars(todayIdx)).extraXP : 0;
// new:
const liveExtra = liveChecks ? calcDayXP(liveChecks, getPillars(todayIdx), getNAMap(todayIdx)).extraXP : 0;
```

**In `renderClosingStep`** summary step (around line 1346):
```javascript
// old:
const { xp, extraXP } = calcDayXP(checks, getPillars(getTodayIndex()));
// new:
const { xp, extraXP } = calcDayXP(checks, getPillars(getTodayIndex()), getNAMap(getTodayIndex()));
```

**In `migratePhantomXP`** (around line 704) — NA didn't exist historically, so pass empty object:
```javascript
// old:
snapshot.forEach(p => (p.tasks || []).forEach(t => {
// new (no change needed — migration runs on old data which has no NA):
// leave migratePhantomXP unchanged — it processes historical days with no NA
```

- [ ] **Step 6: Snapshot NA into submitted day**

In `submitDay` function, find where the day snapshot is saved. Locate the line that builds the day record (search for `state.days[currentDayIdx]` assignment). Add `naMap` to the snapshot:

Find the `submitDay` function and locate where it sets `state.days[currentDayIdx]`. Add `naMap: getNAMap(currentDayIdx)` to that object. For example, if the existing code is:
```javascript
state.days[currentDayIdx] = {
  done: true,
  tasks: { ...checks },
  ...
```
Add:
```javascript
state.days[currentDayIdx] = {
  done: true,
  tasks: { ...checks },
  naMap: { ...(getNAMap(currentDayIdx)) },
  ...
```

- [ ] **Step 7: Verify XP calc**

Open browser. Mark one task as N/A (in next task). Confirm the XP shown on the tasks page denominator adjusts — if max was 110, dismissing the 8 XP `corridor_crowding` task should show max as 102.

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: add N/A state infrastructure and XP calc exclusion"
```

---

## Task 3: N/A UI on Tasks Page

**Files:**
- Modify: `index.html` — CSS section, NA modal HTML, `buildBinaryItem`, `buildActionBtns`, new NA prompt functions

- [ ] **Step 1: Add CSS for NA task state**

In the CSS block (inside `<style>`), after the `.task-item.done` rule (around line 108), add:

```css
.task-item.na { border-color: var(--border); background: transparent; opacity: 0.5; }
.task-item.na .task-text { text-decoration: line-through; color: var(--text-dim); }
.task-item.na .task-check { border-color: var(--border); cursor: default; }
.task-na-reason { display: block; margin-top: 3px; font-size: 12px; color: var(--text-dim); font-style: italic; }
```

- [ ] **Step 2: Add N/A reason modal HTML**

In the HTML, after the `noteModal` div (search for `id="noteModal"`), add this new modal:

```html
<div class="modal-overlay" id="naModal">
  <div class="modal-box">
    <div class="modal-title">لا ينطبق اليوم</div>
    <div class="modal-body">
      <div class="flow-question-label" style="margin-bottom:8px">السبب (مطلوب)</div>
      <input type="text" id="naReasonInput" maxlength="80" placeholder="مثال: ما كانت فيه مشاكل اليوم"
        class="setup-input" style="margin-bottom:0; text-align:right; width:100%; box-sizing:border-box;">
    </div>
    <div class="modal-actions" style="display:flex;gap:8px;margin-top:12px;">
      <button class="flow-secondary" style="flex:1" onclick="closeFlowModal('naModal')">إلغاء</button>
      <button class="flow-primary" style="flex:1" onclick="submitNA()">تأكيد</button>
    </div>
  </div>
</div>
```

- [ ] **Step 3: Add N/A prompt JS functions**

After `clearTaskNA` (added in Task 2), add:

```javascript
let _naPendingTaskId = null;

function openNAPrompt(taskId) {
  _naPendingTaskId = taskId;
  const input = $('naReasonInput');
  if (input) input.value = '';
  openFlowModal('naModal');
  setTimeout(() => input?.focus(), 0);
}

function submitNA() {
  const reason = ($('naReasonInput')?.value || '').trim();
  if (!reason) { $('naReasonInput')?.focus(); return; }
  if (_naPendingTaskId) setTaskNA(_naPendingTaskId, reason);
  _naPendingTaskId = null;
  closeFlowModal('naModal');
  updateAllUI();
}
```

- [ ] **Step 4: Update `buildActionBtns` to add N/A button**

Find `buildActionBtns` (line 1622) and replace it:

```javascript
function buildActionBtns(task, pillar, parentRef) {
  const actions = el('div', 'task-actions');

  const naBtn = el('button', 'task-action-btn', '—');
  naBtn.title = 'لا ينطبق اليوم';
  naBtn.style.fontSize = '11px';
  naBtn.style.color = 'var(--text-dim)';
  naBtn.onclick = e => { e.stopPropagation(); openNAPrompt(task.id); };

  const editBtn = el('button', 'task-action-btn edit', '✏️');
  editBtn.title = 'عدّل';
  editBtn.onclick = e => { e.stopPropagation(); showEditRow(task, pillar, parentRef); };

  const delBtn = el('button', 'task-action-btn del', '✕');
  delBtn.title = 'احذف';
  delBtn.onclick = e => { e.stopPropagation(); deleteTask(task, pillar); };

  actions.append(naBtn, editBtn, delBtn);
  return actions;
}
```

- [ ] **Step 5: Update `buildBinaryItem` to render NA state**

Find `buildBinaryItem` (line 1540) and replace it:

```javascript
function buildBinaryItem(task, isDone, pillar, submitted) {
  const dayIdx = viewDayIdx;
  const isNA = !submitted && isTaskNA(dayIdx, task.id);
  const naData = isNA ? getNAMap(dayIdx)[task.id] : null;

  const item = el('div', 'task-item' + (isDone ? ' done' : '') + (isNA ? ' na' : ''));

  const chk = el('div', 'task-check', isDone ? '✓' : (isNA ? '—' : ''));
  if (!submitted && !isNA) chk.onclick = () => toggleBinary(task.id, task.xp, task._scope);

  const copy = el('div', 'task-copy');
  if (!submitted && !isNA) copy.onclick = () => toggleBinary(task.id, task.xp, task._scope);

  const txt = el('span', 'task-text', task.text);
  copy.appendChild(txt);

  if (isNA && naData?.reason) {
    const reason = el('span', 'task-na-reason', 'لا ينطبق: ' + naData.reason);
    copy.appendChild(reason);
  } else if (task.note) {
    const note = el('span', 'task-note', task.note);
    copy.appendChild(note);
  }

  const xpSpan = el('span', 'task-xp', '+' + task.xp);
  item.append(chk, copy, xpSpan);

  if (!submitted) {
    if (isNA) {
      const undoBtn = el('button', 'task-action-btn', '↩');
      undoBtn.title = 'استعد للمهمة';
      undoBtn.style.color = 'var(--gold-dim)';
      undoBtn.onclick = () => { clearTaskNA(task.id); updateAllUI(); };
      const actions = el('div', 'task-actions');
      actions.appendChild(undoBtn);
      item.appendChild(actions);
    } else {
      item.appendChild(buildActionBtns(task, pillar, item));
    }
  }
  return item;
}
```

- [ ] **Step 6: Remove `buildPartialItem` references**

In `updateQuestUI` (around line 1504), find:
```javascript
tasksDiv.appendChild(task.partial
  ? buildPartialItem(task, val, pillar, submitted)
  : buildBinaryItem(task, val === 100, pillar, submitted));
```
Replace with:
```javascript
tasksDiv.appendChild(buildBinaryItem(task, val === 100, pillar, submitted));
```

The `buildPartialItem` function body can be left in place (dead code, won't be called) or deleted — deleting is cleaner. Find the `// ── PARTIAL TASK ──` comment and delete from there through the closing `}` of `buildPartialItem` (lines 1564–1619).

Also remove the partial-related fields from the edit task form. In the edit row HTML (`showEditRow` function), find:
```javascript
<div class="task-edit-partial-wrap">
  <input type="checkbox" id="editPartialChk" ...>
  <label for="editPartialChk">نسبة الاتمام (يدوي)</label>
```
and remove that entire `task-edit-partial-wrap` div. Remove `partialInput` from the `controls()` call and the `partial` variable in `saveTaskEdit` — set `partial: false` always.

- [ ] **Step 7: Verify N/A flow**

Open browser → Tasks page → tap the "—" button on any task → confirm reason modal appears → enter reason → confirm task shows muted/struck-through with reason beneath → confirm XP denominator decreased → confirm "↩" button restores the task → confirm submitted days show tasks normally (no NA controls).

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: add N/A task UI — dismiss with reason, undo, muted rendering"
```

---

## Task 4: Fix Auto-Complete Logic

**Files:**
- Modify: `index.html` — `submitNoteModal`, `submitQuickCheckModal`, `nextClosingStep`, `submitClosingDay`

- [ ] **Step 1: Remove `log_note` auto-complete from `submitNoteModal`**

Find `submitNoteModal` (around line 1253). Remove this line:
```javascript
autoComplete('log_note', 100);
```

- [ ] **Step 2: Remove facilities auto-completes from `submitQuickCheckModal`**

Find `submitQuickCheckModal` (around line 1271). Remove this entire block:
```javascript
['corridors', 'corridor_crowding', 'fridges', 'fridge_empty', 'cleaning', 'cleaning_weak'].forEach(id => autoComplete(id, 100));
```

- [ ] **Step 3: Remove `update` auto-complete from closing step 3**

In `nextClosingStep` (around line 1359), find the `closingStep === 3` branch. Remove:
```javascript
autoComplete('update', 100);
```

- [ ] **Step 4: Move `close_note` auto-complete to `submitClosingDay`**

In `nextClosingStep`, find the `closingStep === 2` branch and remove:
```javascript
if (closingDraft.hasOpenNotes === 'no') autoComplete('close_note', 100);
```

In `submitClosingDay` (around line 1378), add `autoComplete('close_note', 100)` before `submitDay()`:
```javascript
function submitClosingDay() {
  const currentDayIdx = getTodayIndex();
  autoComplete('close_note', 100);
  viewDayIdx = currentDayIdx;
  state.todayPhase = 'done';
  state.todayPhaseDayIdx = currentDayIdx;
  saveState();
  closeFlowModal('closingModal');
  submitDay();
}
```

- [ ] **Step 5: Verify auto-complete behaviour**

Open browser:
1. Add a note → `log_note` task should NOT auto-complete
2. Submit morning modal with "نعم" → `checkin` and `briefing` should auto-complete ✅
3. Complete Close Day flow → `close_note` should auto-complete ✅, `update` should NOT auto-complete
4. Quick check with round → zone round added, no task auto-completes

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "fix: remove phantom auto-completes — only checkin/briefing/close_note remain"
```

---

## Task 5: Home Screen Task Panel

**Files:**
- Modify: `index.html` — CSS section, home screen HTML, new `renderHomeTasks` function, `updateHomeUI`

- [ ] **Step 1: Add home tasks CSS**

In the CSS block, after the `.today-action-btn` styles (around line 85), add:

```css
/* HOME TASKS PANEL */
.home-tasks-panel { background: var(--bg2); border: 1px solid var(--border); border-radius: var(--radius); overflow: hidden; }
.home-tasks-header { display: flex; align-items: center; gap: 10px; padding: 10px 14px; cursor: pointer; user-select: none; transition: background 0.2s; }
.home-tasks-header:hover { background: rgba(200,151,42,0.06); }
.home-tasks-title { font-family: 'Cinzel Decorative', cursive; font-size: 13px; color: var(--gold-light); flex: 1; }
.home-tasks-chip { font-size: 12px; padding: 2px 10px; border-radius: 10px; border: 1px solid var(--gold-dim); color: var(--gold); background: rgba(200,151,42,0.1); }
.home-tasks-chip.all-done { border-color: var(--teal); color: var(--teal-light); background: rgba(42,122,106,0.12); }
.home-tasks-chevron { font-size: 12px; color: var(--text-dim); transition: transform 0.2s; }
.home-tasks-chevron.open { transform: rotate(180deg); }
.home-tasks-body { border-top: 1px solid var(--border); padding: 10px 14px; display: flex; flex-direction: column; gap: 12px; }
.home-pillar-group { display: flex; flex-direction: column; gap: 6px; }
.home-pillar-label { font-size: 12px; color: var(--text-dim); padding-bottom: 2px; border-bottom: 1px solid var(--border); }
.home-task-row { display: flex; align-items: center; gap: 8px; padding: 7px 0; cursor: pointer; }
.home-task-check { width: 18px; height: 18px; border: 2px solid var(--border); border-radius: 50%; flex-shrink: 0; display: flex; align-items: center; justify-content: center; font-size: 11px; transition: all 0.25s; }
.home-task-row.done .home-task-check { background: var(--teal); border-color: var(--teal); color: white; }
.home-task-row.na .home-task-check { background: transparent; border-color: var(--border); color: var(--text-dim); }
.home-task-row.na .home-task-text { color: var(--text-dim); text-decoration: line-through; opacity: 0.6; }
.home-task-text { flex: 1; font-size: 14px; line-height: 1.4; }
.home-task-na-btn { font-size: 11px; color: var(--text-dim); background: none; border: 1px solid transparent; border-radius: 4px; padding: 2px 6px; cursor: pointer; flex-shrink: 0; }
.home-task-na-btn:hover { border-color: var(--border); }
```

- [ ] **Step 2: Add home tasks panel HTML**

In the home screen HTML, find the `today-dashboard` div (which contains the `today-stats-row` and `today-action-row`). Add the tasks panel between the stats row and the action row:

```html
<!-- HOME TASKS PANEL -->
<div class="home-tasks-panel" id="homeTasksPanel">
  <div class="home-tasks-header" onclick="toggleHomeTasksPanel()">
    <span class="home-tasks-title">مهام اليوم</span>
    <span class="home-tasks-chip" id="homeTasksChip">— / —</span>
    <span class="home-tasks-chevron" id="homeTasksChevron">▼</span>
  </div>
  <div class="home-tasks-body" id="homeTasksBody" style="display:none"></div>
</div>
```

- [ ] **Step 3: Add `toggleHomeTasksPanel` and `renderHomeTasks` functions**

Add these functions after `renderHomeRounds` (after line 1195):

```javascript
let _homeTasksOpen = false;

function toggleHomeTasksPanel() {
  _homeTasksOpen = !_homeTasksOpen;
  const body = $('homeTasksBody');
  const chevron = $('homeTasksChevron');
  if (body) body.style.display = _homeTasksOpen ? 'flex' : 'none';
  if (chevron) chevron.classList.toggle('open', _homeTasksOpen);
  if (_homeTasksOpen) renderHomeTasks(viewDayIdx);
}

function renderHomeTasks(dayIdx) {
  const body = $('homeTasksBody');
  if (!body) return;
  body.innerHTML = '';

  const submitted = isDaySubmitted(dayIdx);
  const checks = submitted
    ? (state.days[dayIdx]?.tasks || {})
    : ((state.todayChecks || {})[dayIdx] || {});
  const naMap = getNAMap(dayIdx);
  const pillars = getPillars(dayIdx);
  const isToday = dayIdx === getTodayIndex();

  let doneCount = 0, totalCount = 0;

  pillars.forEach(pillar => {
    const baseTasks = pillar.tasks.filter(t => t._scope === 'base');
    if (!baseTasks.length) return;

    const group = el('div', 'home-pillar-group');
    const label = el('div', 'home-pillar-label', pillar.icon + ' ' + pillar.name);
    group.appendChild(label);

    baseTasks.forEach(task => {
      const isNA = !submitted && !!naMap[task.id];
      const isDone = getCheckValue(checks, task.id) === 100;
      if (!isNA) totalCount++;
      if (!isNA && isDone) doneCount++;

      const row = el('div', 'home-task-row' + (isDone ? ' done' : '') + (isNA ? ' na' : ''));

      const chk = el('div', 'home-task-check', isDone ? '✓' : (isNA ? '—' : ''));
      if (!submitted && !isNA && isToday) {
        chk.onclick = () => { toggleBinary(task.id, task.xp, task._scope); renderHomeTasks(dayIdx); updateHomeUI(); };
      }

      const txt = el('span', 'home-task-text', task.text);
      if (!submitted && !isNA && isToday) {
        txt.onclick = () => { toggleBinary(task.id, task.xp, task._scope); renderHomeTasks(dayIdx); updateHomeUI(); };
      }

      row.append(chk, txt);

      if (!submitted && isToday) {
        if (isNA) {
          const undo = el('button', 'home-task-na-btn', '↩');
          undo.title = 'استعد';
          undo.onclick = e => { e.stopPropagation(); clearTaskNA(task.id); renderHomeTasks(dayIdx); updateHomeUI(); };
          row.appendChild(undo);
        } else {
          const naBtn = el('button', 'home-task-na-btn', '—');
          naBtn.title = 'لا ينطبق';
          naBtn.onclick = e => { e.stopPropagation(); openNAPrompt(task.id); };
          row.appendChild(naBtn);
        }
      }

      group.appendChild(row);
    });

    body.appendChild(group);
  });

  // Update chip
  const chip = $('homeTasksChip');
  if (chip) {
    const allDone = totalCount > 0 && doneCount === totalCount;
    chip.textContent = doneCount + ' / ' + totalCount;
    chip.className = 'home-tasks-chip' + (allDone ? ' all-done' : '');
  }
}
```

- [ ] **Step 4: Update `updateHomeUI` to refresh the chip and panel**

At the end of `updateHomeUI` (before the closing `}`), add:

```javascript
  // Refresh home tasks chip and panel
  renderHomeTasks(viewDayIdx);
  if (!_homeTasksOpen) {
    // still need chip counts when collapsed — renderHomeTasks handles that
  }
```

But `renderHomeTasks` writes to `homeTasksBody`. If collapsed, the body is `display:none` but the chip update still runs. That's correct. However if `_homeTasksOpen` is false, writing to `homeTasksBody.innerHTML` is wasted work. Optimise by separating the chip update:

Instead, update `renderHomeTasks` to always update the chip regardless, but skip body render if collapsed:

```javascript
function renderHomeTasks(dayIdx) {
  const body = $('homeTasksBody');
  const chip = $('homeTasksChip');
  if (!body && !chip) return;

  const submitted = isDaySubmitted(dayIdx);
  const checks = submitted
    ? (state.days[dayIdx]?.tasks || {})
    : ((state.todayChecks || {})[dayIdx] || {});
  const naMap = getNAMap(dayIdx);
  const pillars = getPillars(dayIdx);
  const isToday = dayIdx === getTodayIndex();

  let doneCount = 0, totalCount = 0;
  const baseTasks = pillars.flatMap(p => p.tasks.filter(t => t._scope === 'base'));
  baseTasks.forEach(task => {
    const isNA = !submitted && !!naMap[task.id];
    if (!isNA) totalCount++;
    if (!isNA && getCheckValue(checks, task.id) === 100) doneCount++;
  });

  if (chip) {
    const allDone = totalCount > 0 && doneCount === totalCount;
    chip.textContent = doneCount + ' / ' + totalCount;
    chip.className = 'home-tasks-chip' + (allDone ? ' all-done' : '');
  }

  if (!body || !_homeTasksOpen) return;
  body.innerHTML = '';

  pillars.forEach(pillar => {
    const tasks = pillar.tasks.filter(t => t._scope === 'base');
    if (!tasks.length) return;

    const group = el('div', 'home-pillar-group');
    group.appendChild(el('div', 'home-pillar-label', pillar.icon + ' ' + pillar.name));

    tasks.forEach(task => {
      const isNA = !submitted && !!naMap[task.id];
      const isDone = getCheckValue(checks, task.id) === 100;

      const row = el('div', 'home-task-row' + (isDone ? ' done' : '') + (isNA ? ' na' : ''));
      const chk = el('div', 'home-task-check', isDone ? '✓' : (isNA ? '—' : ''));
      const txt = el('span', 'home-task-text', task.text);

      if (!submitted && !isNA && isToday) {
        const toggle = () => { toggleBinary(task.id, task.xp, task._scope); renderHomeTasks(dayIdx); updateHomeUI(); };
        chk.onclick = toggle;
        txt.onclick = toggle;
      }

      row.append(chk, txt);

      if (!submitted && isToday) {
        if (isNA) {
          const undo = el('button', 'home-task-na-btn', '↩');
          undo.title = 'استعد';
          undo.onclick = e => { e.stopPropagation(); clearTaskNA(task.id); renderHomeTasks(dayIdx); updateHomeUI(); };
          row.appendChild(undo);
        } else {
          const naBtn = el('button', 'home-task-na-btn', '—');
          naBtn.title = 'لا ينطبق';
          naBtn.onclick = e => { e.stopPropagation(); openNAPrompt(task.id); };
          row.appendChild(naBtn);
        }
      }

      group.appendChild(row);
    });

    body.appendChild(group);
  });
}
```

Replace the earlier version of `renderHomeTasks` in Step 3 with this final version.

In `updateHomeUI`, add one line at the end (before closing `}`):
```javascript
  renderHomeTasks(viewDayIdx);
```

- [ ] **Step 5: Ensure N/A from home screen refreshes the panel**

In `submitNA` (added in Task 3), `updateAllUI()` is already called after `setTaskNA` which will call `updateHomeUI` → `renderHomeTasks`. No extra change needed.

- [ ] **Step 6: Verify home panel**

Open browser → home screen:
1. Confirm "مهام اليوم" row is visible with "X / Y" chip
2. Tap header → panel expands showing 3 pillar groups with tasks
3. Tap a task circle → it toggles to done (teal) and chip updates
4. Tap "—" on a task → N/A modal appears → submit reason → task shows struck-through
5. Chip count decreases (N/A excluded)
6. Navigate to a past day → panel shows tasks read-only, no NA/toggle controls
7. Close and re-open the panel — expanded state persists within session

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add collapsible home screen task panel with binary toggle and N/A support"
```

---

## Self-Review

**Spec coverage check:**
- ✅ Remove `corridors`, `fridges`, `cleaning` — Task 1
- ✅ Convert all partial tasks to binary — Task 1
- ✅ Redesign `shift_ready`, `update`, `checkin` text — Task 1
- ✅ N/A mechanic with reason, excluded from XP both sides — Task 2
- ✅ Perfect day bonus only when all applicable tasks done — Task 2 (`calcDayXP` returns `perfect` based on applicable tasks only)
- ✅ Streak based on applicable tasks — streak uses `coreComplete` which is set in `submitDay` using the updated `calcDayXP`
- ✅ N/A visible in day log — NA is snapshotted into `state.days[idx].naMap`, visible via memory/log views
- ✅ Un-dismiss before submission — Task 3 ↩ button
- ✅ Auto-complete only for binary tasks with real evidence — Task 4
- ✅ Add Note no longer auto-completes — Task 4
- ✅ Close Day auto-completes `close_note` — Task 4
- ✅ Home panel collapsed by default — Task 5
- ✅ Expands to compact pillar groups — Task 5
- ✅ Binary toggle live save from home — Task 5
- ✅ N/A from home panel — Task 5
- ✅ Chip count excludes NA — Task 5

**One gap to watch:** `submitDay` sets `coreComplete` based on core tasks. After Task 2, verify that `submitDay` also passes the `naMap` when checking core task completion. Search for `coreComplete` in `submitDay` and confirm it uses the updated `calcDayXP` result (which already excludes NA). If `coreComplete` is set manually by iterating tasks directly (not via `calcDayXP`), add `if (naMap[t.id]) return` to that loop as well.
