# business-logic — Router Conversation Guide

This file contains the detailed conversation flow for the business-logic router stages. Sub-skill flows are in each sub-skill's own SKILL.md file.

<EXTREMELY-IMPORTANT>
**Rules that apply to every stage in this guide:**

1. **Real timestamps only.** Every timestamp in `.pp-context/skill-state.json` and in output documents must use actual UTC time. Never use placeholders.
2. **Write state after every stage transition.** Update `.pp-context/skill-state.json` immediately when a stage completes.
3. **Wait for developer confirmation** at every gate.
4. **Read sub-skill SKILL.md before dispatch.** When entering SUB_SKILL_EXECUTION, read the sub-skill's SKILL.md from its directory.
5. **Apply the complexity gate** at SUB_SKILL_SELECTION before the developer selects sub-skills.
</EXTREMELY-IMPORTANT>

---

## Stage: INIT

**Actions:**

1. Read foundation files:
   - `.foundation/00-project-identity.md` — project name, publisher prefix
   - `.foundation/01-requirements.md` — requirements for logic identification
   - `.foundation/02-architecture-decisions.md` — architecture context
   - `.foundation/06-logic-map.md` — logic requirements (if exists and not placeholder)
2. Read `docs/schema-physical-model.md` — entity relationships, column types, cascade behaviors
3. Read `docs/ddd-model.md` (if exists) — domain events for trigger mapping
4. Read `docs/ui-form-event-map.md` (if exists) — client script requirements
5. Check for existing business-logic artifacts
6. Load `.pp-context/skill-state.json`

**Presentation:**

> "Starting business-logic for **[Project Name]**. Here's what I loaded:
>
> - **Physical model:** [N] entities with relationships and cascade behaviors
> - **Logic map:** [Available — N requirements / Not available / Placeholder]
> - **DDD model:** [Available — domain events for trigger mapping / Not available]
> - **Form event map:** [Available — N client script events / Not available]
> - **Existing artifacts:** [None / list]
>
> [If re-entry artifacts found, present re-entry options]"

**State write:**
```json
{
  "activeSkill": "business-logic",
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

> "I've loaded your foundation, logic map, and physical model. Before we start implementing business logic, I want to make sure I understand where you are.
>
> A few questions:
>
> 1. **What are you implementing today?** Starting fresh, continuing existing work, or adding to what's already built?
> 2. **Any constraints I should know about?** (Performance requirements, plugin execution context restrictions, flow license tier, specific Dataverse environment version?)
> 3. **Is the logic map current?** [If available: It was last updated [date].] Any requirements added or changed?
>
> If the foundation is current and you want to go straight to sub-skill selection, just say so."

**Gate:** Developer confirms or updates context.

---

## Stage: SUB_SKILL_SELECTION

**Actions:**
1. Read `06-logic-map.md` and classify each entry by logic type
2. Apply business rule complexity gate (see SKILL.md)
3. Map classified entries to sub-skills
4. Present the result

**Presentation:**

> "Based on your logic map, here's what needs to be built:
>
> | Sub-skill | Requirements | Status |
> |---|---|---|
> | **csharp-plugin** | [N] plugin requirements | Not started |
> | **power-automate** | [N] flow requirements | Not started |
> | **business-rule** | [N] business rule requirements ([X] flagged for complexity review) | Not started |
> | **client-script** | [N] client script requirements ([from form event map / from logic map]) | Not started |
>
> [If any business rules were redirected:]
> **Complexity gate:** I've redirected [N] requirements that exceed business rule capabilities:
> | Requirement | Original type | Redirected to | Reason |
> |---|---|---|---|
> | [requirement] | business-rule | csharp-plugin | Requires cross-entity data |
>
> Which sub-skills do you want to work on today, and in what order?"

**Gate:** Developer selects and sequences.

---

## Stage: SUB_SKILL_EXECUTION

Same protocol as ui-design router — read sub-skill SKILL.md, follow its state machine, return to router after COMPLETE, present between-sub-skill prompt if more remain.

---

## Stage: INTEGRATION_REVIEW

**Condition:** Only runs if two or more sub-skills were completed.

**Cross-logic consistency checks:**
1. **No duplicate logic:** Same requirement not implemented in both a plugin and a business rule
2. **No conflicting behaviors:** Client scripts and plugins on the same entity don't create conflicting field value operations
3. **No trigger overlap:** Flow triggers don't overlap with plugin triggers on the same entity/message causing double-execution

**Presentation:**

> "You've completed implementations for [list]. Running a cross-logic consistency check."

Present findings. Developer resolves or accepts.

---

## Stage: COMPLETE

<EXTREMELY-IMPORTANT>
**You MUST do ALL of the following at COMPLETE. Do NOT skip any step.**

1. Write `docs/business-logic-inventory.md` — output manifest
2. Update `.pp-context/skill-state.json` with completion state
3. Present the full handoff summary
4. Suggest the next skill
5. WAIT for the developer's response
</EXTREMELY-IMPORTANT>

**Inventory format (`docs/business-logic-inventory.md`):**

```markdown
# Business Logic Inventory — [Project Name]

Generated by: business-logic
Date: [date]

## C# Plugins ([N] total)
| Plugin class | Entity | Message | Stage | Mode | Artifact |
|---|---|---|---|---|---|
| [PluginName] | [entity] | [message] | [stage] | [sync/async] | src/Plugins/[name].cs |

## Power Automate Flows ([N] total)
| Flow name | Trigger | Connection references | Artifact |
|---|---|---|---|
| [Flow Name] | [trigger] | [list] | docs/bl-flow-[name].md |

## Business Rules ([N] total)
| Rule name | Entity | Scope | Trigger | Artifact |
|---|---|---|---|---|
| [Rule Name] | [entity] | [scope] | [trigger] | docs/bl-rule-[entity].md |

## Client Scripts ([N] total)
| Script file | Entity | Forms | Event count | Artifact |
|---|---|---|---|---|
| [prefix]_[Entity]Form.js | [entity] | [forms] | [N] | src/WebResources/[file] |
```

**Handoff presentation:**

> "Business logic implementation is complete. Here's the inventory:
>
> - [N] C# plugin classes across [M] entities
> - [N] Power Automate flow designs
> - [N] business rules
> - [N] client scripts
>
> **Suggested next:** security — if you haven't designed your security model yet. Otherwise, alm-workflow for solution packaging."
