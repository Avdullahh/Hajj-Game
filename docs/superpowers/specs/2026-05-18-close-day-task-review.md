# Close Day — Incomplete Task Review
**Date:** 2026-05-18
**Status:** Approved for implementation

---

## 1. Problem Statement

The Close Day flow currently allows a day to be closed with tasks that were never acted on. There is no gate, no prompt, and no awareness. Tasks simply remain at zero XP silently. Additionally, the existing closing prompts give no indication of which tasks they auto-complete, creating a disconnect between user actions and the gamification layer.

---

## 2. Design

### 2a. Close Day Gate

When the user triggers Close Day, the app collects all **base tasks** (`_scope === 'base'`) for today that are:
- Not done (`getCheckValue(...) !== 100`)
- Not already N/A (`isTaskNA(dayIdx, taskId) === false`)

If zero such tasks exist → skip Phase 1 entirely, jump straight to Phase 2.

If one or more exist → run Phase 1 before Phase 2.

---

### 2b. Phase 1 — Incomplete Task Review

Each incomplete task gets its own step inside the existing closing modal (`#closingModal`).

**Step layout:**
- Progress header: "مهام اليوم · X من Y" (current task index / total incomplete)
- Task text displayed as the main question
- Three action buttons:
  - **اكتملت** — marks task done (100), awards full XP
  - **لا ينطبق** — reveals inline reason input (required, max 80 chars) + confirm button; on confirm, calls `setTaskNA`
  - **لم تكتمل** — acknowledges zero XP, no state change needed (task stays at 0)
- No back navigation within Phase 1 — each decision is final
- "لا ينطبق" inline reason input appears below the three buttons; confirm advances to next task

**State changes per action:**
| Button | State change |
|--------|-------------|
| اكتملت | `autoComplete(taskId, 100)` |
| لا ينطبق | `setTaskNA(taskId, reason)` |
| لم تكتمل | no state change (task stays at 0) |

---

### 2c. Phase 2 — Trimmed Closing Steps

After Phase 1 (or immediately if no incomplete tasks), the existing closing steps run in this order:

**Step: Readiness slider**
- Label: "كيف كانت جاهزية الموقع؟"
- Small sublabel beneath title: "سيُضاف كجولة إغلاق اليوم"
- Behaviour unchanged: adds a zone round on advance

**Step: Highlight textarea**
- Label: "أهم شيء اليوم"
- No auto-complete label (nothing auto-completes here)
- Behaviour unchanged

**Step: Summary**
- Shows final XP preview
- "أغلق اليوم" button submits

**Removed step:** "فيه ملاحظات مفتوحة؟" — deleted entirely.

---

### 2d. `close_note` Auto-Complete Removal

`autoComplete('close_note', 100)` is removed from `submitClosingDay`. The `close_note` task now appears in Phase 1 if incomplete, handled like every other task. No special wiring.

---

## 3. Rendering Notes

- Phase 1 reuses the existing `#closingModal` / `#closingStepContent` / `#closingActions` layout
- The progress header "مهام اليوم · X من Y" renders inside `#closingStepContent`
- Inline reason input for "لا ينطبق" is appended to the same step content div after the button row, shown/hidden by JS
- Phase 1 steps are not numbered in the existing `closingStep` counter — they run before it
- After Phase 1 completes, `closingStep` resets to 1 and Phase 2 begins normally

---

## 4. Out of Scope

- Custom tasks (day-custom and global-custom) are not shown in Phase 1 — only base tasks
- No back navigation within Phase 1
- No changes to the tasks page or home panel
- No changes to morning modal or note modal
