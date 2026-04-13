# pp-superpowers — security Skill Specification

**Version:** 1.0
**Date:** April 13, 2026
**Author:** SDFX Studios
**Status:** Draft — pending approval
**Parent document:** pp-superpowers Design Roadmap v1.0

---

## 1. Skill Overview

| Attribute | Value |
|---|---|
| **Name** | security |
| **Domain** | Power Platform security design — security roles, field-level security, team-based access, and hierarchy security |
| **Lifecycle group** | Build |
| **Has sub-skills** | No — single-path skill with skippable stages for optional security patterns |
| **Foundation sections consumed** | `00-project-identity`, `01-requirements`, `02-architecture-decisions`, `03-entity-map`, `05-ui-plan`, `08-security-profile` |
| **Physical model consumed** | `docs/schema-physical-model.md` (entity inventory, column inventory, table ownership type, relationship behaviors) |
| **DDD model consumed** | `docs/ddd-model.md` (bounded context assignments for role scoping, aggregate boundaries for access patterns) |
| **UI design consumed** | `docs/ui-design-spec.md` (app type determines which entities are surfaced to which personas — informs role-to-entity privilege mapping) |
| **Upstream dependency** | schema-design (physical model must exist); ui-design (recommended — persona-to-app mapping informs role design) |
| **Downstream handoff** | alm-workflow (security role XML for solution packaging), environment-setup (role assignment configuration) |
| **Agents** | security-reviewer |

### 1.1 Design Philosophy

security is a **single-path skill** that walks through security design in layers: first identify who needs access (roles), then define what they can do (privileges), then lock down sensitive data (field-level security), then handle organizational structure (teams and hierarchy). Each layer builds on the previous.

Three locked design decisions:

1. **Least privilege by default** — every role starts with zero privileges and adds only what is required. The skill never proposes Organization-level access without explicit justification.
2. **Schema-driven completeness** — the physical model is the source of truth for which entities and columns exist. Every entity in the physical model must appear in the privilege matrix (even if the assignment is "no access"). Every column flagged as sensitive in the security profile must have a field-level security decision.
3. **Separation of design from implementation** — security produces a design document (`docs/security-design.md`) that specifies roles, privileges, FLS profiles, and team configurations. It does not create security role XML, configure environments, or assign roles to users — those are alm-workflow and environment-setup responsibilities.

### 1.2 Relationship to Other Skills

**Upstream:** schema-design produces `docs/schema-physical-model.md` — the entity and column inventory that drives the privilege matrix. The physical model's table ownership type (User/Team vs. Organization-owned) directly determines which privilege levels are available per entity. ui-design produces `docs/ui-design-spec.md`, which maps personas to app types — this informs which entities each role needs access to.

**Downstream:** alm-workflow reads `docs/security-design.md` to generate security role XML for solution packaging. environment-setup reads the team configuration section to set up teams and assign roles during environment provisioning.

**Cross-skill signals:** If the security analysis reveals that the foundation's `08-security-profile` is incomplete (e.g., no personas defined, no sensitive data flagged), security surfaces this gap and suggests the developer update the foundation via solution-discovery UPDATE mode before proceeding. It does not modify the foundation.

### 1.3 Boundary: security vs. environment-setup

security **designs** the security model — roles, privileges, FLS profiles, team structures. environment-setup **implements** the security model — creates teams in environments, assigns roles to users, activates field-level security profiles. The design document is the handoff artifact between them.

---

## 2. State Machine

```
INIT → ROLE_ANALYSIS → PRIVILEGE_MATRIX → FIELD_SECURITY → TEAM_HIERARCHY → REVIEW → COMPLETE
```

### 2.1 Stage Definitions

| Stage | Purpose | Gate to enter | Can skip? |
|---|---|---|---|
| INIT | Read all inputs, detect re-entry, check prerequisites | Foundation exists, physical model exists | No |
| ROLE_ANALYSIS | Identify roles from security profile personas, map to Dataverse security role patterns | INIT complete | No |
| PRIVILEGE_MATRIX | Build full privilege matrix: every entity × every role × every privilege type | ROLE_ANALYSIS complete | No |
| FIELD_SECURITY | Identify FLS candidates, define field security profiles and column assignments | PRIVILEGE_MATRIX complete | Yes — if no sensitive columns flagged in security profile |
| TEAM_HIERARCHY | Design team-based security and/or hierarchy security for org-structure-aware access | FIELD_SECURITY complete (or skipped) | Yes — if neither team nor hierarchy security is in scope |
| REVIEW | Dispatch security-reviewer agent for least-privilege validation and completeness check | TEAM_HIERARCHY complete (or skipped) | No |
| COMPLETE | Write completion state, produce handoff artifact, suggest next skill | REVIEW confirmed | No |

