# Fix cRPC client error guards across entrypoints

Objective:
Make `kitcn/crpc` client-error guards recognise `CRPCClientError`s thrown
through `kitcn/react` and `kitcn/solid` without accepting lookalikes, as one
not-yet-created PR slice.

Goal plan:
docs/plans/2026-09-10-fix-crpc-client-error-guards.md

Template:
docs/plans/templates/task.md

Primary template:
docs/plans/templates/task.md

Applied packs:
- package-api (docs/plans/templates/packs/package-api.md)

Task source:
- type: plain task text; non-ticket bug fix, so no GitHub issue is required.
- id / link: N/A; Tim's task prompt plus the Track B review corrections.
- title: fix(crpc): recognize client errors across entrypoints
- acceptance criteria: `isCRPCClientError`, `isCRPCError` and
  `isCRPCErrorCode` from `kitcn/crpc` recognise genuine client errors produced
  through `kitcn/react` and `kitcn/solid`; the guard requires
  `instanceof Error`, `name === 'CRPCClientError'`, a supported `code`, and
  string `functionName` and `message`; correctness does not depend on
  `instanceof CRPCClientError`; lookalikes, including complete plain objects,
  are rejected; HTTP 4xx stays deterministic, HTTP 5xx and transport errors
  stay retryable; public import paths and types are unchanged.
- caveats: Tim explicitly owns every Git and GitHub operation (stage, commit,
  push, PR). `HttpClientError`, `AuthMutationError` and the wider tsdown
  duplication are out of scope.
- likely files: `packages/kitcn/src/crpc/error.ts` and its tests.
- browser surface: none.
- root-cause layer: the public guard in `src/crpc/error.ts`; the duplicate
  classes come from tsdown's separate client, Solid and server build groups.

Task PR:
- Not yet created. This plan owns exactly one future PR from branch
  `fix/crpc-client-error-guards` (based on `upstream/main` 5b0bd5a1). Record the
  PR number and URL here once Tim opens it.

Timed checkpoint:
- requested duration: N/A; no duration requested.
- semantics: N/A.
- initial confidence score: N/A; regression tests are the threshold.
- improvement loop: N/A.
- final score / loop closure: N/A.

Completion threshold:
- The built-entrypoint regression fails on the `upstream/main` build and passes
  after the fix for both `kitcn/react` and `kitcn/solid`.
- Source guard tests cover recognition, code matching, lookalike rejection and
  HTTP 4xx/5xx behaviour.
- Package build, kitcn and root typecheck, lint and the full `bun check` pass.
- An existing unreleased changeset records the patch.
- The local read-only autoreview returns no accepted/actionable findings.
- Commit and PR gates are recorded as user-owned.
- Task closure is legal only when the source-of-truth acceptance criteria are
  satisfied or explicitly narrowed, required verification evidence is recorded,
  code-review and release-artifact gates are closed when applicable, verified
  code changes are committed and PR'd unless explicitly declined or blocked,
  task-style PR body sync is complete or marked N/A with reason,
  GitHub issue/PR sync is complete or marked N/A with reason, and
  `node .agents/skills/autogoal/scripts/check-complete.mjs docs/plans/2026-09-10-fix-crpc-client-error-guards.md` passes.

Verification surface:
- `bunx bun@1.3.9 test packages/kitcn/src/crpc` (source and built-entrypoint
  tests)
- `bun --cwd packages/kitcn build`
- `bun --cwd packages/kitcn typecheck` and `bun typecheck`
- `bun lint`
- `bunx bun@1.3.9 run check`
- Execution of the emitted `dist/crpc/index.js` guards against genuine errors
  and lookalikes
- `.claude/skills/autoreview/scripts/autoreview --mode local`
- `node .agents/skills/autogoal/scripts/check-complete.mjs docs/plans/2026-09-10-fix-crpc-client-error-guards.md`

Constraints:
- Fix the kitcn library guard; no application-specific workaround.
- Do not restore `instanceof CRPCClientError` as a requirement or fast path.
- No bundler reconfiguration; public exports and types stay unchanged.
- Tests must not encode today's duplicated bundle layout.
- Reuse an existing unreleased changeset; never touch the hydration PR's
  changeset or plan.
- Tim explicitly declined assistant-run Git and GitHub operations: no staging,
  unstaging, commit, push, merge, rebase, or PR creation/update.

