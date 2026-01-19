# Multiagent System Plugin for Claude Code
## Evaluator-Orchestrator Architecture

---

## Overview

This plugin implements a **hierarchical multiagent system** for Claude Code that enables sophisticated product/feature development through coordinated AI agents. The architecture follows an Evaluator-Orchestrator pattern where high-level evaluation and quality control is separated from task execution and coordination.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER REQUEST                            │
│                    (New Product/Feature)                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    EVALUATOR (Opus 4.5)                         │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ • Uses AskUserQuestion to clarify requirements            │  │
│  │ • CREATES STRATEGIC PLAN → writes to .md file             │  │
│  │ • Validates requirements & feasibility                    │  │
│  │ • Evaluates: Code Quality, Clean Code, Security, Reqs     │  │
│  │ • Provides strategic feedback & course corrections        │  │
│  │ • Final sign-off on deliverables                          │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │  📄 PLAN.md     │
                    │  (Source of     │
                    │   Truth)        │
                    └────────┬────────┘
                             │
                              ▼
              ┌───────────────────────────────┐
              │     🧑 HUMAN REVIEW           │
              │  ┌─────────────────────────┐  │
              │  │ • Reviews plan.md       │  │
              │  │ • Approves / Requests   │  │
              │  │   changes via chat      │  │
              │  │ • Says "approved" or    │  │
              │  │   "proceed" to continue │  │
              │  └─────────────────────────┘  │
              └───────────────┬───────────────┘
                              │
                    ┌─────────┴─────────┐
                    │  Human Approved?  │
                    └─────────┬─────────┘
                              │ YES
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR (Sonnet)                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ • READS approved plan.md as source of truth               │  │
│  │ • Decomposes features into technical tasks                │  │
│  │ • DYNAMICALLY SPAWNS required subagents only              │  │
│  │ • Coordinates subagent execution order                    │  │
│  │ • Manages dependencies between components                 │  │
│  │ • Aggregates results from subagents                       │  │
│  │ • Reports progress to Evaluator                           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │  AGENT SPAWNER  │
                    │  (Dynamic)      │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│   BE AGENT    │    │   FE AGENT    │    │ TESTING AGENT │
│   (Sonnet)    │    │   (Sonnet)    │    │   (Sonnet)    │
│   [Optional]  │    │   [Optional]  │    │   [Optional]  │
├───────────────┤    ├───────────────┤    ├───────────────┤
│ • API design  │    │ • UI/UX       │    │ • Unit tests  │
│ • Database    │    │ • Components  │    │ • Integration │
│ • Services    │    │ • State mgmt  │    │ • E2E tests   │
│ • Security    │    │ • Styling     │    │ • Coverage    │
└───────────────┘    └───────────────┘    └───────────────┘
        ▲                    ▲                    ▲
        │                    │                    │
        └────────── Spawned based on ────────────┘
                   task requirements
```

---

## Core Components

### 1. Evaluator Agent (Opus 4.5)

**Purpose**: Strategic planning, quality assurance, user interaction, and final approval

**Model**: `claude-opus-4-5-20251101` (for superior reasoning, planning, and evaluation)

**Tools Available**:
- `AskUserQuestion` - Gather requirements, clarify ambiguities, get user preferences
- `Write` - Create and update the plan.md file (source of truth)
- `Read` - Read existing codebase for context during planning

---

#### Strategic Planning & Plan File

The Evaluator creates a **plan.md file** that serves as the single source of truth. This file:
- Is written to the project folder
- Must be reviewed and approved by a human before implementation
- Can be edited by human or updated by Evaluator based on feedback
- Is read by Orchestrator to guide implementation

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLANNING WORKFLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Evaluator gathers requirements (AskUserQuestion)             │
│                         │                                        │
│                         ▼                                        │
│  2. Evaluator creates strategic plan                             │
│                         │                                        │
│                         ▼                                        │
│  3. Evaluator writes plan to:                                    │
│     📄 ./FEATURE_NAME_plan.md  (project root)                   │
│                         │                                        │
│                         ▼                                        │
│  4. Evaluator notifies human: "Plan ready for review"            │
│                         │                                        │
│                         ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  🧑 HUMAN-IN-THE-LOOP                                    │    │
│  │                                                          │    │
│  │  Human can:                                              │    │
│  │  • Review plan.md in IDE/editor                          │    │
│  │  • Chat: "Change X to Y" → Evaluator updates plan        │    │
│  │  • Edit plan.md directly → Evaluator acknowledges        │    │
│  │  • Chat: "approved" / "proceed" → Implementation begins  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  5. On approval → Orchestrator reads plan.md and executes        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

#### Plan File Structure (plan.md)

The Evaluator generates a structured markdown plan:

```markdown
# Feature Plan: [Feature Name]

## Status: PENDING_REVIEW | APPROVED | IN_PROGRESS | COMPLETED

## Overview
[Brief description of the feature]

## Requirements
### Functional Requirements
- [ ] Requirement 1
- [ ] Requirement 2

### Non-Functional Requirements
- [ ] Performance: [specifics]
- [ ] Security: [specifics]

## Technical Approach
### Architecture Decision
[High-level architecture choices and rationale]

### Technology Stack
- Backend: [tech]
- Frontend: [tech]
- Database: [tech]

## Implementation Plan
### Phase 1: [Name]
- Task 1.1: [description]
- Task 1.2: [description]

### Phase 2: [Name]
- Task 2.1: [description]

## Agents Required
- [ ] Backend Agent: [Yes/No] - [reason]
- [ ] Frontend Agent: [Yes/No] - [reason]
- [ ] Testing Agent: [Yes/No] - [reason]

