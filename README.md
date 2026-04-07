# Cursor Plugin for IntelliJ

A [Cursor](https://cursor.com) Team Marketplace plugin that integrates IntelliJ IDEA's powerful IDE capabilities — debugging, code indexing, and general IDE operations — directly into your Cursor AI workflow via MCP (Model Context Protocol) servers.


---

## What This Does

This plugin connects Cursor to three IntelliJ Marketplace MCP servers, giving your AI assistant deep access to your IntelliJ IDE. Instead of switching between Cursor and IntelliJ, you can debug, navigate, refactor, and build your project through natural language.

### Integrated IntelliJ MCP Servers

| Server | Description |
|---|---|
| **Debugger MCP Server** | Full debugging control — breakpoints, stepping, variable inspection, expression evaluation |
| **IDE Index MCP Server** | Code navigation and refactoring — find references, definitions, implementations, type hierarchies |
| **MCP Server** | General IDE operations — file management, terminal commands, project builds, search |

---

## Available Tools

### Debug Tools (22 tools)

Control IntelliJ's debugger without leaving Cursor:

- **Session management** — `start_debug_session`, `stop_debug_session`, `list_debug_sessions`, `get_debug_session_status`
- **Execution control** — `resume_execution`, `pause_execution`, `step_into`, `step_over`, `step_out`, `run_to_line`
- **Breakpoints** — `set_breakpoint`, `remove_breakpoint`, `list_breakpoints`
- **Inspection** — `get_variables`, `get_stack_trace`, `select_stack_frame`, `evaluate_expression`, `set_variable`
- **Configuration** — `list_run_configurations`, `execute_run_configuration`, `list_threads`, `wait_for_pause`
- **Context** — `get_source_context`

### Index Tools (15 tools)

Navigate and refactor code through IntelliJ's indexing engine:

- **Navigation** — `ide_find_definition`, `ide_find_references`, `ide_find_implementations`, `ide_find_super_methods`
- **Search** — `ide_find_file`, `ide_find_class`, `ide_search_text`
- **Hierarchy** — `ide_type_hierarchy`, `ide_call_hierarchy`
- **Refactoring** — `ide_refactor_rename`, `ide_refactor_safe_delete`, `ide_move_file`
- **Diagnostics** — `ide_diagnostics`, `ide_index_status`, `ide_sync_files`

### General IDE Tools (23 tools)

Manage files, run commands, and interact with IntelliJ's broader features:

- **File operations** — `get_file_text_by_path`, `create_new_file`, `replace_text_in_file`, `open_file_in_editor`, `reformat_file`
- **Search** — `search_in_files_by_text`, `search_in_files_by_regex`, `find_files_by_name_keyword`, `find_files_by_glob`
- **Project** — `get_project_modules`, `get_project_dependencies`, `get_file_problems`, `build_project`, `list_directory_tree`
- **Execution** — `execute_terminal_command`, `get_run_configurations`, `execute_run_configuration`, `runNotebookCell`
- **Code intelligence** — `get_symbol_info`, `rename_refactoring`, `get_all_open_file_paths`
- **Other** — `permission_prompt`, `get_repositories`

---

## Repository Structure

```
.
├── .cursor-plugin/          # Marketplace manifest and plugin registry
│   └── marketplace.json
├── .cursor/                 # Cursor editor configuration
├── assets/                  # Plugin assets (logos, icons)
├── docs/                    # Documentation
├── plugins/
│   └── intellij/            # IntelliJ plugin definition
│       ├── .cursor-plugin/
│       │   └── plugin.json  # Plugin metadata
│       ├── rules/           # Rule files (.mdc)
│       ├── skills/          # Skill definitions (SKILL.md)
│       ├── agents/          # Subagent definitions
│       └── mcp.json         # MCP server configuration
├── scripts/                 # Validation and utility scripts
├── .gitignore
└── README.md
```

---

## Getting Started

### Prerequisites

- [Cursor](https://cursor.com) editor installed
- [IntelliJ IDEA](https://www.jetbrains.com/idea/) with the following plugins installed from the JetBrains Marketplace:
  - Debugger MCP Server
  - IDE Index MCP Server
  - MCP Server

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/sanatshah/Cursor-Plugin-Intellij.git
   ```

2. **Open your project in IntelliJ IDEA** and ensure the three MCP server plugins are installed and running.

3. **Add this plugin to your Cursor Team Marketplace** by referencing the repository in your Cursor configuration.

4. **Validate the setup:**
   ```bash
   node scripts/validate-template.mjs
   ```
   This checks marketplace paths, plugin manifests, and required frontmatter in all rule/skill/agent files.

---

## License

This project is forked from the [Cursor Plugin Template](https://github.com/cursor/plugin-template). See the repository for license details.
