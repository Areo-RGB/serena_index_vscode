---
name: serena-index-workflow
description: Use when working in VS Code with the Serena Index agent plugin. Explains project activation, the lean Serena tool surface, Index MCP-backed navigation, and project switching.
---

# Serena Index workflow for VS Code

This plugin exposes **one MCP server: Serena** with a deliberately small tool surface.

## Exposed Serena tools

Discovery/navigation:

- `find_file` -> internal `ide_find_file`
- `get_symbols_overview` -> internal `ide_file_structure`
- `find_symbol` -> internal `ide_find_symbol`
- `find_referencing_symbols` -> internal `ide_find_references`
- `search_for_pattern` -> internal `ide_search_text`
- `open_file` -> internal `ide_open_file`
- `switch_project` -> internal `ide_open_project`, then Serena activation
- `activate_project` -> Serena-only activation when no project is active yet

Editing:

- `replace_symbol_body`
- `insert_before_symbol`
- `insert_after_symbol`
- `replace_content`
- `replace_in_files`

The plugin intentionally does **not** expose Serena config/diagnostic helpers, memories, cross-project query helpers, raw file/shell tools, or the separate Serena JetBrains-plugin `jet_brains_*` family.

## Start by activating the workspace

Because VS Code Agent Plugins start the MCP server from the plugin installation directory, Serena may begin with no project. In that case call `activate_project` with the current workspace root.

Once a project is active, use `switch_project(path, auto_link=false, timeout_seconds=600)` to move to another project. It opens/indexes the target in JetBrains first and then activates the same absolute path in Serena.

## Runtime expectations

Serena is launched with:

```text
uvx -p 3.13 --from git+https://github.com/Areo-RGB/serena-main serena start-mcp-server --context=vscode --language-backend LSP
```

Forcing the LSP backend prevents the unrelated Serena JetBrains-plugin mode from injecting `jet_brains_*` tools. The selected discovery/navigation wrappers still call JetBrains Index MCP internally.

Index MCP endpoint:

```text
http://127.0.0.1:29170/index-mcp/streamable-http
```

Enable `ide_open_file` and `ide_open_project` in **Settings > Tools > Index MCP Server > Exposed Tools** when using `open_file` or `switch_project`.

Do not call `initial_instructions` routinely. This skill and the Serena `vscode` context already provide the relevant guidance.
