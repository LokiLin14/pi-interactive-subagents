# Defer the cross-cwd artifact-path fix and use a same-cwd workaround

## Requirements

Subagents must be able to start in a different working directory without losing the task, system prompt, or activity artifact paths created by the parent. Startup errors should remain diagnosable even when Pi exits before creating a session.

While reviewing pi-theory-tutor from a parent Pi Developer session, two implementation workers and two reviewers exited with code 1 before producing a result. The parent used a relative session directory (`--session-dir ./.pi-session`); each child was launched with a different `cwd`. Explicitly selecting the parent's working model did not resolve the failure.

## Final Decision

Record the bug here and defer extension changes to a separate, explicitly approved implementation plan. For the immediate read-only reviews, launch children in the parent's working directory and tell them to inspect the target repository using absolute file paths and `git -C <target>`.

This avoids the broken launch-time path resolution without modifying the installed extension, user configuration, or the repository under review. It is a review workaround, not equivalent to loading the target directory's Pi setup: the child retains the parent's startup configuration. Work that requires target-cwd startup still needs the eventual fix.

This branch contains documentation only. No model-policy, sandbox, renderer, or consumer-launcher changes are included.

## Findings and evidence

Observed with Pi `0.86.1`, Node `v26.8.1`, tmux, and this fork at `d3b9a6f947c08035f80adad3498b5f647b7e1d2f`.

In [the subagents implementation](../../pi-extension/subagents/index.ts):

- `getArtifactDir(sessionDir, sessionId)` returns `join(sessionDir, "artifacts", sessionId)` without making a relative session directory absolute.
- `launchSubagent()` writes parent-relative task/system-prompt paths, then prefixes the child invocation with `cd <effectiveCwd>`. The same path strings now refer to files under the child directory instead of the parent's artifact directory. Activity-file paths are also relative in the observed launch scripts.
- All four failed launch scripts pointed to task and system-prompt files that existed relative to the parent and did not exist relative to the child.
- Replaying only the missing task argument in an offline Pi invocation returned `Error: File not found: <child>/.pi-session/artifacts/.../context/<task>.md`, exit 1. This is a local startup failure, not evidence of an authentication or provider failure.
- `watchSubagent()` reports the last session message, a sidecar error, or a generic exit-code summary for Pi children; it does not capture their terminal startup output before closing the pane. With no child session or sidecar, the parent only receives `Sub-agent exited with code 1`. A resume attempt then reports that the session file is gone. This does not establish that a session was ever created.

### Minimal offline reproduction of the path failure

This isolates Pi's resolution of the generated relative task argument. It does not invoke the complete extension or make a model request: the missing file stops startup first.

```sh
probe=$(mktemp -d)
mkdir -p "$probe/parent/.pi-session/artifacts/probe/context" "$probe/parent/child"
printf 'Reply with OK.\n' > "$probe/parent/.pi-session/artifacts/probe/context/task.md"
(
  cd "$probe/parent/child" &&
  pi --offline --no-extensions --no-skills --no-session -p \
    '@.pi-session/artifacts/probe/context/task.md'
)
printf 'child exit: %s\n' "$?"
```

Executed during diagnosis: Pi reported the nonexistent child-relative task path and exited 1. The task file existed under `parent/.pi-session/`. Only generated launch artifacts and source were inspected; no lesson transcripts or credentials are included in this record.

## Deferred repair direction and validation

A future plan should consider normalizing artifact paths against the parent cwd before changing the child's cwd, covering task files, system-prompt files, launch scripts, activity files, and resume paths. Preserve the child's intended cwd and sandbox. Confirm the exact interfaces before choosing where normalization belongs.

Required regression coverage should include relative and absolute parent session directories, same and different child cwd, spaces in paths, and fresh/resumed launches. Prove that the child reads the intended task and prompt and writes activity to the parent's expected location. Separately consider capturing a bounded startup error before pane teardown so pre-session failures are actionable.

These are investigation/acceptance directions, not an approved implementation design. Do not bundle the fix into consumer dependency recovery.

## Alternatives considered

- Keep retrying or override the model: does not repair missing file paths and already failed in this incident.
- Launch from the parent cwd with absolute target paths: chosen immediate workaround for read-only reviews; does not reproduce target-cwd configuration loading.
- Restart the parent with an absolute session directory: plausible mitigation, but interrupts the active session and was not required for the chosen workaround; not validated here.
- Copy/symlink parent artifacts into the target checkout: pollutes the target and masks the bug; rejected.
- Patch the installed package in place: risks untracked local divergence; defer a reviewed fix in this fork instead.
