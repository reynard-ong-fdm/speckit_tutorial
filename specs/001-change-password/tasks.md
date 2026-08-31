---

description: "Task list for implementing Password Changing feature"
---

# Tasks: Password Changing

**Input**: Design documents from `specs/001-change-password/`

**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/

**Tests**: Included. The specification and constitution require strong QA-focused coverage.

**Organization**: Tasks are grouped by user story so each story can be implemented and validated independently.

## Format: `[ID] [P?] [Story] Description`

- [P] indicates tasks that can run in parallel.
- [Story] is present only on user story tasks: [US1], [US2], [US3].
- Every task includes an exact file path.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Initialize project scaffolding and baseline configuration for frontend and backend work.

- [ ] T001 Create backend and frontend module structure in backend/src/.gitkeep
- [ ] T002 Initialize backend dependencies and scripts for testing in backend/package.json
- [ ] T003 [P] Initialize frontend dependencies and scripts for testing in frontend/package.json
- [ ] T004 [P] Add shared lint and formatting configuration in .eslintrc.cjs
- [ ] T005 Create environment variable templates for local setup in backend/.env.example
- [ ] T006 [P] Create frontend environment template in frontend/.env.example

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Build shared capabilities required before any user story implementation.

**Critical**: User story implementation begins only after this phase is complete.

- [ ] T007 Create foundational persistence migration for throttle, audit, and session invalidation state in backend/src/models/migrations/001_password_change_foundation.sql
- [ ] T008 [P] Implement authenticated-request context middleware in backend/src/security/auth_middleware.ts
- [ ] T009 [P] Define deterministic password-change outcome code model in backend/src/models/password_change_outcome.ts
- [ ] T010 [P] Implement standardized API error and success response mapper in backend/src/api/password_change_response_mapper.ts
- [ ] T011 Implement throttle-state service interface and data access contract in backend/src/services/password_throttle_service.ts
- [ ] T012 [P] Implement audit-record service interface and sink adapter in backend/src/services/password_change_audit_service.ts
- [ ] T013 [P] Implement session invalidation service interface in backend/src/services/session_invalidation_service.ts
- [ ] T014 Implement frontend password-change API client with typed outcomes in frontend/src/services/passwordChangeApi.ts
- [ ] T015 [P] Implement frontend validation utility module for mismatch and format checks in frontend/src/services/passwordValidation.ts

**Checkpoint**: Foundational services, contracts, and shared utilities are ready.

---

## Phase 3: User Story 1 - Change Own Password Successfully (Priority: P1)

**Goal**: Allow an authenticated user to change password successfully and return to the previous page.

**Independent Test**: User can complete the flow with valid input, receive success feedback, and observe old password rejection on next sign-in.

### Tests for User Story 1

- [ ] T016 [P] [US1] Add frontend integration test for successful change-password journey in frontend/tests/integration/password-change-success.spec.ts
- [ ] T017 [P] [US1] Add backend integration test for successful update and session invalidation in backend/tests/integration/password-change-success.spec.ts
- [ ] T018 [P] [US1] Add contract test for 200 PASSWORD_CHANGE_SUCCESS response in backend/tests/contract/password-change-success.contract.spec.ts

### Implementation for User Story 1

- [ ] T019 [P] [US1] Create change-password page with required fields and submit action in frontend/src/pages/ChangePasswordPage.tsx
- [ ] T020 [P] [US1] Add success-message component and return-navigation behavior in frontend/src/components/PasswordChangeSuccessBanner.tsx
- [ ] T021 [US1] Wire frontend submit flow to call API only after local validation pass in frontend/src/pages/ChangePasswordPage.tsx
- [ ] T022 [P] [US1] Implement password-update domain service with secure hashing in backend/src/services/password_update_service.ts
- [ ] T023 [US1] Implement password-change controller success orchestration in backend/src/api/password_change_controller.ts
- [ ] T024 [US1] Invalidate all non-current sessions after successful update in backend/src/services/session_invalidation_service.ts
- [ ] T025 [US1] Persist success outcome audit entry in backend/src/services/password_change_audit_service.ts

**Checkpoint**: User Story 1 is independently functional and testable.

---

## Phase 4: User Story 2 - Prevent Invalid Password Changes (Priority: P2)

