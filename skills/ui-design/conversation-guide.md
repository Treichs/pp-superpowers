# ui-design — Router Conversation Guide

This file contains the detailed conversation flow for the ui-design router stages. Sub-skill flows are in each sub-skill's own SKILL.md file.

<EXTREMELY-IMPORTANT>
**Rules that apply to every stage in this guide:**

1. **Real timestamps only.** Every timestamp in `.pp-context/skill-state.json` and in output documents must use actual UTC time from the system. Never use placeholder values like `T00:00:00Z`.
2. **Write state after every stage transition.** Update `.pp-context/skill-state.json` immediately when a stage completes. Do not batch state writes.
3. **Wait for developer confirmation** at every gate. Never auto-advance past a gate condition.
4. **Read sub-skill SKILL.md before dispatch.** When entering SUB_SKILL_EXECUTION, read the sub-skill's SKILL.md from its directory. Do not improvise the sub-skill flow.
</EXTREMELY-IMPORTANT>

---

## Stage: INIT

**Actions:**

1. Read foundation files:
   - `.foundation/00-project-identity.md` — project name, publisher prefix
   - `.foundation/02-architecture-decisions.md` — declared app types
   - `.foundation/05-ui-plan.md` — persona-to-app mapping (if exists and not placeholder)
2. Read `docs/schema-physical-model.md` — entity and column inventory
3. Read `docs/ddd-model.md` (if exists) — aggregate boundaries for form grouping
4. Check for existing ui-design artifacts (`docs/ui-design-spec.md`, `docs/ui-mda-design.md`, etc.)
5. Load `.pp-context/skill-state.json` for session continuity

**Presentation:**

> "Starting ui-design for **[Project Name]**. Here's what I loaded:
>
> - **Physical model:** [N] entities, [M] total columns
> - **Architecture:** [app types from 02-architecture-decisions]
> - **UI plan:** [Available / Not available / Placeholder]
> - **DDD model:** [Available — N aggregates / Not available]
> - **Existing UI artifacts:** [None / List of existing design docs]
>
> [If re-entry artifacts found, present re-entry options per SKILL.md]"

**State write:**
```json
{
  "activeSkill": "ui-design",
  "activeStage": "INIT",
  "activeSubSkill": null,
  "subSkillQueue": [],
  "completedSubSkills": [],
  "stageHistory": [
    { "stage": "INIT", "completedAt": "[UTC timestamp]" }
  ]
}
```

---

## Stage: CONTEXT_GATHER

**Presentation:**

