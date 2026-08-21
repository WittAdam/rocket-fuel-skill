# rocket-fuel-skill (Adam's fork)

This folder is a **fork**, not the upstream project. Read this before changing anything.

## What this is

A Claude Code skill that runs Claude as Visionary (plan, review, run the proof) and OpenAI
Codex as Integrator (critique the plan read-only, then build in a sandbox). Upstream is
`NulightJens/rocket-fuel-skill` (MIT), by Jens Heitmann.

## Remotes — push to the right one

```
origin  https://github.com/NulightJens/rocket-fuel-skill.git   # upstream, PULL only
fork    https://github.com/WittAdam/rocket-fuel-skill.git      # Adam's, PUSH here
```

`git push fork HEAD:main`. Never push to `origin`.

## Keep the diff small — that is the whole strategy

The fork carries a deliberately tiny patch (~34 lines, one file) so upstream keeps merging
cleanly. Everything else must stay byte-identical to upstream. If you are about to change a
second file, stop and ask whether it belongs in this fork at all.

The patch is entirely in `rocket-fuel/SKILL.md`:

1. **Phase 0 lease gate.** Before anything else, the run claims the repo through the PDP
   session relay and releases on every exit path. See below.
2. **Rule 6** in the rules list, stating the same thing.
3. **Corrected Windows preflight.** Upstream tells you to fix a failing `codex --version`
   with `npm i -g @openai/codex@latest`. On Adam's machine that is wrong twice over — see
   "Windows traps".

## Why the gate exists

The skill writes `VTO.md`, `PLAN.md`, `ISSUES.md` and `SAME-PAGE-LOG.md` as **fixed
filenames at the repo root**, and lets Codex write files unsupervised. Upstream's built-in
`RF-` rename only fires on *unrelated* content, so two rocket-fuel runs against one repo land
on top of each other rather than being renamed apart. Adam and Russ share repos, so that is a
real collision, not a theoretical one.

```bash
py ~/.claude/session-mailbox.py claim --resource "rocket-fuel:$(basename "$PWD")" --ttl 180
py ~/.claude/session-mailbox.py release --resource "rocket-fuel:$(basename "$PWD")"
```

Exit contract, and the third line is the one that matters:

| Exit | Meaning | Action |
|---|---|---|
| 0 | lease held | proceed |
| 1 | someone else holds it | stop, name the holder |
| 3 | relay unreachable / leases dir unwritable | **stop** — a claim that could not be *written* is an unknown repo, not a free one |

If you touch that table, keep exit 3 distinct from exit 1. Collapsing them makes a broken
share indistinguishable from a colleague working, which is the exact failure the gate exists
to prevent.

**The gate is live as of 2026-08-20.** `claim` / `release` / `claims` now exist in the copy the
hook actually runs (`C:\Users\adamp\.claude\session-mailbox.py`) and were smoke-tested end to
end: claim exit 0, release exit 0, real holder name. The unwritable-directory case genuinely
returns 3 rather than 1, so the table above is enforced by the client, not merely documented here.

## Windows traps (both cost real time on 2026-08-19)

- Codex here is the **desktop app** build, not npm. Running `npm i -g @openai/codex@latest`
  installs a *second, competing* copy. Always `codex --version` first.
- Claude Code shells out through git-bash, and `codex` is a `.cmd` only PowerShell resolves.
  Bash reports `codex: command not found` and the skill dies at preflight. The fix is a shim
  named `codex` (no extension) on the bash PATH — Adam's lives at `C:\Users\adamp\bin\codex`
  and discovers `%LOCALAPPDATA%\OpenAI\Codex\bin\<hash>\codex.exe` at runtime, since the hash
  directory changes on app updates.
- **A Windows env var set today is invisible to a process started yesterday.** `codex` inherits
  its environment from whatever launched it, so a token added to the user environment is not
  visible to an already-running Claude Code session, nor to the `codex` that session spawns --
  which is exactly how this skill calls it. A `bearer_token_env_var` then resolves to nothing and
  the MCP server fails auth for no visible reason. One restart of Claude Code fixes it. Bites
  `GITHUB_PERSONAL_ACCESS_TOKEN` and `CLAUDE_MAILBOX_HMAC_KEY` alike.

## Installing / updating

The active skill is a **copy**, not a symlink — a symlink into SMB `Z:` is unreliable, and
the other 15 skills in that directory are plain folders too. So a `git pull` here does not
reach the running skill. After any change:

```bash
git pull origin main          # upstream, if updating
git push fork HEAD:main
cp rocket-fuel/*.md ~/.claude/skills/rocket-fuel/
```

That third line is the step people forget. If the gate misbehaves, check that the installed
copy actually matches this one before debugging anything else.

Both Adam and Russ install from **the fork**. Anyone installing from upstream gets no gate.

Adam's installed copy has been the gated fork since 2026-08-20; before that it was plain upstream.
Russ's setup email predates the gate and still points at upstream -- he has not been re-sent yet.

## Codex MCP plugins

`~/.codex/config.toml` lists MCP servers that load on **every** `codex` run, which is why their
auth state shows up in this skill's output. State as of 2026-08-20:

- **github -- authenticated.** `codex mcp login github` does not work here; GitHub's endpoint
  rejects the dynamic client registration Codex attempts. The route that works is a bearer token
  held by *reference*, so no secret lands in the config file:

  ```bash
  codex mcp add github --url "https://api.githubcopilot.com/mcp/" \
    --bearer-token-env-var GITHUB_PERSONAL_ACCESS_TOKEN
  ```

  Verified by MCP `initialize` handshake: HTTP 200, `serverInfo: github-mcp-server`. Subject to
  the env-var trap above.
- **stripe -- deliberately unauthenticated.** Adam's call, 2026-08-20. It logs an auth failure on
  every run; that noise is expected, not a regression.

## Do not

- Do not add a `CLAUDE.md` to `~/.claude/skills/rocket-fuel/`. Claude Code resolves CLAUDE.md
  by walking up from the *working directory*; the skill folder is never in that chain, so it
  would never load. Skill instructions belong in `SKILL.md` or a file `SKILL.md` links.
- Do not point Codex at `/opt/pdpros/ai-receptionist`, deploy scripts, or anything holding
  credentials. The build step runs it unsupervised with `workspace-write --full-auto`.

## Related

Vault page `Z:\claude-skills\Adam-Vault\Projects\Rocket Fuel\Rocket Fuel.md` ·
relay client `C:\Users\adamp\.claude\session-mailbox.py` (repo: `WittAdam/session-mailbox`)
