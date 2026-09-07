# Serena Index VS Code Agent Plugin

Portable VS Code Agent Plugin for the `Areo-RGB/serena-main` fork.

The agent sees a single MCP server, `serena-index`. Selected Serena discovery tools delegate internally to the JetBrains Index MCP service.

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

## Included skills

### `serena-index-workflow`

Explains project activation and the preferred Serena/Index-backed workflow.

### `serena-index-wrapper-test`

Runs a focused validation of all five Serena -> Index MCP wrappers and produces a PASS / PARTIAL / FAIL report.

Plugin skills are automatically namespaced by VS Code. You can discover them from the `/` menu or **Chat: Configure Skills**.

## Index-backed Serena wrappers

| Serena tool | Internal Index MCP tool |
|---|---|
| `find_file` | `ide_find_file` |
| `get_symbols_overview` | `ide_file_structure` |
| `find_symbol` | `ide_find_symbol` |
| `find_referencing_symbols` | `ide_find_references` |
| `search_for_pattern` | `ide_search_text` |

Additional Index MCP capabilities can be wrapped in Serena later without exposing a second MCP server to VS Code.

## Troubleshooting

If the MCP server does not start, confirm `uvx --version` works in the environment VS Code inherits.

If Serena connects but the five wrappers fail, verify the local Index MCP endpoint on port `29170` and that the intended JetBrains project is open/indexed.

If project-scoped Serena tools report that no project is active, call `activate_project` with the VS Code workspace root.
