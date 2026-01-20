# Feature Plan: CRUD API for [Resource Name]

## Status: PENDING_REVIEW

**Last Updated**: [Timestamp]
**Template**: CRUD API (RESTful Resource)

---

## Overview

[2-3 sentences describing the resource and its purpose in the application]

**Resource**: [Singular resource name, e.g., "User", "Product", "Order"]
**API Pattern**: RESTful CRUD operations

---

## Requirements

### Functional Requirements

- [ ] **Create** - Create new [resource] via POST endpoint
- [ ] **Read** - Retrieve single [resource] by ID via GET endpoint
- [ ] **Read All** - List all [resources] with pagination via GET endpoint
- [ ] **Update** - Update existing [resource] via PUT/PATCH endpoint
- [ ] **Delete** - Delete [resource] via DELETE endpoint
- [ ] **Validation** - Validate all input fields
- [ ] **Error Handling** - Return appropriate HTTP status codes and error messages

### Non-Functional Requirements

- [ ] **Performance**: API response time < 200ms for GET, < 500ms for mutations
- [ ] **Security**: Input validation, SQL injection prevention, authentication required
- [ ] **Scalability**: Support [X] requests per second
- [ ] **Data Validation**: Strong typing, schema validation

---

## API Endpoints

### POST /api/[resources]
**Purpose**: Create new [resource]

**Request Body**:
```json
{
  "field1": "value",
  "field2": "value"
}
```

**Response** (201 Created):
```json
{
  "id": "uuid",
  "field1": "value",
  "field2": "value",
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

**Error Responses**:
- 400 Bad Request: Validation failed
- 401 Unauthorized: Authentication required
- 409 Conflict: Resource already exists

---

### GET /api/[resources]/:id
**Purpose**: Retrieve single [resource]

**Response** (200 OK):
```json
{
  "id": "uuid",
  "field1": "value",
  "field2": "value",
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

**Error Responses**:
- 404 Not Found: Resource doesn't exist
- 401 Unauthorized: Authentication required

---

### GET /api/[resources]
**Purpose**: List all [resources] with pagination

**Query Parameters**:
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 20, max: 100)
- `sort` (optional): Sort field (default: createdAt)
- `order` (optional): Sort order (asc/desc, default: desc)

