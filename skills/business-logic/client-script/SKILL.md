---
name: client-script
description: Dispatched by the business-logic router — designs and scaffolds JavaScript
  client scripts for MDA forms using the Xrm API. Reads the form event map from ui-design
  as primary input. Not directly invokable; use business-logic to access this sub-skill.
---

# Client Script Design

Designs and scaffolds JavaScript that runs in the browser context on Model-Driven App forms using the Xrm API (`formContext`, `Xrm.WebApi`, `Xrm.Navigation`). Reads the form event map from ui-design as its primary input. Testing uses a hybrid strategy: Jest with Xrm mock objects for unit-testable logic, and a manual checklist for form-bound behavior.

## State Machine

```
INIT → DISCOVERY → DESIGN → IMPLEMENT → TEST → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Load form event map, logic map client script requirements, identify scope | No |
| DISCOVERY | Per event: entity, form, event type, trigger condition, behavior | No |
| DESIGN | Script file structure, function signatures, Xrm API usage plan | No |
| IMPLEMENT | Implementation guidance per handler with Xrm API patterns | No |
| TEST | Jest stubs with Xrm mocks + manual test checklist | No |
| REVIEW | Spec compliance + quality + security review | No |
| COMPLETE | Write script files and `docs/bl-script-[entity].md`, update inventory | No |

---

## INIT

**Action:** Read `docs/ui-form-event-map.md` if available. Read `06-logic-map.md` for any client script requirements not in the form event map.

**Presentation:**

> "Starting client-script design. Loading event requirements:
>
> [If form event map available:]
> From your form event map ([N] events across [M] entities):
> | Entity | Form | Event | Behavior needed |
> |---|---|---|---|
> | [entity] | Main | OnChange: [field] | [description] |
>
> [If additional requirements in logic map:]
> Additional from logic map: [list]
>
> Does this capture all client-side logic?"

**Gate:** Developer confirms complete event list.

---

## DISCOVERY

**Presentation (per event):**

> "**Event: [Entity] / [Form] / [Event type] / [Field]**
>
> 1. **What should happen?** [plain language behavior]
> 2. **What Xrm data does it need?** (field values, related records via WebApi)
> 3. **Does it write back to the form?** (set value, visibility, required level, notification)
> 4. **Does it make server calls?** (Xrm.WebApi)
> 5. **Any async behavior?** (form behavior while waiting)"

**Gate:** Developer confirms behavior per event.

---

## DESIGN

**Presentation (per entity):**

> "**Script file: [prefix]_[EntityName]Form.js**
>
> ```javascript
> "use strict";
> var [Prefix] = [Prefix] || {};
> [Prefix].[EntityName] = (function() {
>
>   function onLoad(executionContext) { /* OnLoad handler */ }
>   function onSave(executionContext) { /* OnSave handler */ }
>   function [fieldName]OnChange(executionContext) { /* OnChange handler */ }
>
>   function _getFormContext(ec) { return ec.getFormContext(); }
>
>   return { onLoad, onSave, [fieldName]OnChange };
> })();
> ```
>
> **Registration spec:**
> | Function | Event | Pass execution context | Field |
> |---|---|---|---|
> | [Prefix].[Entity].onLoad | OnLoad | Yes | N/A |
> | [Prefix].[Entity].[field]OnChange | OnChange | Yes | [logicalname] |"

**Gate:** Developer confirms file structure and signatures.

---

## IMPLEMENT

**Presentation (per handler):**

> "**[function] — Implementation**
>
> ```javascript
> function [name](executionContext) {
>   var formContext = executionContext.getFormContext();
>   // Read: formContext.getAttribute("[name]").getValue()
>   // Write: formContext.getAttribute("[name]").setValue(value)
>   // Visibility: formContext.getControl("[name]").setVisible(bool)
>   // Required: formContext.getAttribute("[name]").setRequiredLevel("required")
>   // Notification: formContext.ui.setFormNotification("msg", "INFO", "id")
>   // WebApi: Xrm.WebApi.retrieveRecord("[entity]", id, "?$select=fields")
> }
> ```
>
> **Logic:** [step-by-step for this handler]"

---

## TEST

**Presentation:**

> "**Jest test stubs: [prefix]_[EntityName]Form.test.js**
>
> ```javascript
> global.Xrm = {
>   WebApi: { retrieveRecord: jest.fn(), createRecord: jest.fn(), updateRecord: jest.fn() }
> };
>
> function createFormContextMock(fields = {}) {
>   const attrs = {}, ctrls = {};
>   Object.keys(fields).forEach(k => {
>     attrs[k] = { getValue: jest.fn(() => fields[k]), setValue: jest.fn(), setRequiredLevel: jest.fn() };
>     ctrls[k] = { setVisible: jest.fn() };
>   });
>   return {
>     getAttribute: n => attrs[n], getControl: n => ctrls[n],
>     ui: { setFormNotification: jest.fn(), clearFormNotification: jest.fn() }
>   };
> }
>
> describe('[Entity] form scripts', () => {
>   test('[scenario]', () => {
>     const ec = { getFormContext: () => createFormContextMock({ [field]: [value] }) };
>     [Prefix].[Entity].[function](ec);
>     // Assert
>   });
> });
> ```
>
> **Manual test checklist (live MDA environment):**
> | Test | Steps | Expected | Pass/Fail |
> |---|---|---|---|
> | Event fires | [action] | [expected behavior] | |
> | Event does not fire | [excluded setup] | No change | |
> | WebApi success | [trigger + verify] | [field populated] | |
> | WebApi failure | [cause failure] | Error notification | |"

---

## REVIEW

**Spec compliance:** Every form event map entry has a handler, every handler registered correctly, all WebApi calls have error handlers.

**Quality:** Execution context always passed (no global Xrm for form context), no synchronous XMLHttpRequest, no hardcoded GUIDs, no console.log in production.

**Security:** No eval(), no user-input in WebApi queries without sanitization, no credentials in script variables.

**Gate:** Developer confirms review results.

---

## COMPLETE

Write script files to web resources folder. Write `docs/bl-script-[entity].md`. Update `docs/business-logic-inventory.md`. Return to router.