Boundaries:
- Source of truth: Tim's task prompt, the Track B review corrections, and
  `packages/kitcn/src/crpc/error.ts`.
- Allowed edit scope: `packages/kitcn/src/crpc/error.ts`, its source tests, the
  built-entrypoint regression test, one existing changeset, and this plan.
- Browser surface: N/A; no rendered UI changes.
- GitHub issue sync: N/A; non-ticket task with no issue.
- Non-goals: `isHttpClientError`, `isAuthMutationError`, tsdown chunk
  deduplication, docs changes, and the hydration PR.

Output budget strategy:
- Exact-file reads, capped `rg` and `tail` output, and full suite logs saved
  under the session scratchpad instead of streamed into context.

Blocked condition:
- Stop if a fix requires a bundler or public API change, if the built-entrypoint
  harness cannot reproduce the defect, or if `bun check` fails for reasons
  outside this diff that cannot be ruled out as local environment rot.
- Met: `bun check` fails at `fixtures:check` on external `lucide-react` drift
  that also affects `upstream/main`. It is not local environment rot, and
  fixture sync is outside this branch.

Task state:
- task_type: bug fix with review-correction pass
- task_complexity: non-trivial; one owner module
- current_phase: verification
- current_phase_status: blocked; implementation complete, required `bun check`
  red
- next_phase: rerun `bun check` once the `lucide-react` fixture drift is
  resolved on `upstream/main`, then Tim-owned commit, push and PR
- goal_status: blocked; not complete

Current verdict:
- verdict: implementation complete and accepted; verification blocked
- confidence: high at the guard and built-entrypoint boundary; the task is not
  complete because the required `bun check` threshold is red
- next owner: Tim
- reason: `bun check` fails at `fixtures:check` on `lucide-react` fixture drift
  (`^1.42.0` committed, `^1.44.0` generated) that reproduces independently on
  unchanged `upstream/main`. Fixtures must not be synced or edited in this
  branch, so the threshold cannot be met here.

Implementation readiness:
- verdict: ready and implemented
- exact owner: `packages/kitcn/src/crpc/error.ts`
- contradiction status: none; review corrections were accepted as stated
- source-listed cases complete: yes

Pre-solution issue challenge:
- reporter claim: a `CRPCClientError` produced by `kitcn/react` fails guards
  imported from `kitcn/crpc` because the published package holds separate class
  copies, so `UNAUTHORIZED` refusals are retried.
- suggested diagnosis or fix: replace constructor identity with a structural
  check.
- repro ladder:
  - tests / source-level repro: a complete error from another class copy fails
    `isCRPCClientError` on `upstream/main` source.
  - repo-owned automated browser or integration proof: the built-entrypoint bun
    test drives `ConvexQueryClient.queryFn()` from `kitcn/react` and
    `kitcn/solid` dist and fails on the `upstream/main` build.
  - Browser plugin: N/A; no rendered UI.
  - screenshot / visual proof: N/A; guard results are not visual.
- reproduction verdict: reproduced for React and Solid.
- validity verdict: valid.
- best long-term fix boundary: the public guard; bundler deduplication would not
  fix cross-realm or future chunk splits.
- harsh honest feedback: the old tests built the error from the same module as
  the guard, so they could never see the defect.
- hard-stop decision: proceed; valid and reproduced.

Completion rule:
- Do not call `update_goal(status: complete)` while any required checklist item
  remains unchecked. If an item does not apply, check it and add `N/A: <reason>`.
- Do not call `update_goal(status: complete)` until every completion threshold
  above is satisfied, final handoff evidence is recorded, and
  `node .agents/skills/autogoal/scripts/check-complete.mjs docs/plans/2026-09-10-fix-crpc-client-error-guards.md` passes.
- Do not create hook state for this goal. This file plus the active goal are the
  durable state.

