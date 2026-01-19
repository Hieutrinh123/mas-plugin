# Feature Builder - Multiagent System Plugin for Claude Code

A sophisticated hierarchical AI architecture for building production-ready features through coordinated specialized agents.

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

---

## Key Features

### 🎯 Strategic Planning
- **Requirements Gathering**: Evaluator uses `AskUserQuestion` to clarify ambiguous requirements
- **Comprehensive Plans**: Detailed plan.md files serve as single source of truth
- **Human-in-the-Loop**: Explicit approval required before implementation begins

### 🤖 Dynamic Agent Spawning
- **Smart Selection**: Orchestrator spawns only the agents needed (Backend, Frontend, Testing)
- **Cost Efficient**: Don't spawn 3 agents when 1 will do
- **Parallel Execution**: Independent agents run simultaneously

### ✅ Quality Assurance
- **5 Evaluation Criteria**: Requirements compliance, code quality, clean code, security, performance
- **Feedback Loop**: Iterative fixes until quality standards met (max 3 iterations)
- **Human Escalation**: Persistent issues escalate to user after 3 failed attempts

### 🔒 Production-Ready Code
- **Security First**: Input validation, password hashing, SQL injection prevention
- **Clean Code**: DRY principles, single responsibility, maintainability
- **Comprehensive Testing**: Unit, integration, and E2E tests

---

## Installation

### 1. Copy Plugin to Claude Code Skills Directory

```bash
# Copy the plugin to your Claude Code skills directory
cp -r "mas plugin" ~/.claude/skills/feature-builder
```

### 2. Verify Installation

```bash
# Check that the skill is recognized
ls ~/.claude/skills/feature-builder
```

You should see:
```
skills/
  feature-builder/
    feature-builder.md
    evaluator.md
    orchestrator.md
    subagents/
templates/
README.md
```

---

## Usage

### Basic Usage

```bash
# In Claude Code CLI
/feature-builder Create a user authentication system
```

The system will:
1. **Gather requirements** via interactive questions
2. **Create a strategic plan** in `.mas/plans/user_authentication_plan.md`
3. **Wait for your approval** (blocking until you say "approved")
4. **Implement the feature** using dynamically spawned agents
5. **Review and iterate** until quality standards are met
6. **Deliver production-ready code**

### Example Workflow

```
You: /feature-builder Add a dark mode toggle to settings

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

You: localStorage

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 2: STRATEGIC PLANNING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: Creating strategic plan...
  ✓ Analyzed requirements
  ✓ Selected technology approach
  ✓ Identified required agents

  📄 Plan created at: .mas/plans/dark_mode_plan.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 3: HUMAN REVIEW  ⏸️  WAITING FOR APPROVAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: 📄 Plan ready for review. Please review and respond with:
           • Feedback to update the plan
           • "approved" to begin implementation

⏸️ Waiting for approval...

You: approved

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 4: IMPLEMENTATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Orchestrator: Spawning agents...
  └── [Frontend Agent] Spawned ✓

  [Frontend] Creating ThemeContext... ✓
  [Frontend] Adding toggle component... ✓
  [Frontend] Implementing localStorage persistence... ✓

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 5: EVALUATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Evaluator: Review complete.
  ✓ Requirements met
  ✓ Code quality standards followed
  ✓ No security concerns

  APPROVED ✓ → Feature delivered
```

---

## Architecture

### Agent Roles

#### 1. Evaluator (Opus 4.5)
**Purpose**: Strategic intelligence and quality assurance

**Responsibilities**:
- Requirements gathering via `AskUserQuestion`
- Strategic planning (creates plan.md)
- Human-in-the-loop coordination
- Code quality review (5 criteria)
- Final approval

**Model**: `claude-opus-4-5-20251101` (for superior reasoning)

#### 2. Orchestrator (Sonnet)
**Purpose**: Task decomposition and agent coordination

**Responsibilities**:
- Read approved plan.md
- Determine which agents to spawn (dynamic)
- Decompose plan into technical tasks
- Coordinate execution order (dependency management)
- Aggregate results
- Handle Evaluator feedback

**Model**: `claude-sonnet-4-20250514` (for efficient coordination)

#### 3. Backend Agent (Sonnet)
**Purpose**: Server-side implementation

