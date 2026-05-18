# Task & Home Screen Redesign
**Date:** 2026-05-18  
**Status:** Approved for implementation

---

## 1. Problem Statement

Two compounding issues break the daily flow:

1. **Partial/percentage tasks** are meaningless in context — they duplicate the zone rounds log, use vague language, and cannot be completed naturally from the home screen.
2. **Irrelevant-day penalty** — tasks that simply didn't apply on a given day (no issue arose, no relevant event occurred) still count against XP and progression, punishing performance that was never in the user's control.

---

## 2. Task Redesign

### 2a. Tasks Being Removed

These three tasks duplicate what the **📊 جولات جاهزية الزون** panel already tracks. They are removed entirely from `BASE_PILLARS`.

| ID | Current text |
|----|-------------|
| `corridors` | المرور على صورة الموقع العامة |
| `fridges` | ملاحظة الاحتياجات الأساسية أثناء الحركة |
| `cleaning` | تقدير جاهزية الموقع خلال اليوم |

Their companion binary tasks (`corridor_crowding`, `fridge_empty`, `cleaning_weak`) remain and become the primary tasks for their area.

### 2b. Tasks Being Redesigned

#### `shift_ready` — Personal End-of-Day Reflection
- **Old:** "الحفاظ على حضور ذهني وعملي مستقر" (partial, vague)
- **New:** "هل أنهيت يومك بوضوح ذهني وراجعت ما تم وما لم يتم؟" (binary)
- **When:** End-of-day task. Appears last in the attendance pillar.
- **Type:** Binary (done / not done)
- **XP:** 18 (unchanged)

#### `update` — Escalation Quality
- **Old:** "إبقاء الصورة واضحة لمن يحتاجها" (partial, corporate, vague)
- **New:** "إذا واجهت مشكلة — هل حاولت حلها؟ وإن لم تستطع، هل أوصلتها بوضوح مع مقترح لمسؤولك؟" (binary)
- **Type:** Binary (done / not done)
- **XP:** 22 (unchanged)
- **N/A eligible:** Yes — if no issues arose today, this task can be dismissed as not applicable.

### 2c. No More Partial/Percentage Tasks

All remaining `partial: true` tasks are converted to binary. The 0–100% input is removed from the UI entirely. The `partial` field is deprecated — existing saved data is treated as: value > 0 → done (100), value = 0 → not done.

---

## 3. Not Applicable (لا ينطبق اليوم) Mechanic

### Behaviour

Any task can be dismissed as **not applicable today**. When dismissed:
- The user selects "لا ينطبق" from the task's action menu
- A short reason prompt appears (free text, required, max 80 chars)
- The task is stored as `{ status: 'na', reason: '...' }` for that day
- It renders with a muted style and the reason shown beneath

### Progression Impact

**The dismissed task is excluded from both the numerator and denominator of XP calculation for that day.**

Current formula (simplified):
```
dayXP = sum(completed task XP) + perfect_bonus if all done
maxXP = sum(all task XP) + perfect_bonus
```

New formula with N/A:
```
applicableTasks = tasks where status !== 'na'
dayXP = sum(completed task XP among applicableTasks)
maxXP = sum(XP of applicableTasks) + perfect_bonus if all applicable done
progressPct = dayXP / maxXP
```

This means:
- Dismissing a task **does not penalise** the day score
- Dismissing every task except one and completing it → 100% for the day
- The perfect day bonus still triggers if all **applicable** tasks are completed
- Streak is based on `progressPct >= threshold` using only applicable tasks

### Abuse Prevention

- A reason is required (cannot be empty)
- Dismissals are visible in the day log/memory view with their reasons
- No cap on dismissals — trust the user, but make the record visible

### UI

- Task row gains a third state alongside done/undone: **muted with "لا ينطبق" label and reason**
- In the tasks page: task action menu adds "لا ينطبق اليوم" option
- In the home screen panel: long-press or swipe on a task reveals the N/A option
- A dismissed task can be un-dismissed (restored to pending) before day submission

