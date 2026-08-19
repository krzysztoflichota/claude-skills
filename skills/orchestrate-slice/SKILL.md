---
name: orchestrate-slice
description: "Run one slice's tickets end to end as the orchestrator — select, delegate, verify, accept, push, record."
disable-model-invocation: true
---

# Orchestrate a slice

A **slice** is one spec's worth of tickets. You run it: you take the next ticket off the
**frontier**, hand it to a **cold** agent, answer what that agent cannot answer alone, and
decide whether the finished work is accepted or sent back. You write no product code.

You are the only participant that keeps state across tickets, so every decision you make
ends up in a file.

## Preflight, once

This skill speaks in **roles** — `ready-for-agent`, `needs-triage`, `needs-info`,
`ready-for-human` — and in generic tracker operations. This repo's own files say what each
one is here. Settle all four before the first ticket:

1. **The repo's configuration.** `docs/agents/issue-tracker.md` — every tracker operation
   below runs the way it says. `docs/agents/triage-labels.md` — the label string for each
   role. `docs/agents/domain.md` — where the domain docs and the ADRs live; read those too.
   A missing file means this repo is not set up for these skills: stop, and the human runs
   `/mattpocock-skills:setup-matt-pocock-skills` first.
2. **The slice.** The argument names it; with no argument, take the one spec that still has
   open tickets, and ask the human when more than one does.
3. **The gates.** What CI runs on the landing branch is the list, plus any gate the agent
   doc names that CI does not run. Name each one *and* the command that runs it. You run
   them in step 4, so a gate you did not find here is a gate that never runs.
4. **The branch and the tree.** The branch this slice lands on, a clean tree, and nothing
   unpushed. Either one dirty is an unfinished ticket: stop and ask the human.

_Done when_ you can name each role's label, the tracker's commands, the spec, the landing
branch, and every gate with the command that runs it.

## 1. Take the ticket at the head of the frontier

The frontier is every open ticket in the `ready-for-agent` role whose every blocking edge
is closed. First in the tracker's order wins — the lowest number, where tickets are
numbered.

- A blocking edge is whatever the tracker doc calls one: a native dependency, or a
  `Blocked by:` / `Depends on:` line in the body.
- Read each candidate whole, comments included. A comment can block a ticket that no
  blocking edge does.
- You read roles and the open/closed flag. The implementer and the human set them.
- An empty frontier is a correct state, not a problem to solve — report why it is empty and
  stop. A frontier blocked behind a `ready-for-human` or `needs-triage` ticket is the
  human's to clear.

_Done when_ you can name one ticket, and say why nothing ahead of it is eligible.

## 2. Hand it to a cold agent

One agent per ticket, freshly spawned, never one carried over from an earlier ticket. Its
prompt is the ticket and one standing rule, in the same words every time:

    Invoke the mattpocock-skills:implement skill, and implement ticket NN with it.
    When you find a bug or a gap while you work, take the detour: fix it in this same
    session, in a commit of its own. Open a ticket in the `needs-triage` role instead
    only when the fix is large, or when only the maintainer can answer it.

`implement` is model-invocable, so the agent reaches it with the Skill tool. An agent that
reports it could not invoke the skill has hit a real fault: confirm the skill is installed,
rather than accepting work it improvised from a copy.

A **detour** is a find fixed where it was found, and it is the default: "out of scope" and
"a follow-up" are not on its two-item escape list.

Nothing ticket-specific joins those words. A summary, a hint, a "watch out for", a result
from an earlier ticket — each belongs in the ticket or the doc it is about: write it
there, then hand over the number. Every later implementer starts cold and hits the same
gap otherwise.

One implementer is in flight at a time, because two would share one git index, one
dependency tree and one test run in the same checkout. Inside its own ticket that agent is
free to fan out to subagents as widely as it likes.

_Done when_ exactly one agent is running, and its prompt names the ticket and the standing
rule and nothing else.

