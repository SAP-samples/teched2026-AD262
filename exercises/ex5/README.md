# Exercise 5 — Add Agent with Skills

In this exercise, you will configure your matching service as an AI agent and create a skill that encodes AP (Accounts Payable) domain policy with business rules.

## Learning Objectives

By the end of this exercise, you will:
- Understand what an AI agent is
- Configure a CAP service as an agent
- Create a domain policy skill
- Encode business rules for intelligent matching
- Test the agent with complex workflows
- Require human approval for critical decisions

## What is an AI Agent?

An **AI agent** is an AI system that can:
- Understand natural language requests
- Plan multi-step workflows
- Use tools to complete tasks
- Apply business rules and policies
- Request human approval when needed

**Think of it as:** Maria's intelligent assistant that knows AP policies and helps her make decisions.

### Agent vs MCP

| Feature | MCP (Exercise 3) | Agent (This Exercise) |
|---------|------------------|----------------------|
| **Purpose** | Tool access | Autonomous assistant |
| **Intelligence** | Query/response | Multi-step reasoning |
| **Business Rules** | None | Encoded in skills |
| **Workflow** | User drives | Agent drives |
| **Human Approval** | Not built-in | Configurable gates |


## Exercise 5.1 — Install Agent Package

Let's add the CAP agent package.

👉 Make sure you're in the project root:
```bash
pwd
# Should show: .../invoice-matching
```

👉 Install the CAP agent package:
```bash
npm add @cap-js/agents
```

This adds agent capabilities to your CAP server.

👉 Restart the CAP server:
```bash
cds watch
```

## Exercise 5.2 — Configure Agent with Claude

Now we'll use Claude to configure the matching service as an agent.

> [!TIP]
> Claude will configure the agent and set up human approval requirements.

### The Claude Prompt

👉 Copy this exact prompt into Claude:

```text
/cap-developer Annotate matching service as an agent. Require human approval for match approvals.
```

### What Claude Will Generate

Claude should:

1. **Annotate the Service** (srv/matching-service.cds or srv/agent.cds):
```cds
using { MatchingService } from './matching-service';

annotate MatchingService with @agent;

// Require human approval for approve action
annotate MatchingService.approve with @agent.hitl;
```

> [!TIP]
> hitl stands for (Human In The Loop).

### Review the Generated Code

Check:<br>
✅ Is the service annotated with @agent?<br>
✅ Does approve action require human approval?<br>
✅ Is the agent properly named and described?<br>

Save all the generated files.

## Exercise 5.3 — Access the Agent UI

Let's test the agent interface.

👉 Make sure the CAP server is running:
```bash
cds watch
```

👉 Open the agent preview URL:
```text
http://localhost:4004/a2a/matching/preview/
```

You should see:
- Agent chat interface
- Text input for natural language queries
- History of conversations
- Human approval prompts (when triggered)

<br>![Agent Preview UI](/exercises/ex5/images/05_03_0010.png)

## Exercise 5.4 — Test the Agent

Let's test the agent with a complex workflow.

### Test 1: Find Unmatched Invoices

👉 In the agent UI, type:

```text
Find all unmatched invoices and propose PO matches
```

**What the agent does:**
1. Queries for invoices with status = 'open'
2. For each invoice, searches for matching POs
3. Uses semantic search for description similarity
4. Checks amount proximity
5. Proposes matches with rationale
6. Creates match decisions with status 'proposed'

**Agent response should include:**
- List of unmatched invoices found
- Proposed PO for each
- Match confidence score
- Rationale (why this match makes sense)

<br>![Agent Proposing Matches](/exercises/ex5/images/05_04_0010.png)

### Test 2: Request Approval

👉 Now try to approve a match:

```text
Approve the match for invoice 1
```

**What happens:**
- Agent identifies the match decision
- Sees that approve action requires human approval
- **Prompts you to confirm:** "This action requires approval. Proceed? [Yes/No]"
- If you click Yes → match is approved
- If you click No → action cancelled

This is the human-in-the-loop pattern!

