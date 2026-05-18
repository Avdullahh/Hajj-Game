# Season Log Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove the ذاكرة الموسم panel and enrich each سجل الموسم day row with an algorithmically-generated positives/negatives reflection.

**Architecture:** Single-file vanilla JS/HTML/CSS app (`index.html`). No build step, no test runner. All changes are in that one file. Task 1 removes dead code; Task 2 adds the reflection logic and updates the log renderer.

**Tech Stack:** Vanilla JS, HTML, CSS. State in `localStorage` under key `hajj_rpg_v3`. No external dependencies.

---

### Task 1: Remove ذاكرة الموسم

**Files:**
- Modify: `C:\Users\avhaz\OneDrive\Desktop\VS\Mama\index.html`

- [ ] **Step 1: Remove the memory panel HTML**

Find and delete this block (around line 690–693):
```html
    <div class="panel">
      <div class="section-title">📝 ذاكرة الموسم</div>
      <div id="memoryContainer"><div class="empty">ما فيه ملاحظات</div></div>
    </div>
```

- [ ] **Step 2: Remove the memory CSS block**

Find and delete this CSS block (around lines 386–398):
```css
  /* MEMORY */
  .memory-day-group { margin-bottom: 16px; }
  .memory-day-header { font-size: 13px; color: var(--gold-dim); margin-bottom: 8px; padding-bottom: 6px; border-bottom: 1px solid var(--border); display: flex; gap: 8px; align-items: center; cursor: pointer; }
  .memory-day-header:hover { color: var(--gold); }
  .memory-item { background: var(--bg2); border: 1px solid var(--border); border-radius: var(--radius); padding: 10px 12px; margin-bottom: 6px; cursor: pointer; transition: border-color 0.2s; }
  .memory-item:hover { border-color: var(--gold-dim); }
  .memory-item-top { display: flex; align-items: center; gap: 8px; margin-bottom: 4px; }
  .memory-source-badge { font-size: 11px; border-radius: 10px; padding: 1px 8px; flex-shrink: 0; border: 1px solid; }
  .memory-source-round { background: rgba(200,151,42,0.08); border-color: var(--gold-dim); color: var(--gold-dim); }
  .memory-source-evidence { background: rgba(42,122,106,0.1); border-color: var(--teal); color: var(--teal-light); }
  .memory-source-task { background: rgba(200,151,42,0.06); border-color: var(--border); color: var(--text-dim); }
  .memory-item-label { font-size: 13px; color: var(--text-dim); flex: 1; }
  .memory-item-note { font-size: 14px; color: var(--text); line-height: 1.6; white-space: pre-wrap; overflow-wrap: anywhere; }
```

- [ ] **Step 3: Remove memory touch/active CSS rules**

Find and remove `.memory-item:active` from this rule (around line 418):
```css
  .task-item:active, .task-partial:active, .log-item:active, .memory-item:active { border-color: var(--gold-dim); }
```
Replace with:
```css
  .task-item:active, .task-partial:active, .log-item:active { border-color: var(--gold-dim); }
```

Find and delete this line entirely (around line 434):
```css
  .memory-day-header:active { color: var(--gold); }
```

Find and remove `.memory-item` from this rule (around line 436):
```css
  .task-check, .task-copy, .log-item, .memory-item { min-height: 44px; }
```
Replace with:
```css
  .task-check, .task-copy, .log-item { min-height: 44px; }
```

Find and remove `.memory-item-top` and `.memory-item-label` from this rule (around line 439–440):
```css
  .day-nav, .task-partial-top, .pct-input-row, .zone-round-row, .evidence-card-top, .memory-item-top { flex-wrap: wrap; }
  .task-copy, .log-bar-wrap, .evidence-card-text, .memory-item-label { min-width: 0; }
```
Replace with:
```css
  .day-nav, .task-partial-top, .pct-input-row, .zone-round-row, .evidence-card-top { flex-wrap: wrap; }
  .task-copy, .log-bar-wrap, .evidence-card-text { min-width: 0; }
```

