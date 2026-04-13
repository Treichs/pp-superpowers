---
name: business-rule
description: Dispatched by the business-logic router — designs declarative business
  rules configured on Dataverse tables. Requirements must pass the router's complexity
  gate before reaching this sub-skill. Not directly invokable; use business-logic to access.
---

# Business Rule Design

Designs declarative business rules configured on Dataverse tables. Business rules execute on the client (form) and optionally server-side. They are intentionally limited in capability — the router's complexity gate ensures only in-scope requirements reach this sub-skill.

## State Machine

```
INIT → DISCOVERY → DESIGN → CONFIGURE → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Load in-scope business rule requirements (post complexity gate) | No |
| DISCOVERY | Per rule: entity, scope, trigger event, conditions, actions | No |
| DESIGN | Complete rule specification: condition tree, action set | No |
| CONFIGURE | Step-by-step configuration guidance for Power Apps maker portal | No |
| REVIEW | Spec compliance + quality review | No |
| COMPLETE | Write `docs/bl-rule-[entity].md`, update inventory | No |

---

## INIT

**Action:** Load business rule requirements that passed the router's complexity gate. These have already been validated as within business rule capabilities.

**Presentation:**

> "Starting business-rule design. [N] requirements passed the complexity gate:
>
> | Requirement | Entity | Action type |
> |---|---|---|
> | [name] | [entity] | [Show/Hide / Lock/Unlock / Set value / Set required / Validate] |
>
> Ready to walk through each rule?"

**Gate:** Developer confirms the requirement list.

---

## DISCOVERY

**Presentation (per rule):**

> "**Business Rule: [Requirement Name]**
>
> 1. **Entity:** Which table?
> 2. **Scope:** Form only / All forms / Server-side? (Server-side enforces without form — use with care)
> 3. **Trigger event:** On load / On change (which field?) / On save
> 4. **Condition:** If [field] [operator] [value] — max 3 nesting levels
> 5. **Action(s):** Show/Hide / Lock/Unlock / Set value / Set required / Show error"

**Gate:** Developer confirms each rule's parameters.

---

## DESIGN

**Presentation:**

> "**Rule Specification: [Rule Name]**
>
> Entity: [entity] | Scope: [scope] | Trigger: [trigger]
>
> **Condition tree:**
> ```
> IF [field] [operator] [value]
>   AND [field] [operator] [value]
> THEN
>   [Action 1]
>   [Action 2]
> ELSE
>   [Action 3]
> ```
>
> **Actions:**
> | Action | Field | Value / State |
> |---|---|---|
> | [Show/Hide] | [field logical name] | Visible / Hidden |
> | [Set required] | [field logical name] | Required / Not Required |
>
> Does this specification match the requirement?"

**Gate:** Developer confirms rule specification.

---

## CONFIGURE

**Presentation:**

> "**Configuration steps — Power Apps maker portal:**
>
> 1. Navigate to make.powerapps.com → [solution] → [entity] → Business rules → New
> 2. Set rule name: `[Entity] - [Rule description]`
> 3. Set scope: [scope]
> 4. Add condition: [condition builder steps]
> 5. Add action: [action type] → select field → [value or state]
> 6. Save and activate
>
> **Note:** Business rules must be explicitly activated. Saved-but-inactive rules do not execute."

---

## REVIEW

**Spec compliance:** Rule scope matches enforcement requirement, all condition/action fields exist in physical model, rule is activation-ready.

**Quality:** Server-side scope justified (not default), no more than 3 nesting levels, error messages are user-friendly, rule name follows convention `[Entity] - [Description]`.

**Gate:** Developer confirms review results.

---

## COMPLETE

Write `docs/bl-rule-[entity].md`. Update `docs/business-logic-inventory.md`. Return to router.
