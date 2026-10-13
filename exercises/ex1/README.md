# Exercise 1 - Build React UI

In this exercise, you will use Claude to generate a complete React user interface for Maria's invoice matching workflow.

## Learning Objectives

By the end of this exercise, you will:
- Generate a React application with `cds add react`
- Use Claude to build a split-panel UI
- Implement match recording workflow
- Connect React to CAP OData services
- See immediate visual results

## What You Will Build

Maria's invoice matching interface with:
- **Left Panel:** List of open invoices (supplier, amount, description)
- **Right Panel:** List of open purchase orders (supplier, amount, description)
- **Match Recording:** Select invoice + PO, click "Record Match"
- **Pending Decisions:** List of proposed matches with Approve/Reject buttons
- **Dynamic Updates:** Re-fetch data after every action

## Exercise 1.1 - Add React to Your CAP Project

CAP provides a convenient command to add React applications.

1. Stop the preview on your right terminal and ensure you are in your main project folder:
```bash
pwd
# Should show: .../invoice-matching
```

2. Add React application:
```bash
cds add react --into matching
```

This creates a new React app in the `app/matching/` directory.

3. Navigate to the React app:
```bash
cd app/matching
```

4. Install dependencies:
```bash
npm install
```

5. Go back to project root:
```bash
cd ../..
```

## Exercise 1.2 - Generate the UI with Claude

Now we'll use Claude to generate the complete React UI!

> 💡 **AI-Assisted Development**: Claude will generate a complete, working UI from a single prompt describing Maria's needs.

### The Claude Prompt

Copy this exact prompt into Claude:

```
Build Maria's invoice matching UI in app/matching/.

Left panel: list of open invoices (supplier, amount, description). Right panel: list of open purchase orders (supplier, amount, description).

When Maria selects an invoice and a PO, show a "Record Match" button. Clicking it creates a match via POST, linking the selected invoice and PO with status "proposed". Recording a match should mark the linked invoice as matched and close its purchase order so both leave the open lists; rejecting the match should reopen both, while approving or escalating keeps them closed.

Below the two panels, show a "Pending decisions" section listing all match decisions with status proposed. Each row shows the linked invoice description, PO description, match confidence, match rationale, and two buttons: Approve and Reject. Approve calls match decisions approve action. Reject calls match decisions reject action.

After any action that changes what belongs in a list, re-fetch and re-render every affected list.

Fetch data from the CAP OData service at /odata/v4/matching/. Keep it simple – no routing, no state management library.
```

### What Claude Should Generate

Claude will create a React application with:

1. **App.jsx** - Main component with layout
2. **InvoiceList.jsx** - Left panel showing open invoices
3. **POList.jsx** - Right panel showing open POs
4. **PendingDecisions.jsx** - Bottom section with approve/reject
5. **API utilities** - Functions to fetch and post data
6. **CSS/Styling** - Basic styling for the panels

Run cds watch and check the generated UI. It should look like this:
<br>![Matching UI](/exercises/ex1/images/01_01_0030.png)

### Review the Generated Code

Check these key features:

**Component Structure:**
- ✅ Does it have separate components for invoices, POs, and decisions?
- ✅ Is there state management for selected items?
- ✅ Does it handle loading and error states?

**Data Fetching:**
- ✅ Does it fetch from `/odata/v4/matching/`?
- ✅ Does it filter for open invoices (`status eq 'open'`)?
- ✅ Does it filter for open POs (`isOpen eq true`)?
- ✅ Does it filter for proposed decisions (`outcome eq 'proposed'`)?

**User Interactions:**
- ✅ Can user select one invoice and one PO?
- ✅ Does "Record Match" button appear when both selected?
- ✅ Are there Approve/Reject buttons for each decision?
- ✅ Does it re-fetch after each action?

**UI Layout:**
- ✅ Are panels side-by-side (or stacked on mobile)?
- ✅ Is there visual feedback for selected items?
- ✅ Are amounts formatted with currency?


## Exercise 1.3 - Update CAP Service for UI Actions

We need to ensure the service implementation supports the UI actions.

Ask Claude to verify/generate the service implementation:

```
Make sure srv/matching-service.js has these implementations:
1. When a match is created with status "proposed", mark the invoice as "matched" and set PO isOpen to false
2. Action "approve" changes match outcome to "approved"  
3. Action "reject" changes match outcome to "rejected", reopens the invoice (status "open") and PO (isOpen true)
4. Action "escalate" changes match outcome to "escalated"
```

Claude will update or create the proper service handlers.

## Exercise 1.4 - Run the Application

Time to see it in action!

1. Start the CAP server (from project root):
```bash
cds watch
```

