---
name: serena-index-wrapper-test
description: Validate the lean Serena Index plugin surface, the Index MCP-backed wrappers, project switching, and absence of redundant JetBrains-plugin tools.
---

# Test Serena's lean Index MCP integration

Expected architecture:

```text
VS Code agent -> Serena MCP -> selected Serena wrappers -> JetBrains Index MCP -> JetBrains project index
```

The Serena MCP itself must run with `--language-backend LSP`; the separate Serena JetBrains-plugin backend is not part of this plugin.

## 1. Tool-surface check

Confirm the `serena-index` MCP server exposes exactly the intended small Serena surface (subject only to client/runtime filtering):

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

Explicitly verify these are **not** exposed:

- any `jet_brains_*` tools
- `serena_info`
- `get_current_config`
- `list_queryable_projects`
- `query_project`
- raw Serena `read_file`, `list_dir`, `create_text_file`, or shell execution tools
- standalone `open_project` (the plugin exposes `switch_project` instead)

If redundant tools are present, mark the tool-surface test FAIL even if the wrappers themselves work.

## 2. Project activation

If Serena has no active project, call `activate_project` with the current VS Code workspace root.

## 3. Wrapper tests

### `find_file` -> `ide_find_file`
Search for a known source filename and verify returned paths.

### `get_symbols_overview` -> `ide_file_structure`
Choose a supported source file and verify meaningful structure is returned.

### `find_symbol` -> `ide_find_symbol`
Find a known symbol and verify its definition/location.

### `find_referencing_symbols` -> `ide_find_references`
Choose a symbol with known usages and verify representative references.

### `search_for_pattern` -> `ide_search_text`
Search for a distinctive pattern and verify content/paths.

### `open_file` -> `ide_open_file`
Enable `ide_open_file` in **Settings > Tools > Index MCP Server > Exposed Tools** if necessary. Test opening a known file, optionally at a 0-based Serena line/column. Index MCP is 1-based, so normalize when checking navigation.

### `switch_project` -> `ide_open_project` + Serena activation
This is state-changing; run only when the user allows it. Enable `ide_open_project` if necessary and use an existing absolute project directory.

Verify both outcomes:
1. JetBrains opens/indexes the target project.
2. Serena reports the same target as its active project afterward.

Remember `ide_open_project` needs at least one JetBrains project already open as the request context.

## Coordinate rule

- Index MCP lines/columns are 1-based.
- Serena positions are 0-based.
- Index line 34 == Serena line 33.
- Do not report expected normalization as an off-by-one bug.

## Final report

| Test | Result | Notes |
|---|---|---|
| Lean tool surface | PASS / PARTIAL / FAIL | |
| No `jet_brains_*` tools | PASS / FAIL | |
| `find_file` | PASS / PARTIAL / FAIL | |
| `get_symbols_overview` | PASS / PARTIAL / FAIL | |
| `find_symbol` | PASS / PARTIAL / FAIL | |
| `find_referencing_symbols` | PASS / PARTIAL / FAIL | |
| `search_for_pattern` | PASS / PARTIAL / FAIL | |
| `open_file` | PASS / PARTIAL / FAIL | |
| `switch_project` | PASS / PARTIAL / FAIL / NOT RUN | |

For every failure include exact arguments, result/error, and the likely failing layer.

Do not modify repository files during validation.