<br>![Human Approval Required](/exercises/ex5/images/05_04_0020.png)

### Test 3: Blocked Supplier Handling

👉 Type:

```text
Are there any invoices from blocked suppliers? What should I do with them?
```

**What the agent does:**
- Queries invoices
- Joins with supplier data
- Filters for isBlocked = true
- Recommends escalation
- Explains the blocking reason

**Agent response:**
- "Found 1 invoice from blocked supplier 'Beta Corp'"
- "Blocking reason: Quality issue - invoice dispute"
- "Recommendation: Escalate this match, do not approve"

### Test 4: Complex Multi-Step Query

👉 Type:

```text
Find invoices over 1000 EUR that have been open more than 30 days, then propose matches and highlight any with blocked suppliers
```

**What the agent does:**
1. Filters invoices by amount and date
2. Proposes matches using semantic search
3. Checks supplier blocking status
4. Highlights risks
5. Provides actionable recommendations

This is AI-powered AP processing!

## Exercise 5.5 — Generate additional sample data with not matching amounts

👉 Copy this exact prompt into Claude:

```text
/cap-developer Generate additional sample data where the amount is not exactly matching of the inovices and 
purchase orders. Amounts should diverge between 2-10%. One invoice should have an identifier INV-TEST-001. 
Another should have a blocked supplier called Beta Corp with a perfect match.
```

## Exercise 5.6 — Create AP Domain Policy Skill

Now let's create a skill that encodes business rules.

> [!NOTE]
> Tools are what the agent CAN do (query, create, update). Skills are HOW it SHOULD do them (policies, rules, guidelines).

### The Claude Prompt

👉 Copy this exact prompt into Claude:

```text
Create a skill that gives the matching agent its AP domain policy. It should guide how the agent proposes and 
confirms matches - not what tools it has.

Encode:
- Amount tolerance: within 2% is a clean match; 2-5% requires a price-variance note in the rationale; over 5% 
must escalate, never auto-confirm.
- Confidence rubric: weight an exact supplier + amount match highest; a semantic-description match (from 
the Vector search) lower, a partial lowest.
- Distrust a high similarity score when the amounts diverge.
- Blocked-supplier rule: if the supplier is blocked for posting, never propose confirm regardless of match 
quality — flag it for Maria with the blocking reason.
- Rationale quality: every proposal must state which signals matched and why, in one or two sentences Maria can act on.
```

### What Claude Will Generate

Claude should create:

1. **Skill File** (srv/skills/ap-policy.js):
```javascript
module.exports = {
  name: 'AP Domain Policy',
  description: 'Business rules for invoice-PO matching in Accounts Payable',
  
  rules: {
    amountTolerance: {
      clean: 0.02,      // 2%
      acceptable: 0.05,  // 5%
      escalate: 0.05     // Over 5%
    },
    
    confidenceWeights: {
      exactSupplierAmount: 1.0,
      semanticDescription: 0.7,
      partial: 0.4
    },
    
    blockedSupplier: {
      action: 'flag',
      allowApproval: false
    }
  },
  
  apply: async (context) => {
    // Implementation of policy logic
  }
};
```

2. **Skill Registration** (srv/matching-service.js):
```javascript
const apPolicy = require('./skills/ap-policy');

// Register skill with agent
this.on('agent:plan', async (req) => {
  req.agent.skills.register(apPolicy);
});
```

3. **Policy Application**:
- Calculate amount variance
- Apply confidence rubric
- Check supplier blocking
- Enforce escalation rules
- Generate quality rationales

### Review the Generated Skill

Check the skill includes:

**Amount Tolerance Logic:**<br>
✅ Within 2%: clean match<br>
✅ 2-5%: add price-variance note<br>
✅ Over 5%: must escalate<br>

**Confidence Rubric:**<br>
✅ Exact supplier + amount: highest score<br>
✅ Semantic description match: lower score<br>
✅ Partial match: lowest score<br>
✅ Distrust high similarity with divergent amounts<br>

