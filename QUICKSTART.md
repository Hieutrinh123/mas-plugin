# Quick Start Guide - Feature Builder

Get started with the Feature Builder multiagent system in 5 minutes.

---

## Installation

```bash
# 1. Copy the plugin to your Claude Code skills directory
cp -r "mas plugin" ~/.claude/skills/feature-builder

# 2. Verify installation
ls ~/.claude/skills/feature-builder/skills/feature-builder
```

You should see: `feature-builder.md`, `evaluator.md`, `orchestrator.md`, and `subagents/`

---

## Your First Feature

### Step 1: Invoke the Skill

```bash
# In Claude Code CLI
/feature-builder Create a simple contact form
```

### Step 2: Answer Questions

The Evaluator will ask clarifying questions:

```
┌─────────────────────────────────────────────────────────────────┐
│ Where should this form be used?                                 │
│ ○ Standalone page (Recommended)                                 │
│ ○ Modal dialog                                                  │
│ ○ Embedded component                                            │
└─────────────────────────────────────────────────────────────────┘
```

Answer each question. The Evaluator is gathering requirements to create a solid plan.

### Step 3: Review the Plan

The Evaluator creates a plan file:

```
📄 Plan ready at: .mas/plans/contact_form_plan.md
```

Open this file in your editor and review:
- Does the technical approach make sense?
- Are all requirements captured?
- Are the acceptance criteria correct?

### Step 4: Provide Feedback or Approve

**Option A: Request changes**
```
You: Change the email service from SendGrid to Resend
```

The Evaluator will update the plan and ask for another review.

**Option B: Approve the plan**
```
You: approved
```

### Step 5: Implementation Begins

The Orchestrator takes over:

```
Orchestrator: Spawning agents...
  ├── [Backend Agent] Spawned ✓
  ├── [Frontend Agent] Spawned ✓
  └── [Testing Agent] Spawned ✓

  [Backend] Creating API endpoint... ✓
  [Frontend] Creating ContactForm component... ✓
  [Testing] Writing integration tests... ✓
```

### Step 6: Quality Review

The Evaluator reviews the completed work:

```
Evaluator: Review complete.
  ✓ Requirements met
  ✓ Code quality passed
  ✓ Security review passed

  APPROVED ✓ → Feature delivered
```

---

## What You Get

After approval, you'll have:

**Backend Files** (if needed):
- API endpoints (`src/api/contact/route.ts`)
- Database models (if persistence required)
- Email service integration

