# Exercise 2 - Add Supplier Data

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

This helps Maria identify issues like:
- ❌ Supplier blocked for posting
- ⚠️ Unusual payment terms
- ✅ Verified supplier ready for payment

## Exercise 2.1 - Import the Supplier Data Product

SAP provides pre-built data products that you can import into CAP.

1. On the root folder of this exercise you can find the file sap-s4com-Supplier-v1.json.

2. Download the file.

3. Copy the file to your project root.

4. Go back to VSCode and make sure you're in the project root in your terminal:
```bash
pwd
# Should show: .../invoice-matching
```

5. Verify the data product file exists:
```bash
ls sap-s4com-Supplier-v1.json
```

You should see the file listed.

6. Import the data product:
```bash
cds import --data-product sap-s4com-Supplier-v1.json
```

This command:
- Analyzes the data product schema
- Creates CDS definitions in `apis/imported/`
- Sets up service references
- Configures API metadata

7. Check what was imported:
```bash
ls apis/imported/
```

You should see a folder for the Business Partner API (Supplier is part of this).


## Exercise 2.2 - Add Supplier Details with Claude

Now we'll enhance the application to show supplier information.

> 💡 **AI-Assisted Development**: Claude will integrate the supplier data product and update the UI in one prompt.

### The Claude Prompt

Copy this exact prompt into Claude:

```
 /cap-add-remote-service  The Supplier data product is imported under apis/imported/. For each invoice Maria reviews, I want to see key details about that supplier alongside it, so she can spot problems before releasing payment.

Add a supplier details section to each invoice showing:
- the supplier's name
- their payment terms
- whether they're blocked for posting
- the payment blocking reason, if any

Pull the name from the Supplier record and the payment/blocking fields from its company-code record. A few things to keep in mind:

Invoices only carry the supplier's name today, not an ID, so match suppliers by name for now.

Give me realistic mock supplier data that lines up with the suppliers already in the invoices, and make sure at least one supplier is blocked so I can see how that looks.
```

### What Claude Will Generate

Claude should:

1. **Extend the Data Model** (`db/schema.cds`):
   - Add association from Invoice to Supplier
   - Add supplier details fields

2. **Update Service Implementation** (`srv/matching-service.js`):
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

### Review the Generated Code

Check these key features:

**Data Model:**
- ✅ Is there an association from Invoice to Supplier?
- ✅ Are payment terms and blocking fields included?
- ✅ Is the association based on supplier name matching?

**Mock Data:**
- ✅ Do supplier CSV files exist?
- ✅ Do supplier names match those in invoices?
- ✅ Is at least one supplier blocked?
- ✅ Are payment terms realistic (NET30, NET60, etc.)?

**UI Enhancement:**
- ✅ Does the invoice panel show supplier details?
- ✅ Is there a visual warning for blocked suppliers?
- ✅ Are payment terms displayed?

## Exercise 2.3 - Deploy Updated Schema

After saving the generated code, test the updated schema.

1. Restart the CAP server:
```bash
cds watch
```

2. The React app should auto-reload. If not:
```bash
cd app/matching
npm start
```

## Exercise 2.4 - Test Supplier Integration

Let's verify the supplier information appears correctly.

### Check in Browser

1. Open the UI at `http://localhost:3000`

2. Look at the invoice list - each invoice should now show:
   - Supplier name
   - Payment terms (e.g., "NET30")
   - Blocking status indicator

3. Find a blocked supplier:
   - Should have a ⚠️ warning icon or red indicator
   - Should show blocking reason (e.g., "Quality issue - invoice dispute")

<br>![Supplier Details](/exercises/ex2/images/02_04_0010.png)

### Check via OData

1. Browse to `http://localhost:4004/odata/v4/matching/Invoices?$expand=supplier`

2. Verify the response includes supplier data:
```json
{
  "value": [
    {
      "ID": "...",
      "supplier": "Acme Corp",
      "amount": 1500.00,
      "description": "Office supplies",
      "status": "open",
      "supplier": {
        "name": "Acme Corp",
        "paymentTerms": "NET30",
        "isBlocked": false,
        "blockReason": null
      }
    }
  ]
}
```

## Exercise 2.5 - Understand Supplier Blocking

Let's understand why blocking status matters.

### Why Block Suppliers?

