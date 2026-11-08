> Snapshot backend tại thời điểm đồng bộ 08/11/2026; không phải trạng thái Git của repository tài liệu SRS này.

# Project State

Updated: 2026-11-07
Branch: `main` at `68b0244`, synchronized with `origin/main`.
PR #1 envelope, PR #2 frontend contract and PR #3 profile schema are merged
using merge commits. Original branch commits retain their SHAs.
This final handoff snapshot is a local, uncommitted state update after PR #3.


## Objective and approved baseline

Prepare the standalone coding platform with its own fresh database. Keep one
shared PostgreSQL database, Package → Level → Problem, synchronous sample Run,
and Submit Pending → Redis LPUSH/BRPOP → worker → Piston → persisted verdict.
AI will run only after persisted Accepted, independently of verdict and ranking.
The baseline is [decisions](docs/DECISIONS.md),
[AI context](docs/AI_CONTEXT.md) and the
[approved schema design](docs/superpowers/specs/2026-11-06-database-schema-design.md).

## Completed integration and branch cleanup

- PR #2 merged at `d3f840b`; PR #3 merged at `68b0244`. Review corrected
  the handoff status flag in `35632d0`; no remaining blocking findings.
- Verified all six branch tips and original implementation commits are ancestors
  of main before deleting their local/remote refs: remove-legacy-code,
  docs/api-envelope, database-schema, fix/api-envelope, frontend-api-v1 and
  schema-profile-fields. Only main remains locally/remotely; no open PRs.
- Final `go test -count=1 ./...` and PostgreSQL 16 bootstrap twice plus all SQL
  constraints passed on main `68b0244`. `go vet`, build, gofmt and OpenAPI
  validations passed on `76e018d`; its tested content exactly matches main
  apart from this state document. OpenAPI: 94 full/42 learner operations and
  2939/1303 examples. GitHub has no CI checks configured.
- GitNexus refreshed at main. Dedicated project test container removed.
  AGENTS changes and outputs workbook were byte-checked unchanged/unpublished.
- Integration limitations below remain. No live service/judge/frontend test.

## Completed profile schema correction

- Added nullable `users.avatar_url` (2048 characters) and `users.bio` (1000
  characters), with DB length checks matching target ProfilePatch limits.
- SQL regression first failed on missing `avatar_url`, then passed after adding
  the columns. Verified fresh bootstrap twice plus all constraints using only
  the disposable `codjingrank-profile-schema-test` PostgreSQL 16 container.
- Covers NULL defaults, persistence, maximum lengths including Unicode, clearing
  to NULL and rejecting oversized values. No judge logic or Go symbols changed.
- API/SRS/context documents now distinguish schema-backed profile fields from
  pending Go integration. Password recovery still requires token store/email;
  it is a separate Should feature, excluded from this correction.
- Original verification of the correction based on `ccddc11`:
  schema runner and `go test -count=1 ./...` passed; gofmt ran without Go diffs.
  OpenAPI validation passed: 94 full/42 learner operations, 2939/1303 examples,
  exact learner projection. Diff whitespace and AGENTS/workbook preservation
  checks passed. Schema/test correction committed in `d6c2adf`.
- Profile implementation commits: `d6c2adf`, `e3915c5`; integrated with
  current main through merge commit `76e018d`, without rebase or squash.

## Completed frontend mock contract

- Documentation-only work on `docs/frontend-api-v1`; Go/schema unchanged.
- Full Gateway OpenAPI:94 operations (52 admin), learner projection:42.
  Both cover F01–F13, UUID/camelCase DTOs, auth/errors/pagination, request and
  response examples, empty/lifecycle states. Every operation is target-only.
- [Frontend contract](docs/FRONTEND_API_CONTRACT.md) and
  [handoff](docs/FRONTEND_HANDOFF.md) explain mock commands/limitations and
  compatibility. SRS2.1 uses current packages/levels/shared database instead
  of old category DTOs/SQL/ERD. AI_CONTEXT and NEXT_STEPS reconciled.
- [Design](docs/superpowers/specs/2026-11-07-frontend-api-v1-design.md) and
  [plan](docs/superpowers/plans/2026-11-07-frontend-api-v1.md) record scope.
