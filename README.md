# claude-skills

Reusable agent skills. Each folder is one skill: a `SKILL.md` plus optional `references/` and
`assets/`.

## abc — a review→investigate→implement loop that survives everything

Three specialised agents and a manager, coordinated through one file on disk.

| | Role | Never does |
|---|---|---|
| **A** | Reviewer — measures the build against the spec, produces a ranked difference list and a number | Touch code, or investigate causes |
| **B** | Investigator — finds the *cause* of each difference, writes a fix guide | Touch code |
| **C** | Implementer — works B's guide top to bottom, one commit per item | Judge whether something should be fixed |
| **Manager** | You, the main session — rules on what agents refuse to decide, checks their claims | Close the final gate alone |

Then A measures again, round after round, until the number reaches the target.

### Why bother with three agents

A single agent that measures, diagnoses and fixes will quietly grade its own work. It finds five
problems, fixes three, and reports success. The split exists so that **each role's output is
checked by the next role** — and in practice every round surfaces something the previous role got
wrong, including the manager's own rulings.

### What it is actually for

Work too large for one session, where "correct" is defined by something external — a design spec,
a reference implementation, a written requirement — and you want to stay the decision-maker
rather than hand over judgement.

It carries the parts that are only obvious after they bite you:

- **A shared state file that is true at every moment**, so an agent dying mid-sentence costs
  nothing.
- **One commit per item, pushed immediately.** Two agents doing identical work died the same way;
  the one that committed per item lost almost nothing, the one that saved its notes for the end
  lost everything.
- **Escalation instead of guessing** — an agent that hits a question it can't answer stops and
  flags it, rather than inventing an answer that looks fine.
- **Exclusions that leave the denominator**, so a zero target stays arithmetically reachable and
  the reviewer never starts rounding to please you.
- **Measuring against real inputs, not just a fixture.** The expensive failure this prevents: a
  build scoring *3% different* against a hand-built test case while, on real data, most of the
  spec had nothing behind it at all.
- **An auto-resume clock** for credit-limit stalls, plus a git-push mutex if more than one
  machine can write.

---

## workaholic-mode — the four-part working mode in one phrase

Say **"开启workaholic mode"** (or "workaholic mode", "turn on workaholic mode") and the agent turns on four things together:

1. **Reviewing cycle** — scored rounds against your target percentage, blind readers, every fix verified by the manager with its own mutation; it stops only when every theme reaches the target in two consecutive rounds ([peter14531453/reviewing-cycle](https://github.com/peter14531453/reviewing-cycle)).
2. **Subagent group** — A reviewer, B investigator, C implementer and blind readers, each in its own worktree; the manager rules and lands (`abc/` in this repo).
3. **Hourly clock** — a cheap scheduled check that wakes the manager when the work stalls (`workaholic-mode/references/hourly-clock.md`).
4. **Everyone at once** — every item that can start, starts now, in parallel; a free slot is refilled the moment a worker reports.

The target percentage is the one thing it asks for if you did not give it. `workaholic-mode/SKILL.md` is self-sufficient; install `abc` and `reviewing-cycle` too for the full templates.

### 中文说明

说一句 **“开启workaholic mode”**，就同时打开四件事：review cycling（按目标分数逐轮打分，连续两轮达标才停）、subagent 小组（A 评审 / B 调查 / C 实现 / 盲评员，经理裁决并亲自验证）、每小时时钟（工作停了就自动叫醒经理）、所有能开工的 agent 同时开工（谁先完成就立刻补上下一件）。没给目标分数时，它只问这一个问题。

---

## Installing

### Claude Code / Claude Desktop

Copy the skill folder into your personal skills directory:

```bash
cp -r abc workaholic-mode ~/.claude/skills/
```

They become available as `/abc` and `/workaholic-mode`, and triggers on its own when a task matches its description.

Project-scoped instead of personal: copy it to `.claude/skills/` inside the repo.

### claude.ai (every new chat, and Claude Code on the web)

Zip a skill folder (for example `workaholic-mode/`, with `SKILL.md` at the top of the folder) and upload it under Settings → Capabilities → Skills. An uploaded skill is synced into every new conversation of your account, so the trigger phrase works without installing anything per project.

### Other agents (ChatGPT, Codex, Cursor, or any agent that reads files)

Skills are plain Markdown with YAML frontmatter — nothing Claude-specific in the content. Two
ways to use them elsewhere:

**Paste it.** Open `abc/SKILL.md` and paste the body into the system prompt or custom
instructions. Point the agent at `references/` and `assets/` for the detail; the SKILL.md says
when to read each one.

**Give the agent the folder.** If it can read files, put the folder in the working directory and
tell it: *"Read `abc/SKILL.md` and follow it."* The reference files are loaded on demand, which
keeps the initial context small.

The only Claude-specific assumptions are the words used for spawning background agents. Anywhere
the skill says "spawn an agent", substitute whatever your tool calls it — a subagent, a task, a
separate chat window, or a human. The loop's structure does not depend on the mechanism.

---

## License

Use them however you like.
