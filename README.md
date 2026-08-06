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

## Installing

### Claude Code / Claude Desktop

Copy the skill folder into your personal skills directory:

```bash
cp -r abc ~/.claude/skills/
```

It becomes available as `/abc`, and triggers on its own when a task matches its description.

Project-scoped instead of personal: copy it to `.claude/skills/` inside the repo.

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
