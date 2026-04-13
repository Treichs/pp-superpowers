---
name: model-driven-app
description: Dispatched by the ui-design router — designs MDA forms, views, dashboards,
  charts, sitemap, and command bar for Dataverse entities. Not directly invokable;
  use ui-design to access this sub-skill.
---

# Model-Driven App Design

Designs the full Model-Driven App experience: entity forms (tabs, sections, fields, subgrids), views, dashboards, charts, sitemap navigation, and command bar customizations. Consumes the physical model column by column and produces a complete MDA design specification.

## State Machine

```
INIT → FORM_DESIGN → VIEW_DESIGN → DASHBOARD_CHART_DESIGN → SITEMAP_COMMANDBAR → WIREFRAME → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Load physical model and DDD model, identify entities in scope | No |
| FORM_DESIGN | Design forms per entity: tabs, sections, fields, subgrids, form events | No |
| VIEW_DESIGN | Design default views, quick-find views, column selection | No |
| DASHBOARD_CHART_DESIGN | Design dashboards and charts | No |
| SITEMAP_COMMANDBAR | Design navigation structure and command bar customizations | No |
| WIREFRAME | Generate Mermaid form layout diagrams | No |
| REVIEW | Dispatch ui-reviewer in sub-skill mode | No |
| COMPLETE | Write `docs/ui-mda-design.md`, update form event map | No |

---

## INIT

**Action:** Read `docs/schema-physical-model.md`. Identify all entities. If `docs/ddd-model.md` is available, load aggregate definitions for tab grouping guidance.

**Presentation:**

> "Starting model-driven-app design for **[Project Name]**. I can see [N] entities in your physical model:
>
> [Entity list with column counts]
>
> [If DDD model available:] I have your aggregate map — I'll use aggregate boundaries to suggest tab groupings on complex forms.
>
> Scope check: Do you want to design forms for all [N] entities, or focus on a subset today?"

**Gate:** Developer confirms entity scope.

---

## FORM_DESIGN

**Action:** For each entity in scope, work through form design collaboratively. Process is column-by-column from the physical model.

**Presentation (per entity):**

> "**[Entity Name] Form Design**
>
> Physical model has [N] columns. Here's my proposed form layout:
>
> **Tab 1 — [Name] (suggested from aggregate root fields):**
> | Section | Columns | Control type | Required? |
> |---|---|---|---|
> | [Section name] | [column list] | [Text, Lookup, Choice, etc.] | Yes/No |
>
> **Subgrid candidates:** [related entities that should appear as subgrids, with reason]
>
> **Form event candidates:** [fields likely to need client-script events — visibility rules, conditional required, cross-field calculations]
>
> Does this layout work? What would you move, add, or remove?"

**Gate:** Developer confirms form layout for each entity. Form event candidates are logged for the form event map.

**Column type to control type mapping reference:**

| Dataverse type | Default MDA control | Notes |
|---|---|---|
| Single Line of Text | Text input | Consider email/URL/phone subtypes |
| Multiple Lines of Text | Text area | Enable rich text for user-facing notes |
| Whole Number | Number input | |
| Decimal / Float | Number input with precision | |
| Currency | Currency input | Shows symbol, handles exchange rate |
| Date Only | Date picker | |
| Date and Time | Date + time picker | Timezone: local vs. UTC |
| Choice (single) | Dropdown | |
| Choices (multi-select) | Multi-select picker | |
| Yes/No | Toggle or checkbox | |
| Lookup | Lookup control | Note quick-create form requirement |
| File / Image | File upload / Image viewer | Notes-enabled tables only |
| Calculated | Read-only display | Cannot be edited |
| Rollup | Read-only display with refresh | |

---

## VIEW_DESIGN

**Presentation (per entity):**

> "**[Entity Name] Views**
>
> Minimum views: Active Records, Quick Find. Additional based on your requirements:
>
> **Active Records View:**
> - Columns: [suggested from physical model — name, status, key lookup, modified date]
> - Default sort: [suggested]
>
> **Quick Find View (search columns):**
> - Searchable columns: [suggested from text and lookup columns]
>
> Any persona-specific views? (e.g., 'My Records' filtered by assignment, 'Pending Approval')"

**Gate:** Developer confirms view design per entity.

---

## DASHBOARD_CHART_DESIGN

**Presentation:**

> "**Dashboards and Charts**
>
> Based on your requirements and entity list, here are dashboard candidates:
> - **[Dashboard name]:** [suggested for which persona, what data it surfaces]
>
> Chart candidates per entity:
> | Entity | Chart type | X-axis | Y-axis / measure | Why |
> |---|---|---|---|---|
> | [entity] | Bar / Pie / Funnel | [field] | [field or count] | [reason] |
>
> What should be included vs. deferred?"

**Gate:** Developer confirms dashboard and chart scope.

---

## SITEMAP_COMMANDBAR

**Presentation:**

> "**Sitemap Design**
>
> Proposed navigation structure:
>
> **Area: [Area Name]**
> - Group: [Group Name]
>   - Subarea: [Entity] → [Entity List View]
>   - Subarea: [Entity] → [Entity List View]
>
> Does this match how your users will navigate the app?
>
> **Command Bar Customizations:**
> Any custom buttons needed on forms or views? (e.g., 'Submit for Approval', 'Generate Report')"

**Gate:** Developer confirms sitemap and command bar design.

---

## WIREFRAME

**Action:** Generate Mermaid diagrams showing the structural layout of major entity forms — tabs, sections, field positions, and subgrids.

<EXTREMELY-IMPORTANT>
**Diagram generation rules:**
- Use Mermaid block diagrams or flowcharts to represent form structure
- Show tab structure, section layout, and key field groupings
- These are structural references, not pixel-perfect mockups
- Write diagrams directly into the design document
</EXTREMELY-IMPORTANT>

**Presentation:**

> "Generating form layout diagrams for [N] entity forms. These show the structural layout — tabs, sections, field positions, and subgrids."

---

## REVIEW

**Action:** Dispatch **ui-reviewer** agent in **model-driven-app** sub-skill mode.

**Stage 1 — Spec compliance:**
- Every column in the physical model is on a form, in a view, or explicitly excluded with reason
- Every subgrid has a supporting relationship in the physical model
- Every command bar customization has a named handler
- All personas have a navigation path through the sitemap

**Stage 2 — Quality:**
- No form exceeds 4 tabs
- No tab exceeds 8 sections
- Required fields are in the first visible tab
- Lookup fields have quick-create forms noted
- No more than 3 subgrids per form
- Accessible labels on all fields

**Presentation:**

> "**Review — model-driven-app**
>
> **Spec compliance:** [PASS / N issues]
> **Quality:** [PASS / N recommendations]
>
> [If issues:] Resolve before marking complete, or explicitly accept with documented reason."

**Gate:** Developer confirms review results.

---

## COMPLETE

**Action:**
1. Write `docs/ui-mda-design.md` — complete MDA design specification
2. Contribute form event candidates to the form event map (aggregated by router at COMPLETE)

**Return to router:** After writing the design document, return control to the ui-design router for the next sub-skill or INTEGRATION_REVIEW.