**Frontend Files** (if needed):
- React components (`src/components/ContactForm.tsx`)
- State management (if complex)
- Styling (matching your project's approach)

**Test Files**:
- Unit tests (`src/components/ContactForm.test.tsx`)
- Integration tests (`src/api/contact/contact.integration.test.ts`)
- E2E tests (if critical flow)

**Documentation**:
- Plan file (`.mas/plans/contact_form_plan.md`)
- Implementation notes
- Usage instructions

---

## Common Scenarios

### Scenario 1: Frontend-Only Feature

```
/feature-builder Add a dark mode toggle to settings
```

**What happens**:
- Evaluator asks about persistence (localStorage? User account?)
- Creates plan with ONLY Frontend Agent required
- Orchestrator spawns just the Frontend Agent
- Delivers: Theme context, toggle component, localStorage persistence

### Scenario 2: Backend-Only Feature

```
/feature-builder Create a /api/users/export endpoint that returns CSV
```

**What happens**:
- Evaluator asks about authentication, rate limiting
- Creates plan with ONLY Backend Agent required
- Orchestrator spawns just the Backend Agent
- Delivers: API endpoint, CSV generation logic, tests

### Scenario 3: Full-Stack Feature

```
/feature-builder Build a user dashboard with activity history
```

**What happens**:
- Evaluator asks about data to display, filters, permissions
- Creates plan requiring Backend + Frontend + Testing
- Orchestrator spawns all three agents
- Delivers: API endpoints, React components, database queries, tests

---

## Tips for Best Results

### 1. Be Specific in Your Request

❌ **Vague**: "Add user features"
✅ **Specific**: "Add user registration with email verification and password reset"

### 2. Review Plans Carefully

The plan is your chance to ensure the feature is built correctly. Check:
- Technical approach (does the architecture make sense?)
- Security considerations (is it secure?)
- Scope (is anything missing or extra?)

### 3. Iterate on Plans Before Approving

It's much faster to change a plan than refactor code later.

```
You: "Change the session approach to use JWT instead"
Evaluator: "Updated plan.md with JWT approach. Ready for review."
You: "approved"
```

### 4. Trust the Quality Review

If the Evaluator finds issues, it will:
1. Send structured feedback to the Orchestrator
2. Orchestrator will re-spawn affected agents with fix instructions
3. Agents will apply targeted fixes
4. Re-submit for review

This happens automatically (max 3 iterations, then escalates to you).

---

## Understanding the Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. REQUIREMENTS    │ Evaluator asks questions                   │
│    GATHERING       │ You answer                                 │
│                    │ Complete picture emerges                   │
├────────────────────┼────────────────────────────────────────────┤
│ 2. STRATEGIC       │ Evaluator creates plan.md                  │
│    PLANNING        │ Analyzes architecture options              │
│                    │ Identifies agents needed                   │
├────────────────────┼────────────────────────────────────────────┤
│ 3. HUMAN REVIEW    │ You review plan.md                         │
│    ⏸️ BLOCKS       │ Request changes OR approve                 │
│                    │ No code written until approval             │
├────────────────────┼────────────────────────────────────────────┤
│ 4. IMPLEMENTATION  │ Orchestrator spawns agents dynamically     │
│                    │ Agents work in parallel (where possible)   │
│                    │ Orchestrator coordinates integration       │
├────────────────────┼────────────────────────────────────────────┤
│ 5. QUALITY REVIEW  │ Evaluator reviews against 5 criteria       │
│    (FEEDBACK LOOP) │ If issues: send feedback → fix → re-review │
│                    │ Max 3 iterations, then escalate            │
├────────────────────┼────────────────────────────────────────────┤
│ 6. DELIVERY        │ Evaluator approves                         │
│                    │ Feature is production-ready                │
│                    │ Plan status: COMPLETED                     │
└────────────────────┴────────────────────────────────────────────┘
```

---

## Plan File Location

Plans are stored in: `.mas/plans/{feature_name}_plan.md`

Example:
```
.mas/
  plans/
    contact_form_plan.md
    user_authentication_plan.md
    dark_mode_plan.md
```

These files:
- Serve as documentation of what was built
- Can be version-controlled with git
- Show the decision-making process
- Provide context for future changes

---

## Approval Keywords

To approve a plan, say any of:
- `approved`
- `proceed`
- `looks good`
- `go ahead`
- `start implementation`

To reject a plan:
- `reject`
- `start over`

To request changes:
- `Change X to Y`
- `I want to use Z instead`
- `Add feature W`

---

## Checking Progress

While implementation is running, the plan.md file is updated with progress:

```markdown
## Implementation Progress

### Backend Agent
- [x] User model created
- [x] Session middleware implemented
- [ ] Auth endpoints (in progress)

### Frontend Agent
- [x] LoginForm component created
- [ ] API integration (pending backend completion)
```

Watch this file to see real-time progress.

---

## What If Something Goes Wrong?

### If Evaluator Finds Issues

The system will automatically:
1. Identify the problem
2. Tell the Orchestrator what to fix
3. Re-spawn affected agents
4. Apply fixes
5. Re-review

You'll see messages like:

```
Evaluator: Found 1 issue. Requesting fix...
Orchestrator: Re-spawning Backend Agent with fix instructions...
Backend Agent: Applied fix for SEC-001
Evaluator: Re-reviewing... APPROVED ✓
```

### If Issues Persist After 3 Attempts

The Evaluator will escalate to you:

```
🚨 IMPLEMENTATION BLOCKED

These issues persist after 3 fix attempts:
• [SEC-001] Password hashing still not implemented correctly

Recommended actions:
1. Review the affected file: src/api/auth/login.ts
2. Provide guidance in chat
3. Or: Say "skip SEC-001" to accept current implementation
```

At this point, you can:
- Review the code yourself and provide specific guidance
- Ask the Evaluator to explain the issue in more detail
- Accept the current implementation (if the issue is minor)

---

## Example: End-to-End

```
You: /feature-builder Create a simple blog post creation form

Evaluator: I need to understand the requirements better.
           [Asks 3 questions about rich text, image uploads, draft saving]

You: [Answers: Yes to rich text, No to images, Yes to draft saving]

Evaluator: Creating plan...
           📄 Plan ready at: .mas/plans/blog_post_form_plan.md

You: [Reviews plan in editor]
     Change the rich text editor from TinyMCE to Tiptap

Evaluator: Updated plan.md with Tiptap editor.
           📄 Plan updated. Ready for review.

You: approved

Evaluator: Plan approved. Handing off to Orchestrator.

Orchestrator: Spawning agents... [FE + BE + Testing]
              [Implementation happens]
              Submitting for review...

Evaluator: Review complete.
           ✓ All criteria passed
           APPROVED ✓

           Files created:
           - src/components/BlogPostForm.tsx
           - src/api/posts/route.ts
           - src/api/posts/drafts/route.ts
           - tests/BlogPostForm.test.tsx

           Feature delivered ✅
```

---

## Next Steps

1. **Try a simple feature** to get comfortable with the workflow
2. **Review the plan.md files** to understand how planning works
3. **Experiment with different feature types** (frontend-only, backend-only, full-stack)
4. **Read the full README.md** for advanced usage and architecture details

---

## Getting Help

- **Full documentation**: See [README.md](README.md)
- **Architecture details**: See [MULTIAGENT_SYSTEM_PLAN.md](MULTIAGENT_SYSTEM_PLAN.md)
- **Agent prompts**: See files in `skills/feature-builder/`

---

**You're ready to build production-ready features with AI assistance!** 🚀

Start with: `/feature-builder [your feature description]`
