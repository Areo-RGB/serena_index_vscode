# Serena Code Intelligence VS Code Agent Plugin

Portable VS Code Agent Plugin for the `Areo-RGB/serena-main` fork.

The agent sees one MCP server, `serena-index`, with a deliberately small Serena tool surface. Five semantic-read tools delegate internally to the JetBrains **Code Intelligence MCP (`intellij-mcp`)** plugin.

## Lean Serena tool surface

The VS Code plugin exposes these 11 tools:

- `activate_project`
- `get_symbols_overview`
- `find_symbol`
- `find_referencing_symbols`
- `get_symbol_info`
- `get_type_hierarchy`
- `replace_symbol_body`
- `insert_before_symbol`
- `insert_after_symbol`
- `replace_content`
- `replace_in_files`

VS Code already provides file-name search, text/regex search, file reads, navigation, shell access, and line editing, so Serena intentionally does not duplicate those operations here.

## Architecture

```text
VS Code agent
    |
    v
Serena MCP (`serena-index`)
    |
    +-- get_symbols_overview ---------> get_file_symbols
    +-- find_symbol ------------------> find_symbol
    +-- find_referencing_symbols -----> find_references
    +-- get_symbol_info --------------> get_symbol_info
    +-- get_type_hierarchy -----------> get_type_hierarchy
                                        |
                                        v
                         Code Intelligence MCP
                         http://127.0.0.1:9876/mcp
                         (inside JetBrains)

Serena editing tools
    |
    v
Serena / LSP
```

There is no separate agent-visible IntelliJ/Index MCP server.

## Requirements

- VS Code with Agent Plugins enabled (`chat.plugins.enabled`)
- `uvx` / `uv` available on `PATH`
- Python 3.13 available to `uv`
- JetBrains IDE with Code Intelligence MCP (`intellij-mcp`) running
- target project open in JetBrains
- endpoint available at `http://127.0.0.1:9876/mcp`

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

The plugin starts Serena with its **LSP backend explicitly forced**:

```bash
uvx -p 3.13 \
  --from git+https://github.com/Areo-RGB/serena-main \
  serena start-mcp-server \
  --context=vscode \
  --language-backend LSP \
  --open-web-dashboard=false
```

The LSP backend remains responsible for Serena's native editing capabilities. The five semantic reads above bypass LSP and use Code Intelligence MCP internally.

`--project-from-cwd` is intentionally not used because Agent Plugins start stdio servers from the plugin installation directory. If Serena has no active project or is on the wrong project, call `activate_project` with the current workspace root. The same project must be open in JetBrains for semantic reads.

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

- `serena-index-workflow`: explains the lean Serena + Code Intelligence MCP workflow.
- `serena-index-wrapper-test`: validates the five semantic wrappers and the 11-tool surface.

## Troubleshooting

If Serena or the hooks do not start, confirm `uvx --version` works in the environment VS Code inherits.

If semantic reads fail, verify `http://127.0.0.1:9876/mcp`, confirm the intended project is open in JetBrains, and wait for IDE indexing to finish.

If you still see old tools such as `find_file`, `search_for_pattern`, `open_file`, `switch_project`, `jet_brains_*`, or `serena_info` after updating, restart/reload the plugin so the new Serena context takes effect.
