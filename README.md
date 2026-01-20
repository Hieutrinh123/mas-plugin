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
                    │  EVALUATOR (Opus)    │──► Requirements Gathering
                    │  • AskUserQuestion   │──► Strategic Planning
                    │  • Creates plan.md   │──► Quality Assurance
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │   🧑 HUMAN REVIEW    │──► Approves/Modifies Plan
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │ ORCHESTRATOR (Sonnet)│──► Task Decomposition
                    │  • Spawns agents     │──► Coordination
                    └──────────┬───────────┘
                               ▼
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                      ▼
  ┌──────────┐          ┌──────────┐          ┌──────────┐
  │ Backend  │          │ Frontend │          │ Testing  │
  │  Agent   │          │  Agent   │          │  Agent   │
  └──────────┘          └──────────┘          └──────────┘
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

5. **Implementation begins**:
The Orchestrator creates a git worktree and spawns necessary agents (Backend, Frontend, Testing). You'll see real-time progress:
```
🔧 IMPLEMENTATION IN PROGRESS

Backend Agent:    [████████░░] 80% (4/5 tasks)
Frontend Agent:   [██████████] 100% (3/3 tasks)
Testing Agent:    [████░░░░░░] 40% (2/5 test suites)
```

6. **Quality review**:
The Evaluator reviews code in the worktree against 5 criteria and either approves or requests fixes (max 5 iterations).

7. **Merge to main**:
On approval, the Orchestrator merges the worktree to main branch.

8. **Delivery**:
You receive production-ready code with comprehensive tests!

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

### Phase 1: Requirements Gathering
The Evaluator (Opus 4.5) uses interactive questions to build a complete understanding of what you want.

### Phase 2: Strategic Planning
The Evaluator creates a comprehensive plan covering:
- Functional and non-functional requirements
- Technical approach with rationale
- Implementation phases
- Risk assessment
- Acceptance criteria

**Optional Research**: For complex features, the Evaluator may spawn a Research agent to discover industry best practices, common pitfalls, and proven patterns (using WebSearch for current 2026 standards).

**Plan Templates**: For common patterns (CRUD APIs, Authentication, Dashboards), the Evaluator uses specialized templates that provide better structure and completeness.

### Phase 3: Human Review (BLOCKING)
**Nothing is implemented until you approve the plan.** This is your chance to steer the implementation.

### Phase 4: Implementation (Worktree Isolation)
The Orchestrator:
1. Creates a git worktree: `.mas/worktrees/{feature_name}`
2. Spawns specialized agents dynamically in the worktree:
   - **Backend Agent**: API endpoints, database models, business logic
   - **Frontend Agent**: UI components, styling, state management
   - **Testing Agent**: Unit, integration, and E2E tests
3. Provides real-time progress updates with visual progress bars
4. Agents run in parallel where possible, respecting dependencies

**Benefit**: All implementation happens in isolation. Your main branch stays clean.

### Phase 5: Quality Evaluation (In Worktree)
The Evaluator reviews code **in the worktree** (not main branch) against:
1. ✓ Requirements Compliance
2. ✓ Code Quality
3. ✓ Clean Code Standards
4. ✓ Security Review
5. ✓ Performance

If issues are found, structured feedback is sent to the Orchestrator, who re-spawns affected agents with fix instructions. Max 5 iterations, then escalates to human.

### Phase 6: Merge to Main
**Only on Evaluator approval**, the Orchestrator merges the worktree to main branch and cleans up the worktree.

**Benefit**: Main branch only receives quality-approved code.

### Phase 6: Delivery
Once approved, you receive:
- Production-ready implementation
- Comprehensive tests
- Documentation (plan.md)

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

## Plan Templates

The Evaluator uses specialized templates for common feature types:

### 1. CRUD API Template (`templates/crud-api-plan-template.md`)
**Use for**: RESTful resource endpoints (users, products, orders, etc.)

**Includes**:
- All 5 CRUD operations (Create, Read, Read All, Update, Delete)
- Pagination, sorting, filtering
- Input validation
- Database schema design
- Error handling patterns

---

### 2. Authentication Template (`templates/auth-plan-template.md`)
**Use for**: User authentication systems

**Includes**:
- Registration, login, logout flows
- Password management (hashing, reset, change)
- Session management (server-side sessions vs JWT)
- Email verification
- Rate limiting and account lockout
- OWASP security best practices

---

### 3. Dashboard Template (`templates/dashboard-plan-template.md`)
**Use for**: Analytics, reporting, or admin dashboards

**Includes**:
- KPI metrics cards with change indicators
- Chart/graph specifications
- Data tables with sorting, filtering, pagination
- Real-time updates
- Export functionality
- Mobile-responsive design

---

### 4. Generic Template (`templates/plan-template.md`)
**Use for**: Features that don't fit above patterns

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

### Implementation failed after 3 iterations

The Evaluator will escalate to you. Review the affected files and provide guidance or accept the current implementation.

---

## File Structure

```
feature-builder-plugin/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest
├── commands/
│   └── feature-builder.md             # /feature-builder slash command
├── skills/                            # Strategic agents (full context)
│   ├── feature-builder-evaluator/
│   │   └── SKILL.md                   # Evaluator - strategic planning & QA
│   └── feature-builder-orchestrator/
│       └── SKILL.md                   # Orchestrator - coordination
├── agents/                            # Execution agents (isolated contexts)
│   ├── backend-developer.md           # Backend implementation only
│   ├── frontend-developer.md          # Frontend implementation only
│   └── testing-specialist.md          # Testing implementation only
├── templates/
│   └── plan-template.md               # Plan file template
├── ARCHITECTURE.md                    # Architecture documentation
├── improvements-plan.md               # Roadmap for enhancements
├── README.md                          # This file
└── LICENSE                            # MIT License
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