**Scope**:
- API endpoints
- Database models and migrations
- Business logic
- Authentication/authorization
- Server-side validation

**Model**: `claude-sonnet-4-20250514`

#### 4. Frontend Agent (Sonnet)
**Purpose**: Client-side implementation

**Scope**:
- UI components
- Styling and responsive design
- State management
- API integration (client-side)
- Accessibility

**Model**: `claude-sonnet-4-20250514`

#### 5. Testing Agent (Sonnet)
**Purpose**: Quality assurance

**Scope**:
- Unit tests
- Integration tests
- E2E tests
- Coverage reports

**Model**: `claude-sonnet-4-20250514`

---

## Plan File Structure

Plans are stored in `.mas/plans/{feature_name}_plan.md` and serve as the single source of truth.

### Key Sections

```markdown
# Feature Plan: [Name]

## Status: PENDING_REVIEW | APPROVED | IN_PROGRESS | COMPLETED

## Requirements
- Functional requirements
- Non-functional requirements (performance, security, accessibility)

## Technical Approach
- Architecture decisions with rationale
- Technology stack

## Implementation Plan
- Phase 1: [Tasks]
- Phase 2: [Tasks]
- Phase N: [Tasks]

## Agents Required
- Backend Agent: Yes/No (with reason)
- Frontend Agent: Yes/No (with reason)
- Testing Agent: Yes/No (with reason)

## Acceptance Criteria
1. [Testable criterion]
2. [Testable criterion]

## Risk Assessment
| Risk | Likelihood | Impact | Mitigation |

## Dependencies
- External dependencies
- Internal dependencies
```

### Plan Approval Flow

```
1. Evaluator creates plan.md → Status: PENDING_REVIEW
2. Human reviews plan
   • Option A: Provide feedback → Evaluator updates → Human reviews again
   • Option B: Say "approved" → Status changes to APPROVED
3. Orchestrator reads approved plan
4. Implementation begins → Status: IN_PROGRESS
5. Evaluator reviews completed work → Status: COMPLETED (if approved)
```

---

## Quality Evaluation Criteria

The Evaluator reviews all code against 5 criteria:

### 1. Requirements Compliance ✓
- All functional requirements implemented
- All acceptance criteria met
- No features skipped or partially implemented

### 2. Code Quality ✓
- Readable, self-documenting code
- Small, focused functions (< 50 lines)
- DRY principle (no unnecessary duplication)
- Appropriate error handling

### 3. Clean Code Standards ✓
- Descriptive, consistent naming
- Logical file organization
- SOLID principles (especially Single Responsibility)
- Comments only where code isn't self-explanatory

### 4. Security Review ✓
- Input validation on all endpoints
- Password hashing (bcrypt/argon2)
- Parameterized SQL queries (no injection vulnerabilities)
- No hardcoded secrets
- XSS prevention
- Authentication/authorization checks

### 5. Performance ✓
- Efficient algorithms
- Optimized database queries (no N+1 problems)
- Appropriate caching
- No unnecessary dependencies

---

## Feedback Loop

When issues are found during evaluation:

```
1. Evaluator generates structured feedback report (JSON)
2. Feedback includes:
   • Issue ID, type, severity
   • Affected file and line number
   • Description of problem
   • Required fix (specific instructions)
   • Assigned agent (backend/frontend/testing)

3. Orchestrator receives feedback
4. Orchestrator re-spawns ONLY affected agents with fix instructions
5. Agents apply targeted fixes
6. Orchestrator re-submits for review

7. Repeat until approved (max 3 iterations)
8. If issues persist after 3 iterations → escalate to human
```