> "I've loaded your foundation and schema. Before we dive into UI design, I want to make sure I understand your current focus.
>
> A few questions:
>
> 1. **What are you designing today?** Are you starting fresh, adding to an existing design, or revisiting something?
> 2. **Any constraints I should know about?** (Timeline pressure, specific user feedback you're responding to, technical limits, accessibility requirements?)
> 3. **Any priorities not captured in your foundation?** (A particular entity or persona that needs attention first, a specific form that's been problematic?)
>
> If everything is captured in the foundation and you want to jump straight to sub-skill selection, just say so."

**Gate:** Developer provides context or explicitly confirms the foundation is current and complete.

**Foundation gap surfacing:** If the context conversation reveals information that contradicts or extends the foundation:

> "That sounds like it may not be reflected in your foundation yet. Do you want to note this for a future solution-discovery update, or should we account for it in our design work today without updating the foundation?"

Context notes are held in working memory for sub-skill dispatch — they do not modify the foundation.

---

## Stage: SUB_SKILL_SELECTION

**Actions:**
1. Read `02-architecture-decisions.md` for declared app types
2. Read `05-ui-plan.md` for persona-to-app-type mapping (if available)
3. Map declared app types to available sub-skills
4. Present matched sub-skills with rationale

**Presentation:**

> "Based on your architecture decisions [and UI plan], here are the sub-skills relevant to your project:
>
> | Sub-skill | Why it's relevant | Status |
> |---|---|---|
> | **model-driven-app** | `02-architecture-decisions` specifies MDA as [primary/secondary] app type | Not started |
> | **canvas-app** | `05-ui-plan` maps [Persona X] to a Canvas experience | Not started |
> | **[others as applicable]** | | |
>
> Additional sub-skills available (not indicated by foundation):
> - pcf-control, custom-page, modal-dialog, code-app
>
> Which sub-skills do you want to work on in this session, and in what order? (You can always return to add more.)"

**Gate:** Developer selects one or more sub-skills and confirms the sequence.

**State write:**
```json
{
  "activeSkill": "ui-design",
  "activeStage": "SUB_SKILL_SELECTION",
  "subSkillQueue": ["model-driven-app", "canvas-app"],
  "completedSubSkills": [],
  "stageHistory": [
    { "stage": "INIT", "completedAt": "[timestamp]" },
    { "stage": "CONTEXT_GATHER", "completedAt": "[timestamp]" },
    { "stage": "SUB_SKILL_SELECTION", "completedAt": "[timestamp]" }
  ]
}
```

---

## Stage: SUB_SKILL_EXECUTION

<EXTREMELY-IMPORTANT>
**Sub-skill dispatch protocol:**

1. Take the first sub-skill from the queue
2. Read that sub-skill's SKILL.md from its directory (e.g., `./model-driven-app/SKILL.md`)
3. Follow the sub-skill's complete state machine — every stage, every gate
4. When the sub-skill reaches COMPLETE, update state and check the queue
5. If more sub-skills remain, present the between-sub-skill prompt
6. If no more sub-skills remain, proceed to INTEGRATION_REVIEW (if 2+ completed) or COMPLETE (if 1)

**Between-sub-skill prompt:**

> "[Sub-skill name] design is complete. Next in your queue: **[next sub-skill]**. Ready to continue, or do you want to pause here?"

If the developer pauses, update state with the current position so RESUME mode can pick up.
</EXTREMELY-IMPORTANT>

**State write (during sub-skill execution):**
```json
{
  "activeSkill": "ui-design",
  "activeStage": "SUB_SKILL_EXECUTION",
  "activeSubSkill": "model-driven-app",
  "subSkillQueue": ["canvas-app"],
  "completedSubSkills": []
}
```

**State write (after sub-skill completes):**
```json
{
  "activeSubSkill": "canvas-app",
  "subSkillQueue": [],
  "completedSubSkills": ["model-driven-app"]
}
```

---

## Stage: INTEGRATION_REVIEW

**Condition:** Only runs if two or more sub-skills were completed in this session.

**Action:** Dispatch the **ui-reviewer** agent in **integration mode**. Provide the agent with:
- All completed sub-skill design document paths
- `docs/ui-form-event-map.md` (aggregated during sub-skill execution)
- `docs/schema-physical-model.md`

**Presentation:**

> "You've completed designs for [list of sub-skills]. Running a cross-design consistency check before we wrap up."

Present the agent's findings. If issues are found:

> "The integration review found [N] issues:
>
> [Agent findings]
>
> Would you like to address these now, or accept them as noted?"

**Gate:** Developer confirms integration review results (resolves issues or accepts with documented rationale).

---

## Stage: COMPLETE

<EXTREMELY-IMPORTANT>
**You MUST do ALL of the following at COMPLETE. Do NOT skip any step.**

1. Write `docs/ui-design-spec.md` — master index linking all sub-skill design artifacts
2. Write `docs/ui-form-event-map.md` — aggregated form events from all sub-skills
3. Update `.pp-context/skill-state.json` with completion state
4. Present the full handoff summary
5. Suggest the next skill
6. WAIT for the developer's response — do NOT auto-proceed
</EXTREMELY-IMPORTANT>

**Master index format (`docs/ui-design-spec.md`):**

```markdown
# UI Design Spec — [Project Name]

Generated by: ui-design
Date: [date]

## Design Documents

| Sub-skill | Document | Status |
|---|---|---|
| model-driven-app | docs/ui-mda-design.md | Complete |
| canvas-app | docs/ui-canvas-design.md | Complete |

## Form Event Map

See: docs/ui-form-event-map.md ([N] events across [M] entities)

## Integration Review

[PASS / PASS WITH NOTES / Skipped (single sub-skill)]
```

**Form event map format (`docs/ui-form-event-map.md`):**

```markdown
# Form Event Map — [Project Name]

Generated by: ui-design
Date: [date]

## Entity: [Entity Name]

| Form | Field | Event type | Trigger condition | Business logic required |
|---|---|---|---|---|
| Main | [field] | OnChange | Always | Conditional required on [other field] |
| Main | [field] | OnSave | Always | Cross-entity validation |

## Code App Events

| Component | Event | Dataverse operation | Business logic required |
|---|---|---|---|
| [Component] | [event] | [create/update/delete] | [description] |
```

**Completion state:**
```json
{
  "activeSkill": null,
  "activeStage": null,
  "activeSubSkill": null,
  "subSkillQueue": [],
  "completedSubSkills": ["model-driven-app", "canvas-app"],
  "lastCompleted": "ui-design",
  "suggestedNext": "business-logic",
  "completedSkills": ["solution-discovery", "application-design", "schema-design", "ui-design"],
  "artifacts": [
    "docs/ui-design-spec.md",
    "docs/ui-form-event-map.md",
    "docs/ui-mda-design.md",
    "docs/ui-canvas-design.md"
  ]
}
```

**Handoff presentation:**

> "UI design is complete. Here's what was produced:
>
> - [List of design documents with paths]
> - Form event map: `docs/ui-form-event-map.md` ([N] form events across [M] entities)
>
> **Suggested next:** business-logic — your form event map identifies [N] client scripts and [M] validation requirements that need implementation. The client-script sub-skill will read this map as its starting point."
