---
name: serena-index-wrapper-test
description: Validate the lean Serena + Code Intelligence MCP integration, the five semantic-read wrappers, and the absence of redundant Index/JetBrains-plugin tools.
---

# Test Serena's lean Code Intelligence MCP integration

Expected architecture:

```text
VS Code agent -> Serena MCP -> five semantic wrappers -> Code Intelligence MCP -> JetBrains PSI/index
                                  |
                                  +-> Serena/LSP editing tools
```

The Serena MCP itself must run with `--language-backend LSP`. Code Intelligence MCP (`intellij-mcp`) is an internal backend, not a second agent-visible MCP server.

## 1. Tool-surface check

Confirm `serena-index` exposes exactly these 11 Serena tools (subject only to client/runtime filtering):

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

Explicitly verify these are **not** exposed:

- any `jet_brains_*` tools
- `serena_info`
- `find_file`
- `search_for_pattern`
- `open_file`
- `open_project`
- `switch_project`
- `get_current_config`
- `list_queryable_projects`
- `query_project`
- raw Serena `read_file`, `list_dir`, `create_text_file`, or shell execution tools
- any separate direct Index MCP / intellij-mcp server

## 2. Project activation

If Serena has no active project or is on the wrong one, call `activate_project` with the current VS Code workspace root. Confirm the same project is open in JetBrains.

## 3. Semantic wrapper tests

### `get_symbols_overview` -> `get_file_symbols`
Choose a supported source file. Verify meaningful nested symbols are returned.

### `find_symbol` -> `find_symbol`
Find a known class/function/method/variable. Verify the file, kind and location. Test path-constrained lookup and `include_body=true` where useful.

### `find_referencing_symbols` -> `find_references`
Choose a symbol with multiple known usages. Verify representative paths, preview snippets and reference locations.

### `get_symbol_info` -> `get_symbol_info`
Use a known symbol position. Verify type/signature/documentation metadata is sensible.

### `get_type_hierarchy` -> `get_type_hierarchy`
Choose a class/interface with inheritance where available. Verify base/subtype results. If the project has no useful inheritance example, report NOT RUN rather than inventing one.

## Coordinate rule

- Code Intelligence MCP positions are **1-based**.
- Serena positions are **0-based**.
- MCP line 34 == Serena line 33.
- Normalize coordinates before reporting an adapter bug.

## Editing sanity check

Use only disposable code if editing is allowed. Verify at least one Serena/LSP editing tool still works, preferably `replace_content` or a symbolic edit in a temporary file. Restore/delete the temporary content afterward with VS Code-native tools.

## Failure classification

Distinguish:

- Serena MCP/plugin startup
- Serena adapter
- Code Intelligence MCP transport (`127.0.0.1:9876/mcp`)
- JetBrains project selection
- JetBrains indexing/dumb mode
- Serena LSP editing

Normal JetBrains indexing state is not a Serena adapter bug.

## Final report

| Test | Result | Notes |
|---|---|---|
| Lean 11-tool surface | PASS / PARTIAL / FAIL | |
| No `jet_brains_*` / old Index wrappers | PASS / FAIL | |
| `get_symbols_overview` | PASS / PARTIAL / FAIL | |
| `find_symbol` | PASS / PARTIAL / FAIL | |
| `find_referencing_symbols` | PASS / PARTIAL / FAIL | |
| `get_symbol_info` | PASS / PARTIAL / FAIL | |
| `get_type_hierarchy` | PASS / PARTIAL / FAIL / NOT RUN | |
| Serena/LSP editing sanity | PASS / PARTIAL / FAIL / NOT RUN | |

For every failure include exact arguments, result/error, expected behavior, and the likely failing layer.

Do not modify production repository code during validation.
