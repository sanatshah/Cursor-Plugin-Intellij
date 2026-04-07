---
name: build-and-run
description: >-
  Build and run IntelliJ projects using run configurations. For generic "run
  the app" or "rebuilt and run the app" requests, list configurations first with GeneralIDE, then launch
  one Task subagent per app-style config (parallel Task calls when several
  apply). For a named config or debugging, use direct execute/start_debug.
  Use when the user wants to run their application, execute a run configuration,
  run tests, or build and launch their IntelliJ project.
---

# Building and Running the Project in IntelliJ

Uses the GeneralIDE MCP server (`plugin-intellij-GeneralIDE`) for build and run, and optionally the Debugger server for debug-mode execution.

Always pass `projectPath` (GeneralIDE, camelCase) or `project_path` (Debugger, snake_case) when you know the workspace root — see workspace rules.

## Stopping existing runs

**Always stop existing runs before launching new ones.** This prevents port conflicts and stale processes.

Run these steps in the **parent** turn, **before** spawning any run-configuration subagents:

1. **Stop debug sessions**: Call Debugger **`list_debug_sessions`** with `project_path`. For **each** active session returned, call **`stop_debug_session`** with its `session_id` and `project_path`.
2. **Kill processes on app ports**: Call GeneralIDE **`execute_terminal_command`** to free known app ports. Use `executeInShell: true`, `reuseExistingTerminalWindow: true`, and a short `timeout` (e.g. 5000 ms):
   ```json
   {"command": "lsof -ti:8080 | xargs kill -9 2>/dev/null; exit 0", "executeInShell": true, "reuseExistingTerminalWindow": true, "timeout": 5000, "projectPath": "PROJECT_PATH"}
   ```
   Adjust the port list as needed (e.g. `8080,8081` for multiple services). The `; exit 0` ensures a zero exit code even when no processes are found.

If either step errors (e.g. no active sessions, nothing on the port), that is fine — continue to the run step.

## When the user wants to run the app (no specific configuration named)

Do **not** guess a configuration name. Use this order:

1. **Optional but recommended**: `build_project` with `projectPath` so compile errors surface before any run.
2. **Discover (required)**: `get_run_configurations` with `projectPath` set to the project's **absolute** path. Briefly summarize what exists (names + types if obvious from description).
3. **Filter** in the **parent** turn (same rules as workspace run-app rule): drop `JUnit test configuration` unless the user asked for tests; keep Maven / Gradle / Spring Boot / Application-style configs.
4. **Stop existing runs** — follow the [Stopping existing runs](#stopping-existing-runs) section above.
5. **Execute — one subagent per configuration**: For **each** remaining configuration, launch **`Task`** with:
   - `subagent_type`: `generalPurpose`
   - `model`: `fast` (enough for MCP orchestration unless analysis is very heavy)
   - `prompt`: the **Single-configuration subagent template** below with `PROJECT_PATH`, `CONFIGURATION_NAME`, and (optional) timeout hints filled in.

If **several** configs remain, issue **multiple** `Task` invocations **in parallel** (one per config). Do **not** chain all configs inside one subagent.

### Single-configuration subagent template

Replace `PROJECT_PATH` with the absolute project root and `CONFIGURATION_NAME` with the **exact** `name` from `get_run_configurations`.

```
You run exactly ONE IntelliJ run configuration. Do not execute any other configuration.

- projectPath: PROJECT_PATH
- configurationName: CONFIGURATION_NAME (use this exact string in execute_run_configuration)

1. Optionally verify: GeneralIDE get_run_configurations with projectPath PROJECT_PATH and confirm CONFIGURATION_NAME is still present.
2. GeneralIDE execute_run_configuration with configurationName CONFIGURATION_NAME, projectPath PROJECT_PATH, and timeout: use 600000 ms for heavy Maven/Gradle builds; use 180000 ms or higher for long-lived servers (e.g. name contains spring-boot:run). Set timedOut expectations in your report if the tool times out.
3. Optionally set maxLinesCount (e.g. 500) and truncateMode END if output is huge.
4. Report back: configuration name, exitCode, timedOut, success/failure, and 1–3 lines of stderr/log if failed.

If GeneralIDE errors or the configuration is missing, say so and stop.
```

## Step 1: Build

Call `build_project` to compile first:

```json
{"projectPath": "/absolute/path/to/project"}
```

If the build fails (`isSuccess: false` or `timedOut: true`), **use the MCP response**: read every `problems` entry (and follow with `get_file_problems` / `ide_diagnostics` if needed), fix against those diagnostics, then call `build_project` again. Do not proceed to run or guess fixes without that output. Details: [building-project](../building-project/SKILL.md) and workspace rule [validate-after-edits](../../rules/validate-after-edits.mdc).

## Step 2: Discover run configurations

Call `get_run_configurations` to list available configurations:

```json
{"projectPath": "/absolute/path/to/project"}
```

Returns an array of configurations with `name`, `description`, `commandLine`, `workingDirectory`, and `environment`.

The Debugger server also exposes `list_run_configurations` which returns the same data (`project_path`).

## Step 3: Stop existing runs

Follow the [Stopping existing runs](#stopping-existing-runs) section. This must happen **after** build and discovery but **before** launching new run configurations.

## Step 4: Run

### User named one configuration, or you already picked one

Use **`execute_run_configuration`** or **`start_debug_session`** directly, or still use **one** `Task` with the single-configuration template above.

### Batch "run everything app-like"

Use the **When the user wants to run the app** section: filter in the parent, then **one `Task` per config** (parallel `Task` calls when multiple configs).

### Normal execution — `execute_run_configuration` (GeneralIDE)

```json
{"configurationName": "MyApp", "projectPath": "/absolute/path/to/project"}
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
{"configuration_name": "MyApp", "project_path": "/absolute/path/to/project"}
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

## Combined Workflow (single named configuration)

```
1. build_project → check isSuccess and timedOut
2. If failed → read problems (MCP) + optional get_file_problems / ide_diagnostics → fix → build_project again
3. get_run_configurations → pick the right config (or parent-filter + parallel Task per config if none named)
4. Stop existing runs (debug sessions + port kill)
5. execute_run_configuration → check exitCode, timedOut, and output (use the returned output text to fix)
6. If exit code != 0 → analyze MCP output → fix → build_project → rerun
```

## Tips

- Always build before running to catch compilation errors early with clear diagnostics.
- Always stop existing runs before starting new ones to avoid port conflicts and zombie processes.
- Use `execute_run_configuration` for application/test runs that produce output you need to analyze.
- Use `start_debug_session` when you need to inspect runtime state interactively.
- Use `execute_terminal_command` for custom build commands not covered by IDE run configurations (e.g., `mvn package`, `gradle assemble`, `npm run build`).
- Pass `projectPath` on every call when you know it.

**Related**: [run-all-app-run-configurations](../run-all-app-run-configurations/SKILL.md) — shortcut for "run all app configs" using the same one-subagent-per-config pattern.
