# MediQueue Repository Rules

**Project owner and sole credited author identity: `ChamathDilshanC`.**

These rules apply to every contribution to the main repository and must be copied or linked into the frontend, backend and configuration repositories when they are created. They guide humans and coding agents; they do not replace branch protection or review.

## Attribution and Git identity

- Do not add AI assistants, tools, models or vendors as project authors, maintainers, contributors, co-authors or core team members in README files, docs, package metadata, releases, commit trailers or UI credits.
- Do not add `Co-authored-by`, `Generated-by`, assistant signatures or similar attribution trailers to commits. Preserve any legally required third-party license notices and dependency attribution; this rule does not remove those obligations.
- Project-facing owner/author credit must read exactly `ChamathDilshanC`. Do not claim another human wrote a change or rewrite third-party commit history.
- Before committing, explicitly inspect `git config user.name`, `git config user.email`, the staged diff and the resulting commit metadata. Use `ChamathDilshanC` as `user.name`; use an owner-supplied or GitHub-verified address before pushing. A local draft commit may use `ChamathDilshanC@users.noreply.github.com` provisionally, but verify its GitHub attribution before publishing. Do not infer a private email from a username.
- Never use global Git identity for this project without checking it. Set identity at repository scope. Keep commit messages factual and specific.

## Architecture and code

- Follow `architecture.md` and `TECH_STACK.md`. Changes to service ownership, contracts, authorization, persistence or config flow require updating those docs in the same PR.
- Frontend and mobile call versioned APIs; backend alone performs queue mutations. PostgreSQL is authoritative. All queue commands are transactional, idempotent where retried, audited and tenant scoped.
- Each backend service owns its schema and migrations. Do not write into another service's schema. Publish cross-service changes with versioned events and an outbox.
- Never expose patient-identifying data on public display channels. Enforce role/branch access for staff channels and config. Never commit credentials, service-role keys, tokens, patient records or production exports.
- Validate all configuration against its schema. The configuration repo has no secrets. Vercel builds initialize the pinned configuration submodule and package a validated snapshot; production uses that snapshot and an allowlisted public endpoint, never live Git fetches during queue operations.
- Prefer a small change with a clear migration and rollback path. Add meaningful tests for queue concurrency, state transitions, authorization boundaries and contracts. Avoid tests that merely repeat implementation details.

## Repository and release workflow

- Main repo pins `frontend/`, `backend/` and `configuration/` as submodules. Commit and push child-repo changes first; update and commit their pointers in main afterward.
- CI must checkout submodules recursively with access to private repos. Run each child repo's checks and a main-repo integration smoke test before deployment.
- Protect default branches, require reviewed PRs and passing checks, and deploy immutable commits/image digests. Do not commit directly to protected branches once protection is enabled.
- Use separate development, staging and production environments. Backward-compatible migrations deploy before code relying on them; confirm rollback and restore procedures.

## Change completion

Report what changed, what was verified, and any incomplete deployment step. Do not state that a GitHub repo was created or pushed until a remote URL and successful push are verified.
