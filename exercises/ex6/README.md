# Exercise 6 - Add Audit Log and Change Tracking (Optional)

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

## Exercise 6.1 - Install Audit Packages

Let's add the audit logging and change tracking packages.

1. Make sure you're in the project root:
```bash
pwd
# Should show: .../invoice-matching
```

2. Add audit log support:
```bash
cds add audit-logging
```

This configures audit logging for CAP.

3. Add change tracking package:
```bash
npm add @cap-js/change-tracking
```

This enables temporal versioning of entities.

4. Restart the CAP server:
```bash
cds watch
```

## Exercise 6.2 - Configure Audit and Change Tracking

Now we'll use Claude to add the necessary annotations.

> 💡 **AI-Assisted Development**: Claude will add all data-privacy and change-tracking annotations following GDPR best practices.

### The Claude Prompt

Copy this exact prompt into Claude:

```
/cap-developer Add data-privacy and change-tracking annotations to the AP matching model.

Treat the supplier as the data subject. Add an unmanaged association from Invoice to the S/4 Supplier, joined on supplier name. Use that association as the invoice's DataSubjectID so the audit-log plugin can reach the DataSubject; mark supplier name and description as personal data. Annotate Supplier itself as DataSubject. Annotate the inoice desciption as DataSubjectDetails.

Enable change tracking on the decision's status, confidence, and rationale using the changelog annotation in the domain entity.
```

### What Claude Will Generate

Claude should:

1. **Add Data Privacy Annotations** (db/schema.cds):
```cds
using { API_BUSINESS_PARTNER as Suppliers } from '../apis/imported';

@PersonalData.EntitySemantics: 'DataSubject'
entity Suppliers : Suppliers.A_BusinessPartner {
  // Supplier is the data subject
}

entity Invoices : cuid, managed {
  supplier      : String @PersonalData.FieldSemantics: 'DataSubjectID';
  description   : String @PersonalData.FieldSemantics: 'DataSubjectDetails';
  
  // Unmanaged association for audit log
  supplierData  : Association to one Suppliers on supplierData.name = supplier;
  
  // Other fields...
}
```

2. **Enable Change Tracking** (db/schema.cds):
```cds
@changelog: [status, confidence, rationale]
entity MatchDecisions : cuid, managed {
  invoice       : Association to Invoices;
  purchaseOrder : Association to PurchaseOrders;
  confidence    : Decimal(3,2) @changelog;
  rationale     : String(500) @changelog;
  outcome       : String(20) @changelog;  // proposed, approved, rejected, escalated
  
  // Other fields...
}
```

3. **Configure Audit Logging** (srv/matching-service.cds):
```cds
using { MatchingService } from './matching-service';

// Mark sensitive operations for audit
annotate MatchingService.approve with @AuditLog.Operation.Write;
annotate MatchingService.reject with @AuditLog.Operation.Write;
annotate MatchingService.escalate with @AuditLog.Operation.Write;

// Log data access
annotate MatchingService.Invoices with @AuditLog.Operation.Read;
annotate MatchingService.MatchDecisions with @AuditLog.Operation.Read;
```

4. **Service Configuration** (package.json):
```json
{
  "cds": {
    "requires": {
      "audit-log": {
        "kind": "audit-log-to-console",
        "impl": "@cap-js/audit-logging"
      }
    }
  }
}
```

### Review the Generated Code

Check:
- ✅ Is Supplier marked as DataSubject?
- ✅ Is supplier name marked as DataSubjectID?
- ✅ Is description marked as DataSubjectDetails?
- ✅ Are MatchDecisions fields annotated with @changelog?
- ✅ Are approve/reject/escalate marked for audit logging?


## Exercise 6.3 - Test Updated Schema

Deploy the changes to enable change tracking.

1. Restart the CAP server:
```bash
cds watch
```

2. Check the console for audit log initialization messages.

## Exercise 6.4 - Test Audit Trail

Let's verify the audit logging is working.

### Test 1: Change an Invoice Description

Use Claude with this prompt:

```
Do a REST call to change the description of an invoice
```

**What Claude will do:**
- Find an invoice
- Make a PATCH request to update the description
- Show you the curl command or HTTP request

**Example:**
```bash
curl -X PATCH http://localhost:4004/odata/v4/matching/Invoices(invoice-uuid) \
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
[audit-log] - {
  user: "anonymous",
  object: {
    type: "Invoice",
    id: { ID: "invoice-uuid" }
  },
  dataSubject: {
    type: "Supplier", 
    id: { name: "Acme Corp" }
  },
  attributes: [
    { name: "description", old: "Office supplies", new: "Updated office supplies - paper and pens" }
  ],
  operation: "UPDATE",
  timestamp: "2026-09-30T10:30:00.000Z"
}
```

<br>![Audit Log Entry](/exercises/ex6/images/06_04_0010.png)

