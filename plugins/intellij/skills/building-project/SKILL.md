---
name: building-project
description: Build and compile IntelliJ projects, check for compilation errors, and inspect file-level problems. Use when the user wants to build, compile, check for errors, validate changes, or get diagnostics on their IntelliJ project.
---

# Building the Project in IntelliJ

Uses the GeneralIDE MCP server (`plugin-intellij-GeneralIDE`).

## Build the project — `build_project`

### Incremental build (default)
```json
{}
```

### Full rebuild
```json
{"rebuild": true}
```

### Compile specific files only
```json
{"filesToRebuild": ["src/main/java/com/example/MyClass.java", "src/main/java/com/example/Utils.java"]}
```

### With timeout
```json
{"timeout": 60000}
```

Always pass `projectPath` when you know it.

### Output

Returns:
- `isSuccess`: whether the build succeeded
- `problems`: array of errors/warnings, each with `message`, `kind`, `group`, `description`, `file`, `line`, `column`
- `timedOut`: whether the build exceeded the timeout

## Check file problems — `get_file_problems`

Runs IntelliJ inspections on a single file (errors, warnings, code smells):

```json
{"filePath": "src/main/java/com/example/MyClass.java"}
```

Set `errorsOnly: true` to filter out warnings.

Returns problems with `severity`, `description`, `lineContent`, `line`, and `column`.

## IDE diagnostics — `ide_diagnostics`

Use `ide_diagnostics` (Index server: `plugin-intellij-Index`) for broader project-level diagnostic information.

## Build Workflow

### After making edits
1. Call `build_project` (incremental)
2. If `isSuccess` is false, **read the full MCP response** and drive fixes from it (do not infer errors from generic knowledge)
3. For each `problems` entry, use `file` / `line` / `column` when present, then `message` and `description`; use `kind` / `group` for filtering noisy warnings
4. If `timedOut` is true, treat results as partial — increase `timeout`, rerun, or use `filesToRebuild` for a faster pass
5. If `problems` is empty but the build failed, call `get_file_problems` on edited files and/or `ide_diagnostics` (Index) before editing
6. Rebuild with `build_project` and repeat until `isSuccess` is true

### When `build_project` fails — checklist
1. Copy or mentally map every `problems` item to a concrete code location.
2. Fix and run `build_project` again (same `projectPath`; use `filesToRebuild` when iterating on a small set).
3. Only fall back to terminal Maven/Gradle if GeneralIDE MCP is unavailable or the user asked for it — even then, prefer fixing against the same error text the tool or CI surfaced.

### Targeted validation
1. Call `build_project` with `filesToRebuild` listing only changed files — faster than full build
2. Follow up with `get_file_problems` on the same files for deeper inspection analysis

### Pre-commit validation
1. `build_project` with `rebuild: true` for a clean build
2. Review all problems — fix errors, address warnings as appropriate