**Blocked Supplier Rule:**<br>
✅ Never auto-confirm blocked suppliers<br>
✅ Flag with blocking reason<br>
✅ Force escalation<br>

**Rationale Quality:**<br>
✅ 1-2 sentences<br>
✅ States which signals matched<br>
✅ Actionable for Maria<br>


## Exercise 5.7 — Test the AP Policy Skill

Let's verify the business rules are working.

### Test 1: Clean Match (Within 2%)

👉 Agent query:

```text
Propose a match for invoice INV-TEST-001
```

**Expected behavior:**
- Agent proposes the match
- Confidence: HIGH
- Rationale: "Exact supplier match, amount within 2% tolerance (€1015 vs €1000)"
- No escalation required



### Test 4: Blocked Supplier

👉 Agent query:

```text
Should I match this invoice from Beta Corp?
```

**Expected behavior:**
- Agent warns: "⚠️ Supplier Beta Corp is blocked for posting"
- Blocking reason: "Quality issue - invoice dispute"
- Recommendation: "Do NOT approve any matches for this supplier. Escalate to procurement."
- Even with perfect match, approval blocked

### Test 5: High Semantic, Divergent Amount

👉 Agent query:

```text
Find matches for the cloud storage invoice
```

**Expected behavior:**
- Agent finds the PO
- Notes high semantic similarity
- BUT flags: "⚠️ Semantic match is strong (0.95) but amounts diverge significantly (€5000 vs €1200)"
- Confidence: LOW (despite high semantic score)
- Rationale: "Descriptions match well, but amount mismatch suggests different services - likely annual vs monthly"
- Recommendation: Escalate

<br>![Policy Enforcement](/exercises/ex5/images/05_06_0010.png)

## Exercise 5.8 — Understand Agent Architecture

Let's understand how the agent works.

### Agent Components

```text
┌─────────────────────────────────────────┐
│         User (Maria)                    │
│  "Find unmatched invoices and match"    │
└───────────────┬─────────────────────────┘
                │
                ↓
┌─────────────────────────────────────────┐
│         Agent Brain (LLM)               │
│  - Understands intent                   │
│  - Plans multi-step workflow            │
│  - Applies skills/policies              │
└───────────────┬─────────────────────────┘
                │
                ↓
┌─────────────────────────────────────────┐
│         Skills Layer                    │
│  - AP Domain Policy                     │
│  - Business Rules                       │
│  - Confidence Rubric                    │
└───────────────┬─────────────────────────┘
                │
                ↓
┌─────────────────────────────────────────┐
│         Tools (MCP)                      │
│  - Query invoices                        │
│  - Search matches                        │
│  - Create decisions                      │
│  - Approve/Reject/Escalate               │
└─────────────────────────────────────────┘
```

### How Skills Work

1. **User asks:** "Find matches"
2. **Agent plans:** Query invoices → Search POs → Propose matches
3. **Before executing:** Apply AP policy skill
4. **Skill checks:** Amount tolerance, supplier blocking, confidence
5. **Skill modifies:** Add rationale notes, block certain actions, force escalation
6. **Agent executes:** With policy enforcement
7. **Result:** Policy-compliant match proposals

### Human Approval Gates

```text
Agent wants to: Approve match
                    ↓
             Check @agent.requiresApproval
                    ↓
          Yes → Prompt human → Wait for response
          No → Execute directly
```

**What requires approval:**
- ✅ approve action (high risk)
- ❌ reject action (low risk, audited)
- ❌ escalate action (low risk, goes to human anyway)
- ❌ query/read operations (no risk)

## What You've Accomplished

✅ Installed CAP agent package  
✅ Configured service as an agent  
✅ Created AP domain policy skill  
✅ Encoded business rules  
✅ Tested intelligent matching  
✅ Implemented human approval gates  
✅ Understood agent architecture

## Understanding What Happened

**Traditional Approach:**
- Hard-code business rules in service
- Build custom approval workflow
- Create UI for decision review
- Days of development

**AI-Assisted Approach:**
- Two Claude prompts
- Skills encode policies
- Built-in approval gates
- 25 minutes total