- [ ] **Step 4: Remove `buildMemoryView()` function**

Find and delete the entire function (around lines 2319–2375):
```js
function buildMemoryView() {
  const mc = $('memoryContainer');
  ...
  else setEmpty(mc, 'ما فيه ملاحظات مسجلة');
}
```

- [ ] **Step 5: Remove the `buildMemoryView()` call from `updateLogUI`**

Find this line in `updateLogUI` (around line 2402):
```js
  buildMemoryView();
```
Delete it.

- [ ] **Step 6: Verify in browser**

Open `index.html` in browser. Go to 📜 السجل tab. Confirm:
- ذاكرة الموسم panel is gone
- سجل الموسم still shows (or shows empty state if no days submitted)
- ⚙️ إعدادات الموسم panel still shows
- No JS errors in console

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Remove ذاكرة الموسم panel and dead memory code"
```

---

### Task 2: Enrich سجل الموسم with Day Reflection

**Files:**
- Modify: `C:\Users\avhaz\OneDrive\Desktop\VS\Mama\index.html`

- [ ] **Step 1: Add CSS for reflection and restructure log-item**

Find the `/* LOG */` CSS block (around line 233) and replace it entirely with:

```css
  /* LOG */
  .log-item { background: var(--bg2); border: 1px solid var(--border); border-radius: var(--radius); padding: 14px 16px; margin-bottom: 10px; display: flex; flex-direction: column; gap: 10px; cursor: pointer; transition: border-color 0.2s; }
  .log-item:hover { border-color: var(--gold-dim); }
  .log-item-top { display: flex; align-items: center; gap: 12px; }
  .log-day { font-family: 'Cinzel Decorative', cursive; font-size: 13px; color: var(--gold-dim); width: 32px; flex-shrink: 0; text-align: center; }
  .log-bar-wrap { flex: 1; }
  .log-label { font-size: 14px; color: var(--text-dim); margin-bottom: 4px; }
  .log-bar-track { background: var(--bg); border-radius: 3px; height: 6px; overflow: hidden; }
  .log-bar-fill { height: 100%; border-radius: 3px; transition: width 0.5s; }
  .log-xp { font-size: 14px; color: var(--gold); text-align: end; flex-shrink: 0; }
  .log-reflection { display: flex; flex-direction: column; gap: 4px; padding-top: 2px; border-top: 1px solid var(--border); }
  .log-reflection ul { margin: 0; padding: 0; list-style: none; display: flex; flex-direction: column; gap: 3px; }
  .log-reflection li { font-size: 12px; line-height: 1.5; padding-right: 4px; }
  .log-positives li { color: var(--teal-light); }
  .log-positives li::before { content: '✅ '; }
  .log-negatives li { color: #c97a7a; }
  .log-negatives li::before { content: '❌ '; }
```

- [ ] **Step 2: Add `buildDayReflection(day)` function**

Add this function in the `// ── LOG ──` section, just before `updateLogUI`:

```js
function buildDayReflection(day) {
  const positives = [];
  const negatives = [];

  const rounds = day.zoneRounds || [];
  const evidence = day.evidence || [];
  const readiness = calcReadiness(rounds);

  // Positives
  if (day.perfect) positives.push('يوم مثالي');
  else if (day.coreComplete) positives.push('جميع المهام الأساسية مكتملة');
  if (rounds.length > 0 && readiness >= 80) positives.push('جاهزية عالية · ' + readiness + '%');
  if (evidence.length > 0) positives.push('دليل موثق · ' + evidence.length + ' إدخال');
  if (rounds.length > 0) positives.push(rounds.length + ' جولات مسجلة');

  // Negatives
  if (day.coreComplete === false) negatives.push('مهام أساسية ناقصة');
  if (rounds.length > 0 && readiness < 60) negatives.push('جاهزية منخفضة · ' + readiness + '%');

  const allTasks = (day.pillarsSnapshot || []).flatMap(p => p.tasks || []);
  const naMap = day.naMap || {};
  const incomplete = allTasks.filter(t => (day.tasks?.[t.id] || 0) !== 100 && !naMap[t.id]).length;
  if (incomplete > 0) negatives.push(incomplete + ' مهام غير مكتملة');

  if (rounds.length === 0) negatives.push('لا توجد جولات مسجلة');
  if (evidence.length === 0) negatives.push('لا يوجد دليل موثق');

  return { positives, negatives };
}
```

- [ ] **Step 3: Update the `updateLogUI` log-item template**

Find this block inside `updateLogUI` (around lines 2389–2400):
```js
  c.innerHTML = entries.map(([idx,day])=>{
    const pct = Math.min(100,(day.xp/maxXP)*100);
    const col = day.perfect?'var(--gold)':(pct>60?'var(--teal)':'var(--text-dim)');
    const extra = day.extraXP || Math.max(0, (day.activityXP||day.xp) - day.xp);
    return `<div class="log-item" onclick="jumpToDay(${+idx})">
      <div class="log-day">${+idx+1}</div>
      <div class="log-bar-wrap">
        <div class="log-label">${day.perfect?'⭐ يوم مثالي':'يوم عادي'}${(day.evidence?.length) ? ' · 🏅 '+day.evidence.length+' دليل' : ''}</div>
        <div class="log-bar-track"><div class="log-bar-fill" style="width:${pct}%;background:${col}"></div></div>
      </div>
      <div class="log-xp">${day.xp} XP${extra ? ' +' + extra + ' متابعة' : ''}</div>
    </div>`;
  }).join('');
```

Replace with:
```js
  c.innerHTML = entries.map(([idx,day])=>{
    const pct = Math.min(100,(day.xp/maxXP)*100);
    const col = day.perfect?'var(--gold)':(pct>60?'var(--teal)':'var(--text-dim)');
    const extra = day.extraXP || Math.max(0, (day.activityXP||day.xp) - day.xp);
    const { positives, negatives } = buildDayReflection(day);
    const posHtml = positives.length ? `<ul>${positives.map(t=>`<li>${escapeHtml(t)}</li>`).join('')}</ul>` : '';
    const negHtml = negatives.length ? `<ul>${negatives.map(t=>`<li>${escapeHtml(t)}</li>`).join('')}</ul>` : '';
    const reflectionHtml = (positives.length || negatives.length) ? `
      <div class="log-reflection">
        ${posHtml ? `<div class="log-positives">${posHtml}</div>` : ''}
        ${negHtml ? `<div class="log-negatives">${negHtml}</div>` : ''}
      </div>` : '';
    return `<div class="log-item" onclick="jumpToDay(${+idx})">
      <div class="log-item-top">
        <div class="log-day">${+idx+1}</div>
        <div class="log-bar-wrap">
          <div class="log-label">${day.perfect?'⭐ يوم مثالي':'يوم عادي'}</div>
          <div class="log-bar-track"><div class="log-bar-fill" style="width:${pct}%;background:${col}"></div></div>
        </div>
        <div class="log-xp">${day.xp} XP${extra ? ' +' + extra + ' متابعة' : ''}</div>
      </div>${reflectionHtml}
    </div>`;
  }).join('');
```

- [ ] **Step 4: Verify in browser**

Open `index.html`. Go to 📜 السجل tab. If no submitted days exist, submit today's day (or check that `state.days` has entries in localStorage).

Confirm:
- Each day row shows a top bar (day number, XP bar, XP value) — same as before
- Below the bar: ✅ positives in teal, ❌ negatives in red
- Days with all tasks NA and no rounds/evidence show no reflection (no empty section)
- Clicking a day row still navigates to that day on the tasks page
- No JS errors in console

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Enrich season log with per-day positives/negatives reflection"
```