Start Gates:
| Gate | Applies | Evidence |
|------|---------|----------|
| Timed checkpoint parsed | N/A | No duration requested |
| Walkthrough baseline for possible UI change | N/A | UI or rendered output cannot change; guard logic only |
| Skill analysis before edits | yes | `task`, `changeset` rule, `autoreview` and the package-api pack read |
| Active goal checked or created | yes | This plan is the durable state |
| Source of truth read before edits | yes | Task prompt, Track B corrections, `error.ts`, tsdown config, entrypoints |
| Exact per-PR task ownership | yes | Owns one not-yet-created PR slice; PR number recorded once Tim opens it |
| GitHub comments and attachments read | N/A | No GitHub source |
| Video transcript evidence required | N/A | No video evidence |
| Pre-solution issue challenge required | yes | Recorded above; verdict valid |
| Reproduction verdict before implementation | yes | Built-entrypoint and source tests failed before the fix |
| Repro escalation ladder selected | yes | Source test, then the built-entrypoint bun test |
| Suggested fix reviewed against durable boundary | yes | Guard fix chosen over bundler change |
| `docs/solutions` checked for non-trivial existing-code work | yes | No entry covers cross-entrypoint class identity |
| TDD decision before behavior change or bug fix | yes | Red tests written and run before the guard change |
| Branch decision for code-changing task | yes | `fix/crpc-client-error-guards` from `upstream/main` 5b0bd5a1; no hydration commits |
| Release artifact decision | yes | Update existing unreleased `.changeset/lucky-plums-cough.md` |
| Browser tool decision for browser surface | N/A | No browser surface |
| Commit / PR expectation decision | N/A | Tim explicitly owns commit, push and PR |
| Task-style PR body decision | N/A | No PR yet; Tim creates it |
| Task-plan PR body evidence | N/A | Not-yet-created PR slice; the PR body must add `🧭 Task plan: docs/plans/2026-09-10-fix-crpc-client-error-guards.md` |
| GitHub issue sync expectation decision | N/A | Non-ticket task |
| Output budget strategy recorded | yes | See Output budget strategy |
| Package/API pack selected | yes | Public guard behaviour across package entrypoints |
| Public surface or package boundary identified | yes | `kitcn/crpc` guards versus `kitcn/react`, `kitcn/solid`, `kitcn/auth/client` producers |
| Convex entry/import graph impact identified | yes | No new imports; `error.ts` graph unchanged |
| CLI/scaffold/generated impact identified | N/A | No CLI, scaffold or generated output changes |
| Release artifact path selected | yes | `.changeset` |
| `changeset` skill loaded when `.changeset` is required | yes | `.agents/rules/changeset.mdc` followed |
| Package build / fixture impact decision recorded | yes | Package build required; fixtures unaffected |

Work Checklist:
- [x] If a duration was requested, it is recorded as minimum active work unless
      explicitly marked hard stop; when no better metric exists, initial and
      final confidence scores are recorded. N/A: no duration requested.
- [x] Objective includes outcome, completion threshold, verification surface,
      constraints, boundaries, and blocked condition.
- [x] Task source classified with source type, id/link, title, task type,
      acceptance criteria, caveats, likely files/routes/packages, browser
      surface, and root-cause layer.
- [x] Every GitHub PR in scope has its own task plan. This plan owns one exact
      PR, owns a not-yet-created PR slice, or records N/A because no PR is in
      scope; a batch plan is not used as a substitute. Owns one
      not-yet-created PR slice.
- [x] Required video or screen-recording evidence is cached/read as normalized
      `<video-transcripts>` XML, or marked N/A with reason. N/A: no video.
- [x] For public GitHub bug reports, behavior claims, technical diagnoses, or
      suggested fixes, reporter claims are challenged before implementation
      with a recorded verdict. Verdict: valid.
- [x] Repro escalation ladder followed for bug/behavior claims.
- [x] Hard-stop rule followed for bug/behavior claims. Reproduced; proceeded.
- [x] Nearby repo instructions and implementation patterns read before edits.
- [x] Source-listed case matrix is complete and every contradiction has an
      owner, harness, and verdict before mutation.
- [x] Readiness is classified `ready`, `repair-source`, `major`, `blocked`, or
      `invalid` with evidence. Ready.
- [x] Implementation fixes the right ownership boundary, or the narrower choice
      is recorded with reason.
- [x] Release artifact requirement recorded: active changeset, new changeset, or
      N/A with reason. Existing `.changeset/lucky-plums-cough.md` updated.
- [x] Final handoff shape decided: bug/feature/testing/batch/review/GitHub
      requirements, PR body sync, and issue sync when applicable.
- [x] Commit/PR handling recorded for code-changing work: commit and PR
      completed, no local patch, user explicitly declined, or blocker recorded.
      User explicitly declined: Tim owns all Git and GitHub operations.
- [x] PR body shape recorded: PR #270 emoji task-style body used, N/A reason
      recorded, or blocker recorded. N/A: no PR yet; Tim creates it.