2. In a new terminal, start the React dev server:
```bash
cd app/matching
npm start
```

3. Open your browser at `http://localhost:3000`

You should see:
- **Left Panel:** List of open invoices
- **Right Panel:** List of open purchase orders
- **Bottom Section:** "Pending Decisions" (empty initially)



## Exercise 1.5 - Test the Workflow

Let's test Maria's matching workflow!

### Step 1: Record a Match

1. **Select an invoice** from the left panel (it should highlight)
2. **Select a PO** from the right panel (it should highlight)
3. **Click "Record Match"** button

What happens:
- Both items disappear from the open lists
- A new entry appears in "Pending Decisions"
- The decision shows as status "proposed"



### Step 2: Approve a Match

1. In the "Pending Decisions" section, find your match
2. **Click "Approve"** button

What happens:
- The decision status updates to "approved"
- It remains in the list (approved decisions stay closed)



### Step 3: Test Reject Workflow

1. Record another match (select invoice + PO)
2. This time, **click "Reject"** in pending decisions

What happens:
- The decision status updates to "rejected"
- The invoice reappears in the left panel (status back to "open")
- The PO reappears in the right panel (isOpen back to true)



## Exercise 1.6 - Explore the Code

Take a moment to understand what Claude generated.

### App.jsx Structure

```javascript
function App() {
  const [invoices, setInvoices] = useState([]);
  const [pos, setPOs] = useState([]);
  const [decisions, setDecisions] = useState([]);
  const [selectedInvoice, setSelectedInvoice] = useState(null);
  const [selectedPO, setSelectedPO] = useState(null);

  // Fetch functions
  // Record match function
  // Action functions (approve, reject)
  
  return (
    <div className="app">
      <InvoiceList ... />
      <POList ... />
      <PendingDecisions ... />
    </div>
  );
}
```

**Key Patterns:**
- ✅ State hooks for data and selections
- ✅ Effect hooks to fetch on mount
- ✅ Handler functions passed to child components
- ✅ Re-fetch after mutations

### Why No State Management Library?

We kept it simple:
- useState and useEffect are sufficient
- No complex state synchronization needed
- Easier to understand
- Less boilerplate

For larger apps, you might use Redux, Zustand, or Context API.

## What You've Accomplished

✅ Added React to your CAP project  
✅ Generated a complete UI with Claude  
✅ Connected React to CAP OData services  
✅ Implemented invoice matching workflow  
✅ Tested record, approve, and reject flows  
✅ Understood OData integration patterns

## Understanding What Happened

**Traditional Approach:**
- 2-3 hours writing React components
- Boilerplate for data fetching
- CSS for layout
- Testing and debugging

**AI-Assisted Approach:**
- 5 minutes with Claude
- Complete components generated
- Works immediately
- Focus on understanding, not typing

**What You Learned:**
- How to describe UI requirements to AI
- React component patterns
- OData integration with CAP
- State management basics

## Troubleshooting

### Issue: React app won't start
**Solution:** Check dependencies installed:
```bash
cd app/matching
npm install
```

### Issue: CORS errors
**Solution:** CAP automatically handles CORS in development. Make sure both servers are running.

### Issue: "Record Match" doesn't work
**Solution:** Check browser console for errors. Verify the service implementation has the correct logic.

### Issue: Data doesn't update after action
**Solution:** Ensure the UI re-fetches data after each POST request.

### Issue: Styling looks broken
**Solution:** Ask Claude to provide the CSS:
```
Can you provide the CSS for the invoice matching UI?
```

## Optional Enhancements

Want to customize the UI? Try these prompts:

### Add Search/Filter
```
Add a search box above each panel to filter by supplier name or description
```

### Improve Styling
```
Make the UI look more professional with better spacing, colors, and hover effects
```

### Add Confidence Slider
```
When recording a match, let Maria set the confidence level with a slider (0-100%)
```

### Show Match History
```
Add a "History" tab showing all approved and rejected decisions
```

## Next Steps

Now that Maria can see and match invoices manually, let's add supplier information from S/4HANA!

**Continue to [Exercise 2 - Add Supplier Data](../ex2/README.md)** to integrate the Supplier data product.

---

## Summary

In this exercise, you:
- ✅ Generated a complete React UI with AI
- ✅ Built a split-panel matching interface
- ✅ Implemented workflow state transitions
- ✅ Connected React to CAP OData
- ✅ Tested the full matching workflow

**Time spent:** ~20 minutes  
**React components written:** ~0 (Claude did it!)  
**React components reviewed:** ~4-5  
**Working UI:** ✅ Complete!

Maria can now match invoices visually! Next: Adding supplier details. 🚀
