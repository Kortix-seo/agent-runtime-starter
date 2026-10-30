# agent-runtime-starter

An open-source agent runtime configuration you can copy: two agents, their tool
rules, and the branch-plus-change-request flow that lands their work, read from
the runtime that runs them, [Kortix, the open-source CompanyOS](https://kortix.com).

Most runtime documentation describes the machine. This repository shows the
three things a runtime actually decides for a run, in files you can read in one
sitting:

| What it decides | Where it lives here |
| --- | --- |
| Which tools run, ask, or are refused | `permission` in `kortix.yaml` (`agents.triage` / `agents.release-notes`) |
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

`agents/triage.md` holds the prompt. The policy it runs under is in
`kortix.yaml`, under `agents.triage`, beside that prompt in the same reviewed
repository:

```yaml
agents:
  triage:
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

Grants are deny-by-default. `kortix.yaml` declares the `helpdesk` connector in
`connectors` and the triage agent enables it by name; the release-notes agent
gets no connector, because it works through git. `HELPDESK_API_KEY` is declared
under `env`, so the run is handed a name and never a value, and the triage agent
is the one agent that is granted it. Both agents hold the `kortix-cli` skill and
two Kortix permissions (`project.session.read`, `project.review.submit`), and
both set `workspace: { project_repository: true }`, so each session gets a
checkout of the repository on its own branch. No GitHub connector is granted.

Three keys are open by default and everything else is closed: an omitted `tools`
grants every tool, an omitted `permission` is `allow`, and `codemode` is on.
Every other grant, `skills` included, reaches no agent until the agent names it.
The `skills` list in `kortix.yaml` is what closes skill loading down to
`kortix-cli`.

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
arrives as a change request on its own branch, and every rule it ran under is a
line in `kortix.yaml`, not one of the lines in `agents/triage.md`.

## What to read next

- [Agents](https://kortix.com/docs/guide#agents): the instructions an agent
  runs on, the model it thinks with, and the permissions it works under.
- [Computers](https://kortix.com/docs/guide#computers): the cloud machine every
  session gets, and the machine you can connect instead.
- [The manifest](https://kortix.com/docs/developers#kortixyaml-reference):
  every key in `kortix.yaml`.

## Before you copy it, read the field

The rules above only mean something against the other runtimes a team could
buy, so the same comparison is written out for a reader who has to choose. A
[plain definition of an agent runtime](https://agentruntimesguide.com/what-is-an-agent-runtime)
is the place to start if the words runtime, harness and framework are still
tangled.

Two follow-ups cover the decisions this repository cannot: how a
[runtime differs from a harness](https://agentruntimesguide.com/agent-runtime-vs-agent-harness)
when both are sold as the same thing, and what a
[session-based architecture](https://agentruntimesguide.com/agent-runtime-architecture)
commits you to once an agent writes back to a repository.

If the products themselves are the part you still have to choose between, [the
open-source agent runtimes that keep coming back in search](https://agentruntimesguide.com/open-source-agent-runtimes)
put each one beside what it leaves you to run, usually the machine, the record
and the person who approves the change.

The rest is operational. What it costs to leave a machine parked is on the
[runtime cost comparison](https://agentruntimesguide.com/agent-runtime-cost),
the install on your own hardware is in the
[self-hosted runtime notes](https://agentruntimesguide.com/self-hosted-agent-runtime),
and the [runtime comparison](https://agentruntimesguide.com/best-agent-runtimes)
puts AgentCore, Cloudflare Agents, E2B, Modal, Daytona and LangSmith beside
each other. The [field notes on the whole cluster](https://agentruntimesguide.com/)
collect the rest.

## Where this comes from

The files here follow the runtime Kortix documents on its own site, checked
October 2026. Fork the repository, change a rule, and watch the change land
through the same review a person would get.
