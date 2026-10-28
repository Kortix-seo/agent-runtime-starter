# agent-runtime-starter

An open-source agent runtime configuration you can copy: two agents, their tool
rules, and the branch-plus-change-request flow that lands their work, read from
the runtime that runs them, [Kortix, the open-source AI Operating System](https://kortix.com).

Most runtime documentation describes the machine. This repository shows the
three things a runtime actually decides for a run, in files you can read in one
sitting:

| What it decides | Where it lives here |
| --- | --- |
| Which tools run, ask, or are refused | `permission` in `agents/triage.md` and `agents/release-notes.md` |
| What each agent may reach | `connectors`, `secrets`, `skills` and `kortix_permissions` in `kortix.yaml` |
| How the work reaches main | the session branch and the change request described below |

## The run, end to end

A Kortix session gets its own computer: an isolated Linux machine that clones
the repository, cuts a branch named after the session, and starts the harness
there. The agent commits to that branch. Session work reaches `main` through a
change request that a person reviews, and merge is default-deny for agents. A
merge does not stop the session.

So the runtime's record is the git history itself: which run cut which branch,
what it changed, and who approved the diff.

## Tool rules

`agents/triage.md` is the whole policy for the triage agent:

```yaml
permission:
  read: allow
  edit:
    "*": allow
    "kortix.yaml": ask
    "agents/*": ask
  bash:
    "*": ask
    "git status": allow
    "git diff *": allow
    "git log *": allow
    "rm -rf *": deny
  webfetch: ask
  websearch: ask
```

Each rule resolves to one of three actions: `allow` runs the call, `ask` waits
for a person, `deny` refuses it. The most specific pattern wins, so `git diff *`
outranks the `"*": ask` fallback while `rm -rf *` is refused outright. A tool
with no rule is allowed, which is why the fallback is written out.

## Grants

Grants are deny-by-default. `kortix.yaml` gives the triage agent the helpdesk
and GitHub connectors, one secret, one skill, and two Kortix permissions
(`project.session.read`, `project.review.submit`). The release-notes agent gets
the GitHub connector and nothing writable outside `releases/`.

Two keys behave differently from the rest, and both are deliberate here: an
omitted `tools` key grants every tool, and an omitted `skills` leaves skill
access to the agent file's own rule. The `skills` list in `kortix.yaml` is what
closes skill loading down to `kortix-cli`.

## Run it

The same manifest runs in the managed cloud, in your VPC, or on your own
network. Self-host is free: one Docker Compose stack, the same images the
managed cloud runs, your database and your files on disk you control.

```bash
kortix init                  # write the project's identity and link the tools
kortix self-host start       # install, configure and update the stack locally
kortix hosts use selfhost    # point this checkout at it
```

Then open a session and ask it to triage the queue. Every reply it drafts
arrives as a change request on its own branch, and every rule it ran under is
one of the lines in `agents/triage.md`.

## What to read next

- [Agents and permissions](https://kortix.com/docs/project/agents): every field
  in the agent file, and how the rules resolve.
- [Sessions](https://kortix.com/docs/work/sessions): the sandbox, the branch,
  the change request.
- [The manifest](https://kortix.com/docs/project/manifest): every key in
  `kortix.yaml`.

## Where this comes from

The files here follow the runtime Kortix documents on its own site, checked
October 2026. Fork the repository, change a rule, and watch the change land
through the same review a person would get.
