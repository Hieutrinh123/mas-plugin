# Feature Builder - Claude Code Plugin

A sophisticated hierarchical multiagent system for building production-ready features through coordinated specialized agents.

## Overview

Feature Builder implements an **Evaluator-Orchestrator** pattern where strategic planning and quality control are separated from task execution. This architecture enables Claude Code to handle complex product development tasks with human-in-the-loop oversight.

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER REQUEST                            │
│                    (New Product/Feature)                        │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
                    ┌──────────────────────┐
                    │  EVALUATOR (Opus)    │──► Phase 1: Requirements Gathering
                    │  • AskUserQuestion   │──► Phase 2: Strategic Planning
                    │  • Spawns Research   │    (Optional: Research Agent)
                    │  • Creates plan.md   │    (Uses plan templates)
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │   🧑 HUMAN REVIEW    │──► Phase 3: Plan Approval (BLOCKING)
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │ ORCHESTRATOR (Sonnet)│──► Phase 4: Implementation
                    │  • Creates worktree  │    in .mas/worktrees/{name}
                    │  • Spawns agents     │──► Task Decomposition
                    │  • Progress tracking │──► Real-time updates
                    └──────────┬───────────┘
                               ▼
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                      ▼
  ┌──────────┐          ┌──────────┐          ┌──────────┐
  │ Backend  │          │ Frontend │          │ Testing  │
  │  Agent   │          │  Agent   │          │  Agent   │
  │(worktree)│          │(worktree)│          │(worktree)│
  └─────┬────┘          └─────┬────┘          └─────┬────┘
        └──────────────────────┼──────────────────────┘
                               ▼
                    ┌──────────────────────┐
                    │  EVALUATOR (Opus)    │──► Phase 5: Quality Review
                    │  • Reviews worktree  │    (in worktree, not main)
                    │  • 5 quality checks  │──► Max 5 fix iterations
                    └──────────┬───────────┘
                               │
                      ┌────────┴────────┐
                      ▼                 ▼
               ┌──────────┐      ┌──────────┐
               │ APPROVED │      │ ISSUES   │──► Back to Orchestrator
               └─────┬────┘      │  FOUND   │    (spawns agents for fixes)
                     │           └──────────┘
                     ▼
          ┌──────────────────────┐
          │ ORCHESTRATOR (Sonnet)│──► Phase 6: Merge to Main
          │  • Merges worktree   │    (only if approved)
          │  • Cleans up         │──► git merge feature/{name}
          └──────────┬───────────┘
                     ▼
          ┌──────────────────────┐
          │  EVALUATOR (Opus)    │──► Phase 7: Delivery
          │  • Delivery summary  │──► Production-ready code
          │  • Updates plan.md   │    on main branch
          └──────────────────────┘
```

## Key Features

### 🎯 Strategic Planning
- **Requirements Gathering**: Evaluator uses interactive questions to clarify ambiguous requirements
- **Industry Research**: Optional Research agent discovers best practices and common pitfalls
- **Plan Templates**: Specialized templates for CRUD APIs, Authentication, Dashboards
- **Comprehensive Plans**: Detailed plan files serve as single source of truth
- **Human-in-the-Loop**: Explicit approval required before implementation begins

### 🤖 Dynamic Agent Spawning
- **Smart Selection**: Orchestrator spawns only the agents needed (Backend, Frontend, Testing, Research)
- **Cost Efficient**: Don't spawn 3 agents when 1 will do
- **Parallel Execution**: Independent agents run simultaneously
- **Real-Time Progress**: Visual progress bars show completion status for each agent

### 🔐 Worktree Isolation
- **Safe Experimentation**: All implementation happens in isolated git worktrees
- **Protected Main Branch**: Only Evaluator-approved code merges to main
- **Clean History**: Failed attempts don't pollute git history
- **Easy Rollback**: Simply delete worktree to discard changes

### ✅ Quality Assurance
- **5 Evaluation Criteria**: Requirements compliance, code quality, clean code, security, performance
- **Feedback Loop**: Iterative fixes until quality standards met (max 5 iterations)
- **Worktree Review**: Evaluator reviews in isolation before merge
- **Human Escalation**: Persistent issues escalate to user after failed attempts

### 🔀 Flexible Workflows
- **Full Workflow**: `/feature-builder` - Complete feature development
- **Plan Only**: `/feature-builder:plan` - Create strategic plan without implementation
- **Review Only**: `/feature-builder:review` - Quality evaluation of existing code
- **Test Only**: `/feature-builder:test` - Generate tests for existing code

### 🔒 Production-Ready Code
- **Security First**: Input validation, password hashing, SQL injection prevention
- **Clean Code**: DRY principles, single responsibility, maintainability
- **Comprehensive Testing**: Unit, integration, and E2E tests
- **Best Practices**: Research-informed implementation based on 2026 standards

---

## Installation

### Option 1: Install to Plugins Directory (Recommended)

```bash
# Clone or download this repository
git clone <repository-url> feature-builder-plugin