**Goal**: Prevent invalid changes via frontend-first checks and deterministic backend policy enforcement.

**Independent Test**: Mismatch/format failures on frontend produce immediate errors with zero backend calls; backend returns expected deterministic outcomes for duplicate, throttled, and temporary failure cases.

### Tests for User Story 2

- [ ] T026 [P] [US2] Add frontend unit tests for mismatch and format no-backend-call behavior in frontend/tests/unit/password-change-client-validation.spec.ts
- [ ] T027 [P] [US2] Add backend unit tests for duplicate and format policy decisions in backend/tests/unit/password-policy.spec.ts
- [ ] T028 [P] [US2] Add backend integration tests for progressive throttling and failure accounting in backend/tests/integration/password-throttle.spec.ts
- [ ] T029 [P] [US2] Add contract tests for 400/429/503 outcome mappings in backend/tests/contract/password-change-errors.contract.spec.ts

### Implementation for User Story 2

- [ ] T030 [US2] Implement click-path mismatch and format validation checks in frontend/src/services/passwordValidation.ts
- [ ] T031 [US2] Implement immediate frontend error rendering and backend-call short-circuit in frontend/src/pages/ChangePasswordPage.tsx
- [ ] T032 [US2] Implement duplicate-password detection policy in backend/src/services/password_update_service.ts
- [ ] T033 [US2] Implement backend format-policy validation and WRONG_PASSWORD_FORMAT mapping in backend/src/services/password_policy_service.ts
- [ ] T034 [US2] Implement progressive throttling computation and PASSWORD_CHANGE_THROTTLED responses in backend/src/services/password_throttle_service.ts
- [ ] T035 [US2] Implement temporary-backend-failure handling with non-attributable failure accounting in backend/src/api/password_change_controller.ts
- [ ] T036 [US2] Enforce single deterministic failure precedence order in backend/src/api/password_change_controller.ts
- [ ] T037 [US2] Persist duplicate/format/throttled/temporary failure audit outcomes in backend/src/services/password_change_audit_service.ts

**Checkpoint**: User Story 2 is independently functional and testable.

---

## Phase 5: User Story 3 - Enforce Authorization Boundaries (Priority: P3)

**Goal**: Ensure unauthorized users cannot access page or perform API password changes.

**Independent Test**: Unauthenticated page access redirects to login, and unauthenticated API requests return PASSWORD_CHANGE_UNAUTHORIZED.

### Tests for User Story 3

- [ ] T038 [P] [US3] Add frontend integration test for unauthorized route redirect behavior in frontend/tests/integration/password-change-unauthorized.spec.ts
- [ ] T039 [P] [US3] Add backend integration test for unauthenticated API rejection in backend/tests/integration/password-change-unauthorized.spec.ts
- [ ] T040 [P] [US3] Add contract test for 401 PASSWORD_CHANGE_UNAUTHORIZED response in backend/tests/contract/password-change-unauthorized.contract.spec.ts

### Implementation for User Story 3

- [ ] T041 [US3] Implement frontend route-guard redirect to login for password page in frontend/src/pages/ChangePasswordPage.tsx
- [ ] T042 [US3] Implement unauthorized outcome handling in controller using auth middleware context in backend/src/api/password_change_controller.ts
- [ ] T043 [US3] Persist unauthorized-attempt audit outcomes in backend/src/services/password_change_audit_service.ts

**Checkpoint**: User Story 3 is independently functional and testable.

---

## Phase 6: Polish and Cross-Cutting Concerns

**Purpose**: Harden quality across stories and validate end-to-end readiness.

- [ ] T044 [P] Add observability metrics for outcome-code distribution, throttle events, and session invalidation timing in backend/src/services/password_change_metrics.ts
- [ ] T045 [P] Add performance and concurrency regression integration tests for p95 and 10k-user assumptions in backend/tests/integration/password-change-performance.spec.ts
- [ ] T046 Update operational validation guide with final run steps and expected outputs in specs/001-change-password/quickstart.md
- [ ] T047 Execute full quickstart validation and capture results in specs/001-change-password/validation-report.md
- [ ] T048 Perform code cleanup for readability and maintainability in frontend/src/pages/ChangePasswordPage.tsx

---

