---
name: power-automate
description: Dispatched by the business-logic router — designs Power Automate flows
  for internal business process automation triggered by Dataverse events or schedules.
  Not directly invokable; use business-logic to access this sub-skill.
---

# Power Automate Flow Design

Designs Power Automate flows that automate internal business processes triggered by Dataverse events or schedules. Produces a design document and flow diagram. Does not produce runnable flow definitions — flows are configured in the Power Automate designer. Scope: internal automation only. External integration flows belong in the integration skill.

## State Machine

```
INIT → DISCOVERY → DESIGN → DIAGRAM → TEST → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Load logic map flow requirements, identify trigger types, boundary check | No |
| DISCOVERY | Per flow: trigger, condition, action sequence, error path, concurrency | No |
| DESIGN | Structured flow specification: trigger config, steps, branches, error handling | No |
| DIAGRAM | Generate Mermaid flowchart from confirmed design | No |
| TEST | Define test scenarios and manual test checklist | No |
| REVIEW | Spec compliance + quality review | No |
| COMPLETE | Write `docs/bl-flow-[name].md`, update inventory | No |

---

## INIT

**Presentation:**

> "Starting power-automate design. From your logic map, I see [N] flow requirements:
>
> | Requirement | Trigger type | Scope |
> |---|---|---|
> | [name] | [Dataverse row change / Scheduled / Manual / HTTP] | [internal / boundary check] |
>
> **Boundary check:** [If any are boundary cases:] These may belong in integration: [list]"

**Gate:** Developer confirms flow scope and boundary.

---

## DISCOVERY

**Presentation (per flow):**

> "**Flow: [Requirement Name]**
>
> 1. **Trigger:** [Dataverse row change / Scheduled / Manual]
> 2. **Trigger filters (Dataverse only):** Filter rows / columns?
> 3. **Conditions:** Branching based on data conditions?
> 4. **Action sequence:** [ordered action list]
> 5. **Error handling:** Retry policy? Configure run after? Admin notification?
> 6. **Concurrency:** Multiple instances simultaneously, or limited?"

**Gate:** Developer confirms all flow parameters.

---

## DESIGN

**Presentation:**

> "**Flow Specification: [Flow Name]**
>
> | Attribute | Value |
> |---|---|
> | Trigger | [trigger type + config] |
> | Environment | [solution name] |
> | Connection references | [list] |
>
> **Step sequence:**
> | Step | Type | Action | On failure |
> |---|---|---|---|
> | 1 | Trigger | [config] | N/A |
> | 2 | Condition | [check] | N/A |
> | 3 | Action | [operation] | Terminate / Retry |
>
> **Connection reference requirements:**
> | Ref name | Connector | Purpose |
> |---|---|---|
> | [ref] | Dataverse | [purpose] |
>
> **Naming convention:** `[Solution prefix] - [Entity] - [Action]`
>
> Does this capture the full flow logic?"

**Gate:** Developer confirms specification.

---

## DIAGRAM

**Action:** Generate a Mermaid flowchart from the confirmed specification showing trigger, decision branches, action steps, and error paths.

**Presentation:**

> "Generating Mermaid flow diagram for [Flow Name]."

Write the Mermaid diagram into the design document.

---

## TEST

**Presentation:**

> "**Test Plan: [Flow Name]**
>
> | Scenario | Setup | Expected outcome | Pass/Fail |
> |---|---|---|---|
> | Happy path | [trigger setup] | [expected result] | |
> | Condition false | [false branch setup] | [expected for false] | |
> | Trigger excluded | [should NOT trigger] | Flow does not run | |
> | Error path | [cause failure] | Error notification sent | |
>
> **Manual test checklist:**
> - [ ] Flow is solution-aware (inside a solution)
> - [ ] Connection references used (not hardcoded connections)
> - [ ] Run history shows successful execution
> - [ ] Error path confirmed
> - [ ] Concurrency limit verified (if configured)"

---

## REVIEW

**Spec compliance:** All trigger parameters specified, connection references named, error handling on every fallible action, flow is solution-aware.

**Quality:** No unbounded loops, scheduled flows have concurrency limits, no secrets in expressions, names follow convention.

**Gate:** Developer confirms review results.

---

## COMPLETE

Write `docs/bl-flow-[name].md`. Update `docs/business-logic-inventory.md`. Return to router.