- [x] PR task evidence recorded: body includes `🧭 Task plan: ...`, the plan
      exists at the PR head, and it identifies the exact PR before autoclosure.
      N/A until the PR exists; this plan must then record its number.
- [x] Branch handling recorded for code-changing work: dedicated branch used,
      new branch needed, or N/A with reason.
- [x] Local-env-rot retry policy recorded for any surprising repo-wide failure:
      reinstall/rerun evidence or N/A with reason.
- [x] Workspace authority recorded: every proof command names the cwd/tool that
      owns the changed behavior. All commands ran from the repo root or
      `packages/kitcn` in this checkout.
- [x] Output budget discipline recorded and followed.
- [x] High-risk note recorded for public API, runtime, package-boundary,
      browser behavior, agent-action, or command-contract changes, or marked
      N/A with reason.
- [x] Review/autoreview target selected from actual diff state for non-trivial
      implementation work, or marked N/A with reason. Dirty local diff:
      `--mode local`.
- [x] Agent-native review decision recorded for `.agents/**`, `.claude/**`,
      `.codex/**`, skills, hooks, commands, prompts, or user-action tooling.
      N/A: none touched.
- [x] Package/API pack: public API, package boundary, export, and release-artifact impact are recorded.
- [x] Package/API pack: release artifact matrix is applied: `.changeset` or explicit no-artifact reason.
- [x] Package/API pack: `.changeset` work loads `changeset` and follows its package/version/prose rules.
- [x] Package/API pack: no-artifact decisions state why the diff has no published package user-visible delta from `main`. N/A: a changeset applies.
- [x] Package/API pack: compatibility, migration, or hard-cut decision is explicit when public shape changes. N/A: public shape unchanged.
- [x] Package/API pack: affected Convex static import graphs stay narrow and
      plugin/per-module boundaries are used where appropriate.
- [x] Package/API pack: CLI commands remain deterministic, `--json` capable,
      and non-interactive with explicit confirmation bypass when relevant. N/A:
      no CLI change.
- [x] Package/API pack: docs and `packages/kitcn/skills/kitcn/**` stay
      current-state synchronized when public guidance changes. N/A: documented
      guard usage is unchanged.
- [x] Package/API pack: package-owned typecheck/build/test proof is recorded or marked N/A with reason.
- [x] Package/API pack: `packages/kitcn` build, fixture sync/check, or other owning package proof is recorded when required.

