---
name: canvas-app
description: Dispatched by the ui-design router — designs Canvas app screens, navigation,
  data sources, and responsive layout. Not directly invokable; use ui-design to access
  this sub-skill.
---

# Canvas App Design

Designs a Canvas app experience: screens, navigation flow, data source connections, responsive layout, and delegation risk analysis. Produces a Canva mockup per screen when Canva MCP is available.

## State Machine

```
INIT → SCREEN_DESIGN → NAVIGATION → DATA_SOURCES → RESPONSIVE_LAYOUT → WIREFRAME → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Load UI plan (persona-to-screen mapping), physical model | No |
| SCREEN_DESIGN | Design each screen: purpose, components, data displayed | No |
| NAVIGATION | Screen flow, navigation triggers, conditional routing | No |
| DATA_SOURCES | Dataverse table connections, delegation analysis | No |
| RESPONSIVE_LAYOUT | App format, container layout, device targets | No |
| WIREFRAME | Generate Canva mockup per screen (or Mermaid screen flow) | No |
| REVIEW | Dispatch ui-reviewer in sub-skill mode | No |
| COMPLETE | Write `docs/ui-canvas-design.md`, update form event map | No |

---

## INIT

**Action:** Read `05-ui-plan.md` for persona-to-app mapping. Identify which personas use this Canvas app and their primary workflows.

**Presentation:**

> "Starting canvas-app design. From your UI plan, this app serves:
>
> | Persona | Primary workflow | Screens implied |
> |---|---|---|
> | [persona] | [workflow] | [implied screens] |
>
> Does this capture the full scope, or are there workflows not in the UI plan?"

**Gate:** Developer confirms scope.

---

## SCREEN_DESIGN

**Presentation (per screen):**

> "**[Screen Name]**
>
> Purpose: [what the user does here]
>
> Proposed components:
> - [Gallery showing [entity] records, columns: [list]]
> - [Form for [entity] — create/edit mode]
> - [Buttons: [list of actions]]
> - [Labels/KPI tiles: [list]]
>
> Global variables or collections this screen reads/writes: [list]
>
> Does this layout match your intent?"

**Gate:** Developer confirms per screen.

---

## NAVIGATION

**Presentation:**

> "**Navigation Flow**
>
> Proposed screen flow:
>
> [Screen A] → (on record select) → [Screen B] → (on save) → [Screen A]
>
> Navigation triggers:
> | Trigger | Source screen | Destination | Condition |
> |---|---|---|---|
> | Record tap in gallery | [Screen A] | [Screen B] | Always |
> | Back button | [Screen B] | [Screen A] | Always |
>
> Does this flow cover all user paths?"

**Gate:** Developer confirms navigation flow.

---

## DATA_SOURCES

**Presentation:**

> "**Data Sources**
>
> Dataverse tables this app connects to:
>
> | Table | Usage | Control type | Delegation risk? |
> |---|---|---|---|
> | [table] | [read / read-write] | Gallery / Form / Lookup | [Yes if >500 rows and non-delegable filter] |
>
> **Delegation warnings:** [list non-delegable operations]
> - StartsWith on non-indexed text columns
> - OR conditions combining multiple Filter expressions
> - String manipulation functions in filters
>
> Are there connectors beyond Dataverse? (e.g., Office 365 Users, SharePoint)"

**Gate:** Developer confirms data sources and resolves delegation warnings.

---

## RESPONSIVE_LAYOUT

**Presentation:**

> "**Responsive Layout Design**
>
> App format: [Phone / Tablet / Responsive — confirm or choose]
>
> Container-based layout:
> | Container | Direction | Content | Fills on mobile? |
> |---|---|---|---|
> | AppShell | Vertical | Header + Main + Footer | Yes |
> | MainContent | Horizontal | SideNav + Content | Stacks on mobile |
>
> **Minimum screen sizes:** [based on persona context — field worker on phone vs. office on tablet]
>
> Are there specific device types or orientations to support?"

**Gate:** Developer confirms layout approach and device targets.

---

## WIREFRAME

**Action:** If Canva MCP is available, generate a Canva mockup per screen. Otherwise, generate a Mermaid screen flow diagram showing screen relationships and navigation paths.

**Presentation:**

> "Generating [Canva mockups / Mermaid screen flow diagram] for [N] screens. These are design-intent references showing layout, navigation, and data presentation."

---

## REVIEW

**Spec compliance:**
- Every persona workflow from the UI plan has at least one screen
- Every Dataverse table in data sources has a physical model entry
- All delegation risks are documented

**Quality:**
- No gallery loads more than 500 records without a filter
- No dead-end screens
- Containers used for layout (not absolute positioning)
- Accessible labels on all interactive controls

**Gate:** Developer confirms review results.

---

## COMPLETE

Write `docs/ui-canvas-design.md`. Contribute form events (PowerFx formula events needing backend logic) to the form event map. Return to router.