- Transport choices made explicit: current helper400 validation, endpoint422,
  102400-byte code limit, AI missing-row review:null, no public AI trigger.
  Optional reset-token/email persistence requires an extension; avatar/bio
  schema omission is corrected above (Go integration still pending);
  AI provider/quota/rubric/retry policy is not invented or marked implemented.
- Verified on documentation working tree based on `ac71ceb`: OpenAPI3.0.3,
  2,939 full/1,303 learner example validations, references/unique operation IDs,
  coverage/security, exact projection and hidden/provider field exclusion.
  Prism5.16.0 exercised94 full/42 learner operations and11 named lifecycle/
  empty states, missing-auth401 and bodyless logout204. Relative links resolve.
  `go test -count=1 ./...`, gofmt check and `git diff --check`: passed.
- Final review is self-review under owner single-agent instructions. No live
  DB/Redis/Piston/frontend-browser integration. Mock is stateless and cannot
  prove ownership, role enforcement, rank calculations or actual judge behavior.
- API envelope PR #1 merged via merge commit `ac71ceb`, original SHAs retained.
  All earlier branch commits are in main; no new feature runtime implemented.
- Contract work is published and merged through PR #2. Unrelated AGENTS
  changes and outputs workbook remain local and uncommitted.

## Completed

- Schema baseline committed in `0c78986`, guards in `5c01950`, repeatable SQL
  checks in `8567ef0`; 22 tables, 36 foreign keys. `ff17e0b` records its limits.
- Removed the repository/GitHub vertical: repo service, Auth OAuth/App methods,
  Gateway proxies, Accepted commit hook, config, submission metadata and SQL joins.
- Removed Subject/Session/Lesson handlers, DTOs, repositories and admin routes;
  retained language/problem/test/tag functionality for schema integration.
- Removed all seven legacy migrations, old seeds and legacy smoke scripts.
  New deployment starts from `schema.sql`; there is no old-data migration need.
- Removed the old Vite test UI including tracked dependencies/build output,
  its Make/CI targets and the old domain-specific Nginx configuration.
- Updated Compose/Make entrypoints and current README/deployment guidance.
  Historical audits/contracts are explicitly marked superseded for removed code.
- Added a Gateway contract test for removed routes and retained auth protection.

## Verification on cleanup working tree

- `go test -count=1 ./...`, `go vet ./...`, `go build ./...`: passed.
- Gateway removal test was also run against the original route file: failed for
  all nine tested retired GET/POST routes, then passed with the cleaned file.
- `gofmt`, `git diff --check`, both Compose `config --quiet`: passed.
- GitNexus impact checked 126 distinct function names. `Load` is CRITICAL
  (six service callers); only retired config fields were removed and callers
  updated. Ambiguous names/file lookups were cross-checked in source/build.
- GitNexus whole-diff detection reports CRITICAL due to deliberate large legacy
  deletion (including tracked UI dependencies). This is not a runtime safety proof.
- Run/judge/Piston/queue source and `schema.sql` were not modified in cleanup.
- No containers started, no database touched, no remote repositories changed.
  Earlier schema validation used unrelated infrastructure; do not repeat it.

## Integration limits and next work

- Retained Go IDs and SQL names are still incompatible with the target UUID
  schema. Passing Go tests proves compilation/unit behavior, not DB/API operation.
- Gateway `/api/v1`, new catalog APIs, Forum, AI service and target leaderboard
  are not implemented. No fake AI call replaces the removed commit hook.
- Adapt Auth/JWT/middleware and persistence to UUIDs/new schema, then catalog,
  Run/Submit consumers; add DB/worker integration and recovery tests.
- Keep current verdict mechanics while integrating accepted solutions/progress.
- Use a dedicated project PostgreSQL 16 test container, never HAPI or another
  project's service. `schema.sql` destructively bootstraps only a fresh database.
- Full Docker image build, live Piston/Redis judging and DB integration were not
  run in cleanup. SQL checks were not rerun because schema did not change.

## Existing unrelated work

AGENTS working rules and `outputs/` workbook predated this task and remain
uncommitted. Related API/SRS/context edits have been incorporated into this
contract task. Do not publish the workbook or unrelated AGENTS edits.