**What You Learned:**
- How AI agents work
- Skills vs tools distinction
- Business rule encoding
- Human-in-the-loop patterns
- Confidence scoring


## Agent Best Practices

### Do's ✅

1. **Encode domain knowledge in skills** - Don't expect the agent to know your business
2. **Require approval for risky actions** - Approvals, deletions, financial commitments
3. **Provide clear rationales** - Help humans understand agent reasoning
4. **Set confidence thresholds** - Low confidence → escalate
5. **Test edge cases** - Blocked suppliers, large amounts, etc.

### Don'ts ❌

1. **Don't auto-approve everything** - Keep humans in critical loops
2. **Don't skip rationales** - "Trust me" doesn't work for auditors
3. **Don't ignore business rules** - Amount limits, blocking rules, etc.
4. **Don't allow unbounded actions** - Set limits on batch operations
5. **Don't forget audit trails** - Every agent action should be logged

<details><summary><h2>Troubleshooting</h2></summary>

### Issue: Agent UI doesn't load
**Solution:** Check the URL and ensure CAP server is running:
```text
http://localhost:4004/a2a/matching/preview/
```

### Issue: Human approval doesn't prompt
**Solution:** Verify the annotation:
```cds
annotate MatchingService.approve with @agent.hitl;
```

### Issue: Skills not applying
**Solution:** Check skill registration in service implementation:
```javascript
this.on('agent:plan', async (req) => {
  req.agent.skills.register(apPolicy);
});
```

### Issue: Agent makes wrong decisions
**Solution:** Review and adjust the skill rules. The agent follows the policy you encode!

</details>

<details><summary><h2>Optional Enhancements (can be skipped onsite at TechEd)</h2></summary>
### Add More Skills

```text
Create a risk assessment skill that flags invoices over €10,000 or from new suppliers
```

### Multi-Language Support

```text
Make the agent respond in German when Maria asks questions in German
```

### Batch Operations

```text
Add a skill for batch matching with confirmation after every 10 matches
```

### Learning from Feedback

```text
Track when Maria overrides agent recommendations and adjust confidence scoring
```
</details>

## Next Steps

Congratulations! You've completed the core workshop. You now have:<br>
✅ Complete invoice matching application<br>
✅ React UI for Maria<br>
✅ S/4HANA supplier integration<br>
✅ MCP server for AI access<br>
✅ Semantic search<br>
✅ Intelligent agent with business rules<br>

**Optional:** Continue to [Exercise 6 - Audit Log & Change Tracking](../ex6/README.md) to add enterprise governance.

Or explore on your own:
- Customize the React UI
- Add more supplier data fields
- Create additional agent skills
- Test with real scenarios

---

## Summary

In this exercise, you:<br>
✅ Configured CAP service as an agent<br>
✅ Created AP domain policy skill<br>
✅ Encoded complex business rules<br>
✅ Tested intelligent workflows<br>
✅ Implemented human approval gates<br>

**Time spent:** ~25 minutes  
**Business rules encoded:** 5+ policies  
**Hard-coded logic:** 0 lines (rules in skills!)  
**Agent intelligence:** 🤖 Complete!

Maria now has an intelligent assistant that knows AP policies! 🚀

---

## Congratulations! 🎉

You've completed the core TechEd 2026 AD262 workshop!

### What You Built (in 2 hours!)
- Full-stack invoice matching application
- React UI with workflow state management
- S/4HANA Supplier integration
- MCP server for AI access
- Semantic search with local embeddings
- Intelligent AI agent with business rules

### What You Learned
- AI-assisted development with Claude
- CAP application patterns
- React + OData integration
- Data products and SAP integration
- Model Context Protocol (MCP)
- Vector embeddings and semantic search
- AI agents and skills
- Human-in-the-loop patterns

### Optional Next Steps
- [Exercise 6 - Audit Log & Change Tracking](../ex6/README.md)
- [Exercise 7 - SAP-RPT-1 Recommendations](../ex7/README.md)


**Thank you for participating! 🙏**
