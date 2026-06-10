# Issues Touched

Three feature issues were created and each received a comment linking the implementation PR (#4, draft).

## Issue #1 — User authentication
- Created with scope + acceptance criteria.
- Comment: "Initial implementation in #4 (draft): scrypt password hashing, HMAC-signed expiring bearer tokens, register/login, and an authenticate() middleware guarding protected routes."

## Issue #2 — Notes CRUD API
- Created with scope + acceptance criteria.
- Comment: "Initial implementation in #4 (draft): per-user create/read/update/delete with ownership enforcement so users cannot access each other's notes."

## Issue #3 — Sync service
- Created with scope + acceptance criteria.
- Comment: "Initial implementation in #4 (draft): pull (changes since timestamp) and push (client batch) endpoints with last-write-wins conflict resolution."

All three issues are referenced by `Closes #1/#2/#3` in PR #4's body and commit message, so they will close when the (currently draft, unmerged) PR merges.
