# PRD: Move the root-level removal section to the end of the tools text

Status: DRAFT

Issue: mifunedev/agro#1205. Epic: mifunedev/agro#1206.

## User Stories

### US-001: Move "Remove a root-level tool" after the tailscale paragraph

**Description:** As an operator, I want the removal steps last in the tools text, so that no unrelated paragraph renders under the removal heading.

**Acceptance Criteria:**

- [ ] In `docs/installation.md`, the `agent-browser` host install paragraphs and the `tailscale` paragraph come before the `#### Remove a root-level tool` heading.
- [ ] The `#### Remove a root-level tool` section ends immediately before the `### Runtimes & package managers` heading.
- [ ] `diff <(git show origin/development:docs/installation.md | sort) <(sort docs/installation.md)` prints nothing. The move adds no line and removes no line.
- [ ] `docs/installation.md` has exactly one `#### Remove a root-level tool` heading, and the link `[Remove a root-level tool](#remove-a-root-level-tool)` still points to it.
- [ ] `pnpm build:harness`, `pnpm test`, and `pnpm typecheck` exit 0.

## Summary

The tools text in `docs/installation.md` on `development` has this order:

1. The tools paragraph and the host install limits.
2. The host tool table, the `sudo -n` rule, and the desktop steps.
3. `#### Remove a root-level tool`, with the `docker-engine` and `desktop` removal steps.
4. The `agent-browser` host install paragraphs.
5. The `tailscale` paragraph.
6. `### Runtimes & package managers`.

Items 4 and 5 render under the removal heading, but items 4 and 5 do not describe removal. mifunedev/agro#1203 added item 3 at that position.

The site copy in mifunedev/agro-web already uses the correct order: the removal section comes after the `tailscale` paragraph.

Selected approach: move item 3 as one block to the position after item 5. Change no word.

## Key Integration Points

| File | Function(s) / Symbol(s) | Role |
|---|---|---|
| `docs/installation.md` | `#### Remove a root-level tool` section | The block to move |
| `docs/installation.md` | `[Remove a root-level tool](#remove-a-root-level-tool)` | The in-page link to the removal section |

## Interface Integration Points

| Surface | Change Type | Description |
|---|---|---|
| `docs/installation.md` | Section order | The removal section moves to the end of the tools text |

## Storage

N/A. The change edits Markdown only.

## Architectural Decisions

- The harness docs are the source of truth. After this change, the harness order matches the order in mifunedev/agro-web.
- Branch `task/1205-root-tool-removal-order` starts from `development`, and the PR targets `development`.

## Test Plan (TDD)

| Test File | Case(s) | Validates |
|---|---|---|
| `sort` and `diff` check | the sorted lines before and after the move are equal | US-001: no word changes |
| `grep -n` on the headings | the removal heading comes after the `tailscale` paragraph and before `### Runtimes & package managers` | US-001: new order |
| `pnpm test` | the existing docs tests, such as `install-prereqs.test.ts`, still pass | US-001: no regression |

## Design Principles

- Move the block. Do not rewrite it.
- Keep the change to the one section that issue #1205 names.

## Out of Scope

- A `CHANGELOG.md` entry. The move changes no behavior and no text.
- The site sync in mifunedev/agro-web#65.
- The flaky test in mifunedev/agro#1204.

## Open Questions

None.

## Acceptance Criteria

- [ ] US-001 has `passes: true` in `prd.json`.
- [ ] CI is green on the PR.
- [ ] The PR body closes #1205.

## Lessons

Filled by the advisor before undraft.
