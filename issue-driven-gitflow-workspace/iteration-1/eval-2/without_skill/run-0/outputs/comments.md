author:	andybaran
association:	owner
edited:	false
status:	none
--
Addressed in #6. Added `limit`/`offset` pagination to `GET /items` with a default page size of 20 (configurable, capped at 100), documented the params in the README, and added tests. PR targets the `bootstrap` base branch.
--