### Example Feedback Report

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
      "current_code": "if (user.password === inputPassword)",
      "required_fix": "Use bcrypt.compare() for secure verification",
      "assigned_agent": "backend"
    }
  ]
}
```

---

## Best Practices

### For Users

1. **Be specific in your initial request**: The more detail, the better the requirements gathering
2. **Review plans carefully**: This is your chance to steer the implementation
3. **Ask questions**: If something in the plan isn't clear, ask the Evaluator to explain
4. **Iterate on plans before approving**: It's much cheaper to change a plan than refactor code

### For Developers Extending the Plugin

1. **Add new specialized agents** by creating files in `subagents/` and updating the Orchestrator's spawning logic
2. **Customize evaluation criteria** by modifying `evaluator.md`
3. **Adjust plan templates** in `templates/plan-template.md`
4. **Add hooks** for custom validation or notifications

---

## Troubleshooting

### Issue: Evaluator keeps asking questions

**Cause**: Requirements are too vague
**Solution**: Provide more detail in your initial request, or answer the questions to build a complete picture

### Issue: Plan doesn't match my expectations

**Cause**: Evaluator made assumptions based on incomplete information
**Solution**: Provide feedback like "Change X to Y" and the Evaluator will update the plan

### Issue: Implementation failed after 3 iterations

**Cause**: Complex issue that agents can't resolve autonomously
**Solution**: The Evaluator will escalate to you. Review the affected files and provide guidance.

### Issue: Wrong agents spawned

**Cause**: Orchestrator misinterpreted the plan's requirements
**Solution**: Make the "Agents Required" section of the plan more explicit

---

## Extending the System

### Adding a New Agent Type

1. **Create agent prompt**: `subagents/devops.md`
2. **Define scope**: What the agent builds (deployment scripts, CI/CD, infrastructure)
3. **Update Orchestrator**: Add spawning logic in `orchestrator.md`
4. **Update plan template**: Add "DevOps Agent: Yes/No" section

Example use cases for new agents:
- **DevOps Agent**: Docker, Kubernetes, CI/CD pipelines
- **Security Agent**: Penetration testing, security audits
- **Documentation Agent**: API docs, user guides, inline comments
- **Migration Agent**: Database migrations, data transformations

---

## File Structure

```
mas-plugin/
├── skills/
│   └── feature-builder/
│       ├── feature-builder.md       # Entry point skill
│       ├── evaluator.md             # Evaluator agent prompt (Opus)
│       ├── orchestrator.md          # Orchestrator agent prompt (Sonnet)
│       └── subagents/
│           ├── backend.md           # Backend agent prompt
│           ├── frontend.md          # Frontend agent prompt
│           └── testing.md           # Testing agent prompt
├── templates/
│   └── plan-template.md             # Plan file template
└── README.md                        # This file
```

---

## Why This Architecture?

### Separation of Concerns
- **Evaluator**: Strategic thinking, not implementation details
- **Orchestrator**: Coordination, not code writing
- **Subagents**: Focused implementation in their domain

### Quality Gates
Built-in checkpoints prevent poor implementations from reaching production:
1. Requirements gathering (Phase 1)
2. Plan approval (Phase 3)
3. Code review (Phase 5)

### Scalability
Easy to add new agent types without modifying existing agents.

### Feedback Loops
Iterative improvement until quality standards are met (with human escalation as safety valve).

### Transparency
Clear progression through defined phases with human visibility at each stage.

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
- Version-controllable (git tracks plan changes)
- IDE-friendly (opens in VS Code, renders nicely)
- Editable (human can modify directly)
- Persistent (survives session restarts)

### Why Dynamic Agent Spawning?
- Cost efficiency (don't spawn agents you don't need)
- Faster execution (less coordination overhead)
- Focused output (agents only produce what's needed)

---

## Examples

See [MULTIAGENT_SYSTEM_PLAN.md](MULTIAGENT_SYSTEM_PLAN.md) for detailed example workflows:
- Full-stack feature (user authentication)
- Frontend-only task (dark mode toggle)
- Backend-only task (API endpoint)

---

## Contributing

To contribute improvements:

1. Test your changes on real feature requests
2. Ensure agent prompts are clear and unambiguous
3. Update this README if you add new capabilities
4. Submit examples of successful feature builds

---

## License

[Specify your license]

---

## Support

For issues or questions:
- GitHub Issues: [Your repo]
- Documentation: This README
- Examples: See MULTIAGENT_SYSTEM_PLAN.md

---

## Version

**v1.0.0** - Initial release

**Architecture**: Evaluator-Orchestrator Multiagent System
**Platform**: Claude Code Plugin
**Models**: Opus 4.5 (Evaluator) + Sonnet 4.5 (Orchestrator & Subagents)

---

*Built for production-ready feature development with AI*
