# Project instructions

This is a project for Kortix, the open-source CompanyOS.

This repository is a Kortix project: the manifest is `kortix.yaml`, each agent's
prompt is the `.md` file under `agents/`, and every session runs in its own
sandbox on its own branch.

- A permission change is a diff in `kortix.yaml`: propose it in a change
  request, never widen `agents.<name>.permission` inside a run.
- Secrets are referenced by name. Never write a secret value into a file.
- Work lands on `main` through a change request a person reviews.
- Merge is default-deny for agents: a run proposes, a person merges.
