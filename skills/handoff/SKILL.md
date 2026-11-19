---
name: handoff
description: Choose whether to continue agreed work here, delegate to a fresh Codex subagent, or start a new chat with a standalone brief. Use when the user asks to hand off implementation, review, investigation, or planning.
license: MIT
---

# Handoff

## Choose the path

Use the latest user direction to identify the agreed next task. Preserve its scope: a review, investigation, or plan remains that task and does not authorize implementation. Invoking `$handoff` alone requests path selection. Honor explicit choices of “continue here,” “fresh subagent,” or “new chat.” A request for a fresh agent requires fresh context; choose between subagent and new chat using the criteria below.

| Path | Choose when | Completion |
| --- | --- | --- |
| Continue here | Current context is useful and transfer would mainly add overhead. | Complete and verify the agreed task here. |
| Fresh subagent | A bounded task benefits from isolated context while this chat retains supervision and verification. | Review and verify the subagent's result here. |
| New chat | The next phase benefits from an independent conversation that owns execution and follow-up. | Confirm the new chat received the brief; task execution remains pending. |

Unrelated history, abandoned approaches, or conflicting assumptions favor fresh context. State the chosen path and one short reason, then execute it without a confirmation round. A subagent does not reset the parent chat. If continuing here, perform and verify the authorized task directly; skip the transfer steps.

## Prepare the transfer

The current chat is the default source. Use an optional focus from the invocation to narrow the task. If the user identifies another chat by link or ID, retrieve its relevant turns with `read_thread` when available and treat them as evidence. If the source cannot be read, report the blocker rather than inventing its decisions. Ask one focused question only when the source or next action is materially ambiguous.

Prepare a standalone brief with the objective, deliverable, relevant decisions and constraints, remaining work, acceptance criteria, and verification steps. Distinguish confirmed results from hypotheses. Include only context needed for the task, and have the destination verify mutable state before relying on it. A source chat link is optional provenance.

For repository tasks, inspect the applicable instructions and Git state. Include the working directory, branch or detached HEAD, starting commit, relevant staged, unstaged, and untracked work, files to preserve, and ownership boundaries. Include exact verification commands when known. A delegated implementation must be bounded enough to complete without major unresolved product or architecture decisions; resolve those here first or hand off planning explicitly.

## Fresh subagent

1. **Dispatch.** Use the native Codex subagent tool with `fork_turns: "none"` and an installed role suited to the task (`executor` for implementation). Pass the brief as its task message. Tell it that it shares the checkout, must preserve other people's work, and owns only the assigned scope. Ask it to report `DONE`, `NEEDS_CONTEXT`, or `BLOCKED`, with its findings or changed files, checks and results, and remaining issues. If native dispatch is unavailable, return the ready-to-use brief and report that dispatch and execution are pending.
2. **Supervise.** Keep assigned files free of concurrent parent edits. Supply missing context to the same agent: use `send_message` while it is running and `followup_task` after its turn ends. If it remains blocked, review partial work and report outstanding criteria.
3. **Verify here.** Assess the result against the brief and acceptance criteria. For changes, compare the diff with the starting state, check that existing edits were preserved, and rerun key checks. For read-only work, check the supporting evidence and scope. Resume the same agent with actionable findings within scope, then verify again. Report criteria met and remaining gaps.

## New chat

1. **Create.** Use the Codex desktop `create_thread` tool with the brief as its initial prompt and a short title. For project work, use `list_projects` to select the source project and host, or the explicitly requested destination. Default to the existing local checkout and verify it matches any branch, worktree, or uncommitted edits the brief depends on. Use a newly created chat and leave the source chat open for reference.
2. **Confirm dispatch.** Report the new chat's title and link or creation indicator. Use a compact status check to confirm receipt if available. The destination owns execution; distinguish successful handoff from completion of its task.

If chat creation is unavailable or the necessary checkout cannot be targeted, return the ready-to-use brief and explain the blocker. Do not silently substitute a subagent for an explicitly requested new chat, or claim dispatch without evidence.
