# Exercise 6 — Add Audit Log and Change Tracking (Optional)

This is an **optional advanced exercise** that adds enterprise governance and compliance features to your invoice matching system.

## Overview

**Duration:** ~15 minutes  
**Prerequisites:** Completed Exercises 0-5  
**Difficulty:** Intermediate

In this exercise, you will add audit logging and change tracking to ensure compliance and traceability for all AI agent actions and data changes.

## Learning Objectives

By the end of this exercise, you will:
- Understand audit logging requirements for AI systems
- Implement data privacy annotations
- Enable change tracking for critical decisions
- Track all modifications to invoices and match decisions
- Understand data subject concepts
- Test the audit trail

## Why Audit Logging Matters

### Compliance Requirements

AI systems in enterprise applications must:
- **Track every decision** - Who/what approved each match
- **Log data access** - Who viewed sensitive supplier data
- **Record changes** - Full history of modifications
- **Support rollback** - Ability to undo if needed
- **Enable investigation** - Audit trail for disputes


### Risk Management

- **AI accountability** - Prove AI decisions were sound
- **Dispute resolution** - Show evidence for invoice disputes
- **Performance tracking** - Monitor approval/rejection rates
- **Policy enforcement** - Verify business rules followed

## Exercise 6.1 — Install Audit Packages

Let's add the audit logging and change tracking packages.

👉 Make sure you're in the project root:
```bash
pwd
# Should show: .../invoice-matching
```

👉 Add audit log support:
```bash
cds add audit-logging
```

This configures audit logging for CAP.

👉 Add change tracking package:
```bash
npm add @cap-js/change-tracking
```

This enables change history for annotated fields.

👉 Restart the CAP server:
```bash
cds watch
```

## Exercise 6.2 — Configure Audit and Change Tracking

Now we'll use Claude to add the necessary annotations.

> [!TIP]
> Claude will add all data-privacy and change-tracking annotations following GDPR best practices.

### The Claude Prompt

👉 Copy this exact prompt into Claude:

```text
/cap-developer Add data-privacy and change-tracking annotations to the AP matching model.

Treat the supplier as the data subject. Reuse the supplier consumption view and the unmanaged invoice-to-supplier association joined on supplier name from Exercise 2.
Use that association as the invoice's DataSubjectID so the audit-log plugin can reach the DataSubject; mark supplier name and
description as personal data. Annotate Supplier itself as DataSubject and its key as DataSubjectID. Annotate Invoice as Other.

Enable change tracking on the decision's outcome, confidence, and rationale using the changelog annotation on each field in the domain entity.
```

### What Claude Will Generate

Claude should:

1. **Add Data Privacy Annotations** (db/schema.cds):

Append these annotations to the existing `db/schema.cds`. Reuse the imports, supplier consumption view, and invoice association from Exercise 2, shown here as `Suppliers` (`ID`, `name`) and `supplierData`. Use the names in your generated model.

```cds
annotate Suppliers with @PersonalData: {
  EntitySemantics: 'DataSubject',
  DataSubjectRole: 'Supplier'
} {
  ID   @PersonalData.FieldSemantics: 'DataSubjectID';
  name @PersonalData.IsPotentiallyPersonal;
};

annotate Invoices with @PersonalData.EntitySemantics: 'Other' {
  supplierData @PersonalData.FieldSemantics: 'DataSubjectID';
  supplier     @PersonalData.IsPotentiallyPersonal;
  description  @PersonalData.IsPotentiallyPersonal;
};
```

2. **Enable Change Tracking** (db/schema.cds):
```cds
annotate MatchDecisions {
  confidence @changelog;
  rationale  @changelog;
  outcome    @changelog;
};
```

3. **Data Privacy Annotations on the Service** (srv/invoice-matching-service.cds):
```cds
annotate InvoiceMatchingService.Invoices
  with @PersonalData.EntitySemantics: 'Other' {
    supplierData @PersonalData.FieldSemantics: 'DataSubjectID';
    supplier     @PersonalData.IsPotentiallyPersonal;
    description  @PersonalData.IsPotentiallyPersonal;
};
```

4. **Default Audit Logging Configuration**:

The plugin logs to the console in development by default. No changes to `package.json` are needed. Verify the effective configuration:
```bash
cds env requires.audit-log
# kind: 'audit-log-to-console'
```

### Review the Generated Code

