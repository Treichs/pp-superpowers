# security — Conversation Guide

This file contains the detailed conversation flow for all security skill stages.

<EXTREMELY-IMPORTANT>
**Rules that apply to every stage in this guide:**

1. **Real timestamps only.** Every timestamp in `.pp-context/skill-state.json` and in output documents must use actual UTC time. Never use placeholders.
2. **Write state after every stage transition.** Update `.pp-context/skill-state.json` immediately when a stage completes.
3. **Wait for developer confirmation** at every gate. Never auto-advance.
4. **Least privilege by default.** Every role starts with zero privileges. Add only what is required. Organization-level access requires explicit justification.
5. **Schema-driven completeness.** Every entity in the physical model must appear in the privilege matrix. Every sensitive column must have an FLS decision.
</EXTREMELY-IMPORTANT>

---

## Stage: INIT

**Actions:**

1. Read foundation files:
   - `.foundation/00-project-identity.md` — project name, publisher prefix
   - `.foundation/01-requirements.md` — access-related requirements
   - `.foundation/02-architecture-decisions.md` — architecture context
   - `.foundation/03-entity-map.md` — entity inventory
   - `.foundation/05-ui-plan.md` — persona-to-app mapping (if exists)
   - `.foundation/08-security-profile.md` — personas and sensitivity flags
2. Read `docs/schema-physical-model.md` — entity inventory, column inventory, table ownership types
3. Read `docs/ddd-model.md` (if exists) — bounded context assignments
4. Read `docs/ui-design-spec.md` (if exists) — persona-to-entity visibility
5. Check for existing `docs/security-design.md`
6. Load `.pp-context/skill-state.json`

**Presentation:**

> "Starting security design for **[Project Name]**. Here's what I loaded:
>
> - **Physical model:** [N] entities ([M] User/Team-owned, [K] Organization-owned)
> - **Security profile:** [Available — N personas / Not available / Placeholder]
> - **UI design:** [Available — persona-to-app mapping / Not available]
> - **DDD model:** [Available — bounded contexts / Not available]
> - **Existing security artifacts:** [None / docs/security-design.md exists]
>
> [If re-entry, present options per SKILL.md]"

**State write:**
```json
{
  "activeSkill": "security",
  "activeStage": "INIT",
  "stageHistory": [
    { "stage": "INIT", "completedAt": "[UTC timestamp]" }
  ]
}
```

---

## Stage: ROLE_ANALYSIS

**Actions:**
1. Read `08-security-profile.md` for declared personas and access descriptions
2. Cross-reference with `05-ui-plan.md` persona-to-app mapping (if available)
3. Map each persona to a proposed security role

**Presentation:**

> "**Role Analysis for [Project Name]**
>
> From your security profile [and UI plan], I've identified these personas and proposed roles:
>
> | Persona | Proposed role name | Privilege pattern | Rationale |
> |---|---|---|---|
> | [persona] | [prefix]_[RoleName] | [Read-heavy / Full CRUD / Admin / View-only] | [why] |
>
> **Standard roles (always included):**
> - **System Administrator** — built-in, not customized
> - **Basic User** — baseline role, minimal privileges on custom entities
>
> **Naming convention:** `[publisher prefix]_[Role Name]`
>
> Questions:
> 1. Any personas I've missed?
> 2. Should any personas share a single role?
> 3. Any roles needing impersonation privileges?
> 4. Any roles needing custom entity ownership?"

**Gate:** Developer confirms the role list. Each role has a name, pattern, and persona mapping.

---

## Stage: PRIVILEGE_MATRIX

<EXTREMELY-IMPORTANT>
**This stage has two rounds:**
1. **Per-role round:** Present one role at a time. Developer confirms each.
2. **Cross-role round:** After all roles confirmed, present the summary matrix for holistic review.

Both rounds are required. Do NOT skip the cross-role summary.
</EXTREMELY-IMPORTANT>

**Per-role presentation:**

> "**Privilege Matrix: [Role Name]**
>
> | Entity | Ownership | Cr | Rd | Wr | Dl | Ap | ApTo | As | Sh |
> |---|---|---|---|---|---|---|---|---|---|
> | [entity] | User/Team | BU | BU | User | None | User | User | None | None |
> | [entity] | Org | — | Org | — | — | — | — | — | — |
>
> **Key decisions:**
> - [Entity]: [level] because [reason]
> - [Entity]: No Delete because [reason]
>
> Does this match [persona]'s access needs?"

**Gate:** Developer confirms per role.

**Cross-role summary presentation:**

> "**Cross-Role Summary**
>
> | Entity | [Role 1] | [Role 2] | [Role 3] | System Admin |
> |---|---|---|---|---|
> | [entity] | rd:BU, wr:User | rd:User | Full CRUD:Org | Full |
>
> **Privilege escalation check:** [Flag entities where non-admin has Org-level Write/Delete]
>
> Does this cross-role view look correct?"

**Gate:** Developer confirms cross-role summary.

**Privilege level reference:**

| Level | Scope |
|---|---|
| None | No access |
| User | Own records only |
| BU | Business unit records |
| P:C BU | BU + child BUs |
| Org | All records (justify for non-admin) |

**Privilege type abbreviations:**