## Acceptance Criteria
1. [Criterion 1]
2. [Criterion 2]

## Risk Assessment
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk 1] | Medium | High | [Strategy] |

## Dependencies
- [External dependency 1]
- [Existing code dependency]

## Estimated Scope
- Files to create: [count]
- Files to modify: [count]
- New endpoints: [count]
- New components: [count]

---
*Generated by Evaluator (Opus 4.5)*
*Awaiting human approval before implementation*
```

---

#### Human-in-the-Loop Protocol

| Human Action | System Response |
|--------------|-----------------|
| Reviews plan.md | Evaluator waits for feedback |
| "Change the auth to use JWT" | Evaluator updates plan.md, notifies human |
| Edits plan.md directly | Evaluator reads changes, acknowledges |
| "I have concerns about X" | Evaluator explains reasoning or adjusts |
| "approved" / "proceed" / "looks good" | Orchestrator begins implementation |
| "reject" / "start over" | Evaluator discards plan, restarts requirements |

**Important**: The system **BLOCKS** until human approval. No code is written until the human explicitly approves.

---

#### User Interaction via AskUserQuestion

The Evaluator uses `AskUserQuestion` to deeply understand the feature before planning begins:

```
┌─────────────────────────────────────────────────────────────────┐
│                  REQUIREMENTS GATHERING FLOW                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  User Request ──► Evaluator analyzes ──► Identifies gaps         │
│                                              │                   │
│                                              ▼                   │
│                                    ┌─────────────────┐           │
│                                    │ AskUserQuestion │           │
│                                    └────────┬────────┘           │
│                                             │                    │
│  ┌──────────────────────────────────────────┼──────────────────┐ │
│  │  Example Questions:                      │                  │ │
│  │  • "What's the target user for this?"    │                  │ │
│  │  • "Should this integrate with X?"       │                  │ │
│  │  • "What's the auth requirement?"        │                  │ │
│  │  • "Performance expectations?"           │                  │ │
│  │  • "Any existing patterns to follow?"    │                  │ │
│  └──────────────────────────────────────────┼──────────────────┘ │
│                                             │                    │
│                                             ▼                    │
│                              User Responds ──► Complete Picture  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Question Categories**:
| Category | Example Questions |
|----------|-------------------|
| **Functional** | "Should users be able to export data?" |
| **Technical** | "Preferred database: SQL or NoSQL?" |
| **Constraints** | "Any third-party services to avoid?" |
| **UX/Design** | "Desktop-first or mobile-first?" |
| **Security** | "What authentication method is required?" |
| **Integration** | "Which existing APIs should this connect to?" |

---

#### Evaluation Criteria (Code Review)

The Evaluator performs comprehensive reviews against these criteria:

##### 1. Requirements Compliance ✓
| Check | Description |
|-------|-------------|
| Feature completeness | All specified functionality implemented |
| Edge cases | Boundary conditions handled |
| User stories | Acceptance criteria from requirements met |
| API contracts | Endpoints match specifications |

##### 2. Code Quality ✓
| Check | Description |
|-------|-------------|
| Readability | Code is self-documenting and clear |
| Maintainability | Easy to modify and extend |
| DRY principle | No unnecessary duplication |
| Appropriate abstractions | Right level of complexity |
| Error handling | Graceful failure modes |

##### 3. Clean Code Standards ✓
| Check | Description |
|-------|-------------|
| Naming conventions | Descriptive, consistent naming |
| Function size | Small, single-purpose functions |
| File organization | Logical structure and grouping |
| Comments | Meaningful where needed (not obvious) |
| Formatting | Consistent style throughout |
| SOLID principles | Single responsibility, open/closed, etc. |

##### 4. Security Review ✓
| Check | Description |
|-------|-------------|
| Input validation | All user input sanitized |
| Authentication | Proper auth checks in place |
| Authorization | Access control enforced |
| Data protection | Sensitive data encrypted/protected |
| OWASP Top 10 | No common vulnerabilities |
| Secrets management | No hardcoded credentials |
| SQL injection | Parameterized queries used |
| XSS prevention | Output properly escaped |

##### 5. Performance ✓
| Check | Description |
|-------|-------------|
| Algorithm efficiency | Appropriate time/space complexity |
| Database queries | Optimized, no N+1 problems |
| Caching | Applied where beneficial |
| Bundle size | No unnecessary dependencies |

---

#### Responsibilities by Phase

| Phase | Action |
|-------|--------|
| **Requirements** | Use `AskUserQuestion` to gather complete requirements |
| **Pre-Planning** | Validate requirements, identify risks, set acceptance criteria |
| **Plan Review** | Approve/reject Orchestrator's implementation plan |
| **In-Progress** | Periodic checkpoints, course corrections |
| **Post-Completion** | Full evaluation against ALL criteria above |

**Trigger Points**:
- Initial request received → Requirements gathering with `AskUserQuestion`
- Orchestrator submits plan for approval
- Major milestone completed
- All work completed (final comprehensive review)

---

### 2. Orchestrator Agent (Sonnet)

**Purpose**: Task decomposition, dynamic agent management, and execution coordination

**Model**: `claude-sonnet-4-20250514` (for efficient coordination)

**Tools Available**:
- `Task` - Spawn subagents dynamically based on requirements

---

#### Dynamic Agent Spawning

The Orchestrator **does not spawn all agents by default**. It analyzes the task and spawns only the required agents:

