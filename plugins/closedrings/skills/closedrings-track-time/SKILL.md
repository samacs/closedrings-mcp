---
description: Track time on Closed Rings via the closedrings MCP server. Use this skill whenever the user asks about time tracking, mentions starting/stopping work on a project or task, asks "what did I do today / yesterday / this week", asks for a standup, mentions logging past work ("I just spent 2 hours on X"), or wants to edit or delete a recorded time block. Loads the canonical playbook for the Closed Rings tool surface — when to call which tool, how to detect the project from context, when to confirm before a write.
---

# closedrings:track-time

You are connected to Closed Rings (closedrings.sh) — a CLI-first time
tracker — via the `closedrings` MCP server in this plugin. You can
read what the user is working on, start and stop live time blocks,
log past work, and edit or delete blocks after the fact.

> **Sandbox awareness.** When `serverInfo.name` from the MCP
> handshake includes `(development)` or `(test)`, you're talking to a
> sandbox database on a developer's local machine. Writes don't
> affect anyone's real Closed Rings account. Lead with that framing
> if the user asks.

## Available tools

**Read tools** (free to call quietly, never narrate every call,
never confirm before reading):

| Tool | When to call |
|---|---|
| `current_context` | Start of any session, before any write. Avoids blindly auto-stopping an unrelated block. |
| `list_projects` | Before any write tool, when you don't already have a valid `project_slug`. |
| `list_tasks(project_slug?)` | When the user mentions a project but not a task, and the project has multiple open tasks. |
| `recent_time_blocks(limit?, since?, project_slug?)` | "What did I just do?" / "How long was my last X?" / fetching an `id` before `update_time_block` or `delete_time_block`. |
| `today_snapshot(period?)` | End-of-day / standup. Never reconstruct day summaries from chat memory. |

**Write tools** (always confirm in one line first, unless the user
has said "just do it" / "go ahead" / "no need to confirm" earlier
in this session):

| Tool | When to call |
|---|---|
| `start_time_block(project_slug, task_slug?, message?)` | "I'm about to work on X" / "starting X now". Auto-stops any running block. |
| `stop_time_block(message?)` | "I'm done" / "stop the timer" / "I just finished". Optional `message:` replaces the original note. |
| `log_time_block(project_slug, task_slug?, started_at, ended_at, message?)` | Past work: "I worked from 2 to 4pm on X" / "I just spent 45 minutes on Y" / "this morning I did Z for an hour". Does not disturb anything running. |
| `update_time_block(id, …)` | Fixing a wrong timestamp, refining a message, reassigning to a different task. Always get the `id` from `recent_time_blocks` first. |
| `delete_time_block(id)` | The block shouldn't exist at all. For "I forgot to stop", use `update` instead. |

## Trigger-phrase patterns

These map natural-language inputs to the right tool. Examples are
illustrative — don't pattern-match too rigidly, but the intents
hold.

- *"What am I working on?"*, *"What's running?"* → `current_context`
- *"What projects do I have?"*, *"List my projects"* → `list_projects`
- *"What did I do today?"*, *"Generate my standup"*, *"How much time
  did I track today?"* → `today_snapshot(period: "today")`
- *"What did I do yesterday?"* → `today_snapshot(period: "yesterday")`
- *"What did I work on this week?"* → `today_snapshot(period: "week")`
- *"What was my last block?"*, *"Show me my recent time"* →
  `recent_time_blocks(limit: 5)`

Write triggers — confirm first:

- *"Start tracking the refund flow"*, *"I'm starting on auth"*,
  *"Let's work on X"* → `start_time_block`
- *"Stop the timer"*, *"I'm done"*, *"OK that's it"* →
  `stop_time_block`. If the user adds a note ("I'm done — finished
  the refactor"), pass it as `message:`.
- *"I just spent 45 minutes refactoring auth"*, *"Log 2 hours of
  meetings to Acme"*, *"This morning I worked on X from 9 to 11"* →
  `log_time_block`. Compute `started_at` and `ended_at` in the
  user's timezone (or UTC if you're unsure — the server will accept
  any ISO-8601 offset).
- *"Move that block to a different task"*, *"Actually the end time
  was 3:30, not 3:45"* → `update_time_block`. Call
  `recent_time_blocks` first to find the right `id`.
- *"Delete that block"*, *"Cancel the last one"* → confirm
  destructiveness explicitly: "I'll permanently delete the block
  for X (id=N). OK?" Then `delete_time_block`.

## Behavioral rules

