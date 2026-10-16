# Exercise 2 — Add Supplier Data

In this exercise, you will import the S/4HANA Supplier data product and integrate supplier information into Maria's UI so she can spot problems before releasing payment.

## Learning Objectives

By the end of this exercise, you will:
- Import an SAP data product into CAP
- Integrate external S/4HANA data with your application
- Display supplier details including payment terms and blocking status
- Match suppliers by name
- Understand data product patterns

## What You Will Build

Enhanced invoice matching with supplier insights:
- **Supplier Name:** Who we're paying
- **Payment Terms:** When payment is due (e.g., "NET30")
- **Posting Block:** Whether supplier is blocked
- **Block Reason:** Why they're blocked (if applicable)

This helps Maria identify issues like blocked suppliers, unusual payment terms, and verified suppliers ready for payment.

## Exercise 2.1 — Import the Supplier Data Product

SAP provides pre-built data products that you can import into CAP.

👉 Copy the [`sap-s4com-Supplier-v1.json`](../../sap-s4com-Supplier-v1.json) file from the repository root to your project root.

👉 Make sure you're in the project root in your terminal:
```bash
pwd
# Should show: .../invoice-matching
```

👉 Import the data product:
```bash
cds import --data-product sap-s4com-Supplier-v1.json
```

This command analyzes the data product schema, creates CDS definitions in `apis/imported/`, sets up service references, and configures API metadata.

👉 Check what was imported:
```bash
ls apis/imported/
```

You should see a folder for the Business Partner API (Supplier is part of this).


## Exercise 2.2 — Add Supplier Details with Claude

Now we'll enhance the application to show supplier information.

> [!TIP]
> Claude will integrate the supplier data product and update the UI in one prompt.

### The Claude Prompt

👉 Copy this exact prompt into Claude:

```text
/cap-add-remote-service  The Supplier data product is imported under apis/imported/. For each invoice Maria reviews,
I want to see key details about that supplier alongside it, so she can spot problems before releasing payment.

Add a supplier details section to each invoice showing:
- the supplier's name
- their payment terms
- whether they're blocked for posting
- the payment blocking reason, if any

Pull the name from the Supplier record and the payment/blocking fields from its company-code record.
A few things to keep in mind:

Invoices only carry the supplier's name today, not an ID, so match suppliers by name for now.
Ensure to update the schema.cds for that.

Give me realistic mock supplier data that lines up with the suppliers already in the invoices,
and make sure at least one supplier is blocked so I can see how that looks.
```

### What Claude Will Generate

Claude should:

1. **Extend the Data Model**:
   - A new cds file with a supplier consumption view will be created — a projection that maps the imported definitions to your domain model, renaming and flattening as needed
   - should be under apis/imported - supplier.cds

2. **Update Service Implementation** (`srv/invoice-matching-service.js`):
   - Add logic to fetch supplier details
   - Match suppliers by name
   - Pull payment terms and blocking info from company code records

3. **Create Mock Supplier Data** (`db/data/` CSV files):
   - Supplier records matching invoice supplier names
   - Company code records with payment terms
   - At least one blocked supplier with reason

4. **Update React UI** (`app/matching/src/`):
   - Display supplier details in invoice panel
   - Show warning for blocked suppliers
   - Format payment terms clearly

<details><summary><h2>Optional: Review the AI generated code</h2></summary>

Check these key features:

**Data Model:**<br>
✅ Is there an association from Invoice to Supplier?<br>
✅ Are payment terms and blocking fields included?<br>
✅ Is the association based on supplier name matching?<br>

**Mock Data:**<br>
✅ Do supplier CSV files exist?<br>
✅ Do supplier names match those in invoices?<br>
✅ Is at least one supplier blocked?<br>
✅ Are payment terms realistic (NET30, NET60, etc.)?<br>

**UI Enhancement:**<br>
✅ Does the invoice panel show supplier details?<br>
✅ Is there a visual warning for blocked suppliers?<br>
✅ Are payment terms displayed?<br>

</details>

## Exercise 2.3 — Test the Updated Schema

Now it is time to test the updated schema. In case the preview is still running stop it with `Ctrl+C`.

👉 Restart the CAP server:
```bash
cds watch
```

## Exercise 2.4 — Test Supplier Integration

Let's verify the supplier information appears correctly.

👉 Open the UI at `http://localhost:4004`.

Look at the invoice list — each invoice should now show:
- Supplier name
- Payment terms (e.g., "NET30")
- Blocking status indicator

