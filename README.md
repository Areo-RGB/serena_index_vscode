# Serena Index VS Code Agent Plugin

Portable VS Code Agent Plugin for the `Areo-RGB/serena-main` fork.

The agent sees one MCP server, `serena-index`, with a deliberately small Serena tool surface. Selected discovery/navigation tools delegate internally to JetBrains Index MCP.

## Lean Serena tool surface

The VS Code plugin exposes:

- `activate_project`
- `find_file`
- `get_symbols_overview`
- `find_symbol`
- `find_referencing_symbols`
- `search_for_pattern`
- `open_file`
- `switch_project`
- `replace_symbol_body`
- `insert_before_symbol`
- `insert_after_symbol`
- `replace_content`
- `replace_in_files`

Serena diagnostics/config helpers, memories, cross-project query helpers, raw file/shell tools already provided by VS Code, and the separate Serena JetBrains-plugin `jet_brains_*` family are intentionally omitted.

## Architecture

```text
VS Code agent
    |
    v
Serena MCP (`serena-index`)
    |
    +-- find_file --------------------> ide_find_file
    +-- get_symbols_overview ---------> ide_file_structure
    +-- find_symbol ------------------> ide_find_symbol
    +-- find_referencing_symbols -----> ide_find_references
    +-- search_for_pattern -----------> ide_search_text
    +-- open_file --------------------> ide_open_file
    +-- switch_project ---------------> ide_open_project + Serena activation
                                        |
                                        v
                              JetBrains Index MCP
```

There is no separate agent-visible Index MCP server.

## Requirements

- VS Code with Agent Plugins enabled (`chat.plugins.enabled`)
- `uvx` / `uv` available on `PATH`
- Python 3.13 available to `uv`
- JetBrains Index MCP running locally at:

```text
http://127.0.0.1:29170/index-mcp/streamable-http
```

Enable `ide_open_file` and `ide_open_project` under **Settings > Tools > Index MCP Server > Exposed Tools** before using `open_file` or `switch_project`.

## Install directly from GitHub

1. Open the Command Palette.
2. Run **Chat: Install Plugin From Source**.
3. Enter:

```text
https://github.com/Areo-RGB/serena_index_vscode.git
```

4. Review the plugin and accept the trust prompt.
5. Verify `serena-index` under **MCP: List Servers** or Chat **Configure Tools**.

## Serena runtime

The plugin starts Serena with the **LSP backend explicitly forced**:

```bash
uvx -p 3.13 \
  --from git+https://github.com/Areo-RGB/serena-main \
  serena start-mcp-server \
  --context=vscode \
  --language-backend LSP \
  --open-web-dashboard=false
```

This prevents Serena's separate JetBrains-plugin backend from injecting `jet_brains_*` tools. The selected Serena discovery/navigation wrappers still call JetBrains Index MCP internally.

`--project-from-cwd` is intentionally not used because Agent Plugins start stdio servers from the plugin installation directory. If Serena has no active project, call `activate_project` with the current VS Code workspace root.

Once a project is active, use `switch_project(path, auto_link=false, timeout_seconds=600)` to move to another project. It first opens/indexes the absolute path in JetBrains through `ide_open_project`, then activates the same path in Serena.

## VS Code hooks

The package includes Copilot-specific hooks at:

```text
com.github.copilot/hooks/hooks.json
```

They run:

```text
SessionStart -> serena-hooks activate --client=vscode
PreToolUse   -> serena-hooks remind --client=vscode
Stop         -> serena-hooks cleanup --client=vscode
```

## Included skills

- `serena-index-workflow`: explains project activation and the lean Serena/Index-backed workflow.
- `serena-index-wrapper-test`: validates the Index-backed wrappers and the lean exposed tool list.

## Troubleshooting

If the MCP server or hooks do not start, confirm `uvx --version` works in the environment VS Code inherits.

If Index-backed wrappers fail, verify the Index MCP endpoint on port `29170`, the intended JetBrains project is open/indexed, and the required `ide_*` tool is enabled.

If you still see `jet_brains_*`, `serena_info`, `get_current_config`, or cross-project query tools after updating, restart/reload the plugin so the new context and MCP launch command take effect.
