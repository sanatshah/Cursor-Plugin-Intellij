---
name: build-and-run
description: Build and run IntelliJ projects using run configurations. Discovers available configurations, executes them, and captures output. Use when the user wants to run their application, execute a run configuration, run tests, or build and launch their IntelliJ project.
---

# Building and Running the Project in IntelliJ

Uses the GeneralIDE MCP server (`plugin-intellij-GeneralIDE`) for build and run, and optionally the Debugger server for debug-mode execution.

## Step 1: Build

Call `build_project` to compile first:

```json
{}
```

If the build fails (`isSuccess: false`), fix errors before proceeding. See the [building-project](../building-project/SKILL.md) skill for details.

## Step 2: Discover run configurations

Call `get_run_configurations` to list available configurations:

```json
{}
```

Returns an array of configurations with `name`, `description`, `commandLine`, `workingDirectory`, and `environment`.

The Debugger server also exposes `list_run_configurations` which returns the same data.

## Step 3: Run

### Normal execution — `execute_run_configuration` (GeneralIDE)

```json
{"configurationName": "MyApp"}
```

Options:
- `timeout`: max wait time in milliseconds (process killed if exceeded)
- `maxLinesCount`: limit output lines
- `truncateMode`: `"START"`, `"MIDDLE"`, `"END"`, or `"NONE"`

Returns:
- `exitCode`: process exit code
- `output`: captured stdout/stderr
- `timedOut`: whether the timeout was hit

### Debug execution — `start_debug_session` (Debugger)

For running with breakpoints and stepping:

```json
{"configuration_name": "MyApp"}
```

Follow with `wait_for_pause` to block until a breakpoint is hit. See the [debugging-issue](../debugging-issue/SKILL.md) skill for the full debug workflow.

### Terminal execution — `execute_terminal_command` (GeneralIDE)

For arbitrary shell commands (build tools, scripts, custom commands):

```json
{"command": "./gradlew run", "executeInShell": true, "timeout": 30000}
```

Options:
- `executeInShell`: `true` to run in user's shell (preserves environment)
- `reuseExistingTerminalWindow`: `true` to avoid creating multiple terminals

## Combined Workflow

```
1. build_project → check isSuccess
2. If failed → fix errors → rebuild
3. get_run_configurations → pick the right config
4. execute_run_configuration → check exitCode and output
5. If exit code != 0 → analyze output → fix → rebuild → rerun
```

## Tips

- Always build before running to catch compilation errors early with clear diagnostics.
- Use `execute_run_configuration` for application/test runs that produce output you need to analyze.
- Use `start_debug_session` when you need to inspect runtime state interactively.
- Use `execute_terminal_command` for custom build commands not covered by IDE run configurations (e.g., `mvn package`, `gradle assemble`, `npm run build`).
- Pass `projectPath` on every call when you know it.
