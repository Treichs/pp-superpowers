---
name: custom-page
description: Dispatched by the ui-design router — designs a custom page (hybrid Canvas/MDA
  page embedded in a Model-Driven App). Not directly invokable; use ui-design to access
  this sub-skill.
---

# Custom Page Design

Designs a custom page — a hybrid Canvas/MDA page embedded in a Model-Driven App. Uses Canvas app patterns for layout and controls, but runs in the MDA shell. Custom pages give full layout freedom within the MDA navigation shell.

## State Machine

```
INIT → COMPONENT_DESIGN → NAVIGATION → RESPONSIVE_LAYOUT → WIREFRAME → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Confirm embedding context in MDA, load physical model | No |
| COMPONENT_DESIGN | Controls, data binding, layout components | No |
| NAVIGATION | How the page is invoked, how it returns, internal navigation | No |
| RESPONSIVE_LAYOUT | Container layout, MDA chrome considerations | No |
| WIREFRAME | Canva mockup or Mermaid layout diagram | No |
| REVIEW | Dispatch ui-reviewer in sub-skill mode | No |
| COMPLETE | Write `docs/ui-custompage-design.md` | No |

---

## INIT

**Presentation:**

> "Starting custom-page design. A custom page is embedded in a Model-Driven App but built with Canvas controls — full layout freedom within the MDA navigation shell.
>
> Where does this page sit in the MDA?
> - Launched from the sitemap (standalone page)
> - Launched from a form command bar button (contextual)
> - Embedded within a form section (inline)
>
> What entity context does it receive when launched?"

**Gate:** Developer confirms embedding point and context.

---

## COMPONENT_DESIGN

**Presentation:**

> "**Custom Page — Components**
>
> Components available: Modern controls (Fluent UI), PCF controls, galleries, forms, media controls.
>
> Proposed component layout:
> | Component | Type | Data source | Purpose |
> |---|---|---|---|
> | [component] | [type] | [Dataverse table / collection] | [what it does] |
>
> What interactions need to write back to Dataverse vs. just display data?"

**Gate:** Developer confirms component layout.

---

## NAVIGATION

**Presentation:**

> "**Navigation**
>
> How is this page invoked? [Sitemap subarea / Command button with recordId parameter / Form section]
> How does it return? [Back navigation / Saves and closes / Stays open as panel]
> Does it navigate internally between views?"

**Gate:** Developer confirms navigation design.

---

## RESPONSIVE_LAYOUT

**Presentation:**

> "**Layout Design**
>
> Custom pages run inside the MDA shell — left navigation and header consume space. Effective content width is ~70-80% of screen width on desktop.
>
> Proposed container layout:
> | Container | Direction | Content |
> |---|---|---|
> | [name] | Vertical / Horizontal | [content] |
>
> Does this page need to work on mobile (MDA mobile app)?"

**Gate:** Developer confirms layout approach.

---

## WIREFRAME

Generate Canva mockup if available, otherwise Mermaid layout diagram showing component arrangement within MDA shell.

---

## REVIEW

**Spec compliance:** All data bindings reference valid physical model columns, page receives correct entity context, return/close behavior defined.

**Quality:** Layout accounts for MDA chrome, modern controls used over classic where available.

**Gate:** Developer confirms review results.

---

## COMPLETE

Write `docs/ui-custompage-design.md`. Return to router.