👉 Find a blocked supplier in the list. It should have a warning icon or red indicator and show the blocking reason (e.g., "Quality issue - invoice dispute").


## Exercise 2.5 — Understand Supplier Blocking

Let's understand why blocking status matters.

### Why Block Suppliers?

Suppliers get blocked for posting when:
- **Quality issues:** Defective products delivered
- **Invoice disputes:** Pricing disagreements
- **Legal holds:** Pending legal matters
- **Compliance issues:** Failed audits or certifications

Maria needs to know because she can't release payment to a blocked supplier, needs to escalate or resolve the block first, and may need approval from procurement.

### The Data Structure

```text
Supplier (A_BusinessPartner)
└── Company Code Records (A_SupplierCompany)
    ├── PaymentTerms: "NET30"
    ├── PostingIsBlocked: true/false
    └── PostingBlockReason: "Quality issue"
```


## Exercise 2.6 — Test Blocked Supplier Workflow

Let's see what happens when Maria tries to match an invoice with a blocked supplier.

👉 Find a blocked supplier invoice in the left panel — it should show a warning and block reason.

👉 Try to select it and match with a PO, then observe the behavior:
- UI might show a warning message
- Or allow the match but flag it for escalation
- Depends on how Claude implemented it

**Expected Business Logic:**
- Match can be recorded but must be escalated
- Match cannot be approved until block is resolved
- Rationale should mention the blocking reason


<details><summary><h2>Optional: Explore Data Product Patterns</h2></summary>

Let's understand what happened under the hood.

### What is a Data Product?

A **data product** is a packaged, reusable API specification that includes:
- Service definitions (OData metadata)
- Entity schemas
- Sample data
- Authentication requirements
- Documentation

SAP provides many data products from S/4HANA Cloud, SuccessFactors, Ariba, and Concur.

### How CAP Imports Data Products

```bash
cds import --data-product <file.json>
```

This creates:
- `apis/imported/<service>/` folder
- CDS definitions from OData metadata
- Service references in `package.json`
- Mock data configurations

### Using Imported Services

In your code:
```javascript
// Reference the imported service
using { API_BUSINESS_PARTNER as Suppliers } from '../apis/imported';

// Create association
entity Invoices {
  supplier : Association to Suppliers.A_BusinessPartner;
}
```

</details>

## What You've Accomplished

- Imported S/4HANA Supplier data product
- Integrated external data with your app
- Added supplier details to UI
- Implemented blocked supplier warnings
- Created realistic mock supplier data
- Understood data product patterns

## Understanding What Happened

**Traditional Approach:**
- Manually define API entities
- Write integration code
- Create mock data
- Update UI components
- 1-2 hours of work

**AI-Assisted Approach:**
- Import data product (1 command)
- One Claude prompt
- Everything integrated
- 15 minutes total

**What You Learned:**
- How SAP data products work
- How to integrate external SAP data
- Matching strategies (by name vs ID)
- Business logic for supplier blocking

<details><summary><h2>Troubleshooting</h2></summary>

### Issue: Data product import fails
**Solution:** Verify the JSON file exists and is valid:
```bash
cat sap-s4com-Supplier-v1.json | jq .
```

### Issue: Supplier details not showing
**Solution:** Check the association in `schema.cds`.

### Issue: No blocked suppliers in data
**Solution:** Ask Claude to add one:
```text
Add a blocked supplier to the mock data with block reason "Quality issue"
```

### Issue: Mock data doesn't match
**Solution:** Ensure supplier names in CSV exactly match invoice supplier names.

## Optional Enhancements

Want to add more supplier features? Try these:

### Add Supplier History
```text
Show how many invoices and POs exist for each supplier, and the total amount
```

### Add Payment Terms Validation
```text
Warn Maria if payment terms on the invoice differ from supplier's standard terms
```

### Add Supplier Search
```text
Add a supplier lookup feature where Maria can search for supplier details by name
```

### Add Multiple Company Codes
```text
Support suppliers with different payment terms in different company codes
```
</details>



## Summary

In this exercise, you:
- Imported S/4HANA Supplier data product
- Integrated supplier information with invoices
- Added supplier blocking warnings
- Created realistic mock supplier data
- Understood data product patterns

**Time spent:** ~15 minutes  
**Data product APIs used:** 1 (Supplier)  
**Lines of integration code:** ~0 (CDS + CAP Skill + AI generated it!)

## Next Steps

Now that Maria can see supplier details and blocking status, let's make the entire service accessible to AI via MCP!

**Continue to [Exercise 3 — Add MCP Server](../ex3/README.md)** to expose your service over the Model Context Protocol.

