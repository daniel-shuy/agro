# PRD: Fix two harness doc links that break the site copy

Status: DRAFT

Issue: mifunedev/agro#1215. Epic: mifunedev/agro#1206.

## User Stories

### US-001: Fix the stale anchor and the bare autolink

**Description:** As a docs maintainer, I want harness pages that copy to the site unchanged, so that the next docs sync needs no link fix.

**Acceptance Criteria:**

- [ ] In `docs/intro.md`, the link to the security page uses the anchor `#4-sandbox-isolation--the-docker-socket-caveat--enforced-with-a-caveat`.
- [ ] In `docs/runtimes/docker.md`, line 29 writes the link as `[https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)`.
- [ ] `grep -rn '<https\?://' docs --exclude-dir=rfcs` prints nothing.
- [ ] `git diff --stat origin/development` shows 2 changed lines in 2 files, apart from the task files.
- [ ] `pnpm build:harness`, `pnpm test`, and `pnpm typecheck` exit 0.

## Summary

The docs sync in mifunedev/agro-web#66 found two harness links that fail on the site:

- `docs/intro.md` line 25 links to `security-considerations.md#3-sandbox-isolation--the-docker-socket-caveat--enforced-with-a-caveat`. The heading is now `## 4. Sandbox isolation & the Docker-socket caveat`, so the anchor does not resolve.
- `docs/runtimes/docker.md` line 29 uses the bare autolink `<https://docs.docker.com/engine/install/>`. The site MDX build rejects this syntax.

The site copy already carries both fixes. Selected approach: make the same two edits in the harness, so that the harness and the site agree.

## Key Integration Points

| File | Function(s) / Symbol(s) | Role |
|---|---|---|
| `docs/intro.md` | line 25, security link | Stale anchor |
| `docs/runtimes/docker.md` | line 29, Docker install link | Bare autolink |

## Interface Integration Points

| Surface | Change Type | Description |
|---|---|---|
| `docs/intro.md` | Link fix | The anchor points to section 4 |
| `docs/runtimes/docker.md` | Link syntax | The autolink becomes a Markdown link |

## Storage

N/A. The change edits Markdown only.

## Architectural Decisions

- The harness docs are the source of truth. After this change, the site copy of both pages matches the harness text.
- Branch `bug/1215-site-copy-link-fixes` starts from `development`, and the PR targets `development`.

## Test Plan (TDD)

| Test File | Case(s) | Validates |
|---|---|---|
| `grep` on `docs/` | no bare autolink outside `rfcs/` | US-001 |
| `grep` on `docs/security-considerations.md` | the section 4 heading exists | US-001: the anchor resolves |
| `pnpm test` | the existing docs tests pass | US-001: no regression |

## Design Principles

- Change the two links only.

## Out of Scope

- The `rfcs/` pages. The site does not copy them.
- A `CHANGELOG.md` entry. The change fixes two links and changes no behavior.

## Open Questions

None.

## Acceptance Criteria

- [ ] US-001 has `passes: true` in `prd.json`.
- [ ] CI is green on the PR.
- [ ] The PR body closes #1215.

## Lessons

Filled by the advisor before undraft.