# Copy to Claude Code plugins directory
cp -r feature-builder-plugin ~/.claude/plugins/feature-builder
```

Restart Claude Code if it's currently running.

### Option 2: Use with --plugin-dir Flag

```bash
# Load plugin from any location during development
claude --plugin-dir /path/to/feature-builder-plugin
```

### Verify Installation

```bash
# Start Claude Code
claude

# Check that the plugin is loaded
/help
# You should see /feature-builder in the available commands
```

---

## Quick Start

### Your First Feature (Full Workflow)

1. **Invoke the plugin**:
```
/feature-builder Create a simple contact form
```

2. **Answer clarifying questions**:
The Evaluator will ask targeted questions to understand your requirements (form fields, validation, backend handling, etc.)

3. **Review the plan**:
A detailed plan will be created at `.mas/plans/contact_form_plan.md` in your project directory. Review it carefully.

4. **Approve or request changes**:
```
# To approve:
approved

# To request changes:
Change the email service from SendGrid to Resend
```

5. **Worktree created**:
The Orchestrator creates an isolated git worktree at `.mas/worktrees/{feature_name}` on a new branch `feature/{feature_name}`.

6. **Implementation begins**:
Agents work in the worktree. You'll see real-time progress:
```
🔧 IMPLEMENTATION IN PROGRESS (in worktree)

Backend Agent:    [████████░░] 80% (4/5 tasks)
  ✓ User model created (src/models/User.ts)
  ✓ API endpoints (src/api/users.ts)
  ⏳ Tests (in progress)

Frontend Agent:   [██████████] 100% (3/3 tasks)
Testing Agent:    [████░░░░░░] 40% (2/5 test suites)

Overall: 73% complete
```

7. **Quality review in worktree**:
The Evaluator reviews code **in the worktree** (not main branch) against 5 criteria:
- Requirements compliance
- Code quality
- Clean code standards
- Security review
- Performance

If issues found, Orchestrator re-spawns affected agents to fix them (max 5 iterations).

8. **Merge to main**:
**Only after Evaluator approval**, the Orchestrator merges the worktree to main:
```bash
git merge feature/{feature_name}
```

9. **Delivery**:
You receive production-ready code on main branch with comprehensive tests! The worktree is cleaned up automatically.

### Partial Workflows

**Create a Plan Only** (no implementation):
```
/feature-builder:plan Design a user authentication system
```

**Review Existing Code**:
```
/feature-builder:review Review the authentication code in src/auth/
```

**Generate Tests Only**:
```
/feature-builder:test Generate tests for src/api/users.ts
```

---

## How It Works

### Complete Workflow (7 Phases)

```
Phase 1: Requirements → Phase 2: Planning → Phase 3: Approval (HUMAN)
     ↓
Phase 4: Implementation (Worktree) → Phase 5: Quality Review → Fix Loop (max 5x)
     ↓
