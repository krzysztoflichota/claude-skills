---
name: orchestrate
description: "Run a repository's ready backlog with several implementers at once, each in its own worktree — claim, implement, review, prove, merge, record."
disable-model-invocation: true
---

# Orchestrate a backlog

You run a repository's **frontier** — every ticket that is ready and unblocked — with several
implementers at once. Each takes its own worktree and its own branch. You select, answer,
review, prove, merge and record. You write no product code.

You are the only participant with state across tickets, so everything you learn lands in a file
or on a ticket. A restart rebuilds the run from the tracker and from git, and from nothing else.

Reach for `/orchestrate-slice` instead when one spec's tickets must land one at a time.

## Preflight, once

This skill speaks in **roles** — `ready-for-agent`, `needs-triage`, `needs-info`,
`ready-for-human` — and in generic tracker operations. The repo's own files say what each one is
here. Settle all seven before the first ticket:

1. **The repo's configuration.** `docs/agents/issue-tracker.md` — every tracker operation below
   runs the way it says. `docs/agents/triage-labels.md` — the label string for each role.
   `docs/agents/domain.md` — where the domain docs and the ADRs live; read those too. A missing
   file means this repo is not set up for these skills: stop, and the maintainer runs
   `/mattpocock-skills:setup-matt-pocock-skills` first.
2. **The gates.** What CI runs on the landing branch is the list, plus any gate an agent doc names
   that CI does not run. Name each one *and* the command that runs it. **Print the list, and let
   the maintainer confirm it before anything starts** — a gate you did not find is a gate that
   never runs.
3. **The bootstrap.** The commands that turn a fresh checkout into one that can run a gate. The
   lockfile names the package manager; the repo's own README names anything more.
4. **The orchestration config** — `docs/agents/orchestration.md`, holding the three answers no run
   can discover. Read it, and when an answer is missing, grill the maintainer for it and write it
   back. See [`PREFLIGHT-CONFIG.md`](./PREFLIGHT-CONFIG.md) for the questions and the file's shape.
5. **The backlog.** A ready backlog is a precondition of this skill. It needs one ticket in the
   `ready-for-agent` role with no open blocker, and an `issue-tracker.md` that defines **what
   orders the backlog** and **which line says what a ticket buys**. Grill the maintainer for a
   missing convention, and record it in `issue-tracker.md` — that is the doc that owns tracker
   conventions, so `to-tickets` and `triage` then write tickets that conform.
6. **The trunk and the tree.** The branch this work lands on, a clean tree, and nothing unpushed.
   Either one dirty is an unfinished ticket: stop and ask the maintainer.
7. **The run directory.** `$(dirname <repo root>)/.orchestrate-<repo basename>/`, holding
   `worktrees/`, `leases/`, `logs/` and `table.md`. It sits outside the repo, so no worktree
   is ever staged by accident and no ignore rule is needed.

_Done when_ you can name each role's label, the tracker's commands, every gate with its command,
the bootstrap, the three config answers, the ordering convention, the trunk and the run directory.

## The frontier, and how much of it you take

The frontier is every open ticket in the `ready-for-agent` role with no open blocker and **no
assignee**. The repo's ordering convention ranks it.

- A blocking edge is whatever the tracker doc calls one: a native dependency, or a `Blocked by:` /
  `Depends on:` line in the body.
- Read each candidate whole, comments included. A comment can block a ticket that no blocking edge
  does.
- **Re-query the frontier at every pick.** A ticket the maintainer triages while you run joins the
  pool by itself.
- The **cap** is the config's, or `max(1, cores / 2)` with no ceiling. Hold that many implementers
  at once.
- An empty frontier is a correct state. Let the agents in flight finish, then report why it is
  empty and stop.

**Claim by assignee.** Assign a ticket to yourself the moment you pick it, and leave it assigned
until it closes. That is what stops a second orchestrator taking it, and what tells a restart which
tickets are yours.

_Done when_ you hold the cap's worth of tickets, or the frontier is empty, and each one you hold is
assigned.

## A ticket's stages

