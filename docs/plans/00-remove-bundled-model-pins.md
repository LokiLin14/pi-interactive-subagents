# Let bundled agents use Pi startup model defaults

State: COMPLETE

## Goal and linked task

Fix hardcoded-provider selection for fresh bundled subagents in one fork PR. See [task](../tasks/00-portable-model-defaults.md) and [decision](../decisions/00-use-pi-model-defaults.md).

## Approval

The user approved both presented plans: "looked over both plans, i approve please bring this to completion". Recorded APPROVED, then IN PROGRESS. The user subsequently authorized dependency installation, authenticated smoke testing, and committing/pushing the reviewed fork with "yup please do".

## Ordered changes

1. `agents/scout.md`, `agents/researcher.md`, `agents/worker.md`: remove only the `model:` frontmatter field. Preserve all other fields and role text. With no spawn override, existing `launchSubagent` selects no explicit model and delegates selection to Pi.
2. `test/test.ts`, bundled-agent and `applySandboxToParts` tests: assert all three bundled profiles load without a model while retaining their existing tool/identity/spawn/autonomy fields. Add a model-less restricted loadout test: no `--model`, no accidental literal `undefined`, and tool/extension restrictions still present. Exercise the real helper, not a copied algorithm. Retain/add the explicit model plus thinking assertion to protect overrides. Include serialization round-trip of a null model using the existing loadout helpers so resume remains compatible. Test the current behavior that profile thinking is not forwarded without an explicit model; do not silently change it in this PR.
3. `README.md`, Spawning, Bundled agents, Custom agents, and Frontmatter reference: explain precedence (spawn override, profile model, otherwise Pi startup selection). Describe saved defaults/scopes versus current-parent state, model-less thinking defaults, and session restoration on resume. Remove hardcoded models from the bundled-agent table and from the basic custom-agent example; describe explicit model configuration as optional. Do not promise that configured credentials guarantee runtime availability.
4. `README.md`, fork attribution: identify this maintained fork while preserving upstream acknowledgements and license/author attribution. `package.json`, `repository.url`: point to `https://github.com/LokiLin14/pi-interactive-subagents`. Leave package name/version and dependencies unchanged; inspect the lockfile and update repository metadata only if represented there.
5. Update this plan's evidence/state and linked task checks after validation. No functional extension code change is expected: the existing omission path is the mechanism being tested. If tests reveal it cannot satisfy the goal, stop and revise the plan before expanding implementation.

## Validation

Acceptance: fresh bundled agents are model-less by default, explicit models still work, all sandbox/lifecycle fields remain intact, and documentation states limitations rather than promising parent-model inheritance or availability checks.

- Capture a baseline and run `git status --short` / `git diff --check` in this fork. Current checkout was clean before these documentation additions; Node is v26.8.1, package version 3.7.2, and node_modules is absent.
- After installation is authorized, use `npm ci --ignore-scripts` and record its outcome. Do not modify dependency versions or fix unrelated baseline failures.
- Run `npm test` before and after edits. It invokes `node --test test/test.ts`, including real exported extension helpers. Inspect integration prerequisites before running `npm run test:integration`; record blocked or skipped checks honestly.
- Verify frontmatter removals and preserved bodies against the baseline. Check concrete Markdown links and `git diff --check`.
- With separate authorization, load the fork explicitly in an isolated Pi session without changing the consumer's installed dependency. Launch one fresh worker without a model override; ask it to report PI_PROVIDER, PI_MODEL, and PI_REASONING_LEVEL via its shell tool. Verify which extension/profile source was loaded and compare only relevant startup model settings/scopes. Do not inspect private credentials or session traces. Record exact launch/spawn commands and result; do not mistake the installed temporary Astra pins for this fork's behavior.
- Obtain independent read-only feature and style reviews after all writers finish. Re-run affected checks if review leads to changes.

## Risks and open questions

Static investigation of installed Pi 0.86.1 supports omitted-model startup selection; the fork declares older Pi development dependencies, so unit tests alone do not prove current-runtime behavior. Model-less profiles also use Pi thinking defaults because `applySandboxToParts` only forwards profile thinking with an explicit model. Resumed sessions can keep earlier models. Custom providers supplied only by disabled extensions may be unavailable in restricted children. All are documented boundaries, not reasons to loosen sandboxing.

Availability notifications and a better error experience are deferred pending their own requirements and plan. No commits, pushes, remote PRs, package installation, or authenticated smoke test are authorized merely by writing this document.

## Implementation results

The delegated implementation worker exited with code 1 without diagnostics or edits. Main-session fallback implemented the three profile removals, README/metadata updates, and two real-helper regression tests. No functional extension code changed; the lockfile contains no repository metadata to update.

Validation completed after the user's additional authorization:

- Initial `npm test` exited 1 before assertions because `@mariozechner/pi-tui` was not installed. `npm ci --ignore-scripts` then succeeded; it reported 13 vulnerabilities (3 moderate, 9 high, 1 critical). Dependency versions and lockfile remained unchanged; remediation is separate work.
- `npm test`: 149/150 passed. The sole failure is the existing `getToolExtensionPath("web_search")` path assertion, which assumes a host extension not available here. To distinguish regression from baseline, archived unchanged HEAD `c3e8b53c0754ae5ccc19fdab5a7481ec039bc2f7` to `/tmp/pi-fork-head.xFtFSM`, linked the same node_modules, and ran `npm test`: 147/148 passed with the identical assertion failure. Output is at `/tmp/pi-fork-baseline-tests.txt`.
- `node --test --test-name-pattern='bundled agents omit|model-less loadouts|applySandboxToParts replays' test/test.ts`: 3/3 passed.
- `node /tmp/pi-model-default-smoke.mjs /home/lucas/Sessions/pi-developer/workspace/pi-interactive-subagents`: passed. The temporary probe imports this fork's extension directly, registers its actual subagent tool using a minimal callback context and synthetic parent session, asserts worker has no profile model, and invokes `{agent: 'worker', name: 'model-default-smoke', cwd: <scratch>, task: 'run printenv PI_PROVIDER PI_MODEL PI_REASONING_LEVEL'}` with no model argument. It receives completion through the extension callback, not a custom polling loop. The real child returned `openai-codex`, `gpt-6-astra`, `medium`, exit 0, matching saved startup fields. This tests the actual spawn path, not the installed temporary pinned profiles. Script/scratch evidence is local under `/tmp`; no private traces or credentials were inspected.
- `node --test --test-concurrency=1 test/integration/tmux-surface.test.ts`: 3/7 passed. Four assertions failed with markers wrapped across very narrow pane lines; no tmux implementation changed. Broad integration compatibility is not claimed. The lifecycle suite was inspected but not run because it still requests removed `fork`, `caller_ping`, and `systemPrompt` interfaces. Updating that harness is deferred.
- `git diff --check` passed. Python comparisons against HEAD passed for all three profiles: only the model lines changed. Concrete documentation links resolve. No functional extension, dependency, lockfile, or role-body edits occurred.
- Independent `fork-feature-review`: final PASS, no blocking feature findings. Targeted runtime evidence supplied by main accepted with the documented integration limitations; not independently rerun by reviewer.
- Independent `fork-style-review`: PASS, no blocking style/scope findings.

The targeted acceptance criteria are complete, with baseline/environment-wide limitations disclosed above. Runtime `.pi-session/` artifacts are excluded from staging. Commit/push are separately authorized publication steps; no PR creation or merge is authorized by this completion record.
