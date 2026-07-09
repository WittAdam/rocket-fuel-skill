# Rocket Fuel

![Fable 5 and Codex, co-founders](assets/hero.png)

> One sees the future. One makes it happen. Run Claude Fable 5 and OpenAI Codex as co-founders in a single Claude Code command.

One Claude Code skill that seats two frontier models into a real founding team, using the operating system from [*Rocket Fuel*](https://www.eosworldwide.com/rocket-fuel-book) by Gino Wickman and Mark C. Winters. Fable 5 is the **Visionary**: vision, architecture, standards, final review. Codex is the **Integrator**: it filters the plan, kills the scope creep, builds the code, and reports honestly. They argue in a structured meeting until they are on the same page, and only then does anyone write code.

## TL;DR

```bash
npm i -g @openai/codex@latest && codex login   # ChatGPT account works, no API key
git clone https://github.com/NulightJens/rocket-fuel-skill.git ~/rocket-fuel-skill
mkdir -p ~/.claude/skills
ln -s ~/rocket-fuel-skill/rocket-fuel ~/.claude/skills/rocket-fuel
```

Then in Claude Code, from any folder:

> `/rocket-fuel build me a habit tracker with streaks`

That's it. The skill detects what you need, runs the interview, the adversarial planning meeting, the build, and the review. You make exactly three decisions: the interview answers, rare deadlock tie-breaks, and the final commit.

## Why this exists

Every week there is a new benchmark war. GPT beats Fable on one eval, Fable beats GPT on another, and you wonder if you picked the wrong tool. That is the wrong question.

In 2015, *Rocket Fuel* documented that great companies are never built by one genius. They are built by a Visionary who generates ten ideas a week and an Integrator who filters them and makes the best ones real. Only about 3 percent of people are Visionaries, only 5.5 percent can be Integrators, and almost nobody is both.

Here is the uncomfortable part: an unsupervised AI model behaves exactly like the book's Visionary. Ten ideas a week. Changes direction every time you blink. Starts everything, finishes nothing. All gas, no brake. It does not need a better benchmark score. It needs a co-founder with veto power.

Vision without execution is just hallucination.

## How it works

One command, four functions, routed by context:

| You say | It runs |
|---|---|
| `/rocket-fuel build me X` in a fresh folder | **KICKOFF**: grill interview → VTO → Same Page Meeting → 3-7 Rocks → Codex builds → Fable reviews |
| `/rocket-fuel this repo is a mess` | **CLARITY BREAK**: solo audit → ranked Issues List → pick this cycle's Rocks → build |
| `/rocket-fuel review PLAN.md` | **SAME PAGE MEETING**: Codex attacks the plan read-only until the verdict line flips |
| `/rocket-fuel have codex add dark mode` | **BUILD A ROCK**: frozen one-task contract → sandboxed build → diff review with proof |

Under the hood it is the book's toolkit, seat by seat:

- **The Accountability Chart**: one seat per model, five accountabilities each. When more than one is accountable, nobody is.
- **The Same Page Meeting**: Fable submits the plan, Codex triages every finding as `[KILL | DEFER | FIX | CLARIFY]`, Fable answers each one in a log, and the loop resumes the same Codex session so it remembers what it flagged. Bounded at 5 rounds.
- **Rocks**: never more than 7 priorities per cycle. Do less better.
- **No End Runs**: mid-build change requests go on the Issues List, never into the working diff.
- **Level 10 review**: Fable reads the full diff and runs the proof itself. The Integrator's pasted output does not count as proof.

What a real round one looks like (actual output from this skill's verification run):

```
- [FIX] Proof only checks that something runs, not that output is valid ->
  Verify output is exactly one of the 3 quotes and exits 0.
- [FIX] `--all` behavior lacks an ordering/format contract ->
  Define that it prints all 3 quotes, one per line, in stable order.
- [DEFER] "CLI tool" is treated as `python3 quote.py`, not an installed command ->
  Ship the script first; add packaging only after the MVP works.
VERDICT: NOT YET
```

Fable fixes the plan, resumes the session, and Codex checks its own findings were addressed:

```
VERDICT: SAME PAGE
```

Every project leaves four artifacts at the repo root:

```
your-project/
├── VTO.md               # the vision (kickoffs only)
├── PLAN.md              # the what: 3-7 Rocks, each with a proof command
├── SAME-PAGE-LOG.md     # the why: the full argument, round by round
└── ISSUES.md            # everything deferred, ranked by impact
```

`SAME-PAGE-LOG.md` is the file worth reading. It is the visible argument between your two co-founders.

## Install

```bash
npm i -g @openai/codex@latest
codex login
git clone https://github.com/NulightJens/rocket-fuel-skill.git ~/rocket-fuel-skill
mkdir -p ~/.claude/skills
ln -s ~/rocket-fuel-skill/rocket-fuel ~/.claude/skills/rocket-fuel
```

The symlink means `git pull` in `~/rocket-fuel-skill` updates the skill instantly. Restart Claude Code after the first install.

**Smoke test** — in Claude Code from any directory:

> "What skills do you have for running Codex as a co-founder?"

You should see `rocket-fuel` in the response.

## Use it

### 1. Kick off a project

> `/rocket-fuel build me a CLI tool that tracks my reading list`

Fable interviews you one question at a time, each with a recommended answer. Facts it can find itself, it finds. Only decisions come to you. Then watch the Same Page Meeting run in `SAME-PAGE-LOG.md`.

### 2. Refactor an existing repo

> `/rocket-fuel this codebase has gotten messy, help me clean it up`

Fable audits alone first and dumps everything into `ISSUES.md`, the way the book's Visionaries hand their new Integrator a 32-item wish list. You pick 3 to 7 for this cycle. Not 20.

### 3. Pressure-test any plan

> `/rocket-fuel review docs/migration-plan.md`

Five rounds max, triage verdict on every finding, full log. Works on plans Fable wrote and plans it has never seen.

### 4. Delegate a scoped task

> `/rocket-fuel have codex add a --json flag to the export command`

This is the daily driver. Fable freezes a one-Rock contract, Codex builds it in its own sandbox, Fable reviews the diff and runs the proof before anything is called done.

## Requirements

- [Claude Code](https://claude.com/claude-code), installed and authenticated
- [Codex CLI](https://github.com/openai/codex) 0.130 or newer, logged in (a ChatGPT Free/Plus/Pro account is enough, no API key)
- `git` (the clean-tree gate and diff review depend on it)

## Troubleshooting

| Symptom | Fix |
|---|---|
| `codex --version` dies with `spawn ... ENOENT` | The npm install lost its native binary. `npm i -g @openai/codex@latest` again. |
| Codex calls return auth errors | `codex login` again. |
| A review round hangs at 0 CPU | A raw `codex exec` was run without stdin redirect. The skill guards this; if you script your own calls, append `< /dev/null`. |
| "Not inside a trusted directory" | The skill passes `--skip-git-repo-check` everywhere; add it to any manual calls. |
| A build touches files mid-review | Should be impossible: resumes force `-c sandbox_mode="read-only"`. If you ever see it, `git status`, revert, and open an issue here. |

## When to use something else

- **Trivial edits**: a one-line fix does not need a founding team. Just ask Fable.
- **No ChatGPT account**: the Integrator seat is Codex. Without it, use [grill-me](https://github.com/mattpocock/skills) for the interview and skip the cross-model review.
- **You want a second opinion, not a second builder**: run function 3 (the Same Page Meeting) alone and keep the build with Fable.

## Credits

- Operating system from *Rocket Fuel* by Gino Wickman and Mark C. Winters (BenBella, 2015)
- Codex plumbing built on [grill-me-codex](https://github.com/chaseai-yt/grill-me-codex) by Chase AI (MIT) and [skill-codex](https://github.com/skills-directory/skill-codex)
- Interview mechanics and authoring patterns from [Matt Pocock's skills](https://github.com/mattpocock/skills)
- Skill packaging by [@NulightJens](https://github.com/NulightJens)

## License

MIT
