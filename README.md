# Truetask agent skills

Skills for coding agents that work in [Truetask](https://truetask.io), the
project management tool that hands tasks to agents.

## Install

```bash
npx skills add truetask-io/skills            # this project
npx skills add truetask-io/skills -g         # every project on this computer
```

Claude Code, Codex, Cursor and the rest of the agents the
[skills CLI](https://github.com/vercel-labs/skills) knows read the same
`SKILL.md`.

Your own workspace serves the same skill, matched to the version it renders:

```bash
npx skills add https://your-truetask-host
```

## What is in here

- `skills/truetask` teaches an agent how to report work so Truetask draws it
  as components, how to ask so a person answers with one tap, and how to work
  a delegated task without wasting calls.

## Changes

This repository is generated. The source lives in the Truetask repository and
is published on release, so a commit made here is overwritten by the next one.
Open an issue instead, or write to the team through
[truetask.io](https://truetask.io).
