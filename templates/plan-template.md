# Feature Plan: [Feature Name]

## Status: PENDING_REVIEW

**Last Updated**: [Timestamp]

---

## Overview

[2-3 sentence description of the feature and its purpose]

---

## Requirements

### Functional Requirements

- [ ] Requirement 1 - [Description]
- [ ] Requirement 2 - [Description]
- [ ] Requirement 3 - [Description]

### Non-Functional Requirements

- [ ] **Performance**: [Specific metrics - e.g., "Page load < 2s", "API response < 500ms"]
- [ ] **Security**: [Specific requirements - e.g., "Passwords hashed with bcrypt", "HTTPS only"]
- [ ] **Accessibility**: [WCAG level if applicable - e.g., "WCAG 2.1 Level AA compliance"]
- [ ] **Browser Support**: [Required browsers/versions - e.g., "Chrome 90+, Firefox 88+, Safari 14+"]
- [ ] **Scalability**: [Expected load - e.g., "Support 10,000 concurrent users"]

---

## Technical Approach

### Architecture Decision

[Explain the high-level architecture choice and the reasoning behind it]

**Example:**
> Server-side sessions chosen over JWT for better security. Sessions can be revoked server-side immediately (critical for auth), whereas JWTs cannot be invalidated until expiration. The security benefit outweighs the slight performance overhead of session storage.

### Technology Stack

- **Backend**: [Technology and version - e.g., "Next.js 14 API Routes with TypeScript"]
- **Frontend**: [Technology and version - e.g., "React 18 with TypeScript"]
- **Database**: [Technology and version - e.g., "PostgreSQL 15 with Prisma ORM"]
- **External Services**: [Third-party integrations - e.g., "SendGrid for email", "Stripe for payments"]
- **Testing**: [Test framework - e.g., "Vitest for unit tests, Playwright for E2E"]

---

## Implementation Plan

### Phase 1: [Phase Name - e.g., "Backend Foundation"]

**Goal**: [What this phase achieves]

- **Task 1.1**: [Specific, actionable task with file path if known]
  - Details: [Any important implementation notes]
- **Task 1.2**: [Specific, actionable task]
- **Task 1.3**: [Specific, actionable task]

**Dependencies**: [What must be completed before this phase can start]
**Estimated Scope**: [Files to create/modify]

### Phase 2: [Phase Name - e.g., "API Development"]

**Goal**: [What this phase achieves]

- **Task 2.1**: [Specific, actionable task]
- **Task 2.2**: [Specific, actionable task]

**Dependencies**: Phase 1 completion
**Estimated Scope**: [Files to create/modify]

### Phase 3: [Phase Name - e.g., "Frontend Implementation"]

**Goal**: [What this phase achieves]

- **Task 3.1**: [Specific, actionable task]
- **Task 3.2**: [Specific, actionable task]

**Dependencies**: Phase 2 completion (APIs must exist)
**Estimated Scope**: [Files to create/modify]

### Phase 4: [Phase Name - e.g., "Testing & Validation"]

**Goal**: [What this phase achieves]

- **Task 4.1**: [Specific, actionable task]
- **Task 4.2**: [Specific, actionable task]

**Dependencies**: Phases 1-3 completion
**Estimated Scope**: [Test files to create]

---

## Agents Required

This section guides the Orchestrator on which specialized agents to spawn.

- [ ] **Backend Agent**: [Yes/No]
  - **Reason**: [e.g., "API endpoints and database schema changes needed"]
  - **Tasks**: [High-level summary of backend work]

- [ ] **Frontend Agent**: [Yes/No]
  - **Reason**: [e.g., "UI components and state management required"]
  - **Tasks**: [High-level summary of frontend work]

- [ ] **Testing Agent**: [Yes/No]
  - **Reason**: [e.g., "Critical security feature requires comprehensive testing"]
  - **Tasks**: [High-level summary of testing work]

---

## Acceptance Criteria

These are the testable conditions that determine when the feature is complete.

1. [Specific, testable criterion - e.g., "User can register with valid email and password"]
2. [Specific, testable criterion - e.g., "User receives confirmation email after registration"]
3. [Specific, testable criterion - e.g., "Invalid email format is rejected with clear error message"]
4. [Specific, testable criterion - e.g., "Passwords are hashed with bcrypt (cost factor 12)"]
5. [Specific, testable criterion - e.g., "User can login and session persists across page refresh"]

**Verification Method**: [How to verify - e.g., "Unit tests", "Integration tests", "Manual QA checklist"]

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| [Risk description - e.g., "Third-party email service downtime"] | Low/Medium/High | Low/Medium/High | [How to mitigate - e.g., "Implement retry queue, fallback to alternative service"] |
| [Risk description - e.g., "Password reset tokens could be guessed"] | Low | High | [Mitigation - e.g., "Use cryptographically secure random tokens (32 bytes), expire after 1 hour"] |
| [Risk description] | [Likelihood] | [Impact] | [Mitigation] |

