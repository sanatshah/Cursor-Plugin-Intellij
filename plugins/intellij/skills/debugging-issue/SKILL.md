---
name: debugging-issue
description: Debug issues in IntelliJ using the Debugger MCP server. Manages breakpoints, debug sessions, variable inspection, expression evaluation, and stepping through code. Use when the user wants to debug, set breakpoints, inspect variables, step through code, or investigate a bug in their IntelliJ project.
---

# Debugging an Issue in IntelliJ

## Prerequisites

The IntelliJ Debugger MCP server must be running at `http://127.0.0.1:29190/debugger-mcp/streamable-http`. The project must have at least one run configuration defined.

## Workflow

### 1. Discover run configurations

Call `list_run_configurations` (MCP server: `plugin-intellij-Debugger`) to find available configurations before starting a session.

### 2. Set breakpoints

Call `set_breakpoint` with `file_path` (absolute) and `line` (1-based).

**IntelliJ must accept the file path** — use project sources under the repo (e.g. `src/main/java/...`). Do not point at JDK or dependency sources copied to `/tmp` or other ad-hoc paths; breakpoint creation usually fails because the debugger does not map those files to loaded classes. For JDK entrypoints (`Thread.start`, etc.), the MCP cannot create **Java method breakpoints**; tell the user to add those in IntelliJ (**Run → View Breakpoints… → + → Java Method Breakpoint**). Prefer a line in **your code** at the call site (e.g. where an `Executor` submits work) when that matches the user’s intent.

Optional parameters:
- `condition`: boolean expression, e.g. `"count > 10"`
- `log_message`: tracepoint with `{expression}` interpolation, e.g. `"x={x}, y={y}"`
- `suspend_policy`: `"all"` (default), `"thread"`, or `"none"` (logpoint when combined with `log_message`)
- `temporary`: `true` to auto-remove after first hit

### 3. Start a debug session

Call `start_debug_session` with `configuration_name`. Returns a `session_id`.

Immediately follow with `wait_for_pause` (`timeout` required, in seconds) to block until a breakpoint is hit or the program finishes. This avoids manual polling.

### 4. Inspect state when paused

Use these tools (all default to the current session/frame if IDs are omitted):

| Tool | Purpose |
|------|---------|
| `get_debug_session_status` | Current location, variables, stack, and source context in one call |
| `get_variables` | Variables in current or specified `frame_index` |
| `evaluate_expression` | Evaluate arbitrary expressions (full support: Java, Kotlin, Python, JS; limited: Rust, C++, Go) |
| `get_stack_trace` | Full call stack with file/line/class/method per frame |

### 5. Control execution

| Tool | Action |
|------|--------|
| `step_over` | Execute current line, stop at next line |
| `step_into` | Enter the function call on current line |
| `step_out` | Run until current function returns |
| `run_to_line` | Continue to a specific file + line |
| `resume_execution` | Continue to next breakpoint or program end |
| `pause_execution` | Pause a running session |

After every step/resume, call `wait_for_pause` (or `get_debug_session_status`) to see where execution stopped.

### 6. Manage breakpoints and sessions

- `list_breakpoints` — view all breakpoints
- `remove_breakpoint` — remove by breakpoint ID
- `list_debug_sessions` — see all active sessions
- `stop_debug_session` — end a session

## Debugging Loop Pattern

```
1. set_breakpoint at suspect location
2. start_debug_session → wait_for_pause
3. get_debug_session_status (inspect location + variables)
4. evaluate_expression for deeper inspection
5. step_over / step_into to trace logic
6. wait_for_pause after each step
7. Repeat 3-6 until root cause is found
8. stop_debug_session
```

## Tips

- Use `set_variable` to modify variable values mid-session for "what-if" testing.
- Use `select_stack_frame` to switch context to a different frame before inspecting variables.
- Use `list_threads` to see all threads; pass `session_id` to target a specific session.
- Conditional breakpoints (`condition` param) are useful for loops — avoid breaking on every iteration.
- Logpoints (`log_message` + `suspend_policy: "none"`) let you trace without stopping execution.
