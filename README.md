# klichota-skills

Backlog orchestration for repositories set up with the
[mattpocock engineering skills](https://github.com/mattpocock/skills) — that is, a repo carrying
`docs/agents/issue-tracker.md`, `triage-labels.md` and `domain.md`.

Two skills, both user-invoked. Neither loads into context until you type its name.

| Skill | When |
| --- | --- |
| `/orchestrate` | A repository's whole ready backlog, several implementers at once, each in its own worktree. Tickets are taken by the tracker's own ordering, whatever spec they belong to. |
| `/orchestrate-slice` | One spec's tickets, one implementer at a time, in the repository's own checkout. |

`/orchestrate` is not a superset in shape: it needs a ready backlog, a per-repo
`docs/agents/orchestration.md`, and a trunk it may merge to. `/orchestrate-slice` needs none of
those and stays the right tool for a slice worked start to finish.

The two are deliberately self-contained rather than sharing a file. A user-invoked skill carries no
description, so nothing can reach it but a human typing its name — including the other skill.

## Install

```sh
claude plugin marketplace add krzysztoflichota/claude-skills
claude plugin install klichota-skills@klichota
```

## Layout

```
.claude-plugin/
  marketplace.json    the marketplace this repo is
  plugin.json         the one plugin in it
skills/
  orchestrate/
    SKILL.md              the run: preflight, the frontier, a ticket's seven stages
    PREFLIGHT-CONFIG.md   the first-run interview, and docs/agents/orchestration.md's shape
  orchestrate-slice/
    SKILL.md
```