---

## Dependencies

### External Dependencies
- [External service or library - e.g., "SendGrid API for email sending"]
- [External service or library - e.g., "bcrypt library for password hashing"]

### Internal Dependencies
- [Existing code that will be affected - e.g., "Existing User model must be extended"]
- [Infrastructure requirement - e.g., "PostgreSQL database must be running"]

### Environment Requirements
- [Environment variable - e.g., "SENDGRID_API_KEY environment variable"]
- [Configuration - e.g., "Redis server for session storage"]

---

## Estimated Scope

This helps set expectations for the implementation effort.

- **Files to create**: [Approximate count - e.g., "~8 files"]
  - [Example: "3 API route handlers"]
  - [Example: "2 frontend components"]
  - [Example: "3 test files"]

- **Files to modify**: [Approximate count - e.g., "~3 files"]
  - [Example: "User model (add password field)"]
  - [Example: "Main layout (add AuthProvider)"]

- **New API endpoints**: [Count - e.g., "4 endpoints"]
  - `POST /api/auth/register`
  - `POST /api/auth/login`
  - `POST /api/auth/logout`
  - `POST /api/auth/reset-password`

- **New UI components**: [Count - e.g., "3 components"]
  - LoginForm
  - RegisterForm
  - PasswordResetForm

- **Database changes**: [Description - e.g., "Add users table with email, passwordHash, createdAt fields"]

---

## Security Considerations

[Specific security measures for this feature]

**Example:**
- All passwords hashed with bcrypt (cost factor 12, never stored plaintext)
- Session cookies configured with httpOnly, secure, sameSite flags
- Rate limiting on login endpoint (5 attempts per 15 minutes)
- Input validation on all endpoints (email format, password length)
- SQL injection prevention via parameterized queries
- XSS prevention via output escaping

---

## Performance Considerations

[Expected performance characteristics and optimizations]

**Example:**
- Login endpoint expected response time: < 500ms
- Database query optimization: index on users.email field
- Session storage: Redis for fast lookup (< 10ms)
- No N+1 query issues (verified in code review)

---

## Future Enhancements

[Optional features that are out of scope for this phase but could be added later]

- [Future enhancement - e.g., "Two-factor authentication (2FA)"]
- [Future enhancement - e.g., "Social login (OAuth with Google, GitHub)"]
- [Future enhancement - e.g., "Password strength meter"]

**Note**: These are NOT part of the current implementation plan.

---

## Implementation Progress

**Status Tracking** (Updated by Orchestrator during implementation):

### Backend Agent
- [ ] Task 1.1 - [Description]
- [ ] Task 1.2 - [Description]

### Frontend Agent
- [ ] Task 2.1 - [Description]
- [ ] Task 2.2 - [Description]

### Testing Agent
- [ ] Task 3.1 - [Description]
- [ ] Task 3.2 - [Description]

---

## Evaluator Review Notes

[Space for Evaluator to add review comments during quality assurance]

### Review Iteration 1
**Date**: [Timestamp]
**Status**: [APPROVED / REVISION_NEEDED]

**Issues Found**:
- [Issue ID] - [Description]

**Feedback**:
- [Detailed feedback]

---

## Change Log

| Date | Change | Reason |
|------|--------|--------|
| [Date] | Plan created | Initial planning phase |
| [Date] | Updated technical approach from JWT to sessions | User preference for better security |
| [Date] | Status updated to APPROVED | Human approval received |
| [Date] | Status updated to IN_PROGRESS | Implementation started |

---

*Generated by Evaluator (Opus 4.5)*
*Created*: [Timestamp]
*Last Modified*: [Timestamp]
*Awaiting human approval before implementation*

---

## Instructions for Human Review

**Please review this plan and respond with:**

1. **Feedback or changes**: Tell the Evaluator what to modify (e.g., "Change the database from PostgreSQL to MongoDB")
2. **Questions**: Ask about any architecture decisions or approach
3. **Approval**: Say "approved", "proceed", or "looks good" to begin implementation
4. **Rejection**: Say "reject" or "start over" to discard and restart planning

**The system will BLOCK until you explicitly approve. No code will be written before approval.**

---

## Plan Status Guide

- **PENDING_REVIEW**: Plan created, waiting for human review
- **APPROVED**: Human has approved, ready for implementation
- **IN_PROGRESS**: Orchestrator and subagents are implementing
- **COMPLETED**: Implementation finished and passed Evaluator quality review
- **BLOCKED**: Issues found that require human intervention