Check:
- ✅ Is Supplier marked as DataSubject?
- ✅ Is the supplier key marked as DataSubjectID, and supplier name as personal data?
- ✅ Is Invoice marked as Other, with supplierData as DataSubjectID and description as personal data?
- ✅ Are MatchDecisions fields annotated with @changelog?
- ✅ Does the service expose supplierData as DataSubjectID and supplier name and description as personal data?


## Exercise 6.3 — Test Updated Schema

Deploy the changes to enable change tracking.

👉 Restart the CAP server:
```bash
cds watch
```

👉 Check the console for audit log initialization messages.

## Exercise 6.4 — Test Audit Trail

Let's verify the audit logging is working.

### Test 1: Change an Invoice Description

👉 Use Claude with this prompt:

```text
Do a REST call to change the description of an invoice
```

**What Claude will do:**
- Find an invoice
- Make a PATCH request to update the description
- Show you the curl command or HTTP request

**Example:**
```bash
curl -X PATCH "http://localhost:4004/odata/v4/matching/Invoices(invoice-uuid)" \
  -H "Content-Type: application/json" \
  -d '{"description": "Updated office supplies - paper and pens"}'
```

**What gets logged:**
- ✅ **Change Record** - Old vs new description
- ✅ **Data Subject** - Which supplier's data was affected
- ✅ **User Context** - Who made the change (or "agent")
- ✅ **Timestamp** - When it happened
- ✅ **Object Type** - Invoice entity

Check the console - you should see audit log entries like:
```
[audit-log] - PersonalDataModified: {
  user: "anonymous",
  object: {
    type: "InvoiceMatchingService.Invoices",
    id: { ID: "invoice-uuid" }
  },
  data_subject: {
    type: "InvoiceMatchingService.Suppliers",
    role: "Supplier",
    id: { ID: "supplier-id" }
  },
  attributes: [
    { name: "description", old: "Office supplies", new: "Updated office supplies - paper and pens" }
  ],
  uuid: "audit-event-uuid",
  time: "2026-09-30T10:30:00.000Z"
}
```

<br>![Audit Log Entry](/exercises/ex6/images/06_04_0010.png)

### Test 2: Approve a Match Decision

👉 In the agent UI or directly via API, approve a match.

Check the decision's change history shows:
   - ✅ Match decision ID
   - ✅ Old outcome: "proposed"
   - ✅ New outcome: "approved"
   - ✅ Timestamp and user

### Test 3: Query Change History

👉 Ask Claude:

```text
Show me the change history for match decision INV-001
```

Claude will:
- Query the decision's `changes` association
- Show changes to the tracked fields
- Display who changed what and when

**Expected output:**
```text
Match Decision Change History:
1. 2026-09-30 10:00 - Created by agent
   - outcome: proposed
   - confidence: 0.85
   - rationale: "Exact supplier match, semantic description similarity"

2. 2026-09-30 10:15 - Updated by maria@company.com
   - outcome: proposed → approved
```

### Test 4: Data Subject Access Request (GDPR)

👉 Simulate a GDPR request:

```text
Show me all data related to supplier "Acme Corp"
```

Claude should retrieve:
- All invoices from Acme Corp
- All match decisions involving Acme Corp invoices
- Audit log entries where Acme Corp was the data subject
- Change history affecting Acme Corp data

This demonstrates GDPR compliance - you can fulfill "right to access" requests.

<br>![Data Subject Report](/exercises/ex6/images/06_04_0020.png)

## Exercise 6.5 — Understand Change Tracking

Let's understand how change tracking works.

### Change Records

The plugin stores field-level changes in `sap.changelog.Changes` and exposes them through the entity's `changes` association:
```text
MatchDecisions (current state)
├── ID: uuid
├── confidence: 0.85
├── outcome: "approved"
└── ...

sap.changelog.Changes (one record per changed field)
├── entityKey: uuid
├── attribute: "outcome"
├── valueChangedFrom: "proposed"
├── valueChangedTo: "approved"
├── createdAt: 2026-09-30 10:15
└── createdBy: "maria@company.com"
```

### Querying History

```bash
curl "http://localhost:4004/odata/v4/matching/MatchDecisions(decision-uuid)/changes"
```

### Automatic Management

The plugin handles:
- ✅ Recording changes to annotated fields
- ✅ Recording user context
- ✅ Tracking timestamps

Change tracking records differences, not full temporal versions. Unchanged fields do not produce update records.

## Exercise 6.6 — Audit Agent Actions

Let's verify AI agent actions are audited.

