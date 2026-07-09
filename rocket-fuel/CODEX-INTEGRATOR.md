# Codex Integrator: the invocation contract

Every Codex call in this skill goes through these exact patterns. They encode real traps; do not improvise around them. All patterns verified live on codex-cli 0.143.0 (2026-07-08).

## The non-negotiables (every call)

1. `< /dev/null` on every call whose prompt is a positional arg. `codex exec` reads stdin IN ADDITION to the prompt; under a non-TTY harness it hangs forever without it (symptom: 0 bytes output, 0 CPU). Calls that feed the prompt via `- <"$FILE"` are already safe: stdin is the file.
2. `2>/dev/null` always. Codex streams thinking tokens to stderr; they would flood Fable's context.
3. `--json -o <outfile>`: parse the JSONL stream ONLY for the thread id (`grep -m1 '"type":"thread.started"'`); read the actual answer from the `-o` file. Never parse the stream for the report.
4. Resume by explicit thread id, never `--last`. A wrong session looks exactly like success.
5. Never pin `-m`. Model pins 400 on ChatGPT-account auth. Before round 1, echo the active model if `~/.codex/config.toml` has a `model` line, else say "Codex CLI default".
6. Bash tool `timeout: 600000` (10 min) on every call. Codex writes output only at completion; a killed call is silently empty. If a call times out, say so and retry once before surfacing.
7. Long prompts go in a temp file, never inline in the command string.
8. `--skip-git-repo-check` on every call. Codex refuses to run outside a trusted git directory without it (error: "Not inside a trusted directory"), and kickoffs often start in a brand-new folder.
9. A stream event complaining about "skills context budget" is benign Codex housekeeping, not a failure. Ignore it.

## Review calls (Same Page Meeting, read-only)

Fresh session, capture the thread id:

```bash
PROMPT_FILE=/tmp/rf-prompt.md   # write the review prompt here first
OUT=/tmp/rf-review.txt
codex exec -s read-only --skip-git-repo-check --json -o "$OUT" "$(cat "$PROMPT_FILE")" \
  < /dev/null 2>/dev/null | grep -m1 '"type":"thread.started"'
```

Extract the id from the `thread.started` line (`"thread_id":"..."`), keep it as `THREAD_ID`, and echo it visibly before every resume.

Resume the SAME session. THE SAFETY LINE: `resume` rejects `-s`; without `-c sandbox_mode="read-only"` Codex inherits the config default and can WRITE files mid-review.

```bash
codex exec resume "$THREAD_ID" -c sandbox_mode="read-only" --skip-git-repo-check --json \
  -o "$OUT" "<follow-up prompt>" < /dev/null 2>/dev/null >/dev/null
```

Then Read the `$OUT` file and grep its last line for `VERDICT:`.

## Build calls (rock execution, sandboxed write)

Gate first: `git status --porcelain` must be empty. If not, stop and ask the user to commit or stash. A clean tree is what makes the diff reviewable and revertible.

Default build mode is Codex's OWN sandbox with write access to the workspace. Do NOT reach for `--yolo` / `--dangerously-bypass-approvals-and-sandbox` by default: the sandboxed mode builds real features, and the bypass flags run an unsandboxed autonomous agent, which needs the user's explicit, per-run approval (harness permission systems will rightly block it otherwise).

Write the build contract to a file, feed it via stdin:

```bash
P=/tmp/rf-build-contract.md
OUT=/tmp/rf-build.txt
codex exec --sandbox workspace-write --full-auto --skip-git-repo-check --json \
  -o "$OUT" - <"$P" 2>/dev/null | grep -m1 '"type":"thread.started"'
```

If the rock needs network (installing deps), add `-c sandbox_workspace_write.network_access=true` and say so in one line before running. Only if the rock genuinely cannot run sandboxed (rare) ask the user to approve `--dangerously-bypass-approvals-and-sandbox` explicitly.

Fix rounds resume the same session with the sandbox forced via `-c` (same rule as reviews):

```bash
codex exec resume "$THREAD_ID" -c sandbox_mode="workspace-write" --skip-git-repo-check --json \
  -o "$OUT" "<one consolidated fix list>" < /dev/null 2>/dev/null >/dev/null
```

## The build contract template

```
GOAL: <one paragraph: what done looks like for THIS rock>
SPEC: Read <PLAN_FILE> at the repo root, rock <N>. It is frozen and already
  reviewed. Implement it exactly. If a step is impossible as written, implement
  the closest faithful version and report the deviation. Do not redesign.
KEY PATHS: <files/dirs to touch, files to read first>
CONSTRAINTS: <do-not-touch list, style rules, deps that must not change>
NON-GOALS: <explicitly out of scope, including every other rock>
PROOF: Run `<PROOF_CMD>` and include its full output in your report.
OUTPUT: End with a report: files changed (one line each: path + what/why),
  the proof output, and any deviations from the spec with reasons.
```

## Level 10 review mechanics (after every build call)

1. `git diff --stat` then read the FULL diff. Codex's report is a claim, not evidence.
2. Run `PROOF_CMD` yourself. Only your run counts.
3. Findings within the rock's scope: send ONE consolidated fix-round prompt (resume, same session): "Fix these N items, nothing else, re-run the proof." Max 2 fix rounds.
4. Still broken after round 2: the Visionary takes the wheel. Announce it, fix it yourself, note it in the close-out scorecard.
5. Deviations Codex reported: judge each against the Core Focus. Execution-detail deviations stand (the Integrator is the Tie Breaker). Vision or scope deviations get reverted in a fix round, with the reason logged.
6. Commits: user-gated, authored by Fable, conventional format. Codex never commits.

## Failure handling

- Non-zero exit or empty `-o` file: retry once with the same thread. Second failure: stop and surface stderr (rerun the identical command WITHOUT `2>/dev/null` to capture the error).
- `codex login` expired (auth errors): tell the user to run `codex login`; do not attempt to re-auth for them.
- `codex --version` dies with `spawn ... codex-darwin-arm64 ... ENOENT`: the npm install is broken (missing native binary). Fix: `npm i -g @openai/codex@latest`, then re-verify.
- Prerequisite floor: Codex CLI >= 0.130; contract verified on 0.143.0.
