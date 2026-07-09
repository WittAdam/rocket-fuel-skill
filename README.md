# Rocket Fuel

![Fable 5 and Codex, co-founders](assets/hero.png)

> One sees the future. One makes it happen. Run Claude Fable 5 and OpenAI Codex as co-founders in a single Claude Code command.

Fable 5 takes the Visionary seat: it interviews you, writes the plan, sets the standards, and reviews the result. Codex takes the Integrator seat: it filters the plan, kills scope creep, builds the code in its own sandbox, and reports honestly. They argue in a bounded meeting until the verdict line says `SAME PAGE`, and only then does anyone write code. Based on the Visionary/Integrator operating system from *Rocket Fuel* by Gino Wickman and Mark C. Winters.

You make exactly three decisions per run: the interview answers, rare deadlock tie-breaks, and the final commit. The skill guides you through everything else: it tells you which mode it detected, what happens next, and where every artifact lands.

## Install

```bash
npm i -g @openai/codex@latest && codex login   # ChatGPT account works, no API key
git clone https://github.com/NulightJens/rocket-fuel-skill.git ~/rocket-fuel-skill
mkdir -p ~/.claude/skills
ln -s ~/rocket-fuel-skill/rocket-fuel ~/.claude/skills/rocket-fuel
```

Restart Claude Code once. Requirements: [Claude Code](https://claude.com/claude-code), [Codex CLI](https://github.com/openai/codex) 0.130+, git.

## Use

One command, plain English. The skill picks the right mode from context:

| You say | You get |
|---|---|
| `/rocket-fuel build me a habit tracker` | Full kickoff: interview → adversarial plan review → Codex builds → Fable verifies |
| `/rocket-fuel this repo is a mess, clean it up` | Codebase audit → ranked issue list → you pick 3-7 fixes → build |
| `/rocket-fuel review PLAN.md` | Codex attacks your plan read-only until it approves |
| `/rocket-fuel have codex add dark mode` | One scoped task, built by Codex, diff-reviewed and proof-run by Fable |

From there the skill walks you through it: one interview question at a time with a recommended answer attached, live verdicts from Codex each round, and a clear prompt before anything is committed. The full argument between the two models is saved to `SAME-PAGE-LOG.md` in your repo, next to `PLAN.md`.

## Credits

Operating system from *Rocket Fuel* (Wickman & Winters). Codex plumbing built on [grill-me-codex](https://github.com/chaseai-yt/grill-me-codex) by Chase AI and [skill-codex](https://github.com/skills-directory/skill-codex); interview mechanics from [Matt Pocock's skills](https://github.com/mattpocock/skills). Packaged by [@NulightJens](https://github.com/NulightJens).

MIT License.