👉 In the agent UI, ask:
```text
Find unmatched invoices and propose matches
```

The agent will create multiple match decisions.

👉 Check the change history — creation values for `outcome`, `confidence`, and `rationale` are recorded with the request's user context.

👉 Ask the agent to approve one:
```text
Approve the match for invoice INV-001
```

👉 Click Yes on the human approval prompt.

👉 Check the change history — the transition from "proposed" to "approved" is recorded with the request's user context.

**Why This Matters:**
- 🔍 Every AI decision is traceable
- 👤 Human-in-the-loop is documented
- 📋 Compliance requirements met
- ⚖️ Legal defensibility

## Exercise 6.7 — Data Privacy Compliance

Let's verify GDPR compliance features.

### Right to Access

👉 Supplier "Acme Corp" requests their data:
```text
Show all data for supplier Acme Corp including audit logs
```

Response includes:
- ✅ All invoices
- ✅ All match decisions
- ✅ All audit log entries
- ✅ Full change history
- ✅ Exported in readable format

### Right to Erasure (Right to be Forgotten)

👉 If needed, you can:
```text
Delete all data for supplier Beta Corp
```

The system:
- ✅ Identifies all related records
- ✅ Checks for legal holds
- ✅ Creates audit log of deletion
- ✅ Removes data if permitted

### Data Minimization

The annotations help:
- Only marked fields are treated as personal data
- Audit logs only capture necessary information
- Retention policies can be applied
- Automatic cleanup after retention period

## What You've Accomplished

✅ Installed audit logging and change tracking  
✅ Added data privacy annotations (GDPR)  
✅ Enabled change tracking on decisions  
✅ Tested audit trail functionality  
✅ Understood field-level change history
✅ Verified AI agent action logging  
✅ Demonstrated GDPR compliance

## Understanding What Happened

**Traditional Approach:**
- Build custom audit tables
- Write triggers for changes
- Implement GDPR logic
- Create access reports
- Days of development

**AI-Assisted Approach:**
- Add packages (2 commands)
- One Claude prompt
- Automatic audit logging
- Built-in change tracking
- 15 minutes total

**What You Learned:**
- Audit logging for AI systems
- GDPR data privacy concepts
- Change tracking patterns
- Data subject relationships
- Compliance requirements

<details><summary><h2>Troubleshooting</h2></summary>

### Issue: Audit logs not appearing
**Solution:** Check package.json configuration:
```json
"audit-log": {
  "kind": "audit-log-to-console"
}
```

### Issue: Change tracking tables not created
**Solution:** Redeploy the database:
```bash
cds deploy --to sqlite
```

### Issue: Change history is empty
**Solution:** Ensure @changelog annotation is present.

</details>

<details><summary><h2>Optional Enhancements (can be skipped onsite at TechEd)</h2></summary>

### Add Audit Dashboard

```text
Create a React component that displays recent audit log entries with filters by user, entity, and date range
```

### Export Audit Reports

```text
Add an action to export audit logs as CSV for external auditors
```

### Alerting

```text
Send alerts when high-risk operations occur (e.g., mass deletions, bulk approvals)
```

### Anonymization

```text
After retention period, anonymize personal data while keeping audit trail
```

</details>

## Real-World Use Cases

### 1. Invoice Dispute
- Supplier disputes an invoice amount
- Pull audit log showing all changes
- Prove original amount and who changed it
- Resolve dispute with evidence

### 2. GDPR Request
- Supplier requests all their data
- Query by data subject
- Export complete record
- Fulfill request within 30 days

### 3. Compliance Audit
- External auditor reviews processes
- Show audit trail for all AI decisions
- Prove human approval for critical actions
- Demonstrate policy compliance

### 4. AI Monitoring
- Track agent decision quality
- Compare approved vs rejected
- Identify policy violations
- Improve agent skills

## Summary

In this optional exercise, you:
- ✅ Added enterprise audit logging
- ✅ Implemented GDPR compliance
- ✅ Enabled change tracking
- ✅ Tested the audit trail
- ✅ Understood data subject concepts

**Time spent:** ~15 minutes  
**Compliance features:** Multiple (GDPR, SOX, etc.)  
**Audit trail:** ✅ Complete!  
**Production-ready:** ✅ Yes!

Your invoice matching system now has enterprise-grade governance! 🔒

---

**Next:** [Exercise 7 - SAP-RPT-1 Recommendations](../ex7/README.md) (Optional)  
