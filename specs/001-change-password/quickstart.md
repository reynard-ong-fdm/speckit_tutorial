# Quickstart: Password Changing Validation

## Purpose
Validate the password-changing feature end-to-end against the specification and contract.

## Prerequisites
- Running backend and frontend services in a development environment
- Test user account with valid authentication
- Access to audit logging outputs
- Contract reference at `contracts/password-change.openapi.yaml`

## Setup
1. Install project dependencies.
2. Configure environment values for auth/session and datastore access.
3. Start backend and frontend services.

Example command pattern:

```bash
npm install
npm run dev:backend
npm run dev:frontend
```

## Validation Scenarios

### 1) Successful password change
1. Sign in as a test user.
2. Open the change-password page.
3. Enter valid matching password values.
4. Submit and verify:
   - Success response is returned.
   - Success message is displayed.
   - User is returned to previous page.
   - Non-current sessions are invalidated.

### 2) Frontend-only mismatch and format checks
1. Enter mismatched password values and click Confirm Changes.
2. Verify immediate PASSWORD_INPUTS_MISMATCH error on frontend.
3. Verify no backend request is emitted.
4. Enter format-invalid password and click Confirm Changes.
5. Verify immediate format error on frontend.
6. Verify no backend request is emitted.

### 3) Backend domain-policy checks
1. Submit a validly matched password that equals current password.
2. Verify `DUPLICATE_PASSWORD_GIVEN`.
3. Submit a format-invalid new password via direct API request.
4. Verify `WRONG_PASSWORD_FORMAT`.

### 4) Unauthorized behavior
1. Access change-password page without a valid session.
2. Verify redirect to login page.
3. Call API without valid authentication.
4. Verify `PASSWORD_CHANGE_UNAUTHORIZED`.

### 5) Throttling and temporary-failure fairness
1. Trigger 5 user-attributable failed attempts in 15 minutes.
2. Submit another request and verify `PASSWORD_CHANGE_THROTTLED`.
3. Simulate temporary backend failure for a valid request.
4. Verify `PASSWORD_CHANGE_TEMPORARY_FAILURE`.
5. Verify backend failure did not increment throttling counters.

### 6) Concurrency behavior
1. Submit concurrent valid requests for the same user.
2. Verify exactly one succeeds.
3. Verify stale concurrent request receives deterministic conflict outcome.

### 7) Audit and security checks
1. Verify audit record exists for every success and failure outcome.
2. Verify audit entries include user id, timestamp, outcome code, and source metadata.
3. Verify audit entries contain no plaintext or transformed password values.
4. Verify stored credentials remain one-way hashed.

## References
- Specification: `spec.md`
- Data model: `data-model.md`
- API contract: `contracts/password-change.openapi.yaml`

## Test Execution Guidance
Run validation suites in this order:
1. Unit tests for frontend validation and backend policy modules.
2. Contract tests for API status and outcome-code mappings.
3. Integration tests for sessions, throttling, audit persistence, and conflict handling.
4. End-to-end tests for complete user journeys and no-backend-call assertions.

Example command pattern:

```bash
npm run test:unit
npm run test:contract
npm run test:integration
npm run test:e2e
```

## Expected Outcomes
- All scenarios pass.
- Deterministic outcome codes are emitted for failed backend requests.
- Performance and scale criteria from `spec.md` are met.