---

## 4. Home Screen Task Panel

### Structure

The home screen gains a collapsible **"مهام اليوم"** section placed below the day nav and stats row.

**Collapsed state:**
- Single row showing: task icon + "مهام اليوم" label + compact progress chip (e.g. "٥ / ٨") + chevron
- Progress chip colour: gold if incomplete, teal if all applicable done

**Expanded state:**
- Pillars rendered as compact labelled groups
- Each task renders as a compact row: checkbox circle + task text + N/A affordance
- All binary — tap circle to toggle done/undone, live save, XP toast fires
- N/A: long-press task row → bottom sheet with reason input
- Submitted days: all controls disabled, read-only

### Auto-Complete Rules

| Home screen action | Auto-completes task? | Reason |
|---|---|---|
| تثبيت بداية اليوم (check-in) | ✅ `checkin` | The action *is* the task |
| إضافة ملاحظة (Add Note) | ❌ No auto-complete | No way to distinguish a closing note from a regular note |
| إغلاق اليوم (Close Day) | ✅ `close_note` | Closing the day is the task |
| Quick Assessment | ❌ No auto-complete | Removed — tasks it completed are now removed or binary |

**Counter tasks (if introduced in future):** Never auto-complete. Close Day flow prompts for any counter still at 0, skips if already set.

### Seamless Dual-View Principle

- Completing a task from the home panel and from the tasks page updates identical state
- No task requires both views to fully complete a day
- A user who never opens the tasks page can finish their full day from home
- A user who never uses the home panel can finish their full day from the tasks page

---

## 5. Revised Pillar Structure (After Redesign)

### الحضور والانضباط
| ID | Text | Type | XP | Core |
|----|------|------|----|------|
| `checkin` | حضور الاجتماع الصباحي و والاستعداد للعمل | Binary | 12 | ✅ |
| `briefing` | فهم أولويات اليوم قبل الانشغال بالتفاصيل | Binary | 10 | ✅ |
| `shift_ready` | هل أنهيت يومك بوضوح ذهني وراجعت ما تم وما لم يتم؟ | Binary | 18 | — |

### الموقع والجولات
| ID | Text | Type | XP | Core |
|----|------|------|----|------|
| `corridor_crowding` | مراجعة وجود ما يستحق الانتباه | Binary | 8 | — |
| `fridge_empty` | التأكد من عدم ترك احتياج واضح بلا متابعة | Binary | 8 | — |
| `cleaning_weak` | مراجعة ما يحتاج متابعة لاحقة | Binary | 8 | — |

### التقارير والمتابعة
| ID | Text | Type | XP | Core |
|----|------|------|----|------|
| `update` | إذا واجهت مشكلة — هل حاولت حلها؟ وإن لم تستطع، هل أوصلتها بوضوح مع مقترح لمسؤولك؟ | Binary | 22 | ✅ |
| `close_note` | مراجعة المفتوح قبل إغلاق اليوم | Binary | 24 | ✅ |

---

## 6. XP Impact Summary

**Old max XP (before):** checkin(12) + briefing(10) + shift_ready(18) + corridors(18) + corridor_crowding(8) + fridges(16) + fridge_empty(8) + cleaning(16) + cleaning_weak(8) + update(22) + close_note(24) = **160 + perfect bonus**

**New max XP (after):** checkin(12) + briefing(10) + shift_ready(18) + corridor_crowding(8) + fridge_empty(8) + cleaning_weak(8) + update(22) + close_note(24) = **110 + perfect bonus**

The total is lower but the tasks are all genuinely completable. Combined with the N/A mechanic, daily scores will better reflect real performance. Historical XP is not retroactively changed.

---

## 7. Out of Scope

- Counter-based partial tasks (deferred — no partial tasks remain after this redesign, counters are a future addition if needed)
- Zone rounds log changes (unchanged)
- Any task added by the user as a custom task (custom tasks inherit the N/A mechanic automatically)
