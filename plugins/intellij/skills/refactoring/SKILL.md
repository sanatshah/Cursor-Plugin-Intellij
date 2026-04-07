---
name: refactoring
description: Perform safe refactoring operations in IntelliJ including rename, move, safe delete, and code reformatting. Uses the IDE's semantic understanding to update all references automatically. Use when the user wants to rename a symbol, move a file, delete unused code, reformat files, or perform any structural code refactoring.
---

# Refactoring in IntelliJ

Two MCP servers provide refactoring capabilities:
- **Index** (`plugin-intellij-Index`): `ide_refactor_rename`, `ide_refactor_safe_delete`, `ide_move_file`
- **GeneralIDE** (`plugin-intellij-GeneralIDE`): `rename_refactoring`, `reformat_file`

## Rename

### Index server — `ide_refactor_rename` (recommended)

Supports both symbol and file renames. Updates all references project-wide.

**Symbol rename** (requires file + line + column + newName):
```json
{"file": "src/UserService.java", "line": 15, "column": 18, "newName": "CustomerService"}
```

**File rename** (file + newName, no line/column):
```json
{"file": "res/mipmap-hdpi/ic_launcher.webp", "newName": "ic_app_icon.webp"}
```

Options:
- `overrideStrategy`: controls behavior when renaming overriding methods
  - `"rename_base"` (default): rename base method and all overrides
  - `"rename_only_current"`: rename only the target method
  - `"ask"`: show IDE dialog
- `relatedRenamingStrategy`: controls renaming of related symbols (getters/setters, test classes, variables)
  - `"all"` (default): rename everything related
  - `"none"`: rename only the target
  - `"accessors_and_tests"`: rename getters/setters and test classes only
  - `"ask"`: show IDE dialog per related symbol

### GeneralIDE server — `rename_refactoring`

Simpler interface using symbol name instead of position:
```json
{"pathInProject": "src/api/controllers/userController.js", "symbolName": "getUserData", "newName": "fetchUserData"}
```

Use this when you know the exact symbol name but not its precise line/column.

## Move File — `ide_move_file`

Relocates a file and updates all references, imports, and package declarations.

```json
{"file": "src/main/java/com/old/MyClass.java", "destination": "src/main/java/com/new"}
```

- `destination` is auto-created if it doesn't exist
- Set `update_references: false` to skip reference updates (for config/resource files)

## Safe Delete — `ide_refactor_safe_delete`

Checks for usages before deleting. Prevents accidental breakage.

**Symbol delete:**
```json
{"file": "src/OldClass.java", "line": 10, "column": 14}
```

**File delete:**
```json
{"file": "src/UnusedUtils.java", "target_type": "file"}
```

- If usages exist and `force: false` (default), returns the usage list instead of deleting
- Use `force: true` to delete anyway (may break compilation)

## Reformat — `reformat_file`

Apply the project's code formatting rules to a file:
```json
{"path": "src/main/java/com/example/MyClass.java"}
```

## Refactoring Workflow

### Rename workflow
1. Use `ide_find_references` to understand current usage scope
2. Call `ide_refactor_rename` with the new name
3. Verify with `build_project` (GeneralIDE) to confirm no compilation errors

### Safe delete workflow
1. Call `ide_refactor_safe_delete` without `force`
2. If usages are returned, review them — decide whether to remove callers first or abort
3. Once no usages remain, delete succeeds

### Move workflow
1. Call `ide_move_file` with source and destination
2. Verify imports are updated with `get_file_problems` on affected files

## Tips

- All rename operations support undo (Ctrl+Z in IntelliJ).
- Prefer `ide_refactor_rename` over `rename_refactoring` when you have exact file position — it offers finer control over override and related symbol strategies.
- Always pass `project_path` when multiple projects are open to avoid ambiguity.
- After any refactoring, run `build_project` to validate the project still compiles.
