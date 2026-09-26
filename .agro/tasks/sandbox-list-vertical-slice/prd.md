# PRD: Sandbox List Vertical Slice

Status: DRAFT

## User Stories

### US-001: Lock the sandbox list contract

**Description:** As a CLI maintainer, I want tests for the `sandbox list` contract so that I can move its code without changing its behavior.

**Acceptance Criteria:**

- [ ] Tests cover `sandbox list` with no entries, text rows, JSON rows, and a failed status probe.
- [ ] Tests cover the current `sandbox list` argument and help behavior, including `--json`, unexpected positionals, and unknown flags.
- [ ] Tests assert the exact JSON field names and order, text output, exit codes, and relevant stderr output.
- [ ] The new tests pass against the original implementation before any production code moves.

### US-002: Isolate sandbox list without changing its contract

**Description:** As a CLI maintainer, I want a bounded `sandbox list` command slice so that later commands can follow a tested pattern.

**Acceptance Criteria:**

- [ ] `cli.ts` retains top-level command selection but delegates `sandbox list` behavior to a focused module under `.agro/cli/src/commands/`.
- [ ] The slice owns list-specific dispatch and output. `runSandboxInstall` remains in its current command module.
- [ ] The slice reuses `listEntries`, `entryRoot`, configuration readers, and `ExecutionTarget`. It does not add a second registry or execution implementation.
- [ ] The old `sandbox list` behavior does not remain as a second implementation in `sandbox.ts` or `cli.ts`.
- [ ] Tests from US-001 pass without changed assertions after the move.
- [ ] `pnpm run typecheck`, `pnpm exec vitest run .agro/cli/src/__tests__/sandbox.test.ts`, and `pnpm run build:harness` exit with code 0.

## Summary

The current `.agro/cli/src/cli.ts` parses `sandbox` arguments and dispatches `sandbox list`. The current `.agro/cli/src/commands/sandbox.ts` loads registry entries, reads their config, checks execution status, and renders text or JSON. Existing tests in `.agro/cli/src/__tests__/sandbox.test.ts` cover text and JSON rows. Extend those tests before moving production code. Keep the CLI contract unchanged. The task establishes one command-oriented slice without a global layer layout.

## Key Integration Points

| File | Function(s) / Symbol(s) | Role |
|---|---|---|
| `.agro/cli/src/cli.ts` | `parseSandboxArgs`, `main` | Preserve parsing and top-level selection; remove list-specific dispatch from the entry point. |
| `.agro/cli/src/commands/sandbox.ts` | `runSandboxList`, `runSandboxInstall` | Move list behavior; keep install behavior in place. |
| `.agro/cli/src/commands/sandbox-list.ts` | `runSandboxList` | Own list execution and rendering after extraction. |
| `.agro/cli/src/lib/registry.ts` | `listEntries`, `entryRoot`, `registryRoot` | Keep registry reads in the existing source of truth. |
| `.agro/cli/src/lib/execution/target.ts` | `ExecutionTarget.status` | Keep status probes behind the existing execution boundary. |
| `.agro/cli/src/__tests__/sandbox.test.ts` | `agro sandbox list` | Lock behavior before extraction. |

## Interface Integration Points

| Surface | Change Type | Description |
|---|---|---|
| `agro sandbox list [--json]` | Internal only | Preserve arguments, help, output, and exit codes. |
| `.agro/cli/src/commands/sandbox-list.ts` | New internal module | Establish a focused command boundary. |

## Storage

The sandbox registry remains under `${AGRO_HOME:-~/.agro}/sandboxes/`. This task changes no schema or persisted data. `registry.ts` remains the registry access point.

## Architectural Decisions

- Keep the command workflow as the unit of organization. Do not add HTTP-style routes, controllers, or models to the CLI.
- Keep one parser contract. If extraction moves `parseSandboxArgs`, preserve its install behavior and exported API; do not create a second parser for list.
- Keep `cli.ts` responsible for selecting the top-level command. Keep list formatting and status lookup in the focused command module.
- Preserve current behavior even where the parser accepts flags that `sandbox list` does not use. A later change can revise that contract separately.
- Run code and tests inside the sandbox. This task does not start a service or change host lifecycle behavior.

## Test Plan (TDD)

| Test File | Case(s) | Validates |
|---|---|---|
| `.agro/cli/src/__tests__/sandbox.test.ts` | Empty registry, aligned text rows, JSON row keys and order, status probe failure | List rendering and status behavior. |
| `.agro/cli/src/__tests__/cli.property.test.ts` | List argument parsing for `--json`, extra arguments, and unknown flags | Existing parser contract. |
| `.agro/cli/src/__tests__/cli-first-help.test.ts` | `sandbox` help and list dispatch output | Public command behavior after extraction. |

Add tests that detect output or dispatch changes; run the tests against the original code first. Then move the list code and rerun the same tests. Run `pnpm run typecheck` and `pnpm run build:harness` from the repository root.

## Design Principles

- Change one command path, not the full CLI.
- Reuse the registry and execution boundaries.
- Delete moved code instead of keeping duplicate implementations.
- Add no explanatory comments to tracked code.
- Keep host and sandbox boundaries unchanged. The command works after a terminal disconnect and does not share mutable state between agents.

## Out of Scope

- Refactoring `sandbox install` behavior or other command families.
- Adding application scaffolds, routes, controllers, models, services, or repositories.
- Changing CLI flags, help text, output, registry layout, or public documentation.
- Adding dependencies or changing the packaged CLI build layout.

## Open Questions

None.

## Acceptance Criteria

- [ ] US-001 tests pass against the original implementation before the code move.
- [ ] Both stories meet every listed criterion.
- [ ] `pnpm run typecheck` exits with code 0 from the repository root.
- [ ] `pnpm exec vitest run .agro/cli/src/__tests__/sandbox.test.ts .agro/cli/src/__tests__/cli.property.test.ts .agro/cli/src/__tests__/cli-first-help.test.ts` exits with code 0 from the repository root.
- [ ] `pnpm run build:harness` exits with code 0 from the repository root.
- [ ] `git diff` shows no change to public CLI output contracts or persisted data.

## Lessons

None.
