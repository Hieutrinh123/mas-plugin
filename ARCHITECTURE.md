# Feature Builder Architecture

## Directory Structure

```
feature-builder-plugin/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest
├── commands/
│   └── feature-builder.md             # /feature-builder slash command
├── skills/                            # Strategic agents (need full context)
│   ├── feature-builder-evaluator/
│   │   └── SKILL.md                   # Opus 4.5 - Requirements, planning, QA
│   └── feature-builder-orchestrator/
│       └── SKILL.md                   # Sonnet - Coordination, task decomposition
├── agents/                            # Execution agents (isolated contexts)
│   ├── backend-developer.md           # Sonnet - Backend implementation only
│   ├── frontend-developer.md          # Sonnet - Frontend implementation only
│   └── testing-specialist.md          # Sonnet - Testing implementation only
├── templates/
│   └── plan-template.md               # Plan file template
├── improvements-plan.md               # Roadmap for future enhancements
├── ARCHITECTURE.md                    # This file
├── README.md                          # User documentation
└── LICENSE                            # MIT License
```

## Architectural Pattern: Hybrid Skills + Agents

### Why the Split?

**Skills (`skills/` folder):**
- Auto-invoked based on context matching
- Share conversation context
- Have full tool access
- Cross-platform (web, API, Code)
- **Use case**: Strategic roles that need visibility across all domains

**Agents (`agents/` folder):**
- Explicitly spawned with isolated contexts
- Run in separate conversation contexts
- Can have controlled tool permissions
- Claude Code only
- **Use case**: Execution roles that need domain isolation

### Problem This Solves

**Before (all skills):**
- Frontend Agent could accidentally access backend skill knowledge
- Backend Agent might use frontend patterns in API code
- No domain boundaries between execution agents
- Risk of cross-contamination

**After (hybrid):**
- Evaluator & Orchestrator remain skills (need full context for planning/coordination)
- Backend, Frontend, Testing become agents (isolated execution contexts)
- Clear domain separation prevents cross-contamination
- Each execution agent has single responsibility

---

## Agent Roles & Responsibilities

### Strategic Layer (Skills)

#### 1. Evaluator (Opus 4.5)
**File**: `skills/feature-builder-evaluator/SKILL.md`

**Responsibilities:**
- Phase 1: Requirements gathering via AskUserQuestion
- Phase 2: Strategic planning and plan.md creation
- Phase 3: Human approval coordination
- Phase 5: Quality evaluation against 5 criteria
- Phase 6: Final delivery

**Why a Skill:**
- Needs full visibility across all domains for quality review
- Must understand backend, frontend, and testing to evaluate properly
- Coordinates with humans (AskUserQuestion)
- Strategic role requiring comprehensive context

**Model**: Opus 4.5 (superior reasoning for complex analysis)

---

#### 2. Orchestrator (Sonnet)
**File**: `skills/feature-builder-orchestrator/SKILL.md`

**Responsibilities:**
- Read and analyze approved plan.md
- Determine which agents to spawn (dynamic selection)
- Decompose plan into specific tasks per agent
- Manage dependencies between agents
- Coordinate parallel execution
- Submit integrated work to Evaluator
- Handle feedback and coordinate fixes

**Why a Skill:**
- Needs full context to coordinate across domains
- Must understand all agent roles to orchestrate effectively
- Strategic coordination role
- Manages communication between Evaluator and execution agents

**Model**: Sonnet (cost-effective for coordination tasks)

---

### Execution Layer (Agents)

#### 3. Backend Developer
**File**: `agents/backend-developer.md`

**Responsibilities:**
- API endpoints (REST, GraphQL, RPC)
- Database models, schemas, migrations
- Business logic and services
- Authentication & authorization
- Server-side validation
- Background jobs, email, external integrations

**Why an Agent:**
- Isolated context prevents frontend knowledge contamination
- Single domain responsibility (backend only)
- Can be given restricted tool permissions (future: no browser tools)
- Execution-focused, not strategic

