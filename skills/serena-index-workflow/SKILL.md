---
name: serena-index-workflow
description: Use when working in VS Code with the Serena plugin. Explains project activation, the lean Serena tool surface, Code Intelligence MCP semantic reads, and Serena/LSP editing.
---

# Serena + Code Intelligence MCP workflow for VS Code

This plugin exposes **one MCP server: Serena** with a deliberately small tool surface.

## Exposed Serena tools

Semantic reads through Code Intelligence MCP (`intellij-mcp`):

- `get_symbols_overview` -> internal `get_file_symbols`
- `find_symbol` -> internal `find_symbol`
- `find_referencing_symbols` -> internal `find_references`
- `get_symbol_info` -> internal `get_symbol_info`
- `get_type_hierarchy` -> internal `get_type_hierarchy`

Project selection:

- `activate_project`

Editing through Serena/LSP:

- `replace_symbol_body`
- `insert_before_symbol`
- `insert_after_symbol`
- `replace_content`
- `replace_in_files`

Use VS Code's native file-name search, text/regex search, reads, navigation, shell, and ordinary editing tools rather than duplicating them through Serena.

## Project activation

Because VS Code Agent Plugins start the MCP process from the plugin installation directory, Serena can begin without the intended workspace active. Call `activate_project` with the current workspace root when needed.

The corresponding project must already be open in JetBrains for Code Intelligence MCP semantic calls to resolve against it.

## Runtime expectations

Serena is launched with:

```text
uvx -p 3.13 --from git+https://github.com/Areo-RGB/serena-main serena start-mcp-server --context=vscode --language-backend LSP
```

Forcing LSP preserves Serena's native editing implementation and prevents the unrelated Serena JetBrains-plugin `jet_brains_*` tool family from appearing.

Code Intelligence MCP endpoint:

```text
http://127.0.0.1:9876/mcp
```

Code Intelligence MCP uses 1-based source coordinates. Serena's public tools use 0-based coordinates and normalize at the adapter boundary.

If the IDE is still indexing, wait for indexing to finish and retry once. Do not classify normal JetBrains dumb-mode state as a Serena adapter failure.

Do not call `initial_instructions` routinely. This skill and Serena's `vscode` context already provide the relevant guidance.