| Abbr | Privilege |
|---|---|
| Cr | Create |
| Rd | Read |
| Wr | Write |
| Dl | Delete |
| Ap | Append |
| ApTo | Append To |
| As | Assign |
| Sh | Share |

---

## Stage: FIELD_SECURITY

**Skip check:** If `08-security-profile.md` does not flag any sensitive columns AND no columns in the physical model are marked sensitive:

> "No sensitive columns flagged in your security profile or physical model. Skipping field-level security. You can re-run in UPDATE mode to add FLS later."

**Presentation (if not skipped):**

> "**Field-Level Security Design**
>
> Sensitive columns identified:
> | Entity | Column | Sensitivity | Current access |
> |---|---|---|---|
> | [entity] | [column] | [PII / financial / confidential] | All roles can read/write |
>
> **Proposed field security profiles:**
> | Profile name | Purpose | Columns |
> |---|---|---|
> | [prefix]_SensitiveData | Restrict PII | [column list] |
>
> **Profile-to-role assignment:**
> | Profile | [Role 1] | [Role 2] | [Role 3] |
> |---|---|---|---|
> | [prefix]_SensitiveData | Read+Update | Read only | No access |
>
> Does this FLS design match your data protection requirements?"

**Gate:** Developer confirms FLS profiles, column assignments, and role mapping. Every sensitive column covered.

**FLS rules to enforce:**
- Never secure the primary name column (breaks views and lookups)
- FLS is enforced server-side — API access is also restricted
- Secured columns show `*****` to unauthorized users
- Plugins run in system context and bypass FLS unless coded to respect it

---

## Stage: TEAM_HIERARCHY

**Skip check:** If no organizational hierarchy requirements (single BU, no team collaboration, no manager-reports access):

> "No team-based or hierarchy security requirements identified. Skipping. Re-run in UPDATE mode if needed."

**Presentation (if not skipped):**

> "**Team and Hierarchy Security Design**
>
> **Team types:**
> | Team type | Purpose | Scope |
> |---|---|---|
> | Owner team | [description] | BU-level |
> | Access team | [description] | per-record |
>
> **Proposed teams:**
> | Team name | Type | Business unit | Security role(s) | Purpose |
> |---|---|---|---|---|
> | [team] | Owner | [BU] | [role(s)] | [purpose] |
>
> **Hierarchy security:**
> - Model: [Manager / Position / Not needed]
> - [If manager:] Depth: [N levels]
>
> **Access team entities:**
> | Entity | Access team enabled | Purpose |
> |---|---|---|
> | [entity] | Yes | [per-record collaboration need] |
>
> Does this match your organizational structure?"

**Gate:** Developer confirms teams, hierarchy model, and access team entities.

---

## Stage: REVIEW

<EXTREMELY-IMPORTANT>
**You MUST dispatch the security-reviewer agent at this stage.**
Do NOT substitute a manual review or skip the agent dispatch.
Provide the agent with:
- `docs/security-design.md` (draft assembled from prior stages)
- `docs/schema-physical-model.md`
- Foundation sections (00, 01, 03, 05, 08)
- `docs/ddd-model.md` (if available)

Wait for the agent to return its findings before presenting results.
All HIGH findings MUST be resolved before proceeding to COMPLETE.
</EXTREMELY-IMPORTANT>

**Pre-review action:** Assemble the draft `docs/security-design.md` from all confirmed decisions before dispatching the agent.

**Presentation:**

> "Running security review. Checking for over-privileged roles, privilege matrix completeness, FLS coverage, and team alignment."

Present agent findings:

> "**Security Review Results**
>
> **Overall:** [PASS / PASS WITH NOTES / FAIL]
>
> **HIGH findings ([N]):** [must resolve]
> **MEDIUM findings ([N]):** [should resolve or accept with rationale]
> **LOW findings ([N]):** [noted]
>
> [Detailed findings from agent]"

**Gate:** All HIGH resolved. MEDIUM resolved or accepted. Developer confirms.

---

## Stage: COMPLETE

<EXTREMELY-IMPORTANT>
**You MUST do ALL of the following at COMPLETE. Do NOT skip any step.**

1. Write final `docs/security-design.md` (incorporating review fixes)
2. Update `.pp-context/skill-state.json` with completion state
3. Present the full handoff summary
4. Suggest the next skill
5. WAIT for the developer's response — do NOT auto-proceed
</EXTREMELY-IMPORTANT>

**Completion state:**
```json
{
  "activeSkill": null,
  "activeStage": null,
  "lastCompleted": "security",
  "suggestedNext": "alm-workflow",
  "completedSkills": ["solution-discovery", "application-design", "schema-design", "ui-design", "business-logic", "security"],
  "artifacts": ["docs/security-design.md"]
}
```

**Handoff presentation:**

> "Security design is complete. Here's what was produced:
>
> - **Security roles:** [N] custom roles defined
> - **Privilege matrix:** [N] entities × [N] roles — all privileges documented
> - **Field-level security:** [N] profiles covering [M] columns [or "Not applicable"]
> - **Team/hierarchy:** [summary] [or "Not applicable"]
>
> Output: `docs/security-design.md`
>
> **Suggested next:** alm-workflow — your security roles need to be packaged into the solution. Or environment-setup if you're ready to provision environments."
