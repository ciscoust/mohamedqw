# Security Specification for Goal Achievement App

## 1. Data Invariants
- A goal must belong to the user who created it (`userId == request.auth.uid`).
- Ritual logs must be linked to a user.
- Goal state transitions (phases) check: Users can only move phases forward or modify fields relevant to the current phase.
- `magnetStrength` is updated based on ritual completions (backend-like logic on client, but rules must allow it).
- Immutability: `createdAt` and `userId` cannot be changed after creation.

## 2. The Dirty Dozen Payloads (Negative Tests)

1. **Identity Theft**: Try to create a goal with `userId` of another user.
2. **Phase Skipping**: Try to set phase to `completed` without going through `before`, `during`, etc.
3. **Ghost Field**: Try to add `isAdmin: true` to a goal document.
4. **Time Spoofing**: Try to set `createdAt` to a date in the past instead of `request.time`.
5. **ID Poisoning**: Try to create a goal with a document ID that is 2KB of junk.
6. **Ritual Spoofing**: Try to log a ritual for a date in the future.
7. **Unauthorized Read**: User A tries to read User B's goals.
8. **Shadow Update**: Try to update `magnetStrength` to `999999` directly.
9. **Email Spoofing**: Rule uses email but `email_verified` is false.
10. **Terminal State Break**: Try to edit a goal that is already `completed`.
11. **PII Leak**: Accessing `/users/{userId}` without being that user.
12. **Orphaned Ritual**: Ritual log with non-existent `goalId`.

## 3. Test Runner Concept
The tests will verify `PERMISSION_DENIED` for all above payloads.