A ticket runs these seven in order. Tickets run concurrently, so at any moment each of yours sits
at a different stage. Advance whichever one just reported.

### 1. Isolate it

    git worktree add <run dir>/worktrees/<ticket> -b <branch> <trunk>

Run the bootstrap there, then **run the gate there once, before any work starts.** A worktree that
cannot green is not ready, and its failure belongs to the environment rather than to the ticket:
stop and tell the maintainer.

_Done when_ the worktree greens on unmodified code.

### 2. Hand it to a cold agent

One **sonnet** agent per ticket, freshly spawned, never one carried over from an earlier ticket.
Its prompt is the ticket, one standing rule, the worktree, and pointers:

    Invoke the mattpocock-skills:implement skill, and implement ticket NN with it.

    Work only inside the worktree at <path>. Every git command runs as `git -C <path>`.

    When you find a bug or a gap while you work, take the detour: fix it in this same
    session, in a commit of its own, whenever the fix needs no decision the ticket did
    not already settle. When it needs one, open a ticket in the `needs-triage` role.

    Read before you start: <pointers>

A **detour** is a find fixed where it was found, and it is the default.

**Pointers are file paths, never prose.** The repo's own `CLAUDE.md` or `AGENTS.md` is the standing
reading list, because that is what it is for. You add only what you learned *this run*: the file
holding an answer you gave earlier, and the ticket numbers in flight beside this one. A summary, a
hint or a "watch out for" belongs in the ticket or the doc it is about — write it there, then hand
over the path. Every later implementer starts cold and hits the same gap otherwise.

_Done when_ the agent is running, and its prompt holds the ticket, the standing rule, the worktree
path and a list of paths.

### 3. Answer from the best available evidence

A subagent runs to the end. It ends its turn to ask you a question, and `SendMessage` wakes that
same agent with your answer and its whole context. Send-backs travel the same way.

- Ground the answer in the repo's authorities, in this order: the ADRs, the domain docs, the spec,
  the repo's other documentation, the ticket itself.
- When those do not settle it, spawn a research agent and settle it before you reply. A guess from
  you is worse than a guess from the implementer, because yours carries authority.
- When the question exposes a gap in a ticket or a doc, write the answer into that file as well as
  replying — then the path is a pointer for the next agent.
- When the answer would change scope, an ADR, or an acceptance criterion, the maintainer decides.
- "Whichever you prefer" is not an answer.

_Done when_ the answer names the authority it rests on, and the file that had the gap now holds it.

### 4. Review, then prove

The agent's summary is a **claim**. The diff is the **evidence**.

1. **Review.** Spawn one **opus** reviewer against that branch. It reads the diff against the
   ticket's acceptance criteria and against whatever the repo's agent docs state as
   non-negotiable, and it returns findings. You read findings; you do not read diffs.
2. **Rebase.** Have the implementer rebase onto the current trunk and green it there, in its own
   worktree.
3. **Prove.** **You** run the gates the diff touches, in that worktree, by the commands preflight
   named. Send the output to `<run dir>/logs/<ticket>-<gate>.log` and read the exit code. On a
   failure, hand the implementer the log *path* — it reads the log, you do not.
4. **Prove it runs.** Whatever the config's end-to-end proof names, on top of the gates. When the
   config records that the repo has none, say so plainly in the accept record rather than treating
   its absence as a pass.

_Done when_ each acceptance line carries a citation from the reviewer's findings, every gate that
applies has exited zero under your own hand, and every commit no acceptance line claims is a named
detour.

### 5. Accept, or send it back naming the fix

Work reaches the bar. It never **moves the bar**. Send it back when the diff moved one:

- a check weakened rather than satisfied — a loosened rule, a suppression comment, a widened type,
  a skipped or deleted test, a relaxed assertion, a widened tolerance;
- an architecture, layout or golden test edited so a violation passes;
- the spec, an ADR, or the ticket's own acceptance criteria edited so the work fits them;
- an acceptance line quietly dropped, or met by weaker means than it asked for;
- a detour that needed a decision the ticket had not settled. Order it split out into a ticket in
  the `needs-triage` role, and the commit reverted.