Completion Gates:
| Gate | Applies | Required action | Evidence |
|------|---------|-----------------|----------|
| Named verification threshold | yes | Run the command, proof, source audit, or artifact check named in this plan | Blocked: every named check passes except the required `bun check` (red); see Verification evidence |
| Exact per-PR task ownership | yes | Record the exact PR and dedicated plan, or the not-yet-created single-PR slice | Not-yet-created single-PR slice |
| Pre-solution issue challenge verdict | yes | Record reporter claim, suggested fix, repro verdict, validity verdict, durable boundary, and hard-stop/pivot decision before implementation | Recorded above |
| Repro escalation ladder | yes | For bug/behavior claims, record test/source-level, automated browser/integration, Browser, and screenshot/visual-proof outcomes or N/A/blocker reasons before `not reproduced` | Source and built-entrypoint repro; Browser and visual N/A |
| Bug reproduced before fix | yes | Record failing test/repro or N/A with reason | 3 failures before the fix: React and Solid built-entrypoint cases, source other-copy case |
| Targeted behavior verification | yes | Run focused test/proof for changed behavior or record N/A | `bunx bun@1.3.9 test packages/kitcn/src/crpc`: 36 pass |
| TypeScript or typed config changed | yes | Run relevant typecheck | kitcn typecheck exit 0; root `bun typecheck` 5/5 |
| Package exports or file layout changed | N/A | Run the relevant package build before final verification and keep generated updates | No export or layout change; package build still run and passes |
| Package manifests, lockfile, or install graph changed | N/A | Run `bun install` and relevant package checks | No manifest or lockfile change |
| Agent rules or skills changed | N/A | Run `bun install` and verify generated skill sync | No rules or skills changed |
| Workspace authority proof | yes | Run verification in the owning repo/package/app/route/tool and record cwd; do not count the wrong workspace as proof | Repo root and `packages/kitcn` of this checkout |
| Browser surface changed | N/A | Capture Browser Use proof or record explicit waiver/blocker | No browser surface |
| Browser final proof | N/A | Attach screenshot or exact browser verification caveat when browser proof applies | No browser surface |
| UI walkthrough | N/A | If UI or rendered output changed, run `.agents/skills/walkthrough/SKILL.md` after final proof and show annotated images in the final handoff; otherwise record N/A | No UI or rendered output |
| Scaffold or fixture output changed | N/A | Run `bun run fixtures:sync` and `bun run fixtures:check`, or record N/A | No scaffold change; `fixtures:check` still runs inside `bun check` |
| Package behavior or public API changed | yes | Add a changeset or record why no changeset applies | `.changeset/lucky-plums-cough.md` patch bullet |
| Docs and kitcn skill sync changed | N/A | Keep `www/**` and `packages/kitcn/skills/kitcn/**` in sync, or record N/A | Guidance unchanged |
| Docs or content changed | N/A | For docs-heavy work, use `--template docs`; for incidental docs, verify source-backed claims, links, examples, and rendered output or record N/A | Only this plan and the changeset |
| High-risk mini gate | yes | For public API/runtime/package-boundary/browser/agent-action/command-contract changes, record realistic failure mode, proof plan, and why the chosen boundary is right; otherwise N/A | See Open risks and Decisions |
| Agent-native review for agent/tooling changes | N/A | For `.agents/**`, `.claude/**`, `.codex/**`, skills, hooks, commands, prompts, or user-action tooling, load `.agents/skills/agent-native-reviewer/SKILL.md` and close accepted/actionable findings, or record N/A | None touched |
| Local install corruption suspected | yes | Run `bun install` once, rerun the exact failing command, or record N/A | See Error attempts |
| Commit created | N/A | For verified code-changing work, stage the entire current checkout per repo policy and create a commit; N/A only for no local patch, explicit user decline, analytical/blocked/inconclusive work, or recorded external blocker | Explicit user decline: Tim owns Git |
| PR create or update | N/A | For verified code-changing work, run `check`, push, create or update the PR, and sync PR body to the task-style final handoff; N/A only for no local patch, explicit user decline, analytical/blocked/inconclusive work, or recorded external blocker | Explicit user decline; `bun check` run locally and red on external fixture drift |
| Task-style PR body verified | N/A | Verify the PR body with `gh pr view --json body`; it must preserve auto-release blocks when applicable, must not include a current-PR self-link, and must use the PR #270 emoji format | No PR yet |
| PR task evidence verified | N/A | Verify body plan line, plan at PR head, and exact PR ownership | No PR yet |
| PR proof image hosting | N/A | If PR body needs browser proof, replace local image paths with hosted GitHub URLs or record N/A | No images |
| GitHub issue sync-back | N/A | Post concise issue sync after PR exists, or record N/A/blocker | No issue |
| Final handoff contract | yes | Fill the final handoff fields below with exact PR/issue/confidence/tests/browser/outcome/caveats/design/verification content or N/A reason | Filled below |
| Final lint | yes | Run `bun lint:fix` or scoped equivalent | `bun lint` clean |
| Output budget discipline | yes | Verify no unbounded high-volume command output was streamed, or record the accidental output and recovery | Suite logs saved to scratchpad |
| Timed checkpoint | N/A | If duration was requested, keep improving until elapsed, then finish the current loop cleanly; otherwise N/A | No duration |
| Autoreview for non-trivial implementation changes | yes | Load `.agents/skills/autoreview/SKILL.md`; use dirty local `--mode local`, branch/PR `--mode branch --base <base>`, or committed slice `--mode commit --commit <ref>` until no accepted/actionable findings, or record N/A for docs-only/trivial/no local patch | `.claude/skills/autoreview/scripts/autoreview --engine claude --mode local` (`claude-fable-5`; Codex CLI absent): TruffleHog clean, no accepted/actionable findings |
| Goal plan complete | yes | Run `node .agents/skills/autogoal/scripts/check-complete.mjs docs/plans/2026-09-10-fix-crpc-client-error-guards.md` | Not complete: blocked on the required `bun check` threshold. A mechanical checker pass does not override that. The PR number is added once Tim opens the PR |
| Public API / package boundary proof | yes | Source-audit public API, exports, and package boundary impact | Emitted `.d.ts` signatures unchanged; guards executed from `dist/crpc` |
| Convex bundle/import proof | yes | Audit affected function-entry static graphs or record N/A | `error.ts` adds no imports |
| CLI/scaffold/generated proof | N/A | Prove command contract and regenerate owned output or record N/A | No CLI or scaffold change |
| Release artifact classification | yes | Record whether the change is published package behavior/API/types/config/runtime or no published user-visible delta | Published runtime behaviour: patch |
| Published package changeset | yes | If published package users see a delta, load `changeset` and add/update one `.changeset/*.md` per package | `.changeset/lucky-plums-cough.md` updated |
| No release artifact | N/A | If no artifact is needed, record the exact reason: internal-only, docs-only, agent-only, test-only, or no user-visible delta from `main` | A changeset applies |
| Package typecheck/build/test | yes | Run owning package checks or record N/A with reason | Build 4/4 groups; typecheck exit 0; cRPC tests 36 pass |
| Fixture/scaffold generation | N/A | Run `bun run fixtures:sync` and `bun run fixtures:check` when scaffold output changed, otherwise N/A | No scaffold change |
| Docs/package skill sync | N/A | Synchronize current-state public guidance or record N/A | Guidance unchanged |
| Full repository gate | yes | Run `bun check` | Blocked (red): exit 1 only on external `lucide-react` fixture drift, which reproduces on unchanged `upstream/main`; every other step passes (see Verification evidence). Rerun after the fixtures are resynced upstream |