```
┌─────────────────────────────────────────────────────────────────┐
│                    AGENT SPAWNING DECISION TREE                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Analyze Task Requirements                                       │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Does it need server-side logic?                        │    │
│  │  (APIs, database, business logic, auth)                 │    │
│  │                                                         │    │
│  │  YES ──► Spawn BE Agent                                 │    │
│  │  NO  ──► Skip BE Agent                                  │    │
│  └─────────────────────────────────────────────────────────┘    │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Does it need client-side UI?                           │    │
│  │  (Components, styling, state, user interactions)        │    │
│  │                                                         │    │
│  │  YES ──► Spawn FE Agent                                 │    │
│  │  NO  ──► Skip FE Agent                                  │    │
│  └─────────────────────────────────────────────────────────┘    │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Does it need automated tests?                          │    │
│  │  (Unit, integration, E2E, coverage)                     │    │
│  │                                                         │    │
│  │  YES ──► Spawn Testing Agent                            │    │
│  │  NO  ──► Skip Testing Agent                             │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Spawning Examples

| Task Type | Agents Spawned |
|-----------|----------------|
| "Add API endpoint for user data" | BE only |
| "Create login form component" | FE only |
| "Write tests for auth module" | Testing only |
| "Build user dashboard feature" | BE + FE + Testing |
| "Add client-side form validation" | FE only |
| "Refactor database schema" | BE + Testing |
| "Create landing page" | FE only |
| "Full CRUD for products" | BE + FE + Testing |

#### Spawning Logic (Pseudocode)

```python
def determine_agents_to_spawn(task_requirements):
    agents_needed = []

    # Check for backend needs
    if any([
        task_requires_api_endpoints,
        task_requires_database_changes,
        task_requires_business_logic,
        task_requires_auth_changes,
        task_requires_server_side_validation
    ]):
        agents_needed.append("backend")

    # Check for frontend needs
    if any([
        task_requires_ui_components,
        task_requires_styling,
        task_requires_state_management,
        task_requires_client_routing,
        task_requires_user_interactions
    ]):
        agents_needed.append("frontend")

    # Check for testing needs
    if any([
        evaluator_requires_tests,
        task_modifies_critical_path,
        task_has_complex_logic,
        existing_tests_need_updates
    ]):
        agents_needed.append("testing")

    return agents_needed
```

---

#### Core Responsibilities

- **Task Analysis**: Understand what the feature requires
- **Agent Selection**: Determine which subagents are needed
- **Task Decomposition**: Break down features into discrete tasks
- **Dependency Management**: Order tasks correctly (BE before FE integration, etc.)
- **Parallel Execution**: Run independent agents simultaneously
- **Result Aggregation**: Compile outputs from all subagents
- **Progress Reporting**: Keep Evaluator informed

---

#### Key Workflows

```
1. Receive approved plan from Evaluator
2. Analyze task requirements
3. Determine which agents to spawn (dynamic decision)
4. Decompose into technical tasks per agent
5. Create dependency graph
6. Spawn required agents (parallel where possible)
7. Coordinate inter-agent dependencies
8. Handle failures/retries
9. Compile results
10. Submit to Evaluator for review
```

---

#### Evaluator Feedback Loop (Error Correction)

When the Evaluator reviews completed work and finds issues, it sends structured feedback back to the Orchestrator to coordinate fixes. This creates an iterative improvement cycle until quality standards are met.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    EVALUATOR → ORCHESTRATOR FEEDBACK LOOP               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐                                                    │
│  │   ORCHESTRATOR  │──────── Submits completed work ─────────┐          │
│  │                 │                                         │          │
│  └────────▲────────┘                                         ▼          │
│           │                                         ┌─────────────────┐ │
│           │                                         │    EVALUATOR    │ │
│           │                                         │                 │ │
│           │                                         │  Reviews code:  │ │
│           │                                         │  • Requirements │ │
│           │                                         │  • Code Quality │ │
│           │                                         │  • Security     │ │
│           │                                         │  • Clean Code   │ │
│           │                                         └────────┬────────┘ │
│           │                                                  │          │
│           │                              ┌───────────────────┴───────┐  │
│           │                              ▼                           ▼  │
│           │                     ┌─────────────────┐         ┌─────────┐ │
│           │                     │  Issues Found?  │         │ APPROVED│ │
│           │                     └────────┬────────┘         │   ✓     │ │
│           │                              │ YES              └─────────┘ │
│           │                              ▼                              │
│           │                     ┌─────────────────────────────────────┐ │
│           │                     │      FEEDBACK REPORT                │ │
│           │                     ├─────────────────────────────────────┤ │
│           │                     │ • Error type (security/quality/etc) │ │
│           │                     │ • Affected files & line numbers     │ │
│           │                     │ • Description of issue              │ │
│           │                     │ • Required fix (specific guidance)  │ │
│           │                     │ • Priority (critical/high/medium)   │ │
│           │                     └────────────────┬────────────────────┘ │
│           │                                      │                      │
│           └────────────── Receives feedback ─────┘                      │
│                                                                         │
│  ORCHESTRATOR ACTIONS ON FEEDBACK:                                      │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ 1. Parse feedback report                                        │    │
│  │ 2. Identify which agent(s) need to make fixes                   │    │
│  │ 3. Re-spawn affected agent(s) with fix instructions             │    │
│  │ 4. Agent makes targeted fix (not full re-implementation)        │    │
│  │ 5. Orchestrator re-submits for Evaluator review                 │    │
│  │ 6. Repeat until Evaluator approves                              │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

##### Feedback Report Structure

The Evaluator generates structured feedback when issues are found:

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
      "description": "Password not hashed before database comparison",
      "current_code": "if (user.password === inputPassword)",
      "required_fix": "Use bcrypt.compare() for secure password verification",
      "assigned_agent": "backend"
    },
    {
      "id": "REQ-002",
      "type": "missing_requirement",
      "severity": "high",
      "file": null,
      "line": null,
      "description": "Password reset email functionality not implemented",
      "plan_reference": "Phase 2, Task 2.2",
      "required_fix": "Implement /api/auth/forgot-password endpoint per plan.md",
      "assigned_agent": "backend"
    },
    {
      "id": "QUAL-003",
      "type": "code_quality",
      "severity": "medium",
      "file": "src/components/LoginForm.tsx",
      "line": 78,
      "description": "Form validation logic duplicated across components",
      "required_fix": "Extract validation into shared useFormValidation hook",
      "assigned_agent": "frontend"
    }
  ],
  "summary": {
    "total_issues": 3,
    "critical": 1,
    "high": 1,
    "medium": 1,
    "agents_needing_fixes": ["backend", "frontend"]
  }
}
```