Suppliers get blocked for posting when:
- **Quality issues:** Defective products delivered
- **Invoice disputes:** Pricing disagreements
- **Legal holds:** Pending legal matters
- **Compliance issues:** Failed audits or certifications

Maria needs to know because:
- ❌ Can't release payment to blocked supplier
- ⚠️ Need to escalate or resolve the block first
- 📋 May need approval from procurement

### The Data Structure

```
Supplier (A_BusinessPartner)
└── Company Code Records (A_SupplierCompany)
    ├── PaymentTerms: "NET30"
    ├── PostingIsBlocked: true/false
    └── PostingBlockReason: "Quality issue"
```

### Matching by Name

Since invoices only have supplier names (not IDs), we match like:
```sql
SELECT * FROM Suppliers 
WHERE name = invoice.supplier
```

In production, you'd use supplier IDs, but names work for this demo.

## Exercise 2.6 - Test Blocked Supplier Workflow

Let's see what happens when Maria tries to match an invoice with a blocked supplier.

1. **Find a blocked supplier invoice** in the left panel
   - Should show ⚠️ warning and block reason

2. **Try to select it** and match with a PO

3. **Observe the behavior:**
   - UI might show a warning message
   - Or allow the match but flag it for escalation
   - Depends on how Claude implemented it

**Expected Business Logic:**
- ⚠️ Match can be recorded but must be escalated
- ❌ Match cannot be approved until block is resolved
- 📋 Rationale should mention the blocking reason

## Exercise 2.7 - Explore Data Product Patterns

Let's understand what happened under the hood.

### What is a Data Product?

A **data product** is a packaged, reusable API specification that includes:
- Service definitions (OData metadata)
- Entity schemas
- Sample data
- Authentication requirements
- Documentation

SAP provides many data products from:
- S/4HANA Cloud
- SuccessFactors
- Ariba
- Concur

### How CAP Imports Data Products

```bash
cds import --data-product <file.json>
```

This creates:
- `apis/imported/<service>/` folder
- CDS definitions from OData metadata
- Service references in package.json
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

### Local Development

For local dev, CAP uses mock data. In production:
- Configure SAP BTP destination
- Connect to real S/4HANA system
- Use OAuth2 authentication
- CAP handles the rest!

## What You've Accomplished

✅ Imported S/4HANA Supplier data product  
✅ Integrated external data with your app  
✅ Added supplier details to UI  
✅ Implemented blocked supplier warnings  
✅ Created realistic mock supplier data  
✅ Understood data product patterns

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

## Troubleshooting

### Issue: Data product import fails
**Solution:** Verify the JSON file exists and is valid:
```bash
cat sap-s4com-Supplier-v1.json | jq .
```

### Issue: Supplier details not showing
**Solution:** Check the association in schema.cds:
```cds
using { API_BUSINESS_PARTNER as Suppliers } from '../apis/imported';
entity Invoices {
  supplier : Association to Suppliers.A_BusinessPartner;
}
```

### Issue: No blocked suppliers in data
**Solution:** Ask Claude to add one:
```
Add a blocked supplier to the mock data with block reason "Quality issue"
```

### Issue: Mock data doesn't match
**Solution:** Ensure supplier names in CSV exactly match invoice supplier names.

## Optional Enhancements

Want to add more supplier features? Try these:

### Add Supplier History
```
Show how many invoices and POs exist for each supplier, and the total amount
```

### Add Payment Terms Validation
```
Warn Maria if payment terms on the invoice differ from supplier's standard terms
```

### Add Supplier Search
```
Add a supplier lookup feature where Maria can search for supplier details by name
```

### Add Multiple Company Codes
```
Support suppliers with different payment terms in different company codes
```

## Next Steps

Now that Maria can see supplier details and blocking status, let's make the entire service accessible to AI via MCP!

**Continue to [Exercise 3 - Add MCP Server](../ex3/README.md)** to expose your service over the Model Context Protocol.

---

## Summary

In this exercise, you:
- ✅ Imported S/4HANA Supplier data product
- ✅ Integrated supplier information with invoices
- ✅ Added supplier blocking warnings
- ✅ Created realistic mock supplier data
- ✅ Understood data product patterns

**Time spent:** ~15 minutes  
**Data product APIs used:** 1 (Business Partner)  
**Lines of integration code:** ~0 (Claude generated it!)  
**Supplier data:** ✅ Integrated!

Maria can now see critical supplier info! Next: Making it AI-accessible with MCP. 🚀