### Test 2: Approve a Match Decision

1. In the agent UI or directly via API, approve a match

2. Check the audit log shows:
   - ✅ Action: approve
   - ✅ Match decision ID
   - ✅ Old outcome: "proposed"
   - ✅ New outcome: "approved"
   - ✅ Confidence score
   - ✅ Rationale
   - ✅ Timestamp and user

### Test 3: Query Change History

Ask Claude:
```
Show me the change history for match decision INV-001
```

Claude will:
- Query the change tracking tables
- Show all versions of the decision
- Display who changed what and when

**Expected output:**
```
Match Decision Change History:
1. 2026-09-30 10:00 - Created by agent
   - outcome: proposed
   - confidence: 0.85
   - rationale: "Exact supplier match, semantic description similarity"

2. 2026-09-30 10:15 - Updated by maria@company.com
   - outcome: proposed → approved
   - confidence: 0.85 (no change)
   - rationale: unchanged
```

### Test 4: Data Subject Access Request (GDPR)

Simulate a GDPR request:
```
Show me all data related to supplier "Acme Corp"
```

Claude should retrieve:
- All invoices from Acme Corp
- All match decisions involving Acme Corp invoices
- Audit log entries where Acme Corp was the data subject
- Change history affecting Acme Corp data

This demonstrates GDPR compliance - you can fulfill "right to access" requests.

<br>![Data Subject Report](/exercises/ex6/images/06_04_0020.png)

## Exercise 6.5 - Understand Change Tracking

Let's understand how change tracking works.

### Temporal Tables

For each entity with `@changelog`:
```
MatchDecisions (current state)
├── ID: uuid
├── confidence: 0.85
├── outcome: "approved"
└── ...

MatchDecisions_Changes (history)
├── ID: uuid (same)
├── validFrom: 2026-09-30 10:00
├── validTo: 2026-09-30 10:15
├── confidence: 0.80
├── outcome: "proposed"
└── modifiedBy: "agent"

├── ID: uuid (same)
├── validFrom: 2026-09-30 10:15
├── validTo: 9999-12-31
├── confidence: 0.85
├── outcome: "approved"
└── modifiedBy: "maria@company.com"
```

### Querying History

```sql
-- Current state
SELECT * FROM MatchDecisions WHERE ID = 'uuid';

-- Point in time (as of 10:10)
SELECT * FROM MatchDecisions_Changes 
WHERE ID = 'uuid' 
  AND validFrom <= '2026-09-30 10:10'
  AND validTo > '2026-09-30 10:10';

-- All changes
SELECT * FROM MatchDecisions_Changes 
WHERE ID = 'uuid'
ORDER BY validFrom;
```

### Automatic Management

CAP handles:
- ✅ Creating history records on update
- ✅ Closing previous versions (setting validTo)
- ✅ Recording user context
- ✅ Tracking timestamps
- ✅ Maintaining referential integrity

## Exercise 6.6 - Audit Agent Actions

Let's verify AI agent actions are audited.

1. In the agent UI, ask:
```
Find unmatched invoices and propose matches
```

2. The agent will create multiple match decisions

3. Check the audit log:
   - Each decision creation is logged
   - User context shows "agent"
   - Data subject is identified (supplier)
   - All fields are recorded

4. Ask the agent to approve one:
```
Approve the match for invoice INV-001
```

5. Human approval prompt appears - click Yes

6. Check the audit log:
   - Approval action logged
   - Shows human approved (not just agent)
   - Records the state transition

**Why This Matters:**
- 🔍 Every AI decision is traceable
- 👤 Human-in-the-loop is documented
- 📋 Compliance requirements met
- ⚖️ Legal defensibility

## Exercise 6.7 - Data Privacy Compliance

Let's verify GDPR compliance features.

### Right to Access

Supplier "Acme Corp" requests their data:
```
Show all data for supplier Acme Corp including audit logs
```

Response includes:
- ✅ All invoices
- ✅ All match decisions
- ✅ All audit log entries
- ✅ Full change history
- ✅ Exported in readable format

### Right to Erasure (Right to be Forgotten)

If needed, you can:
```
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
✅ Understood temporal versioning  
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

## Troubleshooting

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

### Issue: Data subject not identified
**Solution:** Verify the unmanaged association:
```cds
supplierData : Association to one Suppliers on supplierData.name = supplier;
```

### Issue: Change history is empty
**Solution:** Ensure @changelog annotation is present.


## Optional Enhancements

### Add Audit Dashboard

```
Create a React component that displays recent audit log entries with filters by user, entity, and date range
```

### Export Audit Reports

```
Add an action to export audit logs as CSV for external auditors
```

### Alerting

```
Send alerts when high-risk operations occur (e.g., mass deletions, bulk approvals)
```

### Anonymization

```
After retention period, anonymize personal data while keeping audit trail
```

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