##### Issue Types & Handling

| Issue Type | Description | Severity Range | Agent Assignment |
|------------|-------------|----------------|------------------|
| `security` | Vulnerabilities, auth flaws, injection risks | Critical-High | BE (usually) |
| `missing_requirement` | Feature from plan.md not implemented | High | Per plan.md |
| `missing_code` | Incomplete implementation, missing files | High | Per context |
| `code_quality` | DRY violations, poor structure, complexity | Medium | Original agent |
| `clean_code` | Naming, formatting, function size | Low-Medium | Original agent |
| `performance` | N+1 queries, inefficient algorithms | Medium-High | Original agent |
| `test_failure` | Tests not passing, missing test coverage | High | Testing agent |

##### Orchestrator Fix Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│            ORCHESTRATOR FIX HANDLING WORKFLOW                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. RECEIVE FEEDBACK                                                    │
│     └── Parse issues from Evaluator feedback report                     │
│                                                                         │
│  2. PRIORITIZE FIXES                                                    │
│     └── Sort by severity: Critical → High → Medium → Low                │
│                                                                         │
│  3. GROUP BY AGENT                                                      │
│     ┌─────────────────────────────────────────────────────────────────┐ │
│     │  BE Agent fixes:        FE Agent fixes:      Testing fixes:     │ │
│     │  • SEC-001 (critical)   • QUAL-003 (medium)  • [none]           │ │
│     │  • REQ-002 (high)                                               │ │
│     └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  4. SPAWN AGENTS WITH FIX CONTEXT                                       │
│     ┌─────────────────────────────────────────────────────────────────┐ │
│     │  Orchestrator → BE Agent:                                       │ │
│     │  "Fix these issues:                                             │ │
│     │   1. [SEC-001] In login.ts:42, use bcrypt.compare()             │ │
│     │   2. [REQ-002] Implement forgot-password endpoint per plan.md"  │ │
│     │                                                                 │ │
│     │  Orchestrator → FE Agent:                                       │ │
│     │  "Fix this issue:                                               │ │
│     │   1. [QUAL-003] Extract validation into useFormValidation hook" │ │
│     └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  5. AGENTS MAKE TARGETED FIXES                                          │
│     └── Agents fix ONLY the specified issues (no scope creep)           │
│                                                                         │
│  6. RE-SUBMIT TO EVALUATOR                                              │
│     └── Orchestrator compiles fixes and requests re-review              │
│                                                                         │
│  7. REPEAT UNTIL APPROVED                                               │
│     └── Max iterations: 3 (then escalate to human)                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

##### Iteration Limits & Escalation

To prevent infinite loops, the system has safeguards:

| Iteration | Action |
|-----------|--------|
| 1st review | Normal feedback → fix cycle |
| 2nd review | Track persistent issues, warn Orchestrator |
| 3rd review | If same issues persist → escalate to human |
| Max (3) exceeded | **BLOCK**: Human intervention required |

