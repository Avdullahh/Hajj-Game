# Season Log Redesign
**Date:** 2026-05-18
**Status:** Approved for implementation

---

## 1. Problem Statement

The 📜 السجل tab has two panels that serve no real purpose:

- **سجل الموسم** — shows completed days as XP bars with a click-to-navigate affordance. Functional but bare — gives no insight into how each day actually went.
- **ذاكرة الموسم** — aggregates zone round notes and evidence entries into a timeline. Rarely has content, has no clear purpose, and duplicates what the tasks page already shows.

---

## 2. Design

### 2a. سجل الموسم — Enriched with Day Reflection

Each completed day row keeps its existing structure (day number, XP progress bar, XP value, perfect badge) and gains a **positives/negatives reflection** rendered directly beneath the bar — always visible, no tap required.

**Positives (✅)** — shown when condition is true:
| Condition | Label |
|-----------|-------|
| `day.perfect === true` | يوم مثالي |
| `day.coreComplete === true` | جميع المهام الأساسية مكتملة |
| readiness ≥ 80% | جاهزية عالية · X% |
| `day.evidence.length > 0` | دليل موثق · X إدخال |
| `day.zoneRounds.length > 0` | X جولات مسجلة |

**Negatives (❌)** — shown when condition is true:
| Condition | Label |
|-----------|-------|
| `day.coreComplete === false` | مهام أساسية ناقصة |
| readiness < 60% and rounds exist | جاهزية منخفضة · X% |
| incomplete task count > 0 | X مهام غير مكتملة |
| `day.zoneRounds.length === 0` | لا توجد جولات مسجلة |
| `day.evidence.length === 0` | لا يوجد دليل موثق |

**Incomplete task count** = tasks in `pillarsSnapshot` where `day.tasks[id] !== 100` and `day.naMap[id]` is falsy.

**Readiness** = `calcReadiness(day.zoneRounds)` — same function used elsewhere.

If a day has zero positives and zero negatives (edge case — all tasks NA, no rounds, no evidence), the reflection is omitted entirely.

If both lists are empty but the day is done, show nothing (no empty state needed).

---

### 2b. ذاكرة الموسم — Removed

The panel and its `#memoryContainer` div are removed from the HTML. The `buildMemoryView()` function and its call in `updateLogUI()` are removed. No data is deleted — `state.days` entries retain their `evidence` and `zoneRounds` arrays unchanged.

---

## 3. Rendering Notes

- Reflection renders inside the existing `.log-item` div, below the `.log-bar-wrap`
- Positives list uses a `<ul class="log-reflection log-positives">` with `<li>` per item
- Negatives list uses a `<ul class="log-reflection log-negatives">` with `<li>` per item
- Each list only renders if it has at least one item
- CSS: compact font (12–13px), muted color for list container, green tint for positives, red tint for negatives
- The click handler on `.log-item` (`onclick="jumpToDay(...)"`) is unchanged

---

## 4. Out of Scope

- No changes to `state.days` data structure
- No changes to how days are submitted or what gets stored
- No search or filtering on the log
- No changes to the tasks page or home screen
- The settings panel (⚙️ إعدادات الموسم) is unchanged
