---
name: csharp-plugin
description: Dispatched by the business-logic router — designs and scaffolds C# plugin
  classes registered on Dataverse entity messages. Development-heavy with code generation
  and plugin-auditor agent review. Not directly invokable; use business-logic to access.
---

# C# Plugin Design

Designs and scaffolds C# plugin classes registered on Dataverse entity messages. Development-heavy — produces plugin class code, registration specifications, and test stubs. The plugin-auditor agent runs a comprehensive audit during REVIEW.

## State Machine

```
INIT → DISCOVERY → DESIGN → SCAFFOLD → IMPLEMENT → TEST → REVIEW → COMPLETE
```

| Stage | Purpose | Can skip? |
|---|---|---|
| INIT | Load logic map plugin requirements, physical model, domain events | No |
| DISCOVERY | For each requirement: entity, message, stage, execution context | No |
| DESIGN | Plugin class structure, IOrganizationService usage, error handling | No |
| SCAFFOLD | Generate plugin class stubs, .csproj references, registration spec | No |
| IMPLEMENT | Implementation guidance per plugin class | No |
| TEST | Generate unit test stubs (FakeXrmEasy or mock pattern) | No |
| REVIEW | Dispatch plugin-auditor agent: full audit (1-100 score) | No |
| COMPLETE | Write `docs/bl-plugin-[name].md`, update inventory | No |

---

## INIT

**Action:** Read `06-logic-map.md` plugin requirements. Read `docs/schema-physical-model.md` for entity relationships. Read `docs/ddd-model.md` for domain events if available.

**Presentation:**

> "Starting csharp-plugin design for **[Project Name]**. From your logic map, I see [N] plugin requirements:
>
> | Requirement | Entity | Trigger | Type |
> |---|---|---|---|
> | [name] | [entity] | [Create/Update/Delete] | [Validation/Calculation/etc.] |
>
> [If DDD model available:]
> Domain events that map to plugin triggers:
> | Domain event | Suggested trigger |
> |---|---|
> | [event] | [entity] [message] [stage] |
>
> Any requirements I've missed?"

**Gate:** Developer confirms the complete plugin requirement list.

---

## DISCOVERY

**Action:** For each requirement, determine the five registration parameters.

**Presentation (per plugin):**

> "**Plugin: [Requirement Name]**
>
> 1. **Entity:** [suggest from logic map]
> 2. **Message:** Create / Update / Delete / Retrieve / RetrieveMultiple / Associate / Custom?
> 3. **Stage:** Pre-Validation (10) / Pre-Operation (20) / Post-Operation (40)?
> 4. **Execution mode:** Synchronous / Asynchronous? (async for Post-Operation only)
> 5. **Filtering attributes (Update only):** Which columns trigger this?
>
> Registration: `[entity] [message] [stage] [sync/async]`"

**Gate:** Developer confirms registration per plugin.

---

## DESIGN

**Presentation (per plugin):**

> "**[Plugin Name] — Class Design**
>
> ```
> [PluginName] : IPlugin
> ├── Execute(IServiceProvider)
> │   ├── Extract context (IPluginExecutionContext)
> │   ├── Get service factory + service
> │   ├── Get tracing service
> │   └── [Core logic method]
> ```
>
> **Error handling:** InvalidPluginExecutionException for validation failures; wrap all others with tracing.
>
> **IOrganizationService usage:** [list of entity operations]
>
> **Performance notes:** [N+1 risks, bounded queries]
>
> Does this match the requirement?"

**Gate:** Developer confirms class design.

---

## SCAFFOLD

**Action:** Generate the plugin class stub and registration specification.

**Presentation:**

> "**Scaffold: [Plugin Name]**
>
> ```csharp
> using Microsoft.Xrm.Sdk;
> using System;
>
> namespace [Namespace].Plugins
> {
>     public class [PluginName] : IPlugin
>     {
>         public void Execute(IServiceProvider serviceProvider)
>         {
>             var context = (IPluginExecutionContext)
>                 serviceProvider.GetService(typeof(IPluginExecutionContext));
>             var serviceFactory = (IOrganizationServiceFactory)
>                 serviceProvider.GetService(typeof(IOrganizationServiceFactory));
>             var service = serviceFactory.CreateOrganizationService(context.UserId);
>             var tracingService = (ITracingService)
>                 serviceProvider.GetService(typeof(ITracingService));
>
>             try
>             {
>                 // Core logic — to be implemented
>             }
>             catch (InvalidPluginExecutionException) { throw; }
>             catch (Exception ex)
>             {
>                 tracingService.Trace($"[PluginName] error: {ex.Message}");
>                 throw new InvalidPluginExecutionException(ex.Message, ex);
>             }
>         }
>     }
> }
> ```
>
> **Registration spec:**
> | Field | Value |
> |---|---|
> | Entity | [logical name] |
> | Message | [message] |
> | Stage | [10/20/40] |
> | Mode | [0=sync / 1=async] |
> | Filtering attributes | [comma-separated or empty] |
> | Rank | 1000 |
> | Isolation mode | Sandbox |"

**Gate:** Developer confirms scaffold.

---

## IMPLEMENT

**Presentation (per plugin):**

> "**[Plugin Name] — Implementation Guidance**
>
> **Pre-image / Post-image:** [needed or not, with reason]
>
> **Target access:**
> ```csharp
> var target = context.InputParameters.Contains("Target")
>     ? context.InputParameters["Target"] as Entity : null;
> ```
>
> **Core logic pattern:** [step-by-step implementation notes]
>
> **Performance notes:** [N+1 resolutions, pagination]"

---

## TEST

**Action:** Generate unit test stubs.

**Presentation:**

> "**Test Stubs: [Plugin Name]**
>
> ```csharp
> using FakeXrmEasy;
> using Microsoft.Xrm.Sdk;
> using Xunit;
>
> public class [PluginName]Tests
> {
>     private readonly XrmFakedContext _context = new();
>
>     [Fact]
>     public void [PluginName]_[Scenario]_[Expected]()
>     {
>         var target = new Entity("[entitylogicalname]") { /* fields */ };
>         _context.ExecutePluginWithTarget<[PluginName]>(target);
>         // Assert expected state changes
>     }
> }
> ```
>
> Test scenarios: [happy path + failure path list]"

---

## REVIEW

<EXTREMELY-IMPORTANT>
**You MUST dispatch the plugin-auditor agent at this stage.**
Do NOT substitute a manual review or skip the agent dispatch.
Provide the agent with all plugin class source code, registration specs, and the physical model path.
Wait for the agent to return its scored audit report before presenting results.
</EXTREMELY-IMPORTANT>

**Presentation:**

> "Running plugin audit on [N] plugin classes. This covers security, performance, schema correctness, and architecture."

The plugin-auditor returns a scored report (1-100). **A score below 70 blocks COMPLETE** — all HIGH findings must be resolved.

**Gate:** All HIGH findings resolved or accepted with documented rationale.

---

## COMPLETE

Write `docs/bl-plugin-[name].md` per plugin class. Update `docs/business-logic-inventory.md`. Return to router.