**Response** (200 OK):
```json
{
  "data": [
    { "id": "uuid", "field1": "value", ... }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

---

### PUT /api/[resources]/:id
**Purpose**: Update existing [resource]

**Request Body**:
```json
{
  "field1": "new value",
  "field2": "new value"
}
```

**Response** (200 OK):
```json
{
  "id": "uuid",
  "field1": "new value",
  "field2": "new value",
  "updatedAt": "timestamp"
}
```

**Error Responses**:
- 400 Bad Request: Validation failed
- 404 Not Found: Resource doesn't exist
- 401 Unauthorized: Authentication required

---

### DELETE /api/[resources]/:id
**Purpose**: Delete [resource]

**Response** (204 No Content): Empty body

**Error Responses**:
- 404 Not Found: Resource doesn't exist
- 401 Unauthorized: Authentication required
- 403 Forbidden: User doesn't own this resource

---

## Technical Approach

### Architecture Decision

RESTful API with the following layers:
1. **Route Handler**: HTTP request/response handling
2. **Service Layer**: Business logic
3. **Data Access Layer**: Database operations
4. **Validation Layer**: Input validation with Zod/Yup

**Rationale**: Separation of concerns, testability, maintainability

### Technology Stack

- **Backend**: [Framework and version]
- **Database**: [Database and ORM]
- **Validation**: [Validation library]
- **Testing**: [Test framework]

---

## Implementation Plan

### Phase 1: Database Schema

**Goal**: Create database model and migrations

- **Task 1.1**: Define [Resource] model/schema
  - Fields: id (UUID/auto-increment), [field1], [field2], createdAt, updatedAt
  - Indexes: id (primary key), [commonly queried fields]
  - Constraints: [NOT NULL, UNIQUE, etc.]

- **Task 1.2**: Create database migration
  - Generate migration file
  - Test up/down migrations

**Dependencies**: None
**Estimated Scope**: 2 files (model, migration)

---

### Phase 2: Data Access Layer

**Goal**: Create database operations

- **Task 2.1**: Create [Resource] repository/data access object
  - `create(data)`: Insert new record
  - `findById(id)`: Fetch by primary key
  - `findAll({ page, limit, sort })`: Fetch with pagination
  - `update(id, data)`: Update record
  - `delete(id)`: Delete record

**Dependencies**: Phase 1 completion
**Estimated Scope**: 1-2 files (repository/DAO)

---

### Phase 3: Business Logic Layer

**Goal**: Implement business logic and validation

- **Task 3.1**: Create [Resource] service
  - Input validation (Zod schemas)
  - Business rules enforcement
  - Error handling

- **Task 3.2**: Define validation schemas
  - Create schema (required fields)
  - Update schema (partial required)

**Dependencies**: Phase 2 completion
**Estimated Scope**: 2-3 files (service, validation schemas)

---

### Phase 4: API Route Handlers

**Goal**: Create HTTP endpoints

- **Task 4.1**: POST /api/[resources] (Create)
- **Task 4.2**: GET /api/[resources]/:id (Read one)
- **Task 4.3**: GET /api/[resources] (Read all with pagination)
- **Task 4.4**: PUT /api/[resources]/:id (Update)
- **Task 4.5**: DELETE /api/[resources]/:id (Delete)

- **Task 4.6**: Add authentication middleware
- **Task 4.7**: Add error handling middleware

**Dependencies**: Phase 3 completion
**Estimated Scope**: 5 route files + middleware

---

### Phase 5: Testing

**Goal**: Comprehensive test coverage

- **Task 5.1**: Unit tests for service layer
  - Test validation logic
  - Test business rules
  - Mock data access layer

- **Task 5.2**: Integration tests for API endpoints
  - Test CRUD operations
  - Test pagination
  - Test error responses
  - Test authentication

- **Task 5.3**: E2E tests for critical flows
  - Create → Read → Update → Delete flow

**Dependencies**: Phase 4 completion
**Estimated Scope**: 3-5 test files

---

## Agents Required

- [x] **Backend Agent**: Yes
  - **Reason**: API endpoints, database schema, business logic
  - **Tasks**: All phases (database, service, routes)

- [ ] **Frontend Agent**: No
  - **Reason**: API-only feature, no UI components

- [x] **Testing Agent**: Yes
  - **Reason**: Critical data operations require comprehensive testing
  - **Tasks**: Unit, integration, and E2E tests

---

## Acceptance Criteria

1. POST /api/[resources] creates new resource and returns 201 with created object
2. GET /api/[resources]/:id returns 200 with resource or 404 if not found
3. GET /api/[resources] returns paginated list with correct pagination metadata
4. PUT /api/[resources]/:id updates resource and returns 200 with updated object
5. DELETE /api/[resources]/:id deletes resource and returns 204
6. All endpoints require authentication (401 if not authenticated)
7. Invalid input returns 400 with clear validation error messages
8. Pagination works correctly (page, limit, total calculations)
9. Database queries use proper indexes (no full table scans)
10. All endpoints have >80% test coverage

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| SQL injection via unsanitized input | Low | Critical | Use parameterized queries (ORM handles this), validate all input with Zod |
| N+1 query problems in list endpoint | Medium | High | Use eager loading, add database query logging in tests |
| Missing pagination limits (DOS via large limit) | Medium | Medium | Enforce max limit (100), return 400 if exceeded |
| Concurrent updates (race conditions) | Low | Medium | Use optimistic locking with version field or database transactions |

---

## Dependencies

### External Dependencies
- [ORM library] (e.g., Prisma, TypeORM, Drizzle)
- [Validation library] (e.g., Zod, Yup)

### Internal Dependencies
- Database must be running and accessible
- Authentication middleware must exist

### Environment Requirements
- DATABASE_URL environment variable

---

## Estimated Scope

- **Files to create**: ~10-12 files
  - 1 database model/schema
  - 1 migration file
  - 1 repository/DAO file
  - 1 service file
  - 1-2 validation schema files
  - 5 API route handlers
  - 3-5 test files

- **Files to modify**: ~2 files
  - Database config (if adding new model)
  - API index (if routing setup needed)

- **New API endpoints**: 5 endpoints
  - POST /api/[resources]
  - GET /api/[resources]/:id
  - GET /api/[resources]
  - PUT /api/[resources]/:id
  - DELETE /api/[resources]/:id

- **Database changes**: 1 new table with indexes

---

## Security Considerations

- **Authentication**: All endpoints require valid authentication token
- **Authorization**: Users can only access their own resources (add ownership check)
- **Input Validation**: Zod schemas validate all input, reject unexpected fields
- **SQL Injection Prevention**: ORM uses parameterized queries
- **Rate Limiting**: Add rate limiting middleware (100 requests/minute per user)
- **Data Sanitization**: Trim strings, normalize email addresses

---

## Performance Considerations

- **Database Indexes**: Index on id (primary key) and commonly queried fields
- **Pagination**: Required for list endpoint, prevents loading all records
- **Query Optimization**: Use SELECT only needed fields, avoid SELECT *
- **Caching**: Consider Redis caching for frequently accessed resources
- **Connection Pooling**: Use database connection pool (ORM handles this)

**Expected Performance**:
- GET single: < 50ms (with database index)
- GET list: < 200ms (with pagination)
- POST/PUT/DELETE: < 500ms

---

## Implementation Progress

**Status Tracking** (Updated by Orchestrator during implementation):

### Backend Agent
- [ ] Phase 1: Database schema
- [ ] Phase 2: Data access layer
- [ ] Phase 3: Business logic layer
- [ ] Phase 4: API route handlers
- [ ] Phase 5: Testing

---

*Generated by Evaluator (Opus 4.5)*
*Template: CRUD API*
*Created*: [Timestamp]
