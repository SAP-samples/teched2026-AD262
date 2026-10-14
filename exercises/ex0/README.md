# Exercise 0 - Initialize and Generate App

In this exercise, you will initialize a CAP project and use Claude to generate a complete invoice matching application for Maria, an AP (Accounts Payable) clerk.

## Learning Objectives

By the end of this exercise, you will:
- Initialize a CAP project with Node.js and SQLite
- Use Claude with the [CAP Skills library](https://github.com/capire/skills) to generate a complete data model
- Understand workflow state transitions
- Have realistic sample data ready to use

## What You Will Build

An intelligent invoice matching system where Maria can:
- View incoming supplier invoices
- See open purchase orders
- Record match decisions linking invoices to POs
- Track confidence, rationale, and outcome for each decision
- Follow a workflow with valid state transitions

## Exercise 0.1 - Initialize CAP Project

First, let's create the basic project structure.

1. Open your terminal and navigate to your working directory.

2. Initialize a new Node.js CAP project:
```bash
cds init invoice-matching
```

This creates a new CAP project with folders for database (`db/`), services (`srv/`), and app (`app/`).

3. Navigate into the project:
```bash
cd invoice-matching
```

4. Add Node.js runtime:
```bash
cds add nodejs
```

5. Add SQLite database:
```bash
cds add sqlite
```

This configures your project for local development with Node.js and SQLite.

6. Verify the setup, split your terminal and run:
```bash
cds watch
```

You should see the CAP server starting. With `Ctrl+C` you can stop the server whenever you want.

<br>![Empty preview](/exercises/ex0/images/00_00_0010.png)

## Exercise 0.2 - Generate the Application with Claude

Now comes the exciting part - we'll use Claude to generate the complete application!

> 💡 **AI-Assisted Development**: Instead of manually writing hundreds of lines of code, you'll use a single Claude prompt to generate everything.

Start Claude Code by entering claude in the terminal, which is not running the preview. 

```bash
claude
```

### Plugin: Claude Code
In this exercise we are using the [CAP SKills library](https://github.com/capire/skills). A shared library of reusable skills, agents, and prompts for the team. Compatible with OpenCode, Claude Code, and other AI coding tools that follow the open standards.

Inside Claude Code, add the plugin:

```bash
/plugin marketplace add capire/skills
```

And install it. While installing select "project scope":
```bash
/plugin install cap-developer@cap
```

### The Claude Prompt

Copy this exact prompt into Claude:

```
/cap-developer Build a CAP app for Maria, an AP clerk who matches incoming supplier invoices to open purchase orders before releasing payment.

She needs to see her invoices and the open POs side by side. Each invoice has a supplier, an amount with currency, a short description, and a status (open, matched, or escalated). She can also see which PO it's been matched to, if any.

Purchase orders have a supplier, a line item description, an amount, and a flag for whether they're still open.

When Maria decides on a match she records it as a match decision – linking the invoice and the PO, noting her confidence, a rationale, and the outcome (proposed, approved, rejected, or escalated). The system should track who made the decision and when.

The decision workflow should enforce valid transitions: a proposed match can be approved, rejected, or escalated; once approved it can only be escalated; rejected and escalated are final.

Generate realistic sample data. Use SQLite for local dev. Ensure to use cap status-flows https://cap.cloud.sap/docs/guides/services/status-flows
```

Simply approve all changes. That's it. Do *not* switch on auto approve that you can follow the changes it does.

### What Claude Will Generate

Claude should generate:

1. **Data Model** (`db/schema.cds`):
   - `Invoices` entity with supplier, amount, description, status
   - `PurchaseOrders` entity with supplier, amount, description, isOpen flag
   - `MatchDecisions` entity linking invoices to POs with confidence, rationale, outcome

2. **Service Definition** (`srv/matching-service.cds`):
   - Service exposing all entities
   - Actions for approve, reject, escalate operations

3. **Service Implementation** (`srv/matching-service.js`):
   - Workflow state transition logic
   - Business rules enforcement

4. **Sample Data** (`db/data/` folder):
   - CSV files with realistic invoices
   - CSV files with matching purchase orders
   - Test data for development

### Review the Generated Code

Before saving, review what Claude generated:

**Check the Data Model:**
- ✅ Does it have Invoices, PurchaseOrders, and MatchDecisions?
- ✅ Are there proper associations between entities?
- ✅ Does it include managed aspects (createdAt, modifiedAt, etc.)?

**Check the Service:**
- ✅ Are all entities exposed?
- ✅ Are there actions for approve, reject, escalate?
- ✅ Is the service implementation included?

**Check the Sample Data:**
- ✅ Are there CSV files for each entity?
- ✅ Do the descriptions vary (to test semantic search later)?
- ✅ Are amounts similar but not identical?

### Save the Generated Code

1. Create the files as Claude suggests (usually):
   - `db/schema.cds`
   - `srv/matching-service.cds`
   - `srv/matching-service.js`
   - `db/data/` CSV files

2. Save each file in your project

## Exercise 0.3 - Preview and Test

This creates the SQLite database and loads the sample data.

1. If you ended the preview simply start the CAP server again. If it is still running in the other terminal skip this step.
```bash
cds watch
```

2. Open your browser at `http://localhost:4004`

You should see:
- Welcome page with the service endpoints
- Link to the matching service
- Sample data loaded

3. Click on the matching service to explore:
- Browse Invoices - you should see sample invoices
- Browse PurchaseOrders - you should see sample POs
- Browse MatchDecisions - should be empty initially

<br>![CAP Welcome Page](/exercises/ex0/images/00_00_0020.png)

## Exercise 0.4 - Understand the Data Model

Let's examine what was generated in more detail.

### Invoices Entity

Each invoice represents a bill from a supplier:
- **supplier**: Name of the supplier company
- **amount**: Total invoice amount
- **currency**: Currency code (EUR, USD, etc.)
- **description**: What was purchased
- **status**: Current state (open, matched, escalated)
- **matchedPO**: Link to the matched purchase order (if any)

### PurchaseOrders Entity

Each PO represents an approved purchase:
- **supplier**: Name of the supplier company
- **amount**: Total PO amount
- **description**: What was ordered
- **isOpen**: Whether this PO can still be matched

### MatchDecisions Entity

Each decision records Maria's matching work:
- **invoice**: Link to the invoice
- **purchaseOrder**: Link to the PO
- **confidence**: How confident (0.0 to 1.0)
- **rationale**: Why this match makes sense
- **outcome**: Current state (proposed, approved, rejected, escalated)

### State Transitions

The workflow enforces valid transitions. You can find the implementation under srv/inoice-matching.cds
```
proposed → approved ✓
proposed → rejected ✓
proposed → escalated ✓
approved → escalated ✓
rejected → [final state] ✗
escalated → [final state] ✗
```

## Exercise 0.5 - Test Sample Data

Let's verify the sample data is realistic and useful. If your service is named differently than invoice-matching, replace the name in the url.

1. In your browser, navigate to `http://localhost:4004/odata/v4/invoice-matching/Invoices`

2. Check that invoices have:
   - Different suppliers
   - Varying amounts
   - Descriptive text (not just "Invoice 1")
   - All status = "open" (nothing matched yet)

3. Navigate to `http://localhost:4004/odata/v4/invoice-matching/PurchaseOrders`

4. Check that POs have:
   - Same suppliers as invoices (some overlap)
   - Similar but not identical amounts
   - Descriptions that are semantically similar to invoice descriptions
   - All isOpen = true

**Why similar but not identical?**
- Real-world data is messy!
- Amounts differ due to taxes, discounts, partial deliveries
- Descriptions use different words for the same things
- This is why we need semantic search (coming in Exercise 4)

## What You've Accomplished

✅ Initialized a CAP project with Node.js and SQLite  
✅ Used Claude to generate a complete data model  
✅ Created service definitions and implementations  
✅ Generated realistic sample data  
✅ Deployed and tested the application  
✅ Understood the workflow state transitions

## Understanding What Happened

**Traditional Approach:**
- 30-45 minutes of typing
- Manual CSV creation
- Prone to typos and errors

**AI-Assisted Approach:**
- 5 minutes with Claude
- Generated in seconds
- Consistent and complete

**What You Learned:**
- How to describe business requirements to AI
- How to review AI-generated code
- The structure of a CAP application

## Troubleshooting

### Issue: `cds` command not found
**Solution:** Install CAP Development Kit:
```bash
npm install -g @sap/cds-dk
```

### Issue: Claude generated incomplete code
**Solution:** Ask Claude to complete specific parts:
```
Can you complete the service implementation with the workflow logic?
```

### Issue: Sample data doesn't load
**Solution:** Check CSV files have correct headers matching entity names:
```bash
cds compile db/schema.cds --to sql
```

### Issue: Syntax errors in generated code
**Solution:** Ask Claude to fix:
```
I got this error: [paste error]. Please fix the code.
```

## Next Steps

Now that you have a working CAP application with data, you're ready to build the user interface!

**Continue to [Exercise 1 - Build React UI](../ex1/README.md)** to create Maria's matching interface.

---

## Summary

In this exercise, you:
- ✅ Created a CAP project structure
- ✅ Used AI including CAP Skills Library to generate a complete application
- ✅ Learned about invoice matching workflows
- ✅ Deployed sample data
- ✅ Understood state transitions

**Time spent:** ~20 minutes  
**Lines of code written:** ~0 (AI did it all!)  
**Lines of code reviewed:** ~200-300

Next up: [Exercise 1 - Build React UI](../ex1/README.md) in Exercise 1! 🚀