### 2.2 Progress Tracking

```json
{
  "activeSkill": "security",
  "activeStage": "PRIVILEGE_MATRIX",
  "stageHistory": [
    { "stage": "INIT", "completedAt": "2026-04-13T10:00:00Z" },
    { "stage": "ROLE_ANALYSIS", "completedAt": "2026-04-13T10:15:00Z" },
    { "stage": "PRIVILEGE_MATRIX", "startedAt": "2026-04-13T10:16:00Z" }
  ],
  "lastCompleted": "ui-design",
  "suggestedNext": null,
  "completedSkills": ["solution-discovery", "application-design", "schema-design", "ui-design"]
}
```

On session resume:

> "You're in security, currently at the **PRIVILEGE_MATRIX** stage. You've defined [N] roles. Want to pick up where you left off?"

---

## 3. Conversation Flow and Gating Logic

### 3.1 Stage: INIT

**Gate:** Foundation directory exists with at minimum `00-project-identity.md`, `03-entity-map.md`, and `08-security-profile.md`. If `08-security-profile.md` is absent or placeholder, warn but allow:

> "No security profile found (or it's a placeholder). I can still design security, but all role and sensitivity decisions will require manual input. Consider running solution-discovery UPDATE mode to populate your security profile first."

**Action:**
1. Read foundation sections: `00-project-identity.md`, `01-requirements.md`, `02-architecture-decisions.md`, `03-entity-map.md`, `05-ui-plan.md` (if exists), `08-security-profile.md`
2. Read `docs/schema-physical-model.md` — if absent, block: "No physical model found. I need the entity and column inventory to build a privilege matrix. Run schema-design first."
3. Read `docs/ddd-model.md` — if present, note bounded context assignments for role scoping
4. Read `docs/ui-design-spec.md` — if present, extract persona-to-app-type mapping for role-entity correlation
5. Check for existing security artifacts in `docs/` — if found, offer re-entry
6. Load `.pp-context/skill-state.json` for session continuity

**Re-entry presentation (if artifacts found):**

> "I found existing security artifacts:
>
> - `docs/security-design.md` (last updated [date])
>
> How would you like to proceed?
> - **Continue** — pick up from where we left off
> - **Update** — revise the existing security design (e.g., add a new role, update privileges)
> - **Full re-run** — start fresh, diff against previous design"

### 3.2 Stage: ROLE_ANALYSIS

**Purpose:** Identify the security roles needed for the project and map each to a Dataverse security role pattern.

**Action:**
1. Read `08-security-profile.md` for declared personas and their access descriptions
2. Cross-reference with `05-ui-plan.md` persona-to-app mapping (if available)
3. Map each persona to a proposed security role with a privilege pattern

**Presentation:**

> "**Role Analysis for [Project Name]**
>
> From your security profile [and UI plan], I've identified these personas and their proposed security roles:
>
> | Persona | Proposed role name | Privilege pattern | Rationale |
> |---|---|---|---|
> | [persona] | [prefix]_[RoleName] | [Read-heavy / Full CRUD / Admin / View-only] | [why this pattern] |
>
> **Standard roles to include:**
> - **System Administrator** — built-in, not customized. Full access for admin tasks.
> - **Basic User** — baseline role all users receive. Minimal privileges on custom entities.
>
> **Naming convention:** `[publisher prefix]_[Role Name]` (e.g., `c3m_ProjectManager`)
>
> Questions:
> 1. Are there personas I've missed that need distinct security roles?
> 2. Should any personas share a single role? (e.g., two personas with identical access needs)
> 3. Any roles that need **impersonation** privileges (acting on behalf of another user)?
> 4. Any roles that need **custom entity ownership** (the ability to own records vs. just access them)?"

**Gate:** Developer confirms the role list. Each role has a name, privilege pattern, and at least one persona mapping.

### 3.3 Stage: PRIVILEGE_MATRIX

**Purpose:** Build the complete privilege matrix — every entity × every role × every privilege type.