```
┌─────────────────────────────────────────────────────────────────────────┐
│  ⚠️  ESCALATION TRIGGER                                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  After 3 iterations with unresolved issues:                             │
│                                                                         │
│  Evaluator → Human:                                                     │
│  "🚨 Implementation blocked. These issues persist after 3 fix cycles:   │
│                                                                         │
│   • [SEC-001] Password hashing still not implemented correctly          │
│     - Attempted fixes: bcrypt import missing, wrong API usage           │
│                                                                         │
│   Recommended actions:                                                  │
│   1. Review the affected file: src/api/auth/login.ts                    │
│   2. Provide guidance in chat                                           │
│   3. Or: 'skip SEC-001' to accept current implementation"               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

##### Feedback Examples by Issue Type

**Security Issue Feedback:**
```
Issue: SQL injection vulnerability
File: src/api/users/search.ts:28
Current: `db.query("SELECT * FROM users WHERE name = '" + name + "'")`
Fix: Use parameterized query: `db.query("SELECT * FROM users WHERE name = $1", [name])`
Severity: CRITICAL
Agent: backend
```

**Missing Requirement Feedback:**
```
Issue: Feature from plan.md not implemented
Plan Reference: Phase 2, Task 2.3 - "Create /api/auth/reset-password endpoint"
Current State: Endpoint does not exist
Fix: Implement endpoint per plan.md specification
Severity: HIGH
Agent: backend
```

**Code Quality Feedback:**
```
Issue: Function exceeds recommended size (150 lines)
File: src/services/orderProcessor.ts:45-195
Current: Single monolithic processOrder() function
Fix: Break into smaller functions: validateOrder(), calculateTotal(), applyDiscounts(), saveOrder()
Severity: MEDIUM
Agent: backend
```

---

### 3. Subagents

#### Backend (BE) Agent

**Model**: `claude-sonnet-4-20250514`

**Tools Available**:
- `Read` - Read existing code files for context
- `Write` - Create new files (models, services, routes)
- `Edit` - Modify existing code files
- `Bash` - Run commands (migrations, package installs, server commands)
- `Glob` - Find files by pattern
- `Grep` - Search code content

**Scope**: Server-side implementation
- API endpoint design and implementation
- Database schema and migrations
- Business logic and services
- Authentication/authorization
- Data validation

---

#### Frontend (FE) Agent

**Model**: `claude-sonnet-4-20250514`

**Tools Available**:
- `Read` - Read existing code files for context
- `Write` - Create new files (components, pages, styles)
- `Edit` - Modify existing code files
- `Bash` - Run commands (build, lint, package installs)
- `Glob` - Find files by pattern
- `Grep` - Search code content

**Scope**: Client-side implementation
- UI component creation
- State management
- API integration
- Styling and responsive design
- Accessibility compliance

---

#### Testing Agent

**Model**: `claude-sonnet-4-20250514`

**Tools Available**:
- `Read` - Read source code to understand what to test
- `Write` - Create new test files
- `Edit` - Modify existing test files
- `Bash` - Run test commands (test runners, coverage tools)
- `Glob` - Find test and source files
- `Grep` - Search for patterns in code

**Restricted From**:
- Writing/editing non-test files (production code)

**Scope**: Quality assurance
- Unit test creation
- Integration test creation
- E2E test scenarios
- Test coverage analysis
- Bug identification

---

## Workflow: New Feature Development

```
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 1: REQUIREMENTS GATHERING (Evaluator)                      │
├──────────────────────────────────────────────────────────────────┤
│ 1. User submits feature request                                  │
│ 2. Evaluator analyzes request, identifies information gaps       │
│ 3. Evaluator uses AskUserQuestion to clarify:                    │
│    • Functional requirements                                     │
│    • Technical preferences                                       │
│    • Security requirements                                       │
│    • Integration needs                                           │
│ 4. Evaluator compiles complete requirements                      │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 2: STRATEGIC PLANNING (Evaluator)                          │
├──────────────────────────────────────────────────────────────────┤
│ 1. Evaluator creates comprehensive strategic plan                │
│ 2. Plan includes:                                                │
│    • Architecture decisions & rationale                          │
│    • Technology stack choices                                    │
│    • Implementation phases & tasks                               │
│    • Which agents will be needed                                 │
│    • Acceptance criteria                                         │
│    • Risk assessment                                             │
│ 3. Evaluator writes plan to: .mas/plans/FEATURE_NAME_plan.md     │
│ 4. Evaluator notifies: "📄 Plan ready for review"                │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 3: HUMAN REVIEW & APPROVAL  ⏸️  BLOCKING                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  🧑 Human reviews .mas/plans/FEATURE_NAME_plan.md                │
│                                                                  │
│  Options:                                                        │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ • "Change X to Y"     → Evaluator updates plan.md          │  │
│  │ • Edit file directly  → Evaluator acknowledges changes     │  │
│  │ • "Why did you...?"   → Evaluator explains reasoning       │  │
│  │ • "approved"          → Proceed to implementation ✓        │  │
│  │ • "reject"            → Back to Phase 1                    │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ⚠️  NO CODE IS WRITTEN UNTIL HUMAN SAYS "approved"              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼ (only after "approved")
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 4: IMPLEMENTATION (Orchestrator + Dynamic Agents)          │
├──────────────────────────────────────────────────────────────────┤
│ 1. Orchestrator reads approved plan.md as source of truth        │
│ 2. Orchestrator decomposes plan into technical tasks             │
│ 3. Orchestrator spawns ONLY required agents:                     │
│    • BE Agent (if backend work needed)                           │
│    • FE Agent (if frontend work needed)                          │
│    • Testing Agent (if tests required)                           │
│ 4. Agents execute tasks (parallel where possible)                │
│ 5. Orchestrator coordinates dependencies                         │
│ 6. Orchestrator aggregates results                               │
│ 7. Updates plan.md status: IN_PROGRESS → tracks completion       │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 5: EVALUATION & ITERATION (FEEDBACK LOOP)                  │
├──────────────────────────────────────────────────────────────────┤
│ 1. Orchestrator submits completed work to Evaluator              │
│ 2. Evaluator performs comprehensive review:                      │
│    ✓ Requirements compliance (per plan.md)                       │
│    ✓ Code quality                                                │
│    ✓ Clean code standards                                        │
│    ✓ Security review                                             │
│    ✓ Performance check                                           │
│                                                                  │
│ 3. IF ISSUES FOUND → Evaluator sends FEEDBACK REPORT:            │
│    ┌──────────────────────────────────────────────────────────┐  │
│    │ Feedback Report includes:                                │  │
│    │ • Issue ID, type, severity (critical/high/medium/low)    │  │
│    │ • Affected file & line number                            │  │
│    │ • Description of the problem                             │  │
│    │ • Specific fix instructions                              │  │
│    │ • Assigned agent (BE/FE/Testing)                         │  │
│    └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│ 4. Orchestrator receives feedback and:                           │
│    • Parses issues and groups by agent                           │
│    • Re-spawns ONLY affected agents with fix context             │
│    • Agents make targeted fixes (no scope creep)                 │
│                                                                  │
│ 5. Orchestrator re-submits fixed work to Evaluator               │
│                                                                  │
│ 6. REPEAT until Evaluator approves (max 3 iterations)            │
│    • After 3 failed iterations → escalate to human               │
│                                                                  │
│ 7. IF APPROVED → Proceed to Phase 6                              │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 6: DELIVERY                                                │
├──────────────────────────────────────────────────────────────────┤
│ 1. Evaluator provides final sign-off                             │
│ 2. Updates plan.md status: COMPLETED                             │
│ 3. Generate delivery report (what was built, files changed)      │
│ 4. Deliver to user                                               │
└──────────────────────────────────────────────────────────────────┘
```

---

## Plugin File Structure

```
mas-plugin/
├── skills/
│   └── feature-builder/
│       ├── feature-builder.md          # Main skill definition
│       ├── evaluator.md                # Evaluator agent prompt
│       ├── orchestrator.md             # Orchestrator agent prompt
│       └── subagents/
│           ├── backend.md              # BE agent prompt
│           ├── frontend.md             # FE agent prompt
│           └── testing.md              # Testing agent prompt
├── hooks/
│   └── post-task-validation.sh         # Hook for Evaluator checkpoints
├── templates/
│   ├── acceptance-criteria.md          # Template for requirements
│   ├── implementation-plan.md          # Template for plans
│   └── evaluation-report.md            # Template for reviews
└── README.md
```

---

## Implementation Tasks

### Phase 1: Core Framework
- [ ] Create skill definition file (`feature-builder.md`)
- [ ] Implement Evaluator agent prompt with Opus 4.5 configuration
- [ ] Add `AskUserQuestion` integration to Evaluator for requirements gathering
- [ ] Implement Orchestrator agent prompt with task decomposition logic
- [ ] Add dynamic agent spawning logic to Orchestrator
- [ ] Define inter-agent communication protocol
- [ ] Create `.mas/plans/` directory structure

### Phase 2: Planning & Human-in-the-Loop
- [ ] Implement strategic planning logic in Evaluator
- [ ] Create plan.md template and generation logic
- [ ] Build plan file writer (Write tool integration)
- [ ] Implement human approval detection ("approved", "proceed", etc.)
- [ ] Add plan update logic (modify plan.md based on feedback)
- [ ] Implement plan status tracking (PENDING_REVIEW → APPROVED → IN_PROGRESS → COMPLETED)
- [ ] Add blocking mechanism until human approval

### Phase 3: Evaluator Review Capabilities
- [ ] Implement requirements gathering question flow
- [ ] Create evaluation criteria checklist (requirements, code quality, clean code, security)
- [ ] Build code review logic for each criterion
- [ ] Add feedback generation for failed reviews
- [ ] Implement iterative review loop
- [ ] Add plan.md reading for acceptance criteria validation

### Phase 4: Subagents
- [ ] Implement Backend agent with API/database expertise
- [ ] Implement Frontend agent with UI/UX expertise
- [ ] Implement Testing agent with QA expertise
- [ ] Create agent registry for dynamic spawning
- [ ] Implement agent selection heuristics in Orchestrator

### Phase 5: Workflow & Integration
- [ ] Implement feedback loop between Evaluator and Orchestrator
- [ ] Create Feedback Report JSON structure for issue communication
- [ ] Implement issue categorization (security, missing_requirement, code_quality, etc.)
- [ ] Build Orchestrator fix-handling workflow (parse → prioritize → group → spawn)
- [ ] Add iteration tracking and max-iteration limits (3 attempts)
- [ ] Implement human escalation trigger for persistent issues
- [ ] Create templates for structured outputs
- [ ] Add progress tracking and status reporting
- [ ] Implement retry/error handling logic
- [ ] Build dependency graph management
- [ ] Add plan.md status updates during execution

### Phase 6: Polish & Documentation
- [ ] Create user-facing documentation
- [ ] Add example workflows
- [ ] Test end-to-end scenarios
- [ ] Optimize token usage and performance
- [ ] Add extensibility guide for custom agents

---

## Agent Communication Protocol

### Message Format
```json
{
  "from": "orchestrator",
  "to": "evaluator",
  "type": "plan_submission",
  "payload": {
    "feature_name": "User Authentication",
    "tasks": [...],
    "dependencies": [...],
    "estimated_files": [...]
  },
  "status": "awaiting_approval"
}
```

### Status Types
| Status | Description |
|--------|-------------|
| `pending` | Task not started |
| `in_progress` | Currently executing |
| `awaiting_review` | Submitted for Evaluator review |
| `approved` | Evaluator approved |
| `revision_needed` | Requires changes |
| `completed` | Finished and verified |

---

## Key Design Decisions

### Why Opus 4.5 for Evaluator?
- Superior reasoning for complex requirement analysis
- Better judgment for quality evaluation
- More nuanced feedback for improvements
- Strategic thinking for architecture decisions

### Why Sonnet for Orchestrator & Subagents?
- Faster execution for task-level work
- Cost-effective for high-volume operations
- Sufficient capability for focused implementation tasks
- Better latency for iterative development

### Why This Architecture?
1. **Separation of Concerns**: Evaluation logic separate from execution
2. **Quality Gates**: Built-in checkpoints prevent poor implementations
3. **Scalability**: Easy to add more specialized subagents
4. **Feedback Loops**: Iterative improvement until quality standards met
5. **Transparency**: Clear progression through defined phases

### Why AskUserQuestion for Evaluator?
- **Incomplete requirements are the #1 cause of project failure** - gathering info upfront prevents rework
- **User preferences matter** - tech stack, security level, integration needs vary per project
- **Evaluator has the context** - positioned at the start of the flow to set clear criteria
- **Reduces back-and-forth** - batch questions upfront vs. interrupting implementation

### Why Planning Built into Evaluator?
- **Opus 4.5 excels at strategic thinking** - you're paying for the best model, use it for planning
- **Context preservation** - requirements flow directly into planning without handoff losses
- **Single source of truth** - plan.md is authoritative, no conflicting interpretations
- **Natural workflow** - requirements → planning → approval is a logical sequence

### Why Human-in-the-Loop with plan.md?
- **Accountability** - human explicitly approves architecture before code is written
- **Course correction** - cheaper to change a plan than refactor code
- **Transparency** - human sees exactly what will be built, no surprises
- **Auditability** - plan.md serves as documentation of what was agreed
- **Trust building** - gradual autonomy; human can give more freedom over time

### Why .md File as Source of Truth?
- **Human readable** - non-technical stakeholders can review
- **Version controllable** - git tracks plan changes
- **IDE friendly** - opens directly in VS Code, renders nicely
- **Editable** - human can modify directly if needed
- **Persistent** - survives session restarts, serves as project documentation

### Why Dynamic Agent Spawning?
- **Cost efficiency** - don't spawn 3 agents when 1 will do
- **Faster execution** - fewer agents = less coordination overhead
- **Focused output** - agents only produce what's needed
- **Flexibility** - easily extensible to add new agent types (DevOps, Security, etc.)

### Why Comprehensive Code Review?
The Evaluator reviews against 5 criteria because:
| Criterion | Why It Matters |
|-----------|----------------|
| **Requirements** | Ensures what was asked for is actually built |
| **Code Quality** | Prevents technical debt and maintenance burden |
| **Clean Code** | Ensures readability and team collaboration |
| **Security** | Catches vulnerabilities before production |
| **Performance** | Prevents scalability issues early |

---

## Usage Examples

### Example 1: Full-Stack Feature with Human Review

```
User: /feature-builder Create a user authentication system

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 1: REQUIREMENTS GATHERING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: I need to understand your requirements better.