Phase / pass table:
| Phase | Status | Evidence | Next |
|-------|--------|----------|------|
| Intake and source read | complete | Root cause and three emitted class copies confirmed | implementation |
| Implementation | complete | Structural guard with `instanceof Error`; Track B corrections applied | verification |
| Verification | blocked | Focused tests, build, typechecks, lint and autoreview pass; required `bun check` red on external `lucide-react` fixture drift | rerun `bun check` after the upstream fixture sync |
| Commit / PR / GitHub sync | N/A (Tim-owned) | Explicit user decline | Tim |
| Closeout | blocked | Waits on the `bun check` threshold | closeout after `bun check` is green |

Findings:
- tsdown builds client (`react`, `auth/client`, `ratelimit/react`), Solid and
  server entrypoints as separate groups, so the built package held three
  `CRPCClientError` classes: `error-*.js` (`crpc`, `rsc`, `server`,
  `auth/nextjs`, `auth/start/server`), `auth-store-*.js` (`react`,
  `auth/client`) and a copy inlined in `solid/index.js`.
- `isCRPCClientError` and `isCRPCError` used `instanceof CRPCClientError`;
  `isCRPCErrorCode` delegates to `isCRPCClientError`.
- `isHttpClientError` and `isAuthMutationError` share the same defect class and
  are left out of scope.

Decisions and tradeoffs:
- The guard requires `instanceof Error` plus the complete stable shape. The
  global `Error` is shared across entrypoint chunks, so the check survives class
  duplication while rejecting plain objects.
- `instanceof CRPCClientError` is not kept as a fast path; one rule is simpler
  and nothing depends on constructor identity.
- The supported codes are a `const` tuple, and `ClientErrorCode` is derived from
  it so the runtime list and the type cannot drift.
- No bundler change: deduplicating chunks would still leave the guard fragile
  against future splits.
- Changeset: `.agents/rules/changeset.mdc` requires reusing an existing
  unreleased changeset, and no repository tooling names a specific one.
  `.changeset/lucky-plums-cough.md` is the most recently added `patch`
  changeset; the newest overall (`wide-index-union-stays-indexed.md`) is
  `minor` and already edited by the hydration PR.

Implementation notes:
- `packages/kitcn/src/crpc/error.ts`: structural `isCRPCClientError` gated on
  `instanceof Error`; `isCRPCError` reuses it; the HTTP branch is unchanged.
- `packages/kitcn/src/crpc/error.test.ts`: other-copy recognition and a
  lookalike table.
- `packages/kitcn/src/crpc/package-entrypoints.integration.test.ts`: real
  refusals from the `kitcn/react` and `kitcn/solid` dist, checked with the
  `kitcn/crpc` guards.

Review fixes:
- Track B: required `instanceof Error`; added a complete plain object with a
  valid code to the negative table; removed the integration assertions that
  errors are not instances of the `kitcn/crpc` constructor; replaced the new
  changeset with an update to an existing unreleased changeset; created this
  plan.