## Phase 7: Analysis Remediation Coverage

**Purpose**: Close specification-analysis coverage gaps before implementation execution.

- [ ] T049 [US2] Implement centralized payload sanitization before policy evaluation in backend/src/api/password_change_controller.ts
- [ ] T050 [P] [US2] Add tests proving sanitization runs before validation and storage processing in backend/tests/integration/password-change-sanitization.spec.ts
- [ ] T051 [US2] Enforce audit-record field allowlist to exclude credential content and derivatives in backend/src/services/password_change_audit_service.ts
- [ ] T052 [P] [US2] Add tests proving audit records never contain plaintext or transformed password inputs in backend/tests/integration/password-change-audit-redaction.spec.ts
- [ ] T053 [US2] Implement per-user password-change serialization to allow one success and deterministic stale conflict outcomes in backend/src/services/password_update_service.ts
- [ ] T054 [P] [US2] Add concurrency integration test proving single success and deterministic conflict outcomes in backend/tests/integration/password-change-concurrency.spec.ts
- [ ] T055 [P] [US1] Add integration SLA test asserting invalidated non-current sessions are rejected within 60 seconds in backend/tests/integration/password-change-session-invalidation-sla.spec.ts
- [ ] T056 [P] [US2] Add frontend integration performance test asserting mismatch/format validation errors render under 1 second with zero backend calls in frontend/tests/integration/password-change-client-validation-latency.spec.ts
- [ ] T057 [P] [US2] Add integration test asserting deterministic failure precedence order in backend/tests/integration/password-change-precedence.spec.ts
- [ ] T058 [P] [US3] Add guardrail integration test ensuring password-change flow never invokes two-factor challenge behavior in backend/tests/integration/password-change-no-2fa.spec.ts
- [ ] T059 [P] [US2] Add contract test asserting malformed payload type failures map to WRONG_PASSWORD_FORMAT in backend/tests/contract/password-change-malformed-payload.contract.spec.ts

---

## Dependencies and Execution Order

### Phase Dependencies

- Phase 1 (Setup) has no dependencies.
- Phase 2 (Foundational) depends on Phase 1 and blocks all user stories.
- Phase 3 (US1), Phase 4 (US2), and Phase 5 (US3) each depend on Phase 2 completion.
- Phase 6 (Polish) depends on completion of desired user stories.
- Phase 7 (Analysis Remediation Coverage) depends on completion of relevant story implementations and must complete before final validation sign-off.

### User Story Dependencies

- US1 depends only on Foundational phase.
- US2 depends only on Foundational phase, and remains independently testable.
- US3 depends only on Foundational phase, and remains independently testable.

### Within Each Story

- Write tests first for that story, then implement models/services/controllers/pages.
- Complete story-level implementation before polishing cross-cutting concerns.

## Parallel Opportunities

- Setup tasks T003, T004, and T006 can run in parallel after T001.
- Foundational tasks T008, T009, T010, T012, T013, and T015 can run in parallel after T007.
- In each story, test tasks marked [P] can run concurrently.
- Phase 7 test tasks T050, T052, T054, T055, T056, T057, T058, and T059 can run concurrently once their dependent implementation tasks are ready.
- Across stories, US1/US2/US3 can proceed in parallel after Foundational completion.

## Parallel Example: User Story 2

```bash
Task: "T026 [US2] frontend no-backend-call tests in frontend/tests/unit/password-change-client-validation.spec.ts"
Task: "T027 [US2] backend policy unit tests in backend/tests/unit/password-policy.spec.ts"
Task: "T029 [US2] contract tests in backend/tests/contract/password-change-errors.contract.spec.ts"
```

## Implementation Strategy

### MVP First (US1)

1. Complete Phase 1 and Phase 2.
2. Complete US1 tasks (T016-T025).
3. Validate US1 independently before moving to additional stories.

### Incremental Delivery

1. Deliver US1 as the first production-capable increment.
2. Add US2 for robust invalid-case handling.
3. Add US3 for strict authorization boundaries.
4. Finish with Phase 6 hardening and validation.

### Multi-Developer Parallel Plan

1. Team finishes Setup + Foundational together.
2. After checkpoint, assign separate owners to US1, US2, and US3 phases.
3. Merge by contract and integration test gates.
