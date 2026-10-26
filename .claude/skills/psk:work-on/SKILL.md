---
name: psk:work-on
description: Work on a task from tasks.md. Use when the user says "work on <task>", "continue <task>", "resume <task>", "close <task>", "reopen <task>", "add <file> to <task>" or "task status". Keeps a per-task memory folder in memory/ (memory.md plus a docs/ folder for external documents), works in a separate context by default, and archives the folder when the task is closed.
argument-hint: "<task> | close <task> | reopen <task> | add <file|url> to <task> | status  (add 'here' to work in the current context)"
---

# Work On a Task

Tasks live in `tasks.md` in this folder. Every task that is being worked on has
one **task folder** in `memory/`. Closed tasks keep their folder in `memory/archive/`.

```
memory/
  <slug>/
    memory.md      # the task's memory: goal, context, progress log, open questions
    docs/          # external documents for this task (mails, PDFs, exports, notes)
  archive/
    <slug>/        # closed tasks, same layout
```

The user may drop files into `docs/` at any time, by hand or with the **add**
command. Everything in `docs/` is input for the task and must be read when the
task is worked on.

All paths below are relative to this folder (the one containing `tasks.md`).
Today's date is in the system prompt. Use it for every date you write.

## Who runs this

- **Main session** (user typed `/psk:work-on` or said "work on X"): run *Resolve*,
  *Task folder*, then *Dispatch*.
- **task-worker subagent** (this skill is preloaded into you): skip *Dispatch*.
  Run *Work* on the task folder you were given, then report back as described
  in *Report*. Never spawn another task-worker.

## Commands

Decide the command from the user's words:

| Command | Trigger | What to do |
|---------|---------|------------|
| **work** (default) | `work on X`, `continue X`, `resume X`, `/psk:work-on X` | Resolve → Task folder → Dispatch |
| **close** | `close X`, `X is done`, `mark X as done` | Close procedure |
| **reopen** | `reopen X` | Reopen procedure |
| **add** | `add <file> to X`, `attach <file> to X`, `put this in X` | Add document |
| **status** | `/psk:work-on status`, `task status`, `what am I working on` | Status |

Context modifiers (only for **work**):

- Default: the task runs in **its own context** (the task-worker subagent).
- If the user says `here`, `inline`, `in this context`, `in the main context`
  or `don't delegate`: run *Work* **in the current context** instead.

## Resolve

1. Read `tasks.md`.
2. Match the user's words against the task lines. Matching is case-insensitive
   and partial. Ignore the ` — added …` / ` — closed …` suffixes.
3. Exactly one open match: use it.
4. Several matches: ask with `AskUserQuestion`, listing the candidates.
5. Only a closed match (`[x]`): say so and offer **reopen**.
6. No match: ask with `AskUserQuestion` whether to add it as a new task. On yes,
   append `- [ ] <task> — added <today>` to the `## To Do` section, then continue.

The **task text** is the line without the checkbox and without the ` — added …`
suffix. The **slug** is the task text in lower kebab-case, ASCII only
(`ä→ae`, `ö→oe`, `ü→ue`, `ß→ss`), at most 6 words, e.g.
`send-a-reminder-to-gion`.

## Task folder

Path: `memory/<slug>/`. Memory file: `memory/<slug>/memory.md`.
Documents: `memory/<slug>/docs/`.

- `memory/<slug>/memory.md` exists: this is a **resume**. Read it; do not
  overwrite it. Append a line to `## Progress log`: `- <today>: resumed`.
  Create `docs/` if it is missing.
- `memory/archive/<slug>/` exists: the task was closed. Say so and offer **reopen**.
- Old layout `memory/<slug>.md` exists (a single file, no folder): migrate it,
  then treat it as a resume:
  ```bash
  mkdir -p memory/<slug>/docs && mv memory/<slug>.md memory/<slug>/memory.md
  ```
- Nothing exists: create the folder, then the memory file with the Write tool:
  ```bash
  mkdir -p memory/<slug>/docs
  ```

```markdown
---
task: <task text>
slug: <slug>
status: in-progress
created: <today>
closed:
context: own | shared
---

# <task text>

## Goal
<one or two sentences: what "done" means for this task>

## Context
<facts gathered so far: people, links, files, decisions, constraints>

## Documents
<one line per file in docs/: `- <file name> (added <date>): <one-line summary>`, or "none">

## Progress log
- <today>: started

## Open questions
- <things only the user can answer, or empty>

## Outcome
<filled in when the task is closed>
```

`context` is `own` when the task is dispatched to the subagent, `shared` when it
runs in the current context.

Do not touch `tasks.md` at this point. The memory file is the source of truth
for progress; `tasks.md` only changes on close and reopen.

## Add document

Run when the user wants to attach something to a task. Works for open tasks;
for a closed task, offer **reopen** first.

1. Resolve the task. If the task folder does not exist yet, create it as in
   *Task folder* (folder, `docs/`, memory file). Do not dispatch.
2. Put the document into `memory/<slug>/docs/`:
   - **Local file path:** copy it, keep the original file name:
     `cp "<path>" memory/<slug>/docs/`. If the name is taken, add `-<today>`
     before the extension.
   - **URL:** fetch the page (WebFetch, or the Confluence/Jira/GitHub tool that
     fits the URL) and save the content as Markdown with the Write tool. File
     name: a short kebab-case title plus `.md`. First line: `Source: <url>`.
   - **Pasted text** (a mail, a chat excerpt, notes): save it verbatim as
     Markdown with the Write tool. File name: what the user calls it, or a short
     kebab-case description plus `.md`. First line: `Source: pasted by user on <today>`.
   - **Several items:** repeat for each.