Error attempts:
| Error / failed attempt | Count | Next different move | Resolution |
|------------------------|-------|---------------------|------------|
| Server build group failed locally (`better-call` not in `inlineOnly`) under bun 1.2.20 | 2 | Reinstall with the repo-pinned `bun@1.3.9` and `--frozen-lockfile` | Build passes; CI was already green on `upstream/main` |
| bun 1.2.20 `bun install` removed `configVersion` from `bun.lock` | 1 | Restore the file | `bun.lock` unchanged in the final diff |
| `bun check` failed in `fixtures:check`: `lucide-react` `^1.42.0` committed, `^1.44.0` generated | 1 | Check each fixture individually; confirm the drift is external | `expo` and `expo-auth` pass; the six web fixtures drift on that single line only. 1.44.0 was published 2026-09-10, after the last green `upstream/main` CI; `upstream/main` still pins `^1.42.0`. No scaffold source is touched here, so `fixtures:sync` is out of scope |
| `test:runtime` scenario `expo` not ready at `127.0.0.1:3210` | 1 | Identify the port owner | Docker container `dashboard-backend-1` held 3210-3211; Tim stopped it, then the scenario was rerun |

Verification evidence:
- Before the fix (`upstream/main` source and build, bun 1.3.9): the React and
  Solid built-entrypoint tests failed with `isCRPCClientError` returning
  `false`, and the source other-copy test failed.
- After: `bunx bun@1.3.9 test packages/kitcn/src/crpc` gives 36 pass, 0 fail.
- `bun --cwd packages/kitcn build`: exit 0, 4/4 build groups.
- `bun --cwd packages/kitcn typecheck`: exit 0. `bun typecheck`: 5/5 tasks.
- `bun lint`: clean.
- The emitted `dist` has no `instanceof CRPCClientError`. Executing the
  `dist/crpc` guards accepts the same-entrypoint and other-copy errors, rejects
  a complete plain object, the incomplete, unsupported-code and missing-message
  lookalikes, a plain `Error` and a message-only `Error`, keeps HTTP 404
  deterministic, and keeps HTTP 503 and `TypeError('fetch failed')` retryable.
- `bunx bun@1.3.9 run check`: exit 1 at `fixtures:check`, caused only by the
  external `lucide-react` 1.44.0 drift recorded under Error attempts. Its
  lint, typecheck, `bun test` (1427 pass), vitest (1047 pass, 14 skipped),
  `test:cli` and Concave smoke steps passed. The steps the failure skipped
  were run separately: `test:verify` exit 0; `test:runtime` exit 0 once
  3210-3211 were freed; per-fixture checks pass for `expo` and `expo-auth`,
  and each web fixture fails only on the `lucide-react` line.
- Autoreview: `.claude/skills/autoreview/scripts/autoreview --engine claude
  --mode local` exited 0 with no accepted/actionable findings. It noted, as
  non-blocking, the tradeoffs listed under Open risks. The Codex CLI is not
  installed; TruffleHog 3.97.4 was installed with Tim's approval.

Source-listed case matrix:
| Case | Source claim | Harness | Before | Expected after | Evidence | Status |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Same-entrypoint error recognised | `error.test.ts` | true | true | 36 pass | passed |
| 2 | `kitcn/react` error passes `isCRPCClientError` | built-entrypoint test | false | true | red then green | passed |
| 3 | Same error passes `isCRPCError` | built-entrypoint test | false (emitted `instanceof` branch) | true | green | passed |
| 4 | `isCRPCErrorCode` accepts the actual code | built-entrypoint test | false | true | green | passed |
| 5 | `isCRPCErrorCode` rejects another code | both tests | false | false | green | passed |
| 6 | Incomplete lookalikes rejected | `error.test.ts` table | false | false | green | passed |
| 7 | Unsupported codes rejected | `error.test.ts` table | false | false | green | passed |
| 8 | Plain `Error` rejected | `error.test.ts` | false | false | green | passed |
| 9 | Message merely containing `UNAUTHORIZED` rejected | `error.test.ts` table | false | false | green | passed |
| 10 | HTTP 4xx deterministic | `error.test.ts` | true | true | green | passed |
| 11 | HTTP 5xx retryable | `error.test.ts` | false | false | green | passed |
| 12 | `kitcn/solid` boundary covered | built-entrypoint test | false | true | red then green | passed |
| 13 | Complete plain object rejected | `error.test.ts` table | false | false | green | passed |
| 14 | `defaultIsUnauthorized` compatible | built-entrypoint test | true | true | green | passed |

