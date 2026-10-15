---
name: truetask
description: Report work on a Truetask task so the app draws it as components (a truetask-ui block), ask so the person answers with one tap, and work a delegated task without wasting calls. Use when a Truetask MCP server is connected, when a kickoff hands you a card, or when you are about to post a comment, a done proposal or a chat answer in Truetask.
license: MIT
---

# Working Truetask

Truetask is a kanban workspace that agents work through an MCP server.
Its chat surfaces, task comments and done proposals draw components from data you write, and every task carries a contract you follow.

This skill covers the three things that decide whether a person can read your work: how to report, how to ask, and how to spend your calls.

The workspace serves its own current protocol at `<workspace-origin>/v1/agent-setup.md`.
Fetch it when you pick up a task, and believe it over this file wherever the two differ.

## Report with a block, not Markdown art

When an answer carries structure, put that structure in a fenced block whose info string is `truetask-ui`, holding one JSON object.
The app checks it against a closed catalog and draws it with its own components.
Emoji status lines, tree glyphs and Markdown tables are what this replaces.

Write at most one short sentence before the block, then the block, and never the same content again as Markdown.
A conversational answer, a short fact or a single sentence needs no block at all.

```truetask-ui
{"type": "report", "title": "Login form fixed", "status": "done", "summary": "The session now survives a refresh.", "blocks": [{"type": "files", "items": [{"path": "src/auth/session.js", "change": "modified", "added": 12, "removed": 3}]}, {"type": "steps", "items": [{"title": "Reproduced it on a fresh profile", "status": "done"}, {"title": "Tests", "status": "done", "text": "48 passing"}]}]}
```

Pick the shape by the job, not by habit:

- A finished piece of work, or any answer to "where does this stand": one `report`, with the rest nested in its `blocks`.
- A few headline numbers: `stats`. Up to 8.
- A value over time: a `chart`, line for a trend and bar for counts per period. Never stats tiles for a series.
- Rows that share the same fields: a `table`.
- Things that each have a state (tasks, checks, findings): a `list`, each item with a `status`.
- What happened, in order: `steps`.
- The one thing the reader must not miss: a `callout`.
- Facts as label and value: `properties`.
- The next moves you suggest: `actions`, at most 4, at most one `primary`.

Four rules decide whether a block draws at all:

1. Strict JSON. Double quotes, no comments, no trailing commas.
2. Only the catalog's types and fields. An unknown field is dropped silently, and a block that fails the check is shown to the person as raw source, which looks worse than plain prose.
3. No HTML, no images, no colours, no styling. You write data; the app owns the look.
4. Labels in the person's language, the same language you are answering in.

The full catalog, every field, every cap and a worked example per type: `references/ui-blocks.md`.
Read it before you write a type you have not written before.

## Ask so a person can tap

A question in prose stops the work and makes the person type.
Ask with your own question tool instead (`AskUserQuestion`, or whatever your runtime calls its request-for-input tool).
Truetask shows it as a card with your options on it, and the answer comes back inside the same turn.

- Give each question 2 to 4 options, your recommendation first, each with a one-line description of what choosing it means.
- Ask everything you can ask now in one go, up to 4 questions.
- Ask only what the task, the board and the linked notes cannot answer. Facts are your job.
- A question that depends on another open question waits for the next round.

One exception: on a task you were handed to work unattended, a blocking question goes through `cards_ask_question(card_id, question, options?, recommended?)`.
That parks the run as waiting for input and leaves the question on the task, where the person finds it later.
Your own question tool needs someone watching the chat right now.

## Spend your calls like they cost something

`cards_context(card_id)` first, always.
One call returns the card, its checklists, comments, work notes, definition of done, board conventions, git state, dependencies and a `contract`.
The contract is authoritative: follow it exactly.
Re-deriving any of that with separate calls is waste, and guessing instead of reading it is worse.

The server registers hundreds of tools and a task session is handed a short list.
A tool missing from your list is not a tool you may not use: `tools_find(query)` searches every tool by words and returns its schema, and `tools_call(name, arguments)` runs it with your own permissions.
So never tell a person something cannot be done because you cannot see the tool.

Keep results small and writes few:

- Pass `filter` and `per_page`, and ask for the one list or the one card you need rather than pulling a board and sorting it locally.
- Prefer a second targeted call over post-processing a big result.
- Prefer a `bulk_*` tool over a loop of single-item writes.
- Never paste a raw tool result into a comment. The person wants the finding, not the transcript.

Progress belongs on the task, not in your head:

- Tick checklist items as the work passes them, not in one burst at the end.
- Append to the work notes (`cards_context_note_update(card_id, content, mode="append")`) at each checkpoint and before you stop. They are what the next session reads.
- Keep the description short and human. Implementation detail, research and plans go in the work notes.
- Finish with `cards_propose_done(card_id, summary)` and answer the definition of done criterion by criterion in `metadata.verification`. A criterion you leave out reads as unanswered to the reviewer. Report `inconclusive` where you could not tell, instead of claiming a check you did not run.

The protocol around runs, steps, failures, dependencies and memory lives in `references/working-truetask.md` and, current and authoritative, in the workspace's own `agent-setup.md`.