**Model**: Sonnet

**Key Constraint**: Cannot access frontend or testing knowledge

---

#### 4. Frontend Developer
**File**: `agents/frontend-developer.md`

**Responsibilities:**
- UI components (React/Vue/Svelte)
- Styling (Tailwind, CSS Modules, CSS-in-JS)
- State management (Context, Redux, Zustand)
- Forms and client-side validation
- API integration (client-side)
- Responsive design
- Accessibility (WCAG compliance)

**Why an Agent:**
- Isolated context prevents backend knowledge contamination
- Single domain responsibility (frontend only)
- Can be given restricted tool permissions (future: browser tools only)
- Execution-focused, not strategic

**Model**: Sonnet

**Key Constraint**: Cannot access backend or testing knowledge

---

#### 5. Testing Specialist
**File**: `agents/testing-specialist.md`

**Responsibilities:**
- Unit tests (70% of testing)
- Integration tests (20% of testing)
- E2E tests (10% of testing)
- Test pyramid enforcement
- Coverage reporting

**Why an Agent:**
- Isolated context focused on testing only
- Single domain responsibility
- Can be given restricted tools (future: read code but not modify)
- Execution-focused, not strategic

**Model**: Sonnet

**Key Constraint**: Cannot access backend or frontend implementation knowledge

---

## Workflow

### High-Level Flow

```
User Request
    ↓
/feature-builder command
    ↓
Evaluator (skill)
    ├─ Phase 1: Gather requirements
    ├─ Phase 2: Create plan.md
    └─ Phase 3: Wait for human approval
    ↓
Orchestrator (skill)
    ├─ Read plan.md
    ├─ Decide which agents to spawn
    └─ Decompose tasks
    ↓
┌────────────────┼────────────────┐
│                │                │
Backend Agent  Frontend Agent  Testing Agent
(isolated)     (isolated)      (isolated)
    │                │                │
    └────────────────┴────────────────┘
                     ↓
            Orchestrator (skill)
                 integrates
                     ↓
            Evaluator (skill)
                 reviews
                     ↓
        [If issues: re-spawn agents]
        [If approved: delivery]
```

### Phase-by-Phase Breakdown

**Phase 1: Requirements Gathering**
- Evaluator uses AskUserQuestion to clarify requirements
- Iterates until complete understanding

**Phase 2: Strategic Planning**
- Evaluator creates `.mas/plans/{feature}_plan.md`
- Includes: requirements, technical approach, phases, agents needed

**Phase 3: Human Review (BLOCKING)**
- Human must approve plan before implementation
- Changes can be requested
- Critical quality gate

**Phase 4: Orchestrated Implementation**
- Orchestrator spawns agents from `agents/` folder
- Each agent works in isolated context
- Agents run in parallel where possible
- Dependencies managed by Orchestrator

**Phase 5: Quality Evaluation**
- Evaluator reviews against 5 criteria:
  1. Requirements Compliance
  2. Code Quality
  3. Clean Code Standards
  4. Security Review
  5. Performance
- Structured JSON feedback if issues found
- Max 3 iteration cycles

**Phase 6: Delivery**
- Evaluator confirms completion
- User receives production-ready code

---

## Agent Spawning (Technical Details)

### How Orchestrator Spawns Agents

The Orchestrator uses the Task tool with `subagent_type: "general-purpose"`:

```typescript
Task({
  subagent_type: "general-purpose",
  model: "sonnet",
  description: "Backend implementation",
  prompt: `Load the agent definition from agents/backend-developer.md
           and implement these tasks:

           1. [Specific task from plan]
           2. [Another task]

           Technical approach: [From plan.md]

           Report completion with file paths.`
})
```

**Key Insight**: The prompt instructs the spawned agent to load the agent definition file. This gives the agent:
- Its role and identity
- Guidelines and best practices
- Security requirements
- Code patterns and examples
- Success criteria

