---
name: pcf-control
description: Dispatched by the ui-design router — designs a Power Apps Component Framework
  (PCF) control with manifest spec, React component structure, and scaffold commands.
  Not directly invokable; use ui-design to access this sub-skill.
---

# PCF Control Design

Designs a Power Apps Component Framework (PCF) control: a custom React/TypeScript control that runs inside a Canvas or Model-Driven App. Development-heavy — produces a component specification, manifest properties, React component structure, and scaffold commands.

## State Machine

```
INIT → BRAINSTORM → PLAN → EXECUTE → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Establish component context: placement, binding, purpose | No |
| BRAINSTORM | Component purpose, bound vs. unbound, data type, interaction model | No |
| PLAN | Component spec: manifest properties, React structure, state management | No |
| EXECUTE | Scaffold commands, implementation guidance, component sketch | No |
| REVIEW | Dispatch ui-reviewer in sub-skill mode | No |
| COMPLETE | Write `docs/ui-pcf-[name].md` | No |

---

## INIT

**Presentation:**

> "Starting PCF control design. A PCF control is a custom component that runs inside an existing app. It's the right choice when:
> - You need a custom input or display control not provided natively
> - You want to replace a standard field control with a richer UX
> - You need a visualization or interaction pattern (calendar, map, Gantt) bound to Dataverse data
>
> Where will this control be used? (Canvas App / Model-Driven App / Both)
> What field or dataset will it bind to?"

**Gate:** Developer confirms placement and binding.

---

## BRAINSTORM

**Presentation:**

> "**PCF Control — Brainstorm**
>
> 1. What problem does this solve that a standard control doesn't?
> 2. Is this a **field-level** control (bound to a single column) or a **dataset control** (bound to a view or collection)?
> 3. What does the user do with it? (Input only / Display only / Both)
> 4. Are there reference implementations or similar controls in the PCF Gallery?"

**Gate:** Brainstorm output confirmed by developer.

---

## PLAN

**Presentation:**

> "**PCF Component Spec**
>
> **Manifest properties:**
> | Property name | Type | Usage | Required? | Description |
> |---|---|---|---|---|
> | [property] | [SingleLine.Text / etc.] | bound | Yes | [what it does] |
>
> **React component structure:**
> ```
> [ComponentName]/
>   index.ts          <- PCF entry point, implements StandardControl
>   [ComponentName].tsx <- React root component
>   components/
>     [SubComponent].tsx
>   hooks/
>     use[Hook].ts
>   types.ts
> ```
>
> **State management:** [local React state / useReducer / none — justified by complexity]
>
> Does this match your implementation intent?"

**Gate:** Developer confirms component spec.

---

## EXECUTE

**Presentation:**

> "**Scaffold commands:**
> ```bash
> pac pcf init --namespace [Namespace] --name [ComponentName] --template [field|dataset]
> cd [ComponentName]
> npm install
> npm run build
> ```
>
> **Implementation guidance:**
> - [How to read bound property from context]
> - [How to notify change manager on value update]
> - [How to handle disabled/masked modes]"

Generate a Mermaid diagram showing the component's visual structure and interactive zones.

---

## REVIEW

**Spec compliance:**
- Manifest has all required properties with correct types
- Entry point implements full StandardControl interface (init, updateView, getOutputs, destroy)
- All bound properties have corresponding React props

**Quality:**
- Component handles disabled and masked modes
- Component handles resize correctly
- No direct DOM manipulation outside container element

**Gate:** Developer confirms review results.

---

## COMPLETE

Write `docs/ui-pcf-[component-name].md`. Return to router.
