# Serena Index VS Code Agent Plugin

Portable VS Code Agent Plugin for the `Areo-RGB/serena-main` fork.

The agent sees a single MCP server, `serena-index`. Selected Serena discovery and navigation tools delegate internally to the JetBrains Index MCP service.

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
    +-- open_project -----------------> ide_open_project
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

For `open_file` and `open_project`, enable `ide_open_file` and `ide_open_project` under **Settings > Tools > Index MCP Server > Exposed Tools**. They are opt-in/disabled-by-default Index MCP tools.

## Install directly from GitHub

In VS Code:

1. Open the Command Palette.
2. Run **Chat: Install Plugin From Source**.
3. Enter:

```text
https://github.com/Areo-RGB/serena_index_vscode.git
```

4. Review the plugin and accept the trust prompt.
5. Verify `serena-index` under **MCP: List Servers** or Chat **Configure Tools**.

VS Code also supports registering a local clone through `chat.pluginLocations`.

## Serena runtime

The plugin starts Serena with:

```bash
uvx -p 3.13 \
  --from git+https://github.com/Areo-RGB/serena-main \
  serena start-mcp-server \
  --context=vscode \
  --open-web-dashboard=false
```

`--project-from-cwd` is intentionally not used. Agent Plugins start stdio servers from the installed plugin directory, which is not guaranteed to be the open workspace.

When Serena has no active project, use `activate_project` with the current VS Code workspace root.

## VS Code hooks

The Agent Plugins 1.0 package includes Copilot-specific hooks at:

```text
com.github.copilot/hooks/hooks.json
```

They are adapted from Serena's official VS Code hook configuration and run the fork through `uvx`:

```text
SessionStart -> serena-hooks activate --client=vscode
PreToolUse   -> serena-hooks remind --client=vscode
Stop         -> serena-hooks cleanup --client=vscode
```

The actual hook commands use:

```bash
uvx -p 3.13 --from git+https://github.com/Areo-RGB/serena-main serena-hooks <command> --client=vscode
```

## Included skills

### `serena-index-workflow`
Explains project activation and the preferred Serena/Index-backed workflow.

### `serena-index-wrapper-test`
Runs a focused validation of all seven Serena -> Index MCP wrappers and produces a PASS / PARTIAL / FAIL report.

## Index-backed Serena wrappers

| Serena tool | Internal Index MCP tool |
|---|---|
| `find_file` | `ide_find_file` |
| `get_symbols_overview` | `ide_file_structure` |
| `find_symbol` | `ide_find_symbol` |
| `find_referencing_symbols` | `ide_find_references` |
| `search_for_pattern` | `ide_search_text` |
| `open_file` | `ide_open_file` |
| `open_project` | `ide_open_project` |

`open_file(relative_path, line?, column?)` uses Serena's 0-based line/column convention and converts to Index MCP's 1-based navigation coordinates.

`open_project(path, auto_link=false, timeout_seconds=600)` opens an absolute filesystem project path in JetBrains and waits for indexing. It requires at least one JetBrains project already open as the request context. Opening a project does not automatically change Serena's active project; call `activate_project` afterward if needed.

## Troubleshooting

If the MCP server or hooks do not start, confirm `uvx --version` works in the environment VS Code inherits.

If Serena connects but the Index-backed wrappers fail, verify the local Index MCP endpoint on port `29170` and that the intended JetBrains project is open/indexed.

If `open_file` or `open_project` reports the underlying Index tool is disabled, enable it under **Settings > Tools > Index MCP Server > Exposed Tools**.

If project-scoped Serena tools report that no project is active, call `activate_project` with the VS Code workspace root.