Phase 6: Merge to Main → Phase 7: Delivery
```

### Phase 1: Requirements Gathering
**Agent**: Evaluator (Opus 4.5)

The Evaluator uses `AskUserQuestion` tool to build a complete understanding:
- Functional requirements (what should it do?)
- Technical preferences (stack, architecture)
- Security requirements
- Performance expectations
- Integration points

**Example Questions**:
- "Which authentication method should be supported?" (Email/Password, OAuth, Magic Link)
- "Should this be mobile-first or desktop-first?"
- "What are the expected load requirements?"

**Output**: Complete requirements understanding

---

### Phase 2: Strategic Planning
**Agent**: Evaluator (Opus 4.5)
**Optional**: Research Agent (Sonnet) - for complex/security-critical features

**Planning Process**:
1. **Optional Research**: For complex features (auth, payments, etc.), Evaluator spawns Research agent
   - Research agent uses WebSearch for current 2026 best practices
   - Discovers common pitfalls, security standards (OWASP), proven patterns
   - Returns structured research report with recommendations

2. **Select Template**: Evaluator identifies feature type:
   - CRUD API → `crud-api-plan-template.md`
   - Authentication → `auth-plan-template.md`
   - Dashboard → `dashboard-plan-template.md`
   - Other → `plan-template.md`

3. **Create Plan**: Writes comprehensive plan to `.mas/plans/{feature_name}_plan.md`:
   - Functional and non-functional requirements
   - Technical approach with rationale (informed by research if applicable)
   - Implementation phases (broken into specific tasks)
   - Risk assessment with mitigations
   - Acceptance criteria (testable conditions)
   - Which agents are needed (Backend, Frontend, Testing)

**Output**: `.mas/plans/{feature_name}_plan.md` (status: PENDING_REVIEW)

---

### Phase 3: Human Review (BLOCKING)
**Agent**: Evaluator (Opus 4.5)
**Gate**: Human approval required

The Evaluator presents the plan and **blocks** waiting for human response:

```
📄 Plan ready for your review at: .mas/plans/user_auth_plan.md

