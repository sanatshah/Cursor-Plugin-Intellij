---
name: opening-files
description: Open files in the IntelliJ IDE editor. Supports finding files by name, pattern, or path and opening them. Use when the user wants to open a file, navigate to a file, view a file in IntelliJ, or find and open a specific file in their project.
---

# Opening Files in IntelliJ

## Open a known file

Use `open_file_in_editor` (MCP server: `plugin-intellij-GeneralIDE`):

```json
{"filePath": "src/main/java/com/example/MyClass.java"}
```

- `filePath`: relative to project root or absolute
- `projectPath`: always pass when you know it — reduces ambiguity with multiple projects

## Find then open

When the exact path is unknown, search first using the Index server (`plugin-intellij-Index`), then open.

### By file name — `ide_find_file`

```json
{"query": "UserService.java"}
{"query": "*Test.kt"}
{"query": "BG"}
```

Supports camelCase (`"USJ"` matches `UserService.java`), substring, and wildcard matching.

### By class name — `ide_find_class`

```json
{"query": "UserService"}
```

Returns file paths along with class metadata.

### By text content — `ide_search_text`

```json
{"query": "processOrder", "context": "code"}
```

Finds files containing a specific word.

## Workflow

1. **Path known** → call `open_file_in_editor` directly
2. **Name known, path unknown** → `ide_find_file` → pick the match → `open_file_in_editor`
3. **Only know a class name** → `ide_find_class` → use the returned file path → `open_file_in_editor`
4. **Know content but not file** → `ide_search_text` → use the returned file path → `open_file_in_editor`

## Listing open files

Use `get_all_open_file_paths` (GeneralIDE) to see which files are currently open in the editor.
