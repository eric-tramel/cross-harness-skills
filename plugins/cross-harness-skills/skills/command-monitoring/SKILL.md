---
name: command-monitoring
description: Manage long-running shell commands, tests, servers, sandboxes, and log polling without blocking the harness. Use when starting, waiting on, checking, or cleaning up commands that may outlive one foreground tool call.
---

# Command Monitoring

Use this skill when a command may keep running, produce delayed output, open a server, wait for a condition, or require periodic status checks.

## Workflow

1. Classify the command before running it:
   - Run short, bounded commands in the foreground.
   - Run servers, watch modes, long tests, sandboxes, and migrations with the harness's background or session mechanism.
   - Use a readiness condition for waits, such as a process exit, log line, health endpoint, file creation, or open port.
2. Start long-running work with a durable handle:
   - Prefer the harness's background execution, monitor, job, or session API.
   - Record the returned handle, process id, port, and log path if the harness exposes them.
   - If the harness has no background API, use a simple shell background process only when allowed, redirect output to a known temporary log, and print the process id.
3. Observe the command separately from starting it:
   - Poll the harness handle, process status, or readiness condition in a separate tool call.
   - Do not chain a naked `sleep` with `tail`, `cat`, or another status command just to wait.
   - When a shell wait is appropriate, use a bounded condition loop that exits on success or failure instead of sleeping for a fixed delay.
   - Bound each readiness probe too; the check inside the loop must not be able to hang past the loop deadline.
4. Keep output bounded:
   - Read only the tail or relevant slices of logs while the command is still running.
   - Preserve full logs on disk only in approved temporary or workspace locations, keep retention bounded, and clean them up when they are no longer needed.
   - Redact secrets, private paths, session identifiers, and other sensitive output before quoting or summarizing logs.
   - If output is noisy, search for concrete markers such as exit status, panic, traceback, failing test name, readiness line, or listening port.
5. Finish with explicit state:
   - Report whether the command is still running, completed successfully, failed, or was stopped.
   - Stop servers, sandboxes, and watchers you started unless the user asked to keep them running.
   - Do not end the task while a command needed for the user's request is still running.

## Harness Notes

- In Codex-style exec sessions, let long foreground commands return a session id, then poll that session rather than starting a separate sleep-and-tail command.
- In Claude-style harnesses, use the available background-run or monitor tools for long commands. If a command pattern is blocked, change to the supported monitor pattern instead of retrying the same blocked sleep chain.
- In CI-like or noninteractive shells, prefer a bounded readiness loop:

```bash
deadline=$((SECONDS + 60))
until curl -fsS --connect-timeout 2 --max-time 5 http://127.0.0.1:3000/health >/dev/null; do
  if [ "$SECONDS" -ge "$deadline" ]; then
    exit 1
  fi
  sleep 2
done
```

Use a condition loop only when it directly checks the state the task needs. Avoid using it as a generic delay.

## Permission Boundaries

Starting local development services, containers, GUI applications, network listeners, or externally mutating commands may require user approval in some harnesses. Ask or request approval at the point of mutation, then continue with the same monitoring workflow after approval.