Final handoff contract:
- Commit line: N/A; Tim owns commit.
- PR line: N/A; not-yet-created PR slice.
- Issue line: N/A; non-ticket task.
- Confidence line: high for the implementation, where every case row has
  direct proof; the task is not complete while the required `bun check` is red.
- Flow table:
  - Reproduced: tests red (React, Solid, other-copy source case), browser N/A
  - Verified: focused tests green; `bun check` red on external fixture drift;
    browser N/A
- Browser check: N/A; no rendered UI.
- Outcome: `kitcn/crpc` guards recognise client errors from `kitcn/react` and
  `kitcn/solid`, so deterministic refusals are no longer retried.
- Caveat: `isHttpClientError`, `isAuthMutationError` and tsdown duplication are
  out of scope.
- Design:
  - Chosen boundary: the public guard in `src/crpc/error.ts`.
  - Why not quick patch: an app-side workaround would leave every consumer
    exposed.
  - Why not broader change: bundler deduplication is wider and still leaves the
    guard dependent on constructor identity.
- Verified: see Verification evidence; the full repository gate is blocked.
- PR body verified: N/A; no PR yet.

Task-style PR body contract:
- Preserve any existing `<!-- auto-release:start -->` block. If a changeset is
  part of the diff and repo policy expects auto release, include that block.
- Use the accepted PR #270 visual format. The body starts with an emoji
  issue/fix line, for example `🐛 Fixes #123` or `🐛 Fixes ➖ N/A`, then
  `🧭 Task plan: docs/plans/<plan>.md`, then an emoji confidence line like
  `🟢 95-100% confidence`.
- Use this exact table header: `| Phase | 🧪 Tests | 🌐 Browser |`.
- Use `Reproduced` and `Verified` rows. Mark passing proof with `🟢`, repro or
  failing proof with `🔴`, and non-applicable cells with `➖ N/A`.
- Use bold emoji section headings: `**✅ Outcome**`, `**⚠️ Caveat**`,
  `**🏗️ Design**`, and `**🧪 Verified**`.
- Never include a line that links to the current PR itself. The current PR URL
  belongs in the final response, not in its own description.
- Do not replace this with a generic `Summary` / `Verification` PR body, an
  adaptive prose body from a git helper skill, plain `## Outcome` sections, or
  an unrelated generated badge footer unless the caller or repo template
  explicitly asks for it.
- Proof is `gh pr view --json body` output or a concise source-backed summary
  of that output.

Final handoff / sync:
- Commit: N/A; Tim-owned.
- PR: N/A; Tim-owned, not yet created.
- Issue: N/A; non-ticket task.
- Browser proof: N/A; no rendered UI.
- Caveats: `bun check` is red on external `lucide-react` fixture drift that
  reproduces on `upstream/main`, so verification is blocked. Record the PR
  number in this plan once the PR exists.

Timeline:
- 2026-09-10 Branch created from `upstream/main`; defect reproduced; guard fixed.
- 2026-09-10 Task goal plan created during the Track B correction pass.
- 2026-09-10 Implementation accepted. Plan state corrected: verification is
  blocked on the red `bun check` threshold, and the task is not complete.

Reboot status:
| Question | Answer |
|----------|--------|
| Where am I? | Verification blocked; implementation complete and accepted |
| Where am I going? | Rerun `bun check` after the upstream `lucide-react` fixture sync, then Tim-owned commit, push and PR |
| What is the goal? | `kitcn/crpc` guards recognise client errors from other entrypoints and reject lookalikes |
| What have I learned? | See Findings |
| What have I done? | See Timeline |

Open risks:
- A deliberately forged `Error` with the complete shape passes; this is
  inherent to structural checks and matches the existing `HttpClientError` name
  check.
- A `CRPCClientError` constructed at runtime with an unsupported code (only by
  bypassing the types) is now rejected.
- Errors from another JavaScript realm fail `instanceof Error`; the required
  cross-entrypoint case shares one realm.

Hard closeout guard:
- A local-only final response for verified code-changing work is invalid unless
  this plan records an explicit user decline, no local patch, analytical/
  blocked/inconclusive outcome, or a real commit/PR blocker. Recorded: explicit
  user decline for Git and GitHub, and verification blocked on the red
  `bun check` threshold.
