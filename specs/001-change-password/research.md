# Research: Password Changing

## Decision: Validation boundary split
- Decision: Keep mismatch and format validation on the frontend click path before any API call.
- Rationale: Immediate feedback and lower backend load; aligns with clarified spec decisions.
- Alternatives considered: Backend-first validation for all failures (rejected due to slower UX and unnecessary round trips).

## Decision: Backend mismatch handling
- Decision: Backend does not implement explicit mismatch-comparison logic.
- Rationale: Confirm-password exists for client-side confirmation only in this flow.
- Alternatives considered: Backend mismatch code path (rejected per feature clarification).

## Decision: Input persistence policy
- Decision: New-password and confirm-password pair is transient and not stored as a standalone record.
- Rationale: Reduces sensitive data exposure and unnecessary persistence.
- Alternatives considered: Persist form submissions (rejected due to security risk and no business value).

## Decision: Backend policy checks
- Decision: Backend enforces duplicate-password, password-strength, authorization, throttling,
  temporary-failure handling, and deterministic outcome mapping.
- Rationale: These are trust-boundary checks that cannot rely only on frontend behavior.
- Alternatives considered: Client-only policy enforcement (rejected as insecure).

## Decision: Progressive throttling fairness
- Decision: After 5 user-attributable failures in a rolling 15-minute window, progressive delay
  applies; temporary backend failures do not increment throttle counters.
- Rationale: Security control with user-fair failure accounting.
- Alternatives considered: Count all failures (rejected), fixed lockout (rejected for UX cost).

## Decision: Session invalidation behavior
- Decision: On successful password change, invalidate all non-current sessions and reject them
  within 60 seconds.
- Rationale: Limits compromised-session window while preserving current-flow continuity.
- Alternatives considered: Keep all sessions active (rejected), force logout of current session too
  (rejected for UX friction).

## Decision: Audit coverage
- Decision: Audit every attempt with user id, timestamp, outcome code, and source metadata;
  never log plaintext or transformed password values.
- Rationale: Forensic traceability without credential leakage.
- Alternatives considered: Success-only audits (rejected as insufficient).

## Decision: Contract strategy
- Decision: Publish one authenticated password-change endpoint contract with deterministic status
  and outcome-code mapping.
- Rationale: Stable integration boundary for frontend, tests, and backend behavior.
- Alternatives considered: Multiple specialized endpoints (rejected as unnecessary complexity).

## Decision: Test strategy
- Decision: Require unit, integration, contract, and end-to-end validation.
- Rationale: Satisfies constitution requirements for component confidence and cross-boundary checks.
- Alternatives considered: Unit-only or integration-only strategies (rejected due to coverage gaps).