**Action:**
1. Load entity list from `docs/schema-physical-model.md`
2. Determine table ownership type for each entity (User/Team-owned → all privilege levels available; Organization-owned → only Organization level)
3. For each role, propose privilege levels per entity based on the role's privilege pattern

**Privilege types (Dataverse standard):**

| Privilege | Code | Description |
|---|---|---|
| Create | cr | Create new records |
| Read | rd | Read existing records |
| Write | wr | Update existing records |
| Delete | dl | Delete existing records |
| Append | ap | Attach a record to this entity as a related record |
| Append To | apt | Allow other records to be attached to this entity |
| Assign | as | Change ownership of a record (User/Team-owned only) |
| Share | sh | Share a record with another user/team (User/Team-owned only) |

**Privilege levels (Dataverse standard):**

| Level | Scope | When to use |
|---|---|---|
| None | No access | Default — least privilege |
| User | Own records only | Standard for record creators |
| Business Unit | Own BU records | Managers who see their team's work |
| Parent: Child BU | BU + child BUs | Regional managers |
| Organization | All records | Administrators only (justify every use) |

**Presentation (per role):**

> "**Privilege Matrix: [Role Name]**
>
> | Entity | Ownership | Create | Read | Write | Delete | Append | AppendTo | Assign | Share |
> |---|---|---|---|---|---|---|---|---|---|
> | [entity] | User/Team | BU | BU | User | None | User | User | None | None |
> | [entity] | Organization | — | Org | — | — | — | — | — | — |
>
> **Key decisions for this role:**
> - [Entity]: Read at BU level because [reason — e.g., managers need to see all team records]
> - [Entity]: No Delete because [reason — e.g., audit trail requirements]
> - [Entity]: Organization-owned, so only Organization-level Read is available
>
> Does this privilege assignment match [persona]'s access needs?"

