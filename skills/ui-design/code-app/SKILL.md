---
name: code-app
description: Dispatched by the ui-design router — designs a standalone React/Vite/TypeScript
  application hosted on Power Platform with Power Apps SDK integration. Not directly
  invokable; use ui-design to access this sub-skill.
---

# Code App Design

Designs a standalone React/Vite/TypeScript application hosted on Power Platform. Requirement-driven — not tied to MDA patterns. Development-heavy. Uses PAC CLI `code` commands for scaffold and deployment. Produces a component specification, scaffold command set, and implementation guidance.

## State Machine

```
INIT → BRAINSTORM → COMPONENT_DESIGN → SCAFFOLD → IMPLEMENT → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Confirm app purpose, licensing prerequisites, environment requirements | No |
| BRAINSTORM | User flows, data sources, authentication context, app type fit check | No |
| COMPONENT_DESIGN | React component tree, state management, routing, UI library | No |
| SCAFFOLD | PAC CLI init + data source commands, project structure | No |
| IMPLEMENT | Implementation guidance per component, SDK patterns, mockup | No |
| REVIEW | Dispatch ui-reviewer in sub-skill mode | No |
| COMPLETE | Write `docs/ui-codeapp-design.md` | No |

---

## INIT

**Presentation:**

> "Starting code-app design. Power Apps Code Apps are standalone React/Vite/TypeScript applications that run inside the Power Platform shell with native Dataverse access via the Power Apps SDK.
>
> **Prerequisites check:**
> - Power Apps Premium license with Code Apps enabled ✓/✗
> - PAC CLI version 1.50.1+ (stable data source generation) ✓/✗
> - Node.js in PATH ✓/✗
>
> **Fit check:** A Code App is right when:
> - You need full DOM/layout/interaction control
> - Canvas, MDA, or custom-page constraints won't work
> - You want the full React ecosystem
>
> Does your requirement justify a Code App, or would a simpler app type suffice?"

**Gate:** Developer confirms Code App is the right choice.

---

## BRAINSTORM

**Presentation:**

> "**Code App — Brainstorm**
>
> 1. **What does this app do?** (2-3 sentences from user's perspective)
> 2. **Who uses it?** (personas from UI plan or new)
> 3. **What Dataverse data does it read and write?** (tables from physical model)
> 4. **Any connectors beyond Dataverse?** (Office 365 Users, SharePoint, custom)
> 5. **Any real-time requirements?** (live updates, polling, push notifications)
> 6. **Embedded in MDA navigation, or standalone URL?**"

**Gate:** Developer confirms brainstorm output.

---

## COMPONENT_DESIGN

**Presentation:**

> "**React Component Tree**
>
> ```
> App
> ├── PowerProvider          <- SDK initialization (required)
> ├── AppShell
> │   ├── Header
> │   ├── SideNav (if applicable)
> │   └── MainContent
> │       ├── [Page components]
> │       └── [Shared components]
> └── [Modal/Dialog layer]
> ```
>
> **Routing:** [React Router / single-page — justified by complexity]
>
> **State management:**
> | State type | Approach | Reason |
> |---|---|---|
> | Server state | Generated services from pac code add-data-source | Type-safe, maintained by PAC CLI |
> | UI state | useState / useReducer | Local interaction state |
> | Global state | [Zustand if needed / none] | Only if 3+ components share state |
>
> **UI library:** Fluent UI React v9 (matches Power Platform chrome; justify alternatives)
>
> Does this structure match your app's complexity?"

**Gate:** Developer confirms component design.

---

## SCAFFOLD

**Presentation:**

> "**Scaffold Commands**
>
> ```bash
> npm create vite@latest [app-name] -- --template react-ts
> cd [app-name]
> npm install @microsoft/power-apps @fluentui/react-components
> pac code init --displayName "[Display Name]" --description "[Description]"
> pac code add-data-source -d dataverse -t [tablename]
> ```
>
> **Generated files:**
> - `power.config.json` — app config (do not edit manually)
> - `src/generated/[Table]Service.ts` — type-safe Dataverse services (do not edit, regenerate via PAC CLI)
>
> Confirm project name and table list."

**Gate:** Developer confirms scaffold parameters.

---

## IMPLEMENT

**Presentation:**

> "**SDK Initialization (required):**
> ```typescript
> import { initialize } from '@microsoft/power-apps/app';
> export function PowerProvider({ children }) {
>   const [ready, setReady] = useState(false);
>   useEffect(() => { initialize().then(() => setReady(true)); }, []);
>   if (!ready) return <LoadingSpinner />;
>   return <>{children}</>;
> }
> ```
>
> **Per-component guidance:**
> | Component | Key pattern | Dataverse service |
> |---|---|---|
> | [Component] | [pattern] | [service method] |"

Generate Canva mockup if available, otherwise describe the UI layout textually.

---

## REVIEW

**Spec compliance:** All tables have add-data-source commands, SDK initialization wraps app root, generated services referenced not replicated.

**Quality:** `power.config.json` not manually edited, generated files not modified, state management proportional to complexity, Fluent UI for platform consistency.

**Gate:** Developer confirms review results.

---

## COMPLETE

Write `docs/ui-codeapp-design.md`. Contribute Code App events (Dataverse operations needing server-side logic) to form event map. Return to router.