Please review and respond with:
• Feedback or questions (I'll update the plan)
• "approved" or "proceed" to begin implementation
• "reject" to discard and restart

⏸️ Waiting for human approval...
```

**Human Options**:
- Provide feedback → Evaluator updates plan → Review again
- Approve → Proceed to Phase 4
- Reject → Start over

**Output**: Approved plan (status: APPROVED)

---

### Phase 4: Implementation (Worktree Isolation)
**Agent**: Orchestrator (Sonnet)
**Subagents**: Backend, Frontend, Testing (Sonnet)

**Step 1**: Create isolated worktree
```bash
git worktree add .mas/worktrees/{feature_name} -b feature/{feature_name}
cd .mas/worktrees/{feature_name}
```

**Step 2**: Orchestrator analyzes plan and spawns only needed agents
- Read plan's "Agents Required" section
- Decompose tasks per agent
- Identify dependencies

**Step 3**: Spawn agents in parallel (where no dependencies)
```
Backend Agent spawned: implementing in worktree
Frontend Agent spawned: implementing in worktree
Testing Agent spawned: implementing in worktree
```

**Step 4**: Real-time progress tracking
```
🔧 IMPLEMENTATION IN PROGRESS (in worktree)

Backend Agent:    [████████░░] 80% (4/5 tasks completed)
  ✓ User model created (src/models/User.ts)
  ✓ Session middleware (src/middleware/session.ts)
  ✓ Register endpoint (src/api/auth/register.ts)
  ✓ Login endpoint (src/api/auth/login.ts)
  ⏳ Password reset endpoint (in progress)

Frontend Agent:   [██████████] 100% (3/3 tasks completed)
  ✓ LoginForm component (src/components/LoginForm.tsx)
  ✓ RegisterForm component (src/components/RegisterForm.tsx)
  ✓ Auth context (src/contexts/AuthContext.tsx)

Testing Agent:    [████░░░░░░] 40% (2/5 test suites)
  ✓ Unit tests for auth utilities
  ✓ Integration tests for login
  ⏳ Integration tests for registration (in progress)

Overall Progress: [███████░░░] 73% (9/13 total tasks)
```

**Step 5**: Submit to Evaluator when all agents complete
```
✅ IMPLEMENTATION COMPLETE

Worktree: .mas/worktrees/user_auth
Branch: feature/user_auth
Files created: 8 files
Files modified: 2 files

Ready for quality review.
```

**Output**: Completed implementation in worktree (main branch untouched)

---

### Phase 5: Quality Evaluation (In Worktree)
**Agent**: Evaluator (Opus 4.5)
**Location**: Reviews code in `.mas/worktrees/{feature_name}` (NOT main branch)

**Evaluation Process**:
1. **Change to worktree**: `cd .mas/worktrees/{feature_name}`

2. **Review against 5 criteria**:
   - ✓ **Requirements Compliance**: All requirements implemented?
   - ✓ **Code Quality**: Readable, DRY, appropriate abstractions?
   - ✓ **Clean Code**: Good naming, file organization, SOLID principles?
   - ✓ **Security**: Input validation, password hashing, SQL injection prevention, OWASP compliance?
   - ✓ **Performance**: Efficient algorithms, optimized queries, no N+1 problems?

3. **Outcome A - APPROVED**:
```markdown
✓ EVALUATION COMPLETE - APPROVED

Requirements Compliance: ✓ PASS
Code Quality: ✓ PASS
Clean Code Standards: ✓ PASS
Security Review: ✓ PASS
Performance: ✓ PASS

🔀 ORCHESTRATOR: Proceed with merge to main branch
```
→ Go to Phase 6

4. **Outcome B - REVISION NEEDED**:
```json
{
  "status": "revision_needed",
  "iteration": 1,
  "issues": [
    {
      "id": "SEC-001",
      "type": "security",
      "severity": "critical",
      "file": "src/api/auth/login.ts",
      "line": 42,
      "description": "Password not hashed before comparison",
      "required_fix": "Use bcrypt.compare() for secure verification",
      "assigned_agent": "backend"
    }
  ]
}
```
→ Orchestrator re-spawns Backend Agent with fix instructions
→ Backend Agent fixes in worktree
→ Submit for re-review
→ Repeat (max 5 iterations)

If issues persist after 5 iterations → Escalate to human

**Output**: Either approval or structured feedback

---

### Phase 6: Merge to Main
**Agent**: Orchestrator (Sonnet)
**Trigger**: Evaluator approval only

**Merge Process**:
```bash
# 1. Return to main repo
cd /path/to/main/repo

# 2. Ensure main is up to date
git checkout main
git pull origin main

# 3. Merge feature branch
git merge feature/{feature_name} --no-ff -m "feat: [Feature name]

[Description]

Plan: .mas/plans/{filename}_plan.md
Reviewed and approved by Evaluator"

# 4. Cleanup worktree
git worktree remove .mas/worktrees/{feature_name}

# 5. Delete feature branch (optional)
git branch -d feature/{feature_name}
```

**Output**:
- Feature merged to main branch
- Worktree cleaned up
- Main branch now has production-ready code

---

### Phase 7: Delivery
**Agent**: Evaluator (Opus 4.5)

**Delivery Summary**:
```markdown
✅ FEATURE DELIVERED: User Authentication System

Implementation Summary:
• Files created: 8 files
  - src/models/User.ts (User model with password hashing)
  - src/middleware/session.ts (Session management)
  - src/api/auth/register.ts, login.ts, logout.ts
  - src/components/LoginForm.tsx, RegisterForm.tsx
  - src/tests/auth.integration.test.ts

• Files modified: 2 files
  - package.json (added bcrypt, express-session)
  - src/app/layout.tsx (added AuthProvider)

• New endpoints: 3 API routes
  - POST /api/auth/register
  - POST /api/auth/login
  - POST /api/auth/logout

Quality Assurance:
✓ All requirements met (per plan.md)
✓ Code quality standards passed
✓ Security review passed (OWASP compliant)
✓ Tests passing (85% coverage)

The feature is now live on main branch.
```

**Plan Updated**: Status → COMPLETED

**Output**: Production-ready feature on main branch with full documentation

---

## Git Worktree Workflow

Feature Builder uses git worktrees to isolate implementation from your main branch.

### What Happens

**Before Implementation**:
```
your-project/              # Your main working directory
├── src/                   # Clean, untouched
├── package.json           # Clean, untouched
└── .mas/
    └── plans/
        └── feature_plan.md
```

**During Implementation** (Phase 4-5):
```
your-project/              # Main directory (unchanged)
└── .mas/
    ├── plans/
    │   └── feature_plan.md (status: IN_PROGRESS)
    └── worktrees/
        └── feature_name/  # Isolated worktree
            ├── src/       # Implementation happens here
            │   ├── models/User.ts (NEW)
            │   └── api/users.ts (NEW)
            └── package.json (MODIFIED)
```

**Key Points**:
- Your main branch is **untouched** during implementation
- All agent work happens in `.mas/worktrees/{feature_name}`
- Evaluator reviews code in the worktree
- If implementation fails, just delete the worktree - main is clean

**After Approval** (Phase 6-7):
```bash
# Orchestrator merges worktree to main
git checkout main
git merge feature/{feature_name}

# Worktree cleaned up automatically
# .mas/worktrees/{feature_name} removed
```

**Result**:
```
your-project/              # Now has the approved changes
├── src/
│   ├── models/User.ts     # Merged from worktree
│   └── api/users.ts       # Merged from worktree
├── package.json           # Merged changes
└── .mas/
    └── plans/
        └── feature_plan.md (status: COMPLETED)
```

### Benefits

✅ **Safe Experimentation**: Try different approaches without risk
✅ **Protected Main**: Your main branch only gets quality-approved code
✅ **Easy Rollback**: `git worktree remove` - no trace left
✅ **Parallel Development**: Multiple features in separate worktrees
✅ **Clean History**: Failed attempts never appear in git log

---

## Usage Examples

### Frontend-Only Feature

```
/feature-builder Add a dark mode toggle to settings
```

**Result**: Only Frontend Agent spawned, delivers theme context + toggle component + localStorage persistence

### Backend-Only Feature

```
/feature-builder Create a /api/users/export endpoint that returns CSV
```

**Result**: Only Backend Agent spawned, delivers API endpoint + CSV generation + authentication

### Full-Stack Feature

```
/feature-builder Build a user dashboard with activity history
```

**Result**: All agents spawned, delivers complete feature with backend API + frontend UI + comprehensive tests

---

## Plan Files

Plans are stored in `.mas/plans/` in your project directory (not in the plugin):

```
your-project/
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
- Track implementation progress

---

## Architecture

### Hybrid Skills + Agents Pattern

Feature Builder uses a **hybrid architecture** to prevent cross-contamination between domains:

**Strategic Layer (Skills)** - Need full context for coordination:
- **Evaluator** (`skills/`) - Requirements, planning, quality assurance
- **Orchestrator** (`skills/`) - Task decomposition, agent coordination

**Execution Layer (Agents)** - Isolated contexts for domain-specific work:
- **Backend** (`agents/`) - API, database, business logic (isolated from frontend knowledge)
- **Frontend** (`agents/`) - UI, styling, state management (isolated from backend knowledge)
- **Testing** (`agents/`) - Unit, integration, E2E tests (isolated from implementation)

**Why This Matters**: Agents run in isolated contexts, preventing the Frontend Agent from accidentally accessing backend skill knowledge and vice versa. See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed explanation.

### Agent Roles

**Evaluator (Opus 4.5)** - Strategic Skill
- Strategic intelligence
- Requirements gathering
- Plan creation
- Quality assurance
- Human-in-the-loop coordination

**Orchestrator (Sonnet)** - Coordination Skill
- Task decomposition
- Dynamic agent spawning from `agents/` folder
- Dependency management
- Progress tracking
- Feedback handling

**Backend Agent (Sonnet)** - Isolated Execution Agent
- API endpoints
- Database models/migrations
- Business logic
- Authentication/authorization
- Server-side validation

**Frontend Agent (Sonnet)** - Isolated Execution Agent
- UI components
- Styling & responsive design
- State management
- Forms & client validation
- Accessibility

**Testing Agent (Sonnet)** - Isolated Execution Agent
- Unit tests (70%)
- Integration tests (20%)
- E2E tests (10%)
- Coverage reporting

**Research Agent (Sonnet)** - Best Practices Discovery
- Web search for current best practices (2026)
- Analyze common pitfalls and vulnerabilities
- Provide evidence-based recommendations
- Cite authoritative sources (OWASP, official docs, etc.)

---

## Available Commands

### Full Workflow
**`/feature-builder [description]`** - Complete feature development lifecycle
- Requirements gathering → Planning → Approval → Implementation → Review → Delivery
- Creates worktree for isolated implementation
- Real-time progress tracking
- Quality evaluation with up to 5 fix iterations
- Merges to main only after Evaluator approval

**Example**:
```
/feature-builder Create a user authentication system with email/password
```

---

### Partial Workflows

**`/feature-builder:plan [description]`** - Strategic planning only (no implementation)
- Requirements gathering
- Creates comprehensive plan at `.mas/plans/{feature_name}_plan.md`
- Waits for human approval
- **STOPS** after approval (does not implement)

**Use when**: You want to explore a feature idea, get a plan document, or review with your team before coding.

**Example**:
```
/feature-builder:plan Design a payment processing system with Stripe
```

---

**`/feature-builder:review [scope]`** - Quality evaluation only
- Reviews existing code using Evaluator's 5 quality criteria
- Generates detailed feedback report
- No implementation or fixes (report only)

**Use when**: You want to evaluate existing code quality, find security issues, or get improvement recommendations.

**Example**:
```
/feature-builder:review Review the authentication code in src/auth/
/feature-builder:review Review src/api/users.ts
```

---

**`/feature-builder:test [scope]`** - Test generation only
- Spawns Testing Agent to create tests for existing code
- Follows test pyramid (70% unit, 20% integration, 10% E2E)
- No application code changes

**Use when**: You have existing code that needs tests, or you want to improve test coverage.

**Example**:
```
/feature-builder:test Generate tests for src/api/users.ts
/feature-builder:test Add integration tests for the authentication system
```

---

## Quality Criteria

All code is evaluated against:

### 1. Requirements Compliance
- All functional requirements implemented
- All acceptance criteria met
- No features skipped

### 2. Code Quality
- Readable, self-documenting code
- Small, focused functions
- DRY principle followed
- Appropriate error handling

### 3. Clean Code Standards
- Descriptive naming
- Logical organization
- SOLID principles
- Comments only where needed

### 4. Security Review
- Input validation
- Password hashing (bcrypt/argon2)
- Parameterized queries (no SQL injection)
- No hardcoded secrets
- XSS prevention
- OWASP Top 10 compliance

### 5. Performance
- Efficient algorithms
- Optimized database queries
- Appropriate caching
- No unnecessary dependencies

---

## Troubleshooting

### Plugin not found

```bash
# Check plugin is in the right location
ls ~/.claude/plugins/feature-builder/.claude-plugin/plugin.json

# Or use --plugin-dir flag
claude --plugin-dir /path/to/feature-builder-plugin
```

### /feature-builder command not showing

Restart Claude Code after installing the plugin.

### Evaluator keeps asking questions

This means your initial request was too vague. Provide more detail or answer the questions to build a complete picture.

### Plan doesn't match expectations

Provide feedback! The Evaluator will update the plan based on your input. Don't approve until you're satisfied.

### Implementation failed after 5 iterations

The Evaluator will escalate to you. Review the affected files in the worktree (`.mas/worktrees/{feature_name}`) and provide guidance or accept the current implementation.

### Worktree still exists after delivery

If something went wrong, you can manually clean up:
```bash
# Remove worktree
git worktree remove .mas/worktrees/{feature_name} --force

# Delete feature branch
git branch -D feature/{feature_name}
```

### Changes not showing on main branch

Check if the worktree merge completed:
```bash
# See if feature branch was merged
git log --oneline --graph -10

# Check worktree status
git worktree list
```

If the worktree is still listed, the merge didn't complete. This usually means the Evaluator found issues.

### Want to manually inspect worktree

```bash
# Navigate to worktree
cd .mas/worktrees/{feature_name}

# This is a full git working directory
git status
git log
```

### Merge conflicts

If merge conflicts occur, the Orchestrator will report them. You can:
1. Let the Orchestrator handle them (recommended)
2. Manually resolve in the worktree and let Orchestrator retry merge
3. Abandon the worktree and start over

---

## File Structure

```
feature-builder-plugin/
├── .claude-plugin/
│   └── plugin.json                         # Plugin manifest
├── commands/                               # Entry point commands
│   ├── feature-builder.md                  # /feature-builder (full workflow)
│   ├── feature-builder-plan.md             # /feature-builder:plan (plan only)
│   ├── feature-builder-review.md           # /feature-builder:review (review only)
│   └── feature-builder-test.md             # /feature-builder:test (test only)
├── skills/                                 # Strategic agents (full context)
│   ├── feature-builder-evaluator/
│   │   └── SKILL.md                        # Evaluator - planning & QA & research
│   └── feature-builder-orchestrator/
│       └── SKILL.md                        # Orchestrator - worktree & coordination
├── agents/                                 # Execution agents (isolated contexts)
│   ├── backend-developer.md                # Backend implementation only
│   ├── frontend-developer.md               # Frontend implementation only
│   ├── testing-specialist.md               # Testing implementation only
│   └── researcher.md                       # Research best practices (NEW)
├── templates/                              # Plan templates
│   ├── plan-template.md                    # Generic template
│   ├── crud-api-plan-template.md           # CRUD API template (NEW)
│   ├── auth-plan-template.md               # Authentication template (NEW)
│   └── dashboard-plan-template.md          # Dashboard template (NEW)
├── README.md                               # This file
└── LICENSE                                 # MIT License
```

---

## Design Decisions

### Why Opus 4.5 for Evaluator?
- Superior reasoning for complex requirement analysis
- Better judgment for quality evaluation
- More nuanced feedback for improvements
- Strategic thinking for architecture decisions

### Why Sonnet for Orchestrator & Subagents?
- Faster execution for task-level work
- Cost-effective for high-volume operations
- Sufficient capability for focused implementation
- Better latency for iterative development

### Why plan.md as Source of Truth?
- Human-readable (non-technical stakeholders can review)
- Version-controllable (git tracks changes)
- IDE-friendly (opens in VS Code, renders nicely)
- Editable (humans can modify directly)
- Persistent (survives session restarts)

### Why Dynamic Agent Spawning?
- Cost efficiency (don't spawn agents you don't need)
- Faster execution (less coordination overhead)
- Focused output (agents only produce what's needed)

---

## Best Practices

### For Users

1. **Be specific in your initial request**: More detail = better requirements gathering
2. **Review plans carefully**: This is your chance to steer the implementation
3. **Ask questions**: If something in the plan isn't clear, ask the Evaluator
4. **Iterate on plans before approving**: Changing a plan is cheaper than refactoring code

### For Feature Requests

**Good requests**:
- "Add user registration with email verification and password reset"
- "Create a /api/products endpoint with filtering by category and price range"
- "Build a responsive navigation menu with dropdown support"

**Vague requests** (will need more clarification):
- "Add user features"
- "Make it better"
- "Fix the thing"

---

## License

MIT License - see [LICENSE](LICENSE) file for details

---

## Version

**v1.0.0** - Initial plugin release

**Architecture**: Evaluator-Orchestrator Multiagent System
**Platform**: Claude Code Plugin
**Models**: Opus 4.5 (Evaluator) + Sonnet (Orchestrator & Subagents)

---

*Built for production-ready feature development with AI*
