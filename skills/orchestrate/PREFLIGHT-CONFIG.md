# The orchestration config

`docs/agents/orchestration.md` holds the four answers **no run can discover**. Everything else
preflight needs — the gates, the bootstrap, the ordering convention — is read from CI, from the
lockfile and from `issue-tracker.md` on every run, so it never goes stale against them.

Read the file at preflight. When an answer is missing, grill the maintainer for it and write it
back. A repo whose answer to every question is "none" still gets the file, so nothing is asked
twice.

## The four questions

Ask them one at a time, each with your recommended answer, so the maintainer can accept in a word.

**1. Exclusive resources.** A resource is exclusive when it is a **singleton by identity**, not
when it is merely busy: a fixed port, a booted simulator, a device on a bus, a named container, a
shared database, a paid quota. CPU is not one.

> Which resources can only one implementer hold at a time, and which command needs each?
> Recommended: none — most repos have none, and the cap already holds the machine.

For each one, take a lease name, the commands that need it, and a stale timeout.

**2. The end-to-end proof.** The gates say the code compiles and its tests pass. This says it
*runs*.

> Beyond the gates, what must an implementer show to prove the change works?
> Recommended: name the command, or say there is none.

A repo with no such layer records "none". Then an accept record states plainly that no end-to-end
proof exists, rather than treating its absence as a pass.

**3. The cap.**

> How many implementers at once? Recommended: `max(1, cores / 2)`, computed per machine.

Record a number only when the maintainer overrides the default.

**4. The models.** One implements and one reviews, and a model name goes stale faster than this
file, so the default is worded to survive a generation.

> Which model implements each ticket, and which reviews it?
> Recommended: _tiers_ — a mid-tier model per implementer, the strongest available for review.

Record names only when the maintainer overrides the default.

## The file

```markdown
# Orchestration

What `/orchestrate` cannot discover on its own. The gates, the bootstrap and the backlog's
ordering come from CI, the lockfile and `issue-tracker.md` on every run, and are not repeated here.

## Exclusive resources

One implementer holds each of these at a time, under a lease of that name.

| Lease | What it guards | Commands that need it | Stale after |
| --- | --- | --- | --- |
| `simulator` | the one booted iOS simulator | `xcodebuild`, `xcrun simctl` | 30m |
| `port-3000` | the dev server's port | `npm run dev` | 30m |

_None_ is a complete answer, and the commonest one.

## End-to-end proof

What an implementer shows beyond the gates, to prove the change runs.

    <command, or `none`>

## Cap

`<number>`, or _machine_ for `max(1, cores / 2)`.

## Models

`<implementer> / <reviewer>`, or _tiers_ for a mid-tier implementer and the strongest reviewer.
```

## Keeping it honest

An entry that has become discoverable leaves this file. A gate that moves into CI is read off CI
from then on, and its line here goes — a copy that disagrees with CI is worse than no copy.