┌─────────────────────────────────────────────────────────────────┐
│ AskUserQuestion: Authentication Method                          │
├─────────────────────────────────────────────────────────────────┤
│ What authentication method should be supported?                 │
│                                                                 │
│ ○ Email/Password (Recommended)                                  │
│ ○ OAuth (Google, GitHub)                                        │
│ ○ Magic Link (passwordless)                                     │
│ ○ Other                                                         │
└─────────────────────────────────────────────────────────────────┘

User: Email/Password

┌─────────────────────────────────────────────────────────────────┐
│ AskUserQuestion: Security Requirements                          │
├─────────────────────────────────────────────────────────────────┤
│ What security features are required?                            │
│                                                                 │
│ ☑ Password reset via email                                      │
│ ☑ Session management                                            │
│ ☐ Two-factor authentication (2FA)                               │
│ ☐ Rate limiting                                                 │
└─────────────────────────────────────────────────────────────────┘

User: [Selects password reset and session management]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 2: STRATEGIC PLANNING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: Creating strategic plan...

  ✓ Analyzed requirements
  ✓ Evaluated architecture options
  ✓ Selected technology approach
  ✓ Identified required agents
  ✓ Defined acceptance criteria

  📄 Writing plan to: .mas/plans/user_authentication_plan.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 FILE CREATED: .mas/plans/user_authentication_plan.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Feature Plan: User Authentication

