# Exercise 7 - Add AI-Powered Recommendations with @cap-js/ai (Optional)

This is an **optional advanced exercise** that integrates the `@cap-js/ai` plugin to provide intelligent field recommendations and AI Core insights in your invoice matching system.

## Overview

**Duration:** ~15 minutes  
**Prerequisites:** Completed Exercises 0-5  
**Difficulty:** Intermediate

In this exercise, you will use the `@cap-js/ai` CDS plugin to automatically add SAP-RPT-1 recommendations to your invoice entity. The plugin requires almost no code — just annotations and an optional service binding.

## Learning Objectives

By the end of this exercise, you will:
- Install and configure the `@cap-js/ai` plugin
- Annotate a draft-enabled entity for automatic field recommendations
- Understand how CAP plugins extend the framework declaratively
- Test AI recommendations locally without any service credentials
- (Optional) Connect to a real SAP AI Core instance for production-quality recommendations

## What is `@cap-js/ai`?

**`@cap-js/ai`** is a CDS plugin that brings AI capabilities directly into CAP's model-driven paradigm:

- **UI Recommendations** — adds SAP-RPT-1 powered suggestions to draft entity fields automatically, using existing value help annotations as context
- **SAP AI Core Service** — exposes resource groups, deployments, and configurations as a standard CAP service you query with CQL
- **Local Vector Embeddings** — experimental support for semantic search via SQLite and ONNX models (no external service needed for development)

The key insight: **no custom service code required**. The plugin reads your CDS model, finds draft-enabled entities with value-help fields, and wires up recommendations automatically.

## Prerequisites

### Local Development (No Credentials Needed)

The plugin ships with a mock implementation that returns the first non-null value for each recommendation field. This is enough to verify the full UI and data flow during development.

### Production

You need:
- SAP BTP account with SAP AI Core entitlement
- AI Core service instance (Cloud Foundry managed service)
- No additional env vars — binding the service is all that is required

## Exercise 7.1 - Install the Plugin

1. Make sure you are in the project root:

```bash
pwd
# Should show: .../invoice-matching
```

2. Install `@cap-js/ai`:

```bash
npm add @cap-js/ai
```

That is all the installation required. The plugin activates automatically as a CDS plugin — no `require` needed in your service code.

3. Restart the CAP server:

```bash
cds watch
```

The startup log should show a line like `[cds] - loaded plugin @cap-js/ai`.

## Exercise 7.2 - Make the Invoice Entity Draft-Enabled

Recommendations only work on draft-enabled entities. Open your invoice service definition and check whether drafts are already enabled.

Open `srv/invoice-service.cds` and look for `@odata.draft.enabled`. If it is missing on the `Invoices` entity, add it:

```cds
service InvoiceService {

  @odata.draft.enabled
  entity Invoices as projection on invoice.Invoices;

  // ... rest of your service
}
```

## Exercise 7.3 - Annotate Fields for Recommendations

The plugin automatically includes any field that has `@Common.ValueList` or whose association target has `@cds.odata.valuelist`. You opt fields in or out with `@UI.RecommendationState`.

Open `db/schema.cds` (or wherever your Invoices entity is defined) and add the following annotations:

```cds
using { InvoiceService } from '../srv/invoice-service';

// Fields with value helps are included automatically.
// Use @UI.RecommendationState to control individual fields:
//   0 = excluded
//   1 = included (default for eligible fields)

annotate InvoiceService.Invoices with {
  // Include the supplier field for SAP-RPT-1 recommendations
  supplier     @UI.RecommendationState;

  // Exclude status — controlled by business rules, not AI
  status       @UI.RecommendationState: 0;
}
```

> **How it works:** When Fiori Elements opens a draft for editing, it reads the `SAP_Recommendations` expand. The plugin intercepts that read, sends the last 2,000 active rows of the same entity as context to SAP-RPT-1, and returns ranked field-value suggestions. The top suggestion appears as a soft-fill default in the form.

## Exercise 7.4 - Verify the Generated Service Shape

The plugin extends your service automatically. You can inspect what it added with:

```bash
cds compile srv/ --to json | grep -A 20 "SAP_Recommendations"
```

You should see a generated `<Entity>_Recommendations` companion entity and a `@UI.Recommendations` annotation pointing to it. This is all handled by the plugin — no hand-written service code.

## Exercise 7.5 - Test Locally with Mock Recommendations

1. Start the server if it is not already running:

```bash
cds watch
```

2. Open the Fiori Elements UI for invoices (typically at `http://localhost:4004`).

3. Create or open a draft invoice.