## 3. Answer to the standard of the best available evidence

When the agent asks a question, or reports an assumption it had to make, you answer it.

- Ground the answer in the repo's authorities, in this order: the ADRs, the domain docs,
  the spec, the repo's other documentation, the ticket itself.
- When those do not settle it, spawn a research agent and settle it before you reply. A
  guess from you is worse than a guess from the implementer, because yours carries
  authority.
- When the question exposes a gap in a ticket or a doc, write the answer into that file as
  well as replying.
- When the answer would change scope, an ADR, or an acceptance criterion, the human decides.
  Stop and ask.
- "Whichever you prefer" is not an answer.

_Done when_ the answer names the authority it rests on, and the file that had the gap now
holds it.

## 4. Verify the work yourself

The agent's summary is a **claim**. The diff is the **evidence**. Read one against the other
and look for the gap between them.

- `git log` and `git diff` for what actually changed.
- Every gate preflight found — **green**, not "green apart from". Run each one the diff
  touches, by the command preflight named.
- Every line of the ticket's acceptance criteria, matched to a named file, test or commit
  in the diff.
- Every commit no acceptance line claims. Each one is a detour or it is scope creep, and
  the diff says which. A detour is verified like the rest of the work.

_Done when_ each acceptance line carries a diff citation, every gate that applies has run,
every unclaimed commit is a named detour, and you can say where the summary claims more
than what landed.

## 5. Accept, or send it back naming the fix

Work reaches the bar. It never **moves the bar**. Send it back when the diff moved one:

- a check weakened rather than satisfied — a loosened rule, a suppression comment, a
  widened type, a skipped or deleted test, a relaxed assertion, a widened tolerance;
- an architecture, layout or golden test edited so a violation passes;
- the spec, an ADR, or the ticket's own acceptance criteria edited so the work fits them;
- an acceptance line quietly dropped, or met by weaker means than it asked for.

Whatever the agent doc states as non-negotiable is the rest of the list. Walk those rules
over the diff one at a time, as review criteria rather than as background.

"Out of scope", "can be a follow-up" and "the test is flaky" are reasons to send work back,
not reasons to accept it. When the ticket genuinely cannot be finished as written, move it
to the `needs-info` role with the reason in a comment and stop — a smaller version of the
ticket is not the ticket.

Send it back to the same agent, which still has its context, then re-verify from step 4.
After two send-backs on one ticket, the human takes it.

Accepted work reaches the remote. The implementer commits; you push, so that nothing you
have not accepted is published. Push the landing branch once the last send-back has landed,
and before you close the ticket.

_Done when_ the ticket is closed with a comment naming the commit that completed it, its
commits follow the repo's commit convention, and the landing branch is pushed — or you have
escalated.

## 6. Leave the record on the ticket, then take the next one cold

The ticket and the doc that owns it are where the record goes, and you keep no log of your
own. You are the only participant with state across tickets, so anything you keep to
yourself is something the next cold agent has to work out again:

- an answer you gave in step 3 → the ticket, and the doc whose gap the question exposed;
- what you sent back and why → a comment on the ticket;
- something you found that is not this ticket's → back to the implementer as a detour while
  it still has its context, and a new ticket in the `needs-triage` role once it does not.

_Done when_ nothing you learned on this ticket lives only in your own context. Then return
to step 1 with a new agent.

## Stop and ask the human when

- this repo has no `docs/agents/` configuration;
- the tree is dirty at the start of a ticket;
- the frontier is empty, or blocked behind a `ready-for-human` or `needs-triage` ticket;
- an answer would change scope, an ADR or an acceptance criterion;
- one ticket has been sent back twice;
- a gate fails for a reason outside the ticket's change;
- a push is rejected, because the landing branch moved under you.

In every one of these the ticket stays unimplemented by you. Your hands are for selecting,
answering, verifying, pushing and recording.