**Isolation Benefits**:
- Agent sees only its own definition + the prompt
- Cannot access other agent definitions (no cross-contamination)
- Focused execution without distraction

---

## Future Enhancements (from improvements-plan.md)

### Planned Agent Additions

**Specialized Review Agents:**
- `agents/security-auditor.md` - OWASP focused, read-only
- `agents/performance-analyzer.md` - N+1 queries, bundle size
- `agents/accessibility-reviewer.md` - WCAG compliance

**Knowledge Management:**
- `agents/librarian.md` - Catalogs patterns, suggests past solutions
- `.mas/knowledge/` folder for pattern storage

**Research & Design:**
- `agents/researcher.md` - Best practices lookup, current patterns
- `agents/designer.md` - Figma integration, design verification

### Tool Permission Evolution

Future versions will enforce tool restrictions:

```typescript
// Backend Agent permissions
{
  allowed: ["Write", "Edit", "Read", "Bash", "Grep", "Glob"],
  denied: ["browser tools", "design tools"]
}

// Security Auditor permissions
{
  allowed: ["Read", "Grep", "Glob"],
  denied: ["Write", "Edit", "Bash"]  // Read-only reviewer
}
```

---

## Comparison: Skills vs Agents Decision Matrix

| Characteristic | Use Skill | Use Agent |
|----------------|-----------|-----------|
| Needs full context across domains | ✓ | |
| Strategic/coordination role | ✓ | |
| Must interact with user (AskUserQuestion) | ✓ | |
| Domain-specific execution | | ✓ |
| Needs isolation from other domains | | ✓ |
| Benefits from tool restrictions | | ✓ |
| Parallel execution with others | | ✓ |
| Single responsibility | | ✓ |

---

## Migration from Previous Architecture

### What Changed

**Before (v1.0):**
```
skills/
  ├── feature-builder-evaluator/SKILL.md
  ├── feature-builder-orchestrator/SKILL.md
  ├── feature-builder-backend/SKILL.md      ← Moved
  ├── feature-builder-frontend/SKILL.md     ← Moved
  └── feature-builder-testing/SKILL.md      ← Moved
```

**After (v2.0):**
```
skills/
  ├── feature-builder-evaluator/SKILL.md    (unchanged)
  └── feature-builder-orchestrator/SKILL.md (updated to spawn from agents/)

agents/
  ├── backend-developer.md                  (moved from skills)
  ├── frontend-developer.md                 (moved from skills)
  └── testing-specialist.md                 (moved from skills)
```

### Breaking Changes

- Orchestrator prompts updated to reference `agents/` folder
- Agent files converted from SKILL.md format to plain markdown
- YAML frontmatter removed from agent files

### Benefits of Migration

✅ **Domain Isolation**: Frontend can't accidentally use backend patterns
✅ **Clear Boundaries**: Each execution agent has single responsibility
✅ **Scalability**: Easy to add specialized reviewers without contamination
✅ **Tool Control**: Foundation for future permission restrictions
✅ **Alignment**: Matches Compound Engineering's proven pattern

---

## Success Metrics

The architecture is successful if:

1. **No Cross-Contamination**: Frontend agents never produce backend code
2. **Clear Separation**: Each agent stays within its domain
3. **Cost Efficiency**: Dynamic spawning prevents unnecessary agent invocations
4. **Quality**: Evaluator catches issues across all domains
5. **Maintainability**: Easy to add new specialized agents

---

## Credits & Inspiration

This architecture is inspired by:
- [Compound Engineering Plugin](https://github.com/kieranklaassen/compound-engineering-plugin) - Agent isolation pattern
- Claude Code's MCP plugin architecture - Skills vs agents distinction
- The Evaluator-Orchestrator pattern - Strategic vs execution separation

---

*Architecture Version: 2.0*
*Last Updated: 2026-01-20*
*Author: Jones Trinh*