## Status: PENDING_REVIEW

## Overview
Email/password authentication system with password reset
and session management for Next.js application.

## Requirements
### Functional Requirements
- [ ] User registration with email/password
- [ ] User login with email/password
- [ ] Password reset via email link
- [ ] Session management with secure cookies

### Non-Functional Requirements
- [ ] Passwords hashed with bcrypt (cost factor 12)
- [ ] Sessions expire after 7 days
- [ ] Rate limiting: 5 failed attempts per 15 min

## Technical Approach
### Architecture Decision
Server-side sessions with httpOnly cookies chosen over JWT
for better security (revocable sessions, no token exposure).

### Technology Stack
- Backend: Next.js API Routes
- Frontend: React with Next.js
- Database: Existing project database
- Email: Project's configured email service

## Implementation Plan
### Phase 1: Backend Foundation
- Task 1.1: Create User model with password hashing
- Task 1.2: Implement session middleware
- Task 1.3: Create /api/auth/register endpoint
- Task 1.4: Create /api/auth/login endpoint
- Task 1.5: Create /api/auth/logout endpoint

### Phase 2: Password Reset
- Task 2.1: Create password reset token model
- Task 2.2: Create /api/auth/forgot-password endpoint
- Task 2.3: Create /api/auth/reset-password endpoint

### Phase 3: Frontend
- Task 3.1: Create LoginForm component
- Task 3.2: Create RegisterForm component
- Task 3.3: Create ForgotPasswordForm component
- Task 3.4: Create ResetPasswordForm component
- Task 3.5: Add auth state management (context)

## Agents Required
- [x] Backend Agent: Yes - API routes, models, middleware
- [x] Frontend Agent: Yes - Forms, state management
- [x] Testing Agent: Yes - Critical security feature

## Acceptance Criteria
1. User can register with email/password
2. User can login and receives session cookie
3. User can request password reset email
4. User can reset password via email link
5. Invalid credentials return proper error messages
6. Sessions persist across browser refresh
7. Logout clears session

