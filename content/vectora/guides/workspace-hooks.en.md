---
title: Workspace Hooks
weight: 11
---

Hooks are deterministic scripts that run at fixed points of the agent loop. Use them to enforce a
rule the model cannot be trusted to follow on its own: block a dangerous command, check a tool's
result, or reject a final answer that breaks a convention. Hooks are available from version 0.1.26.

## Defining a hook

Hooks live in the workspace's `vectora.toml`, one `[[hooks.rules]]` table per rule:

```toml
[[hooks.rules]]
id = "block-rm"
event = "pre_tool"                       # "pre_tool" | "post_tool" | "pre_response"
tools = ["run_command"]                  # fnmatch patterns; empty matches every tool
command = ["python", ".vectora/hooks/check.py"]   # argument list, no shell
mode = "blocking"                        # "blocking" | "informative"
timeout_s = 10                           # 0.5 to 60 seconds
```

| Field       | Meaning                                                                                                  |
| ----------- | -------------------------------------------------------------------------------------------------------- |
| `id`        | Lowercase letters, digits, `-` and `_`, up to 64 characters.                                             |
| `event`     | `pre_tool` runs before a tool, `post_tool` after it, `pre_response` before the final answer is accepted. |
| `tools`     | Tool-name patterns. Ignored in `pre_response`, which has no tool.                                        |
| `command`   | The program and its arguments as a list. Vectora never passes it through a shell.                        |
| `mode`      | `blocking` can stop the action; `informative` only records the result.                                   |
| `timeout_s` | Time limit for the script.                                                                               |

A rule with an invalid field is discarded with a warning, so a typo does not break the rest of the
configuration.

## What the script receives and returns

The script reads a JSON object from standard input: `version`, `event`, `hook_id`, `run_id`,
`thread_id`, and then `tool` (`name` and `args`) plus `result` for `post_tool`, or `response` for
`pre_response`. Values of sensitive keys (tokens, passwords, API keys) are replaced before the
script sees them, and long results are truncated. The environment is minimal: a few base variables
plus `VECTORA_HOOK_ID` and `VECTORA_HOOK_EVENT`.

It answers on standard output with `{"decision": "allow"}` or
`{"decision": "deny", "reason": "..."}`. Empty output with exit code 0 means `allow`.

```python
import json, sys

payload = json.load(sys.stdin)
command = payload["tool"]["args"].get("command", "")
if "rm -rf" in command:
    print(json.dumps({"decision": "deny", "reason": "rm -rf is not allowed here"}))
```

## What happens on each event

- **`pre_tool`**: a denial stops the call, and the agent receives the error `hook_denied` with your
  reason. Permission to run the tool is checked again after the hook finishes, so a policy change
  that arrives while the hook runs still applies.
- **`post_tool`**: a blocking denial withholds the tool's result from the model.
- **`pre_response`**: a denial sends your reason back to the model and asks it to revise the answer.
  This happens at most twice per run; after that, the last answer is delivered.

Hooks also apply to the tools a subagent runs.

## Failing closed

In `blocking` mode, a denial, a timeout, a non-zero exit code, or an answer that is not valid JSON
all stop the action. A script that breaks never lets the action through by accident. In
`informative` mode the result is only recorded. Each execution leaves a `hook_run` entry in the
run's evidence, without the call's arguments.

If the sandbox is active for the workspace, the script runs inside the same isolated worker as the
tools. When a script times out, its whole process tree is ended, not only the main process.

## Approval

A `vectora.toml` can come from a cloned repository, so its commands do not run on their own. The
workspace must be approved first, and the approval is tied to a digest of the rules: changing a
command, event, tool list, mode or timeout requires approving again. The same digest covers the
`[hooks].post_file_write` commands. Until then the hooks are listed as pending and skipped.

Review and approve the rules in the **Workspace hooks** section of the Terminal workbench settings.

## Testing from the command line

```bash
vectora hooks list --workspace .
vectora hooks test block-rm --tool run_command --args '{"command": "rm -rf /tmp/x"}'
```

`list` shows each rule and the approval state. `test` runs one rule once with a synthetic call and
prints the decision, without affecting any run. Use `--result` for `post_tool` and `--response` for
`pre_response`. Because you choose the rule yourself, `test` does not require the workspace
approval.

## See also

- [Using the chat](../using-the-chat) — permission modes and approvals
- [Egress policies](../egress-policies)