**Gate:** Developer confirms privilege matrix per role. All entities in the physical model are accounted for (even if the assignment is "None" across the board for a role that doesn't touch that entity).

**Round structure:** Present one role at a time. After each role is confirmed, present the next. After all custom roles are confirmed, present a summary matrix showing all roles side by side for cross-role review.

**Cross-role review presentation:**

> "**Cross-Role Summary**
>
> | Entity | [Role 1] | [Role 2] | [Role 3] | System Admin |
> |---|---|---|---|---|
> | [entity] | rd:BU, wr:User | rd:User | Full CRUD:Org | Full |
>
> **Privilege escalation check:** [Flag any entity where a non-admin role has Organization-level Write or Delete]
>
> Does this cross-role view look correct? Any privilege levels that seem too high or too low?"

### 3.4 Stage: FIELD_SECURITY

**Skip condition:** If `08-security-profile.md` does not flag any sensitive columns AND no columns in the physical model are marked as sensitive → skip with note:

> "No sensitive columns flagged in your security profile or physical model. Skipping field-level security. If you need to restrict access to specific columns later, you can re-run security in UPDATE mode."

**Purpose:** For columns flagged as sensitive, define field security profiles that control which roles can read, update, or create values in those columns.

**Action:**
1. Identify all columns flagged as sensitive in `08-security-profile.md` and `docs/schema-physical-model.md`
2. For each sensitive column, determine the appropriate field security profile

**Presentation:**

> "**Field-Level Security Design**
>
> Sensitive columns identified:
>
> | Entity | Column | Sensitivity reason | Current access |
> |---|---|---|---|
> | [entity] | [column] | [PII / financial / confidential] | All roles can read/write |
>
> **Proposed field security profiles:**
>
> | Profile name | Purpose | Columns included |
> |---|---|---|
> | [prefix]_SensitiveData | Restrict PII columns to authorized roles | [column list] |
> | [prefix]_FinancialData | Restrict financial columns to finance roles | [column list] |
>
> **Profile-to-role assignment:**
>
> | Profile | [Role 1] | [Role 2] | [Role 3] |
> |---|---|---|---|
> | [prefix]_SensitiveData | Read+Update | Read only | No access |
> | [prefix]_FinancialData | No access | Read+Update | No access |
>
> For each profile, roles not listed have **no access** to the secured columns.
>
> Does this FLS design match your data protection requirements?"

**Gate:** Developer confirms FLS profiles, column assignments, and role-to-profile mapping. Every sensitive column is covered by exactly one profile.

### 3.5 Stage: TEAM_HIERARCHY

**Skip condition:** If the project has no organizational hierarchy requirements (single BU, no team-based collaboration, no manager-reports access patterns) → skip with note:

> "Your project doesn't indicate team-based or hierarchy security requirements. Skipping. If your organization grows to need BU-scoped access or team ownership, re-run security in UPDATE mode."

**Purpose:** Design team structures and optionally hierarchy security for organization-structure-aware access patterns.

**Action:**
1. Determine if the project needs **owner teams** (teams that own records) vs. **access teams** (teams that share records) vs. both
2. Map organizational structure to Dataverse business unit and team configuration
3. If hierarchy security is needed, determine the model (manager hierarchy vs. position hierarchy)

**Presentation:**

> "**Team and Hierarchy Security Design**
>
> **Team types needed:**
>
> | Team type | Purpose | Scope |
> |---|---|---|
> | Owner team | [description — e.g., regional teams that own project records] | [BU-level] |
> | Access team | [description — e.g., ad-hoc collaboration on specific records] | [per-record] |
>
> **Proposed team structure:**
>
> | Team name | Type | Business unit | Security role(s) | Purpose |
> |---|---|---|---|---|
> | [team] | Owner / Access | [BU] | [role(s)] | [what this team does] |
>
> **Hierarchy security:**
> - Model: [Manager hierarchy / Position hierarchy / Not needed]
> - [If manager hierarchy:] Managers automatically see records owned by their direct reports. Depth: [N levels]
> - [If position hierarchy:] Positions define access independent of reporting structure.
>
> **Access team auto-creation:**
> - Entities with access team enabled: [list]
> - Users can be added to record-level access teams for shared collaboration
>
> Does this team and hierarchy design match your organizational structure?"

**Gate:** Developer confirms team types, team structure, hierarchy model (if any), and access team entity list.

### 3.6 Stage: REVIEW

**Action:** Dispatch the **security-reviewer** agent with:
- `docs/security-design.md` (draft written during prior stages)
- Foundation sections (00, 01, 03, 05, 08)
- `docs/schema-physical-model.md`
- `docs/ddd-model.md` (if available)

The agent returns a findings report with severity levels (HIGH/MEDIUM/LOW) and an overall assessment.

**Presentation:**

> "Running security review. Checking for over-privileged roles, FLS completeness, and team configuration alignment."

Present findings to the developer:

> "**Security Review Results**
>
> **Overall:** [PASS / PASS WITH NOTES / FAIL]
>
> **HIGH findings ([N]):**
> - [Finding]: [description] → [required resolution]
>
> **MEDIUM findings ([N]):**
> - [Finding]: [description] → [suggested resolution]
>
> **LOW findings ([N]):**
> - [Finding]: [description] → [optional improvement]
>
> HIGH findings must be resolved before proceeding. MEDIUM findings should be resolved or explicitly accepted with documented rationale."

**Gate:** All HIGH findings resolved. MEDIUM findings resolved or accepted with rationale. Developer confirms the review is complete.

### 3.7 Stage: COMPLETE

**Action:**
1. Write `docs/security-design.md` (final version incorporating all review fixes)
2. Update `.pp-context/skill-state.json` with completion state
3. Suggest next skill

**Presentation:**

> "Security design is complete. Here's what was produced:
>
> - **Security roles:** [N] custom roles defined
> - **Privilege matrix:** [N] entities × [N] roles — all privilege levels documented
> - **Field-level security:** [N] profiles covering [M] sensitive columns [or "Not applicable"]
> - **Team/hierarchy security:** [team summary] [or "Not applicable"]
>
> Output: `docs/security-design.md`
>
> **Suggested next:** alm-workflow — your security roles need to be packaged into the solution for deployment. Alternatively, environment-setup if you're ready to provision environments with these roles."

---

## 4. Output Specifications

### 4.1 Primary output

| Artifact | Path | When written |
|---|---|---|
| Security design document | `docs/security-design.md` | COMPLETE stage (draft updated throughout) |
| Skill state | `.pp-context/skill-state.json` | Every stage transition |

### 4.2 Security Design Document format

```markdown
# Security Design — [Project Name]

Generated by: security skill
Date: [date]
Physical model version: [date of schema-physical-model.md]

## Roles

### [Role Name]

| Attribute | Value |
|---|---|
| Logical name | [prefix]_[rolename] |
| Personas | [persona list] |
| Privilege pattern | [Read-heavy / Full CRUD / Admin / View-only] |
| Notes | [any special considerations] |

## Privilege Matrix

### [Role Name]

| Entity | Ownership | Cr | Rd | Wr | Dl | Ap | ApTo | As | Sh |
|---|---|---|---|---|---|---|---|---|---|
| [entity] | User/Team | BU | BU | User | None | User | User | None | None |

### Cross-Role Summary

| Entity | [Role 1] | [Role 2] | ... |
|---|---|---|---|
| [entity] | [summary] | [summary] | ... |

## Field-Level Security

### Profile: [Profile Name]

| Attribute | Value |
|---|---|
| Logical name | [prefix]_[profilename] |
| Purpose | [description] |

**Columns:**

| Entity | Column | Sensitivity | Read | Create | Update |
|---|---|---|---|---|---|
| [entity] | [column] | [PII/financial/etc.] | [role list] | [role list] | [role list] |

**Role assignments:**

| Role | Access level |
|---|---|
| [role] | Read + Update |
| [role] | Read only |

## Team Security

### [Team Name]

| Attribute | Value |
|---|---|
| Type | Owner / Access |
| Business unit | [BU] |
| Security roles | [role list] |
| Purpose | [description] |

## Hierarchy Security

| Attribute | Value |
|---|---|
| Model | Manager / Position / Not applicable |
| Depth | [N levels] |
| Notes | [configuration notes] |

## Access Teams

| Entity | Access team enabled | Purpose |
|---|---|---|
| [entity] | Yes / No | [why] |
```

---

## 5. Agent: security-reviewer

```markdown
# security-reviewer

## Role
Reviews security design for least-privilege compliance, privilege matrix completeness,
field-level security coverage, and team configuration alignment. Operates as a single-mode
reviewer dispatched at the security skill's REVIEW stage.

## Invoked by
security skill — REVIEW stage.

## Input context
- docs/security-design.md (the security design document to review)
- docs/schema-physical-model.md (entity and column inventory — source of truth)
- .foundation/08-security-profile.md (declared personas, sensitivity flags)
- .foundation/03-entity-map.md (entity inventory for coverage check)
- .foundation/05-ui-plan.md (persona-to-app mapping, if available)
- docs/ddd-model.md (bounded context assignments, if available)

## Evaluation Criteria (priority order)

### Least-Privilege Compliance (weight: HIGH)
- No custom role has Organization-level Write or Delete without documented justification
- No custom role has privileges that exceed what its persona description requires
- Every "None" assignment is intentional (entity is genuinely not relevant to the role)
- Assign and Share privileges are only granted where ownership transfer or collaboration is required
- No role duplicates System Administrator privileges (if it does, it should use System Administrator)

### Privilege Matrix Completeness (weight: HIGH)
- Every entity in the physical model appears in the privilege matrix
- Every custom role has a complete row for every entity (no missing cells)
- Organization-owned entities only show Organization-level or None (no User/BU levels)
- User/Team-owned entities have appropriate scope justifications for BU and above

### Field-Level Security Coverage (weight: MEDIUM)
- Every column flagged as sensitive in 08-security-profile has a corresponding FLS profile
- Every FLS profile has at least one role with access (no orphan profiles)
- No FLS profile grants broader access than the entity-level privilege matrix allows
- FLS is not applied to primary name columns (breaks usability in views and lookups)

### Team Configuration Alignment (weight: MEDIUM)
- Owner teams have security roles assigned
- Access teams are enabled only on entities that need per-record collaboration
- Team names follow naming conventions
- Business unit structure matches the organizational hierarchy described in security profile

### Naming Convention Compliance (weight: LOW)
- Role names follow [prefix]_[RoleName] convention
- FLS profile names follow [prefix]_[ProfileName] convention
- Team names are descriptive and consistent

## Output format
Return a structured review result:

**Overall:** PASS / PASS WITH NOTES / FAIL

**HIGH findings ([N]):**
- [Finding]: [description] → [required resolution]

**MEDIUM findings ([N]):**
- [Finding]: [description] → [suggested resolution]

**LOW findings ([N]):**
- [Finding]: [description] → [optional improvement]

**Summary:** [One paragraph describing the overall security posture and the most important
patterns to address]

## Does not
- Redesign the security model — it reviews and flags, the developer decides
- Modify the security design document — findings are presented for developer action
- Evaluate runtime security (that is environment-setup's domain)
- Check Power Platform licensing constraints (that is pp-devenv's domain)
- Test security roles against a live environment
```

---

## 6. Reference Material

### 6.1 Dataverse Privilege Types

| Privilege | What it controls | Notes |
|---|---|---|
| Create | Creating new records | Always paired with Append on the created entity if it has lookups |
| Read | Reading record data | Most common privilege — virtually all roles need Read on some entities |
| Write | Modifying existing records | Does not include ownership change (that is Assign) |
| Delete | Deleting records | Rarely needed for non-admin roles; consider soft-delete patterns instead |
| Append | Attaching a record of this entity to another entity | Required when setting lookups FROM this entity |
| Append To | Allowing other entities' records to be attached to this entity | Required when other entities have lookups TO this entity |
| Assign | Changing record ownership | Only applicable to User/Team-owned entities |
| Share | Sharing a record with other users/teams | Enables per-record access grants |

### 6.2 Dataverse Privilege Levels

| Level | Scope | Typical use case |
|---|---|---|
| None | No access | Default. Use for entities the role does not interact with. |
| User | Records owned by the user | Standard for record creators. Most restrictive useful level. |
| Business Unit | Records owned by anyone in the same BU | Managers, team leads who need visibility into team work. |
| Parent: Child BU | Records in the user's BU and all child BUs | Regional managers, directors overseeing multiple teams. |
| Organization | All records in the environment | Administrators. Justify every non-admin use. |

### 6.3 Table Ownership Types and Privilege Implications

| Ownership type | Available privilege levels | Assign/Share available? | Common for |
|---|---|---|---|
| User/Team | All five levels (None through Organization) | Yes | Business entities — projects, tasks, contacts, invoices |
| Organization | Only None or Organization | No | Reference data — currencies, settings, config entities |

**Key implication:** Organization-owned entities cannot have granular access control. Every user who has Read on an Organization-owned entity can read ALL records. This is appropriate for reference/configuration data but not for business data with access restrictions.

### 6.4 Field-Level Security Patterns

| Pattern | When to use | Example |
|---|---|---|
| PII protection | Columns containing personally identifiable information | SSN, date of birth, personal email, home address |
| Financial restriction | Columns containing financial data with limited audience | Salary, revenue, cost, margin |
| Confidential classification | Columns with data restricted by policy or regulation | Internal notes, classification level, compliance status |
| Staged visibility | Columns visible only after a process stage is reached | Approval decision visible only after review complete |

**FLS behavior:**
- Secured columns show as `*****` to users without Read access
- Secured columns are excluded from Quick Find search results for users without Read access
- Secured columns in views show the masked value, not blank
- FLS is enforced server-side — it cannot be bypassed by API access or plugins (plugins run in system context and bypass FLS unless explicitly coded to respect it)

### 6.5 Team Security Patterns

| Team type | When to use | Ownership | Configuration |
|---|---|---|---|
| Owner team | A group of users who collectively own records | Team owns records | Created in admin center, has security roles assigned |
| Access team | Ad-hoc collaboration on specific records | Users share records via team membership | Enabled per entity, auto-created per record, no security roles (inherits from member roles + share privileges) |
| Azure AD group team | Team membership managed by Entra ID (Azure AD) group | Same as Owner team | Requires Entra ID group; membership synced automatically |

### 6.6 Hierarchy Security Models

| Model | How it works | When to use |
|---|---|---|
| Manager hierarchy | Managers automatically see records owned by their direct reports, up to configurable depth | Standard org charts; most common pattern |
| Position hierarchy | Access is based on position in a hierarchy, independent of reporting structure | Matrix organizations, cross-functional access patterns |
| None | No hierarchy-based access | Flat organizations, or when BU-level privileges are sufficient |

**Manager hierarchy depth:** Default is 1 (direct reports only). Can be configured up to 7. Each level adds query overhead — keep depth minimal.

### 6.7 Security Anti-Patterns

| Anti-pattern | Problem | Resolution |
|---|---|---|
| Organization-level Write on all entities for non-admin roles | Violates least privilege; any user can modify any record | Scope Write to User or BU level; elevate only where justified |
| Copying System Administrator role and removing a few privileges | Creates a near-admin role that's hard to audit | Start from zero and add privileges incrementally |
| FLS on primary name columns | Breaks record display in views, lookups, and search results | Never secure the primary name column; secure other identifying columns instead |
| Granting Assign without business justification | Enables ownership transfer attacks (reassigning records to gain access) | Only grant Assign when ownership transfer is a documented workflow |
| Single role for all users | No access differentiation; violates least privilege | Create role per persona or access pattern |
| Access teams on every entity | Performance overhead for team membership queries | Enable only on entities with documented per-record collaboration needs |

---

## 7. Handoff Contract — security → downstream

### 7.1 What alm-workflow receives

alm-workflow reads `docs/security-design.md` to:
- Generate security role XML definitions for solution packaging
- Include FLS profile definitions in the solution
- Document team configurations for manual setup (teams are not solution-portable)

### 7.2 What environment-setup receives

environment-setup reads `docs/security-design.md` to:
- Create teams in target environments
- Assign security roles to teams and users
- Enable access teams on specified entities
- Configure hierarchy security model and depth
- Activate field-level security profiles

### 7.3 Minimum completeness for handoff

security is considered complete for downstream handoff when:
- REVIEW stage has been completed with all HIGH findings resolved
- `docs/security-design.md` is written with at minimum the Roles and Privilege Matrix sections
- `.pp-context/skill-state.json` shows security in completedSkills

### 7.4 Running downstream skills without security

alm-workflow and environment-setup can proceed without security if the developer explicitly chooses to skip security design. In this case:
- alm-workflow will not include custom security roles in the solution (default roles only)
- environment-setup will note: "No security design found. Default security roles will be used. Users will need manual role assignment."

---

## 8. Decision Log

| # | Decision | Rationale |
|---|---|---|
| 1 | Single-path skill, not a router | Security design follows a linear progression: roles → privileges → FLS → teams. There are no app-type-specific sub-skills. Skippable stages (FLS, Team/Hierarchy) handle optional patterns without needing a router. |
| 2 | Least privilege by default | Starting with zero and adding is the only defensible security posture. Starting with "Standard User" and removing creates blind spots. Every privilege must be justified. |
| 3 | Schema-driven completeness | The physical model is the definitive list of what exists. An entity missing from the privilege matrix is an unreviewed access decision — which is worse than an intentional "None" assignment. |
| 4 | TEAM_HIERARCHY combines team and hierarchy security | Team security and hierarchy security are closely related — both handle organizational structure. Splitting into two stages would create an awkward dependency (hierarchy references teams). Combined, the developer makes all org-structure decisions in one conversation. |
| 5 | FLS skippable if no sensitive columns flagged | Many projects have no FLS requirements. Forcing the stage when there's nothing to secure wastes time. The skip note reminds the developer they can return in UPDATE mode. |
| 6 | Security design document, not security role XML | The skill designs security; it does not implement it. XML generation is alm-workflow's responsibility. This keeps security focused on decisions and alm-workflow focused on packaging. |
| 7 | Cross-role summary after all roles confirmed | Reviewing roles individually can miss privilege escalation across roles. The cross-role summary forces a holistic view before FIELD_SECURITY. |
| 8 | Per-role privilege presentation, one role at a time | Presenting all roles at once in a matrix is overwhelming. Walking through one role at a time allows focused discussion of each persona's needs, then the cross-role summary catches inconsistencies. |
| 9 | security-reviewer agent over manual checklist | The review criteria are extensive enough to benefit from agent-level analysis. A manual checklist would be easy to skip or rush through. The agent enforces thoroughness. |

---

## 9. Open Items for Build

- **08-security-profile format:** The foundation section format should define a standard structure for persona declarations and sensitivity flags so security can reliably parse them. Currently, the format is flexible — security should handle both structured and freeform security profiles gracefully.
- **Multi-solution security roles:** If the project uses multiple solutions (from solution-strategy), security roles may need to be split across solutions. Define how the privilege matrix handles cross-solution entities (entities in solution A referenced by roles in solution B).
- **Privilege matrix presentation for large entity counts:** Projects with 20+ entities will produce very large privilege matrices. Consider a grouped presentation (by bounded context or aggregate) for large projects, with the option to expand to full detail.
- **Custom privilege mapping:** Dataverse supports custom privileges (e.g., "prvApproveProject"). Define whether the security skill should propose custom privileges or limit itself to standard CRUD + Append + Share privileges.
- **Column-level security on calculated/rollup columns:** Verify whether FLS can be applied to calculated and rollup columns in Dataverse. If not, the FLS stage should exclude them from candidate lists.
- **Business unit design:** The skill assumes business units are already defined. If the project needs custom business units, this should be flagged as an environment-setup prerequisite rather than designed within the security skill.