4. The `supplier` field should now show a recommendation chip above the input. In local development the mock returns the first non-null value from existing rows, so you will see a real value from your seed data.

> **Expected behavior (mock):** The first non-null `supplier` value from active rows appears as a suggestion. Accept it by clicking the chip or ignore it — the draft activates normally either way.

<br>![AI Recommendations](/exercises/ex7/images/07_05_0010.png)


## Exercise 7.7 - (Optional) Connect to Real SAP AI Core

To switch from mock to real recommendations, bind your AI Core service instance:

```bash
cds bind <your-aicore-service-instance>
```

Then start the server in hybrid mode:

```bash
cds watch --profile hybrid
```

The first recommendation request for a resource group will:
1. Check whether an `sap-rpt-1-small` deployment already exists
2. Create and start one if not (may take ~30 seconds)
3. Use it for all subsequent recommendations in that resource group

No code changes are needed — the same annotations drive both local mock and production SAP-RPT-1.

## What You've Accomplished

✅ Installed `@cap-js/ai` as a zero-config CDS plugin  
✅ Enabled draft mode on the Invoices entity  
✅ Annotated fields for automatic SAP-RPT-1 recommendations  
✅ Verified recommendations work locally without credentials  
✅ Understood how to connect to real SAP AI Core for production

## Understanding What Happened

**Traditional approach:**
- Write a custom recommendation service
- Manage OAuth token lifecycle manually
- Parse LLM responses into structured data
- Build UI components to display suggestions

**`@cap-js/ai` approach:**
- Install one package
- Add `@odata.draft.enabled` and `@UI.RecommendationState` annotations
- The plugin handles service generation, AI Core communication, token lifecycle, and Fiori Elements integration

**Why this matters:**
- Less code = fewer bugs and less maintenance
- Recommendations follow the same draft lifecycle as your data
- Works identically on SAP HANA and SQLite
- Same model in local dev and production — only the binding changes

## Troubleshooting

### Plugin not loading

Check that `@cap-js/ai` is in `dependencies` (not `devDependencies`) in `package.json`, then restart:

```bash
cat package.json | grep cap-js/ai
cds watch
```

### No recommendation chip visible

Confirm the entity has `@odata.draft.enabled`. Recommendations only appear during draft editing, not on active entity reads.

### Mock returns no suggestion

The mock uses the first non-null value from active rows. If the database has no data with a non-null `supplier`, seed some data first:

```bash
cds deploy --to sqlite
```

### Production: deployment creation fails

Check that your AI Core instance has the `foundation-models` scenario enabled and that the service binding includes `resourceGroup`. The plugin defaults to `default`:

```json
{
  "cds": {
    "requires": {
      "AICore": {
        "resourceGroup": "your-resource-group"
      }
    }
  }
}
```

## Summary

In this optional exercise, you:
- ✅ Used `@cap-js/ai` to add AI recommendations with minimal code
- ✅ Leveraged CDS annotations instead of imperative service logic
- ✅ Understood the mock → production pathway via service binding

**Time spent:** ~15 minutes  
**AI Service:** SAP AI Core (via `@cap-js/ai` plugin)  
**Model:** SAP-RPT-1 (managed by the plugin)  
**Business Value:** Intelligent field suggestions with zero custom AI integration code

---

**Workshop Complete!** 

You have now completed ALL exercises (core + optional):
- ✅ Exercises 0-5: Core invoice matching system
- ✅ Exercise 6: Audit logging and compliance
- ✅ Exercise 7: AI-powered recommendations via `@cap-js/ai`

**What You Built:**  
A complete, production-ready AI-powered invoice matching system with a React UI, S/4HANA integration, MCP server, semantic search, an intelligent agent, audit logging, and automatic AI field recommendations.

**Congratulations!**

---

## Notes for Instructors

This exercise is **optional** because:
- Participants need SAP BTP and AI Core access for production behavior
- Local mock is sufficient to demonstrate the pattern
- More meaningful after participants understand draft lifecycle from earlier exercises

**When to include:**
- Extended workshops (2+ hours total)
- Participants with BTP access who want to see real SAP-RPT-1 output
- Sessions focused on SAP AI Core capabilities and BTP integration

**When to skip:**
- Limited workshop time
- Focus on MCP, agents, or semantic search (covered in earlier exercises)
- Groups without BTP access (mock still works but is less impressive)

**Key talking points:**
- The plugin is a first-class example of CAP's model-driven philosophy applied to AI
- Zero-code AI integration = lower risk and faster development
- Service binding is the only difference between local mock and production