**1. Confirm writes.** Restate what you're about to do in one short
line and wait for the user's go-ahead, unless they've said
"just do it" / "go ahead" / "no need to confirm" earlier in this
session.

> *Good:* "Logging 45m to closedrings.sh / refund-flow with the note
> 'fixed the boundary condition'. OK?"
>
> *Bad:* (just calls the tool with no preamble)
>
> *Bad:* "I'm going to call `log_time_block` with these arguments…"
> (don't expose tool plumbing; describe the *outcome*)

**2. Detect the project from context.** If the user's working
directory or active editor folder ends in a name matching a project's
`slug` or a fuzzy match on `name`, prefer that project without
asking. Otherwise call `list_projects` and propose the most likely
match — don't dump the whole list at the user.

**3. Don't invent projects or tasks.** Projects and tasks are
curated by the user in the dashboard at
[closedrings.sh](https://closedrings.sh). If the user mentions one
that doesn't appear in `list_projects` / `list_tasks`, point them to
the dashboard rather than creating it on the fly. The MCP transport
deliberately does not expose `create_project` or `create_task` to
keep this boundary clean.

**4. Prefer `log_time_block` for past work.** *"I just spent an hour
on X"* is past work — they're reporting after the fact. Don't
`start_time_block` then `stop_time_block` to fake it; use
`log_time_block` with explicit timestamps so the dashboard shows the
real interval.

**5. Use ids you got back.** `update_time_block` and
`delete_time_block` take a block `id`. Don't infer "the last block"
— call `recent_time_blocks(limit: 5)` to find the actual id and
echo back which block you're touching ("editing the 14:00–14:30
block on refund-flow") before the user confirms.

**6. Be quiet about reads.** `current_context` and `list_*` are
cheap. Don't announce every call. Surface the result only when the
user asks or when a write tool depends on it.

**7. Boomerang awareness.** If `current_context` shows a running
block and the user pivots topics, ask whether to stop or keep
running — don't silently auto-stop. The server records every
task-change as a `ContextSwitch`, and pivot-then-return within the
boomerang window is flagged in the dashboard, so silent auto-stops
pollute the user's focus metrics.

**8. Timezone discipline.** Server stores `started_at` /
`ended_at` as ISO-8601 with offset. If the user says "2pm", resolve
it against their timezone (which `current_context` and
`today_snapshot` carry implicitly) — don't UTC-shift their intent.
When in doubt, ask: "2pm in your local time?"

**9. Avoid overlap.** The server's model-level overlap validation
will reject a `log_time_block` that intersects an existing block.
If you get `"overlaps with an existing time block"`, call
`recent_time_blocks` to find the conflict and propose either
adjusting the interval or `update_time_block` on the existing one
instead.

**10. Surface env clearly.** If `serverInfo.name` includes
`(development)` or `(test)`, mention it the first time the user
asks the agent to write — "I'll log this to the local development
instance" — so they don't expect it to show up in their real
account.

## Recipes

**End-of-day standup:**

```
> "What did I do today? Format for Slack."

1. Call today_snapshot(period: "today")
2. Render rows as bullets:
   • payments-v3 / refund-flow — 2h 15m
     - Fixed the negative-amount edge case
     - PR #842 review with Sam
   • compliance / soc-2-evidence-collection — 45m
     - Drafted the audit-log retention policy
3. Total: 3h 00m, 2 projects, 3 sessions
```

**Retroactive logging from a stand-up summary:**

```
> "Log: 9-10:30 standup, 10:30-12 refactoring auth on closedrings.sh,
   13-15 client work on Acme."

For each block:
1. Resolve project_slug and task_slug from the context
2. Confirm: "I'll log three blocks: 1h 30m standup, 1h 30m
   auth refactor on closedrings.sh, 2h Acme client work. OK?"
3. After "OK", call log_time_block three times. Report the resulting
   block ids.
```

**Fix a forgotten stop:**

```
> "I forgot to stop the block from this morning."

1. current_context → confirms it's still running
2. recent_time_blocks(limit: 1) → optional, for context
3. Ask: "I'll stop it now. What time did you actually finish?"
4. If user gives a time, update_time_block with the right ended_at.
   If they say "just stop it now", call stop_time_block.
```

## Source

This skill mirrors the canonical playbook the Closed Rings server
ships over `serverInfo.instructions` on every MCP `initialize`
handshake (see `app/api/mcp/playbook.rb` in the `closedrings.sh`
repo). When the server-side copy changes, this file is regenerated
to match. The expanded version of this same content with extra
prose lives at `skill/SKILL.md` in this repo for users who want to
paste it into clients without plugin support.
