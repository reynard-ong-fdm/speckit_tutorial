# Data Model: Password Changing

## Entity: UserAccount
- Purpose: Authenticated user identity that owns credentials.
- Fields:
  - userId (string, unique, immutable)
  - passwordHash (string, required)
  - credentialVersion (integer, increments on successful password change)
  - passwordUpdatedAt (timestamp)
- Constraints:
  - Password hash must be one-way derived.
  - Previous credentials become invalid after successful update.
- Relationships:
  - One-to-many with AuthenticationSession
  - One-to-many with PasswordChangeAttempt
  - One-to-many with PasswordChangeAuditRecord

## Entity: PasswordChangeSubmission (Transient)
- Purpose: Frontend-only form submission context.
- Fields:
  - newPassword (string, required)
  - confirmPassword (string, required)
- Constraints:
  - Fields are used for frontend validation only.
  - Not persisted as a standalone backend record.

## Entity: PasswordChangeAttempt
- Purpose: Single backend password-change processing event.
- Fields:
  - attemptId (string, unique)
  - userId (string)
  - occurredAt (timestamp)
  - outcomeCode (enum)
  - isUserAttributableFailure (boolean)
- Outcome code enum:
  - PASSWORD_CHANGE_SUCCESS
  - DUPLICATE_PASSWORD_GIVEN
  - WRONG_PASSWORD_FORMAT
  - PASSWORD_CHANGE_UNAUTHORIZED
  - PASSWORD_CHANGE_THROTTLED
  - PASSWORD_CHANGE_TEMPORARY_FAILURE
  - PASSWORD_CHANGE_CONFLICT
- Constraints:
  - Exactly one primary outcome code for failed attempts.
  - Backend-caused temporary failures set isUserAttributableFailure=false.

## Entity: PasswordThrottleState
- Purpose: Tracks anti-abuse counters and delay level.
- Fields:
  - userId (string, unique)
  - rollingWindowStartAt (timestamp)
  - attributableFailureCount15m (integer)
  - currentDelaySeconds (integer)
  - lastFailureAt (timestamp)
- Constraints:
  - Count only user-attributable failures.
  - Progressive delay starts after 5 attributable failures in 15 minutes.

## Entity: AuthenticationSession
- Purpose: Session or token state bound to a user.
- Fields:
  - sessionId (string, unique)
  - userId (string)
  - issuedAt (timestamp)
  - invalidatedAt (timestamp, nullable)
  - invalidationReason (enum, nullable)
- Constraints:
  - Invalidate all non-current sessions on successful password change.
  - Invalidated sessions rejected within 60 seconds on subsequent requests.

## Entity: PasswordChangeAuditRecord
- Purpose: Immutable audit trail for each attempt.
- Fields:
  - auditId (string, unique)
  - userId (string)
  - occurredAt (timestamp)
  - outcomeCode (enum)
  - sourceIp (string)
  - sourceDevice (string)
  - requestId (string)
- Constraints:
  - Record exists for success and every failure category.
  - No plaintext or transformed password values in audit content.

## State Transitions

### PasswordChangeAttempt
- Received -> Authorized -> PolicyChecked ->
  - Success
  - RejectedDuplicate
  - RejectedFormat
  - RejectedUnauthorized
  - RejectedThrottled
  - FailedTemporary
  - FailedConflict

### AuthenticationSession
- Active -> Invalidated (non-current sessions after successful password change)
- Current session remains active after successful update

### PasswordThrottleState
- Normal -> ThresholdReached (5 attributable failures in rolling 15 minutes) -> Delayed
- Delayed -> Normal when window decays below threshold
