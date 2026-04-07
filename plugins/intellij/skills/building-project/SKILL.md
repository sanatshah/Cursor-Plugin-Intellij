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
- `problems`: array of errors/warnings, each with `message`, `kind`, `file`, `line`, `column`
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
2. If `isSuccess` is false, review the `problems` array
3. Fix errors at the reported file/line/column locations
4. Rebuild and repeat until clean

### Targeted validation
1. Call `build_project` with `filesToRebuild` listing only changed files — faster than full build
2. Follow up with `get_file_problems` on the same files for deeper inspection analysis

### Pre-commit validation
1. `build_project` with `rebuild: true` for a clean build
2. Review all problems — fix errors, address warnings as appropriate
