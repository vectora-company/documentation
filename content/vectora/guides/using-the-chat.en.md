---
title: Using the Chat
weight: 1
---

## Sending messages

The chat input accepts text, drag-and-dropped files, pasted clipboard images, and file `@mentions`
to bring specific content into context without having to describe it.

## Model selector

Only shows providers with a configured API key — Google Gemini, OpenAI, Anthropic, Cohere, or local
Ollama. Switching models mid-conversation doesn't lose history.

## Permission modes

Control how automatically the agent acts before asking for your approval:

| Mode             | Approval-gate behavior                                                                          |
| ---------------- | ----------------------------------------------------------------------------------------------- |
| **Always ask**   | pauses for tools classified as sensitive or destructive                                         |
| **Accept edits** | allows file writes and edits without pausing; other protected actions remain subject to HITL    |
| **Autonomous**   | does not pause for ordinary human approval; policy and security blocks remain active            |
| **Plan**         | pauses for the first protected action in a turn; each metered media call needs its own approval |
| **Bypass**       | disables ordinary HITL pauses; independent guardrails still apply                               |

These are permission modes, not trust levels for the model. Some operations, such as desktop
control, can still require confirmation in every mode. In **Plan**, an approval already granted in
the same turn lets the plan continue; it is not a permanent authorization. See
[Plan Mode, AgentRun, and decisions](../plan-mode-and-agentrun) for the `/plan` prefix and how Plan
Mode differs from the workbench Plan tab.

With confirmed native worker isolation, the HITL gate exempts `terminal`, `file_write` and
`file_edit`, including the first action in Plan mode. Other policy and tool restrictions still
apply.

A pre-approval is a separate, explicit grant. It can be consumed once for one tool and its exact
arguments, within the same workspace and `run_id`, and expires after a limited time. Irreversible
tools cannot use this grant.

In **Autonomous** mode, a Decision Model participates only when enabled, selected, ready, and
allowed for the action's surface. Without it, no additional automated decision is assumed: static
policy still controls what can run. If an evaluation that has started fails, times out, or requires
human review, the action is not treated as approved.

See [Orchestrator & Subagents](../../concepts/sub-agents) for the HITL flow and
[Decision models](../decision-models) to configure providers and surfaces.

## Messages while the agent is responding

You can keep typing while the agent works. A message sent during a response goes into a queue shown
above the input, and each item has a state:

| State        | Meaning                                                            |
| ------------ | ------------------------------------------------------------------ |
| **Queued**   | Waiting; it is sent when the current response finishes.            |
| **Sending**  | The request is on its way to the server.                           |
| **Not sent** | The request did not reach the server. The item stays in the queue. |

An item leaves the queue only after the server confirms it. If sending fails, use **Try again** to
send it again: the retry reuses the same message identifier, so the message is never duplicated even
if the first attempt did arrive. A failed item holds the ones behind it, which keeps the order;
retry it or remove it to release the rest. You can also send a waiting item right away, which stops
the current response first. The queue has a maximum height, so it does not cover the conversation.

Attachments you have not sent are kept per conversation: switching to another conversation and
coming back restores them along with the text.

## Reliable response stream

Each event of a response carries a sequence number. If the connection repeats or reorders events,
the chat ignores the extras, and after the final event it accepts nothing more, so text is never
shown twice. The final event says why the response ended, and the message shows an indicator when it
was interrupted or when it reached the step limit. Once the final response arrives, the input is
free immediately, with no waiting time before the next message.

## Orchestrator thinking

The "thinking" block shows the orchestrator's decision before acting: answer directly or delegate to
`coder`/`search`, and why. This is real transparency, not just a spinner — you can see the
reasoning, not just wait for the result.

## Memory across sessions

A "🧠 N memories loaded" badge appears when the agent uses persistent memories from previous
conversations in that response. Manage persistent facts in **Settings → Preferences → Memory**; the
workbench Memory tab shows retrieval activity and context used.

## RAG citations

Answers based on indexed content bring clickable `[1] [2]` citations — click to see the original
excerpt and source.

## Multi-user access

Multi-user access depends on server configuration and connected services. See the
[authentication](../../security/authentication) guide.

## See also

- [Using the workbench](../using-the-workbench) — the side panel that accompanies chat
- [Using settings](../using-settings) — where the default model, language, and theme live
