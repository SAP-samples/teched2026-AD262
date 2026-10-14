# Exercise 3 - Add MCP Adapter

In this exercise, you will expose your matching service over the Model Context Protocol (MCP), making it accessible to AI agents like Claude.

## Learning Objectives

By the end of this exercise, you will:
- Understand what MCP is and why it matters
- Install the CAP MCP Adapter
- Expose your service over MCP
- Configure Claude Desktop to connect
- Query your business data conversationally

## What is MCP?

The **Model Context Protocol (MCP)** is an open standard that enables AI applications to securely connect with data sources and tools.

**Think of it as:** A universal adapter that lets AI agents interact with your enterprise applications.

**Key Concepts:**
- **Tools:** Functions AI agents can call (e.g., "find matches", "approve decision")
- **Resources:** Data AI agents can read (e.g., invoice details, supplier info)
- **Prompts:** Pre-built templates to guide AI interactions

**Why MCP Matters:**
- ✅ Standard protocol - works with any MCP-compatible AI
- ✅ Secure - built-in authentication and permissions
- ✅ Automatic - CAP entities become MCP tools
- ✅ No custom API - reuse existing OData services

## Exercise 3.1 - Install MCP Package

Let's add MCP support to your CAP application.

1. Make sure you're in the project root:
```bash
pwd
# Should show: .../invoice-matching
```

2. Install the CAP MCP package:
```bash
npm add @cap-js/mcp
```

This adds MCP capabilities to your CAP server.

3. Restart the CAP server:
```bash
cds watch
```

You should see a message indicating MCP is enabled.

## Exercise 3.2 - Expose Service Over MCP

Now we'll manually configure MCP for your service.


1. **Annotates the service** (in `srv/matching-service.cds`) Just add the line at the end of the file:
```cds
annotate nameOfYourMatchingService with @odata @mcp;
```

Note: If you just add @mcp and have not mentioned @odata in your service, it would only serve mcp. Therefore we also mentioned @odata here.


## Exercise 3.3 - Verify MCP Connection

Let's check that Claude can see your MCP server.

1. Open Claude in a new terminal with the command claude

2. Run /mcp

3. You should see "invoice-matching" listed as a connected server


<br>![MCP Server](/exercises/ex3/images/03_03_0000.png)

## Exercise 3.4 - Test with Conversational Queries

Now for the fun part - talking to your business data!

### Test Prompt 1: Show Potential Matches

In Claude, type:

```
Show me potential matches
```

**What happens:**
- Claude calls your MCP server
- Fetches open invoices
- Fetches open purchase orders
- Analyzes potential matches
- Presents them in a table

<br>![Potential Matches](/exercises/ex3/images/03_03_0010.png)

### Test Prompt 2: Find Specific Invoice

```
Show me details for invoices from  Apex Office Supplies Ltd.
```

**What happens:**
- Claude queries Invoices with filter
- Returns matching records
- Formats nicely for you

### Test Prompt 3: Check Blocked Suppliers

```
Are there any invoices from blocked suppliers?
```

**What happens:**
- Claude joins invoice and supplier data
- Filters for blocked suppliers
- Warns you about payment risks

### Test Prompt 4: Match Analysis

```
For unmatched invoices, suggest the best PO match and explain why
```

**What happens:**
- Claude fetches all unmatched invoices
- For each, analyzes open POs
- Compares amounts, suppliers, descriptions
- Provides reasoning



## Exercise 3.5 - Understand What's Happening

Let's understand the MCP magic.

### How MCP Works

```
You (in Claude):
"Show me potential matches"
         ↓
Claude AI:
- Understands intent
- Plans tool calls
- Executes MCP tools
         ↓
MCP Server (your CAP app):
- Receives tool calls
- Queries database
- Returns results
         ↓
Claude AI:
- Analyzes data
- Formats response
- Shows you the answer
```

### What Tools Are Available?

Claude can now:
- **Read Invoices** - with filters
- **Read PurchaseOrders** - with filters
- **Read MatchDecisions** - with filters
- **Create MatchDecisions** - record matches
- **Call approve** - approve a match
- **Call reject** - reject a match
- **Call escalate** - escalate a match

### Why This is Powerful

**Before MCP:**
- Navigate UI manually
- Click through screens
- Remember filters
- Copy/paste data

**With MCP:**
- Ask in natural language
- Get instant answers
- AI does the navigation
- Formatted, analyzed results

## Exercise 3.6 - Advanced Queries

Try these more complex questions:

### Business Intelligence

```
What's the average invoice amount by supplier?
```

Claude will aggregate and analyze your data.

### Workflow Status

```
How many invoices are in each status (open, matched, escalated)?
```

Claude will count and show distribution.

### Risk Analysis

```
Which invoices have been open the longest without being matched?
```

Claude will sort by date and highlight aging invoices.

### Supplier Analysis

```
Show me all invoices for suppliers with payment terms longer than NET30
```

Claude will join supplier data and filter.

## What You've Accomplished

✅ Installed CAP MCP package  
✅ Exposed matching service over MCP  
✅ Configured Claude Desktop connection  
✅ Queried business data conversationally  
✅ Understood MCP architecture  
✅ Tested complex analytical queries

## Understanding What Happened

**Traditional Approach:**
- Build custom API endpoints
- Write OpenAPI specs
- Create AI integration layer
- Days of development

**AI-Assisted Approach:**
- One `npm add` command
- One Claude prompt
- Automatic MCP exposure
- 15 minutes total

**What You Learned:**
- What MCP is and how it works
- How CAP automatically exposes MCP tools
- How AI can query business data
- The power of conversational interfaces

## Troubleshooting


### Issue: Claude can't see data
**Solution:** Ensure CAP server is running:
```bash
cd /path/to/invoice-matching
cds watch
```

Open localhost:4004 and check if the mcp protocol is served.

### Issue: Permission errors
**Solution:** MCP runs with file system permissions. Check that your user can read the project directory.

## Security Considerations

### What's Exposed

MCP exposes:
- ✅ Read access to business data
- ✅ Write operations you configure
- ✅ Actions you explicitly allow

### What's Protected

- ❌ Database credentials (never exposed)
- ❌ Internal implementation
- ❌ Operations without @mcp annotations

### Best Practices

1. **Limit write operations** - only expose necessary mutations
2. **Use authentication** - in production, configure OAuth
3. **Audit logging** - track all MCP access (we'll add this in Ex 6)
4. **Rate limiting** - configure request throttling

## Optional Enhancements


### Add Descriptions

```
Add detailed descriptions to each MCP tool so Claude understands when to use them
```

### Add Resources

```
Create MCP resources for common queries like "unmatched-invoices" and "pending-decisions"
```

## Next Steps

Now that Claude can access your data via MCP, let's add semantic search so it can find matches based on meaning, not just keywords!

**Continue to [Exercise 4 - Add Semantic Search](../ex4/README.md)** to implement AI-powered similarity matching.

---

## Summary

In this exercise, you:
- ✅ Exposed CAP service over MCP
- ✅ Connected Claude Desktop
- ✅ Queried business data conversationally
- ✅ Understood MCP architecture
- ✅ Tested complex queries

**Time spent:** ~15 minutes  
**MCP tools exposed:** ~10 (entities + actions)  
**Custom API code:** 0 lines!  
**AI access:** ✅ Enabled!

Your business data is now AI-accessible! Next: Semantic search for smarter matching. 🚀
