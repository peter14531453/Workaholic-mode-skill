# The resume clock — surviving credit limits and closed laptops

A loop that takes many rounds will hit usage limits. Without a clock, the user has to keep
coming back to say "continue". With one, the work resumes on its own.

There are two mechanisms. They solve different failure modes, and using both is reasonable.

---

## 1. The in-session wakeup — the default

A self-paced timer in the running session that re-reads §1 and continues whoever's turn it is.

**Set it up** by starting a self-paced loop whose prompt is, in substance:

> Resume the `<name>` loop. Read `<state-file>` §0b (the manager's resume playbook) and §1
> (current state), work out whose turn it is, and continue — spawning the agent whose turn it is
> on `<cheap model>`, in the background, or doing the work directly in the main session if spawns
> keep dying on the credit limit. If the limit is still exhausted, do nothing this tick and wait
> for the next one.

**Cadence: 20–30 minutes.** Tighter is pure waste — a failed spawn tells you nothing a later one
won't, and each idle tick costs tokens.

**What it gives you:** the local environment. Credentials, local services, the real checkout,
the machine's own tooling. If any turn needs those, this is the only mechanism that can run it.

**What it does not survive:** the session closing, or the machine sleeping or hibernating.

**Worth checking on a laptop:** whether the machine auto-sleeps, and what closing the lid does.
Screen-off and lock-screen are harmless — the process keeps running. Sleep and hibernate are
not. On Windows, `powercfg /query SCHEME_CURRENT SUB_SLEEP STANDBYIDLE` reads the sleep timeout;
`0` means never. The lid action is often hidden from that query and is easier to check in the
system's power settings UI.

---

## 2. The scheduled cloud run — for the hours the machine is off

A recurring job in the cloud that checks out the branch and does the same thing.

**It needs the work pushed.** A cloud checkout sees the remote, not the local machine. Push
before setting this up, and make "push after every commit" a ground rule from then on —
otherwise the cloud run works from a stale snapshot.

**Before pushing**, scan what is about to go out. Confirm no credential file was ever committed
and no key-shaped strings appear in the outgoing diff.

**Keep the scheduled prompt short.** Long prompts get truncated by the scheduling API, and the
durable instructions belong in the repo anyway. Point it at the state file:

> Check out `<branch>`, pull, and look at recent commits and status. Then open `<state-file>` and
> read §0b, §0c, §1, the current round's section in §4, and every ruling. It is ~N lines — read
> it in pages. §0b and §0c together are your complete brief; they are written for a session
> exactly like yours, with no memory. Where §0c conflicts with §0b, §0c wins. Do whichever turn
> §1 says is next. If §1 shows nothing pending, change nothing and exit quietly.

**Then write a §0c in the state file** with the constraints only a cloud run has:

1. **What it cannot do, and what to do instead.** A cloud checkout has no local credentials, so
   any turn needing them is impossible. Be explicit: *do not fake it and do not substitute other
   work* — append a note saying a cloud run reached that turn and could not do it, leave §1
   pointing at that turn, commit, and stop. The local machine picks it up.

   This matters more than it sounds. A cloud run that produces a fixture-only measurement and
   calls it the round hides exactly the failure the real-data pass exists to catch.

2. **A quiet no-op is the correct outcome most of the time.** It fires on a schedule; most
   firings have nothing to do. Say so, or it will invent work to justify the run.

3. **Do the work directly; do not spawn subagents.** Nested spawning inside a scheduled run is
   what credit limits kill first.

4. **Commit and push before stopping.** The checkout is discarded when the run ends.

---

## 3. The turn lock — only if both mechanisms exist

Two writers on one branch will diverge: each pulls a snapshot, works from it, and produces a
history the other cannot merge. The conflicts land in the state file itself — and merging two
versions of "whose turn is it" produces a file that is true for neither.

**Note what is *not* a second writer.** Connecting a phone or another computer to the *same*
running session — screen-sharing style remote control — adds a window, not a worker. There is
still one session doing the work. A lock is only needed when two independent sessions can write.

### The protocol

Put a `HELD BY:` line at the top of §1 and a §0d explaining it. To claim:

1. **Pull.** Always, first.
2. Read `HELD BY`. **If it names someone else and its timestamp is under 2 hours old, stand
   down** — change nothing, commit nothing, exit.
3. If free, stale, or already yours: set it to your identifier and the current UTC time, commit,
   and **push**.
4. **If that push is rejected, you lost the race.** Pull, re-read, stand down.

**Step 4 is the actual lock.** Git refuses a non-fast-forward push, so exactly one writer can
win. Everything above it is bookkeeping.

**Release it** — set `HELD BY: free`, commit, push — as part of your final commit. And make it
**stale after ~2 hours**, because an agent that dies mid-task will not release it, and a lock
that cannot expire turns one crash into a permanently jammed loop.

---

## When spawning keeps failing

Two consecutive immediate deaths on a credit limit means spawning is not going to work right
now. **Stop retrying and do the work yourself in the main session.**

Two things to get right when you do:

- **Say so in the log.** A round the manager both performed and graded is less independent than
  one an agent ran. Hiding that costs more later than admitting it now.
- **Do not let it become the habit.** The separation between roles is what makes the loop find
  its own mistakes. Go back to spawning as soon as the limit clears.