3. In `memory.md`, add one line per file to `## Documents`:
   `- <file name> (added <today>): <one-line summary of what it contains>`.
   Add the section if it is missing. Replace a `none` placeholder.
4. If the document answers something listed under `## Open questions`, say so
   in the reply, but leave resolving it to the next **work** run.
5. Append `- <today>: added <file name> to docs/` to `## Progress log`.
6. Confirm: task name, file path(s), and that the next **work** run will pick
   them up.

Tell the user, when it fits, that they can also copy files into
`memory/<slug>/docs/` by hand; the next **work** run reads them.

## Dispatch

**Own context (default):** call the `Agent` tool with `subagent_type: task-worker`
and this prompt (fill in the values):

```
Work on the task "<task text>".
Task folder: <absolute path to memory/<slug>/>
Memory file: <absolute path to memory/<slug>/memory.md>
Documents: <absolute path to memory/<slug>/docs/>
Task list: <absolute path to tasks.md>
Working folder: <absolute path of this folder>
<everything else the user said about the task, verbatim>

Follow the Work and Report sections of the psk:work-on skill. Do not close the task
yourself unless the user explicitly asked for that in the text above.
```

When the agent returns, relay its report to the user. If the report says the task
is complete and the user asked to close it, run the *Close procedure*; otherwise
tell the user they can say `close <task>` when ready.

**Current context:** run *Work* yourself, then continue as after a report.

## Work

Rules for doing the task, whoever runs it:

1. Read the memory file first. The memory file is your context from earlier
   sessions; trust it over guesses.
2. Read the documents. List `docs/` and compare it with `## Documents` in the
   memory file:
   - Files not yet listed are **new input from the user**. Read each one (PDFs
     and images with the Read tool, text as is). Add a line for it to
     `## Documents` with today's date and a one-line summary, and pull the facts
     that matter into `## Context`.
   - Files already listed: re-read the ones the current step needs. The
     summary in `## Documents` tells you which.
   - Files listed but missing from `docs/`: mark the line `(removed)`.
   Never modify or delete files in `docs/`; they belong to the user. Files you
   produce yourself (drafts, exports, scripts) go into the task folder next to
   `memory.md`, not into `docs/`, and get a line in `## Context`.
3. Do the task. Use any tool that helps (shell, web, Jira, Confluence, GitHub,
   files in the working folder). Follow all project rules in effect.
4. Keep the memory file current **while you work**, not only at the end. Use the
   Edit tool. After every meaningful step:
   - add a dated line to `## Progress log`
   - add new facts to `## Context`
   - add anything blocked on the user to `## Open questions`
5. Things you cannot do yourself (send an email, talk to a person, decide on
   someone's behalf): prepare them fully (draft text, list of points, links) and
   put the draft into the memory file under `## Context`. Never pretend you did
   them.
6. Do not edit `tasks.md` while working. Only the close and reopen procedures
   touch it.
7. If working in the current context and a question needs the user, ask it with
   `AskUserQuestion`. In the subagent, write it to `## Open questions` and
   continue with what does not depend on the answer. When a document would
   unblock a question, say which one under `## Open questions`, e.g.
   `- needs the mail thread with X; add it to docs/`.

## Report

End with a short report for the user (the subagent returns it as its final
message):

- **Status:** `done` (nothing left for the user), `needs you` (lists what the
  user has to do), or `blocked` (why)
- **What was done:** 2–5 bullets
- **Documents read:** new files from `docs/` picked up this run, or "none"
- **Open questions:** copied from the memory file, or "none"
- **Task folder:** its path

## Close procedure

Run when the user asks to close a task, or after a `done` report when the user
asked for the task to be closed.

1. Resolve the task (see *Resolve*). If it is already `[x]`, tell the user and
   stop.
2. Finish the memory file (`memory/<slug>/memory.md`):
   - set `status: closed` and `closed: <today>` in the front matter
   - write `## Outcome`: 1–3 sentences on the result
   - append `- <today>: closed` to `## Progress log`
   If no task folder exists (task closed without ever being worked on), create
   the folder and a memory file from the template with just the front matter,
   `## Progress log` (`- <today>: closed without work`) and `## Outcome`.
   If only the old layout `memory/<slug>.md` exists, migrate it first (see
   *Task folder*).
3. Move the whole task folder, documents included, to the archive:
   ```bash
   mv memory/<slug> memory/archive/<slug>
   ```
   If `memory/archive/<slug>/` already exists, use `memory/archive/<slug>-<today>`.
4. Update `tasks.md` with the Edit tool:
   - change `- [ ]` to `- [x]` on the task line
   - append ` — closed <today>` to the line
   - move the line from `## To Do` to the end of `## Done`
5. Confirm to the user: task name, archive path, and the remaining open tasks.

## Reopen procedure

1. Find the task line in `tasks.md` (either section) and the task folder in
   `memory/archive/`.
2. Move the task folder back:
   ```bash
   mv memory/archive/<slug> memory/<slug>
   ```
   If the archive holds an old-layout file `memory/archive/<slug>.md` instead,
   move it to `memory/<slug>/memory.md` and create `memory/<slug>/docs/`.
   Set `status: in-progress`, clear `closed:`, append `- <today>: reopened` to
   `## Progress log`.
3. In `tasks.md`: change `[x]` to `[ ]`, remove the ` — closed …` suffix, move
   the line back to `## To Do`.
4. Continue with **work** if the user asked to work on it; otherwise confirm.

## Status

List, in this order:

1. Tasks with a folder in `memory/` (in progress), each with the last
   `## Progress log` line, the number of files in `docs/` (and how many of them
   are not yet listed in `## Documents`), and any open questions.
2. Open tasks in `tasks.md` without a task folder (not started).
3. Count of archived tasks in `memory/archive/`.
