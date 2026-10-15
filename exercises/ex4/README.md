# Exercise 4 — Add Semantic Search

In this exercise, you will add semantic search capabilities to find matches based on meaning, not just exact keywords.

## Learning Objectives

By the end of this exercise, you will:
- Understand semantic search vs keyword search
- Install AI and vector search packages
- Implement local embedding generation
- Search by meaning using vector similarity
- Test semantic matching with Claude

## What is Semantic Search?

**Keyword Search** (what we have now):
- "laptop" matches "laptop" ✅
- "laptop" does NOT match "notebook computer" ❌
- Exact word matching only

**Semantic Search** (what we'll build):
- "laptop" matches "laptop" ✅
- "laptop" matches "notebook computer" ✅  
- "office furniture" matches "desk and chairs" ✅
- Understands meaning and context

### Why It Matters for Invoice Matching

Real-world data is messy:
- Invoice says: "Office supplies - paper products"
- PO says: "Printer paper and copy paper"
- **Same thing, different words!**

Semantic search finds these matches that humans would spot but keyword search misses.

## Exercise 4.1 — Install AI and Vector Search Packages

Let's add the necessary packages for semantic search.

👉 Make sure you're in the project root:
```bash
pwd
# Should show: .../invoice-matching
```

👉 Install AI and vector search packages:
```bash
npm add -D @cap-js/ai @cap-js/sqlite@^3.1 @huggingface/hub@^2.15.0 @huggingface/tokenizers@0.1.3 onnxruntime-node@1.20.1
```

The packages will download. This may take a minute.

**What these packages do:**
- `@cap-js/ai` - CAP AI integration layer
- `@cap-js/sqlite@^3.1` - SQLite with vector support
- `@huggingface/hub` - Access to Hugging Face models
- `@huggingface/tokenizers` - Text tokenization
- `onnxruntime-node` - Run ML models locally



## Exercise 4.2 — Add Semantic Search with Claude

Now we'll use Claude to implement semantic search.

> [!TIP]
> Claude will add vector embeddings, implement similarity search, and update the service — all from one prompt.

### The Claude Prompt

👉 Copy this exact prompt into Claude:

```text
/cap-developer Add semantic search to the matching service. Invoice descriptions and PO item descriptions should be searchable by meaning, not just keywords. Use the local embedding model – no HANA required.
```

### What Claude Will Generate

Claude should:

1. **Extend the Data Model** (db/schema.cds):
   - Add vector fields to Invoices and PurchaseOrders
   - Configure embedding dimensions (typically 384 or 768)

2. **Configure AI Service** (package.json):
   - Enable @cap-js/ai
   - Configure local embedding model
   - Set vector dimensions

3. **Implement Embedding Generation** (srv/matching-service.js):
   - Generate embeddings on CREATE/UPDATE
   - Store vectors in database
   - Handle errors gracefully

4. **Add Similarity Search** (srv/matching-service.js):
   - Create `searchMatches` action
   - Calculate vector similarity
   - Return ranked results with scores

5. **Extend MCP** (srv/mcp.cds):
   - Expose searchMatches as MCP tool
   - Add description for Claude to understand


<details><summary><h2>Optional Review the Generated Code</h2></summary>

Check these key features:

**Data Model Extensions:**<br>
- Are vector fields added to Invoices and POs? Check the schema file.<br>
- Is the vector dimension specified?<br>
- Are vectors configured correctly?<br>

**AI Configuration:**<br>
- Is @cap-js/ai configured in package.json?<br>
- Is a local model specified (e.g., "sentence-transformers/all-MiniLM-L6-v2")?<br>
- Are dimensions matching the model?<br>

**Service Implementation:**<br>
- Are embeddings generated automatically on create/update?<br>
- Is there a searchMatches action?<br>
- Does it calculate cosine similarity?<br>
- Does it return ranked results?<br>

</details>

## Exercise 4.3 — Test


👉 Start the CAP server:
```bash
cds watch
```

The first time it runs, it will:
- Download the embedding model (~25MB)
- Generate embeddings for existing data
- This may take 30-60 seconds


## Exercise 4.4 — Test Semantic Search with Claude

Now let's test the semantic search through MCP!

### Test 1: Furniture Search

👉 In Claude, type:

```text
Run the search matches function and show similarities for 'office furnitures'
```

**What should happen:**
- Claude calls the searchMatches MCP tool
- Passes "office furnitures" as the query
- Gets back results like:
  - "Desk and chairs for new office" (high similarity)
  - "Conference room furniture" (high similarity)
  - "Office supplies" (lower similarity)


### Test 2: Computer Equipment

👉 Type:

```text
Search for matches related to 'laptop computers'
```

**Expected matches:**
- "Notebook computers for dev team" ✅
- "Portable computers" ✅
- "MacBook Pro" ✅
- "Mobile workstations" ✅



## Exercise 4.5 — Understand Vector Embeddings

Let's understand what's happening under the hood.

### What are Embeddings?

An **embedding** is a numerical representation of text:

```text
"office furniture" → [0.12, -0.45, 0.78, ..., 0.23]
                     (384 or 768 numbers)
```

**Key properties:**
- Similar meanings → similar vectors
- Different meanings → different vectors
- Can calculate similarity with math

### How Similarity Works

**Cosine Similarity:**
```text
similarity = cos(angle between vectors)
  1.0 = identical
  0.8 = very similar
  0.5 = somewhat similar
  0.0 = unrelated
```

**Example:**
```text
"laptop computer"     [0.8, 0.2, -0.1, ...]
"notebook computer"   [0.75, 0.25, -0.05, ...]
                      → similarity: 0.95 (very similar!)

"laptop computer"     [0.8, 0.2, -0.1, ...]
"office chair"        [-0.1, 0.6, 0.4, ...]
                      → similarity: 0.15 (not similar)
```

### The Local Model

We're using a Hugging Face model:
- **Model:** sentence-transformers/all-MiniLM-L6-v2
- **Size:** ~25MB
- **Dimensions:** 384
- **Language:** English
- **Speed:** ~50ms per embedding
- **Quality:** Good for general text

**Why local?**<br>
✅ No external API needed<br>
✅ No API costs<br>
✅ Works offline<br>
✅ Data privacy<br>


## Exercise 4.6 — Integrate with Match Finding

Now semantic search can help find invoice-PO matches.

👉 Ask Claude to update the findMatches logic:

```text
Update the findMatches action to use semantic similarity on descriptions when amounts are close but not exact
```

Claude will:
- Calculate semantic similarity between invoice and PO descriptions
- Combine with amount proximity
- Return matches with confidence scores
- Prioritize semantic+amount matches

Now when Maria looks for matches:
- Amount within 10% → check semantic similarity
- Semantic similarity > 0.8 → suggest as match
- Show both scores to Maria

## What You've Accomplished

✅ Installed AI and vector search packages  
✅ Added semantic search capability  
✅ Implemented local embedding generation  
✅ Tested similarity search with Claude  
✅ Understood vector embeddings  
✅ Compared keyword vs semantic search

## Understanding What Happened

**Traditional Approach:**
- Research vector databases
- Set up HANA or pgvector
- Write embedding generation code
- Implement similarity algorithms
- 2-3 days of work

**AI-Assisted Approach:**
- Install packages
- One Claude prompt
- Everything configured
- 20 minutes total

**What You Learned:**
- How semantic search works
- What embeddings are
- Local AI models
- Vector similarity
- Real-world matching improvements



<details><summary><h2>Optional Enhancements (can be skipped onsite at TechEd)</h2></summary>

### Multi-language Support

```text
Add support for German invoice descriptions using a multilingual embedding model
```

### Tunable Thresholds

```text
Let Maria adjust the similarity threshold (0.6 to 0.9) to see more or fewer matches
```

### Explanation

```text
When showing semantic matches, explain which words/phrases drove the similarity
```

### Hybrid Search

```text
Combine semantic search with exact keyword matching for optimal results
```
</details>



## Summary

In this exercise, you:<br>
✅ Added semantic search with local embeddings<br>
✅ Implemented vector similarity<br>
✅ Tested meaning-based matching<br>
✅ Understood embeddings and vectors<br>
✅ Improved real-world matching<br>

**Time spent:** ~20 minutes  
**Model size:** ~25MB  
**Embedding dimensions:** 384  
**Search quality:** 🚀 Much better!


## Next Steps

Now that you have intelligent semantic matching, let's wrap it all in a conversational AI agent that can help Maria find and approve matches!

**Continue to [Exercise 5 - Add Agent with Skills](../ex5/README.md)** to build an intelligent assistant for Maria.