---
*Generated by Evaluator (Opus 4.5)*
*Awaiting human approval before implementation*

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 3: HUMAN REVIEW  ⏸️  WAITING FOR APPROVAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: 📄 Plan ready for your review at:
           .mas/plans/user_authentication_plan.md

           Please review and respond with:
           • Feedback to update the plan
           • "approved" to begin implementation

⏸️  Waiting for human approval...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
HUMAN FEEDBACK LOOP (example)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

User: "I want to use JWT instead of sessions"

Evaluator: I'll update the plan. Note: JWTs are stateless so
           you won't be able to revoke tokens server-side.
           Is that acceptable?

User: "Actually keep sessions, but add rate limiting"

Evaluator: ✓ Updated plan to include rate limiting.
           📄 Plan updated at: .mas/plans/user_authentication_plan.md

User: "approved"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 4: IMPLEMENTATION (Human Approved ✓)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: Plan approved. Handing off to Orchestrator.
           📄 Status updated: PENDING_REVIEW → APPROVED

Orchestrator: Reading approved plan...
              Analyzing required agents...

  Agents to spawn: [BE, FE, Testing]

Orchestrator: Spawning agents...
  ├── [BE Agent] Spawned ✓
  ├── [FE Agent] Spawned ✓
  └── [Testing Agent] Spawned ✓

Executing Phase 1: Backend Foundation...
  [BE Agent] Creating User model... ✓
  [BE Agent] Implementing session middleware... ✓
  [BE Agent] Creating auth endpoints... ✓

Executing Phase 2: Password Reset...
  [BE Agent] Creating reset token model... ✓
  [BE Agent] Creating reset endpoints... ✓

Executing Phase 3: Frontend (parallel with testing)...
  [FE Agent] Building LoginForm... ✓
  [FE Agent] Building RegisterForm... ✓
  [Testing Agent] Writing unit tests... ✓
  [Testing Agent] Writing integration tests... ✓

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 5: EVALUATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: Performing comprehensive review against plan.md...

  REQUIREMENTS COMPLIANCE (per plan.md):
  ✓ All 7 acceptance criteria met

  CODE QUALITY: ✓ Passed
  CLEAN CODE STANDARDS: ✓ Passed
  SECURITY REVIEW: ✓ Passed

  OVERALL: APPROVED ✓

  📄 Status updated: IN_PROGRESS → COMPLETED

  → Feature ready for delivery
```

---

### Example 2: Frontend-Only Task (Single Agent)

```
User: /feature-builder Add a dark mode toggle to the settings page

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 1: REQUIREMENTS GATHERING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: I need a few clarifications.

┌─────────────────────────────────────────────────────────────────┐
│ AskUserQuestion: Persistence                                    │
├─────────────────────────────────────────────────────────────────┤
│ Should the dark mode preference persist across sessions?        │
│                                                                 │
│ ○ Yes, save to localStorage (Recommended)                       │
│ ○ Yes, save to user account (requires API)                      │
│ ○ No, session only                                              │
└─────────────────────────────────────────────────────────────────┘

User: Yes, save to localStorage

Evaluator: Requirements complete.
  ✓ Dark mode toggle on settings page
  ✓ Persist preference in localStorage
  ✓ No backend changes needed
  → Forwarding to Orchestrator

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 2: PLANNING & AGENT SELECTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Orchestrator: Analyzing task requirements...

  Task analysis:
  • UI component changes only → FE Agent required
  • No API changes → BE Agent NOT needed
  • Simple feature → Testing Agent optional (skipping)

  Agents to spawn: [FE]  ← Only one agent!

  Implementation plan:
  └── FE Agent (3 tasks)
      ├── Create ThemeContext provider
      ├── Add toggle component to settings
      └── Implement localStorage persistence

  ⏳ Awaiting Evaluator approval...

Evaluator: Plan approved.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 3: IMPLEMENTATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Orchestrator: Spawning agents...
  └── [FE Agent] Spawned ✓

  [FE Agent] Creating ThemeContext... ✓
  [FE Agent] Adding toggle to settings... ✓
  [FE Agent] Implementing persistence... ✓

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 4: EVALUATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: Review complete.
  ✓ Requirements met
  ✓ Clean code standards followed
  ✓ No security concerns (client-only)

  APPROVED ✓ → Feature ready
```

---

## Next Steps

1. **Review this plan** and confirm the architecture meets your needs
2. **Decide on additional subagents** (DevOps? Documentation? Security?)
3. **Begin implementation** starting with the core skill definition
4. **Iterate** based on testing with real feature requests

---

*Document created: 2026-01-19*
*Architecture: Evaluator-Orchestrator Multiagent System*
*Target Platform: Claude Code Plugin*


DO LATER

Implementation Plan: Context Synchronization via Orchestrator-Held Shared Memory
Status: PENDING_REVIEW
Overview
Implement a context synchronization mechanism to prevent integration failures when Backend and Frontend agents work on shared interfaces (APIs, schemas, types). The Orchestrator will act as the central "shared memory coordinator" rather than using a file-based approach.

Problem Statement
Currently, if the Backend Agent changes a database schema or API contract, the Frontend Agent may not be aware until Phase 5 integration testing fails. This wastes tokens and requires rework cycles that could be prevented with proactive context sharing.

Solution Architecture
Orchestrator as Shared Memory Holder
The Orchestrator maintains an in-memory technical context object that tracks:

API contracts (endpoints, methods, request/response schemas)
Database schemas (tables, fields, types, indexes)
Shared TypeScript/type definitions
Environment variable requirements
Dependencies between components