---
name: find-definitions
description: Find classes, definitions, symbols, references, and implementations in IntelliJ using the Index MCP server. Use when the user wants to find where a class is defined, look up a symbol, find all usages, navigate to a definition, search for text, explore type hierarchies, or trace call chains.
---

# Finding Classes and Definitions in IntelliJ

Uses the IntelliJ Index MCP server (`plugin-intellij-Index`) for fast, IDE-indexed lookups.

## Quick Reference

| Goal | Tool | Key Parameters |
|------|------|---------------|
| Find a class by name | `ide_find_class` | `query` (supports camelCase, substring, wildcard) |
| Find a file by name | `ide_find_file` | `query` (supports camelCase, substring, wildcard) |
| Go to definition | `ide_find_definition` | `file` + `line` + `column`, OR `language` + `symbol` |
| Find all references | `ide_find_references` | `file` + `line` + `column`, OR `language` + `symbol` |
| Find implementations | `ide_find_implementations` | `file` + `line` + `column`, OR `language` + `symbol` |
| Search text in code | `ide_search_text` | `query` (exact word match using IDE index) |
| Get symbol info | `get_symbol_info` | `filePath` + `line` + `column` (GeneralIDE server) |

## Lookup by Name

### Find a class

```json
{"query": "UserService"}
{"query": "USvc"}
{"query": "User*Impl"}
```

- `matchMode`: `"substring"` (default), `"prefix"`, or `"exact"`
- `language`: filter by language, e.g. `"Kotlin"`, `"Java"`
- `includeLibraries`: `true` to search dependencies
- Supports pagination via `cursor` from previous response

### Find a file

```json
{"query": "UserService.java"}
{"query": "*Test.kt"}
{"query": "BG"}
```

Same pagination and library options as `ide_find_class`.

## Lookup by Position

### Go to definition

Two targeting modes (mutually exclusive):

**Position-based:**
```json
{"file": "src/Main.java", "line": 15, "column": 10}
```

**Symbol-based (Java):**
```json
{"language": "Java", "symbol": "com.example.MyClass#processData(String)"}
```

Set `fullElementPreview: true` to get the complete source of the target element.

### Find all references

Same targeting as `ide_find_definition`. Returns file paths, line numbers, context snippets, and reference types (method_call, field_access, import, etc.).

### Find implementations

Same targeting. Discovers concrete implementations of interfaces, abstract classes, or abstract methods. Works across Java, Kotlin, Python, JS, TS, PHP, Rust.

## Code Intelligence

### Type hierarchy

`ide_type_hierarchy` — get full inheritance chain (supertypes and subtypes).

```json
{"className": "com.example.UserService"}
{"file": "src/MyClass.java", "line": 10, "column": 14}
```

### Call hierarchy

`ide_call_hierarchy` — trace who calls a method (callers) or what it calls (callees).

```json
{"file": "src/Service.java", "line": 42, "column": 10, "direction": "callers"}
```

`depth` controls levels (default 3, max 5).

### Super methods

`ide_find_super_methods` — navigate up inheritance from override to original declaration.

### Text search

`ide_search_text` — fast indexed word search (not regex). Filter by `context`: `"code"`, `"comments"`, `"strings"`, or `"all"`.

```json
{"query": "ConfigManager", "context": "code", "caseSensitive": true}
```

## Strategy

1. **Know the name?** Start with `ide_find_class` or `ide_find_file`.
2. **See a symbol in code?** Use `ide_find_definition` with file + line + column.
3. **Need all usages before changing?** Use `ide_find_references`.
4. **Working with interfaces?** Use `ide_find_implementations` to find concrete types.
5. **Understanding architecture?** Combine `ide_type_hierarchy` and `ide_call_hierarchy`.
