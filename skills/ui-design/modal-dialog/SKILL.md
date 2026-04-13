---
name: modal-dialog
description: Dispatched by the ui-design router — designs a lightweight modal dialog
  triggered from a form, view, or command button. Not directly invokable; use ui-design
  to access this sub-skill.
---

# Modal Dialog Design

Designs a lightweight dialog — a focused interaction triggered from a form, view, or command button. Simpler workflow than custom-page. The right choice when you need user input before an action executes, confirmation with custom options, or a focused interaction without navigating away.

## State Machine

```
INIT → DIALOG_DESIGN → WIREFRAME → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Confirm trigger context, dialog purpose | No |
| DIALOG_DESIGN | Content, fields, actions, return value | No |
| WIREFRAME | Canva mockup or Mermaid dialog layout | No |
| REVIEW | Dispatch ui-reviewer in sub-skill mode | No |
| COMPLETE | Write `docs/ui-dialog-[name].md` | No |

---

## INIT

**Presentation:**

> "Starting modal-dialog design. A dialog is the right choice when:
> - You need a focused interaction that shouldn't navigate away from the current context
> - You need user input before an action executes (e.g., 'Enter rejection reason before declining')
> - You need to show confirmation with custom options
>
> What triggers this dialog? (Command button / Form event / Business process flow step)
> What entity context is available when opened?"

**Gate:** Developer confirms trigger and context.

---

## DIALOG_DESIGN

**Presentation:**

> "**Dialog Design: [Dialog Name]**
>
> **Purpose:** [one sentence]
>
> **Content:**
> | Element | Type | Required? | Notes |
> |---|---|---|---|
> | [field/label/image] | [input/display] | Yes/No | [notes] |
>
> **Action buttons:**
> | Button | Label | Action | Returns |
> |---|---|---|---|
> | Primary | [e.g., Confirm] | [what it does] | [value returned to caller] |
> | Secondary | [e.g., Cancel] | Dismiss | null |
>
> **Return value to calling context:** [what the caller receives]
>
> Does this match the interaction you need?"

**Gate:** Developer confirms dialog design.

---

## WIREFRAME

Generate Canva mockup if available (dialog frame, content, action buttons), otherwise Mermaid diagram showing dialog structure.

---

## REVIEW

**Spec compliance:** All required fields have validation, return value defined and matches caller expectations, cancel/dismiss always available.

**Quality:** Dialog title clearly states purpose, no more than 3-4 input fields (suggest custom-page if more), no destructive actions without confirmation language.

**Gate:** Developer confirms review results.

---

## COMPLETE

Write `docs/ui-dialog-[name].md`. Return to router.