Whatever the agent docs state as non-negotiable is the rest of the list. Walk those rules over the
findings one at a time, as review criteria rather than as background.

"Out of scope", "can be a follow-up" and "the test is flaky" are reasons to send work back. When
the ticket genuinely cannot be finished as written, move it to the `needs-info` role with the reason
in a comment — a smaller version of the ticket is not the ticket.

**Two send-backs, then the ticket parks.** It stays assigned, a comment records both send-backs and
what is unresolved, and the table marks it. The run carries on with everything else, and the
maintainer takes that one.

_Done when_ the work is accepted, or it is back with the same agent naming the fix, or it is parked
with the record on the ticket.

### 6. Merge under the lease

Take the `merge` lease. One ticket merges at a time, and the trunk stands still while it does.

    git -C <repo> merge --ff-only <branch>
    git -C <repo> push

A merge that will not fast-forward means the trunk moved after the rebase. Release the lease, send
the implementer back to stage 4 step 2, and take the lease again when it reports.

Accepted work reaches the remote. The implementer commits; you push, so nothing you have not
accepted is published.

_Done when_ the branch is on the trunk, the trunk is pushed, and the lease is released.

### 7. Record, and release

The ticket and the doc that owns it are where the record goes, and you keep no log of your own:

- an answer you gave in stage 3 → the ticket, and the doc whose gap the question exposed;
- what you sent back and why → a comment on the ticket;
- something you found that is not this ticket's → back to its implementer as a detour while it
  still has its context, and a new ticket in the `needs-triage` role once it does not.

Then close the ticket with a comment naming the commit that *completes* it — the last one its work
needed rather than the first. Work that sha out after the final fix has landed, because a commit
cannot contain its own sha. Remove the worktree, delete the branch, and take the next ticket cold.

Run the full gate on the trunk once, after the last merge of the run.

_Done when_ nothing you learned lives only in your own context, the ticket is closed naming its
commit, and the worktree is gone.

## Leases

A **lease** guards a resource that is a singleton by identity — a fixed port, a booted simulator, a
device on a bus, a paid quota. The config names them, and in most repos the list is empty. CPU is
not one: parallel gate runs are slower, and the cap is what holds that.

`merge` is always a lease, whatever the config says.

One subagent cannot see another, so the lease works with no coordinator:

    until mkdir <run dir>/leases/<name> 2>/dev/null; do sleep 5; done
    # ... hold it ...
    rmdir <run dir>/leases/<name>

`mkdir` is atomic, so the first to arrive wins. A lease older than the config's timeout is stale:
take it, and note in the table that you did. Tell an implementer which leases it must take, and for
which commands, in its prompt.

## The table

`<run dir>/table.md`, rewritten at every state change. It is the only thing the maintainer reads
while the run works, so the chat stays quiet between stop conditions.

One row per open ticket in the pool, in the repo's ordering convention, with the rows in flight
marked:

| Order | Ticket | What it buys | Label | State |
| --- | --- | --- | --- | --- |
| P1 | 143 | the one line the repo's convention names | ready-for-agent | **stage 4 — review** |
| P2 | 146 | … | ready-for-agent | queued |
| P1 | 140 | … | needs-triage | not in the pool |

Carry every open ticket, not only the ready ones, so the maintainer can triage in another agent and
watch a ticket join the pool. The state column is yours; every other column is the tracker's.

## Stop and ask the maintainer when

- this repo has no `docs/agents/` configuration, or no ready backlog;
- the tree or the trunk is dirty at the start of a run;
- a fresh worktree cannot green on unmodified code;
- the frontier is empty, or every ticket left is blocked behind a `ready-for-human` or
  `needs-triage` one;
- an answer would change scope, an ADR or an acceptance criterion;
- a gate fails for a reason outside the ticket's change;
- the trunk's final gate fails;
- a push is rejected for a reason a rebase does not fix.

In every one of these the ticket stays unimplemented by you. Your hands are for selecting,
answering, reviewing, proving, merging and recording.
