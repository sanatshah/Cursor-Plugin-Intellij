---
name: run-all-app-run-configurations
description: >-
  Batch-run all app-style IntelliJ run configurations (skip JUnit-only).
  Parent lists and filters configs via GeneralIDE, then launches one Task
  subagent per configuration (parallel Task calls when several apply). Use
  when the user wants to run all non-test run configurations without naming
  a single config.
---

# Run all app run configurations

This workflow lives in **[build-and-run](../build-and-run/SKILL.md)** under **"When the user wants to run the app (no specific configuration named)"**.

Follow that section exactly:

1. Optionally `build_project` with `projectPath`. If **`isSuccess`** is false, use the MCP **`problems`** (and related diagnostics) to fix and rebuild before spawning run `Task`s — see [building-project](../building-project/SKILL.md).
2. **`get_run_configurations`** with `projectPath` (required before delegating).
3. **Filter** app-style configs in the parent (skip `JUnit test configuration` unless the user asked for tests).
4. **Stop existing runs** before launching new ones — follow the [Stopping existing runs](../build-and-run/SKILL.md#stopping-existing-runs) section in build-and-run: stop active debug sessions via Debugger `list_debug_sessions` / `stop_debug_session`, then kill processes on app ports via GeneralIDE `execute_terminal_command`.
5. Launch **one `Task`** (`subagent_type`: `generalPurpose`, `model`: `fast`) **per** filtered configuration, using the **Single-configuration subagent template** in build-and-run (replace `PROJECT_PATH` and `CONFIGURATION_NAME` each time). Use **parallel** `Task` invocations when **multiple** configs apply so each subagent runs **only** its own `execute_run_configuration`. Have the subagent run in the background.

Parallel runs may contend for ports or IDE resources; mention that if failures look like conflicts.

## Related

- [build-and-run](../build-and-run/SKILL.md) — template, single-config run, debug, and terminal execution.
