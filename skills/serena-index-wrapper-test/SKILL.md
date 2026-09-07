---
name: serena-index-wrapper-test
description: Validate the Serena tools in this fork that delegate internally to JetBrains Index MCP. Use when testing plugin installation, wrapper correctness, path/position handling, IDE navigation, or diagnosing Serena-to-Index integration failures.
---

# Test Serena's Index MCP wrappers

Validate the architecture:

```text
VS Code agent -> Serena MCP -> Index MCP -> JetBrains project index
```

Do not call direct `ide_*` tools as the primary workflow. If a separately configured Index MCP server is available for diagnostics, it may be used only for side-by-side validation.

## Preparation

1. Confirm the `serena-index` MCP server is connected.
2. If no project is active, call `activate_project` with the current VS Code workspace root.
3. Confirm these Serena tools are available:
   - `find_file`
   - `get_symbols_overview`
   - `find_symbol`
   - `find_referencing_symbols`
   - `search_for_pattern`
   - `open_file`
   - `open_project`

## Coordinate-system rule

- Index MCP reports line/column positions as **1-based**.
- Serena's public position model is **0-based**.
- Therefore Index `line: 34` and Serena `start_line: 33` are the same location.
- `open_file(line=33, column=9)` should call Index MCP as `line=34, column=10`.
- Normalize coordinates before reporting an adapter failure.

## Wrapper tests

### 1. `find_file` -> `ide_find_file`
Search for a known source filename and verify returned paths.

### 2. `get_symbols_overview` -> `ide_file_structure`
Choose a real source file and verify meaningful structure is returned.

### 3. `find_symbol` -> `ide_find_symbol`
Find a known symbol and verify its definition and normalized location.

### 4. `find_referencing_symbols` -> `ide_find_references`
Choose a symbol with known usages and verify representative references.

### 5. `search_for_pattern` -> `ide_search_text`
Search for a distinctive pattern and verify returned content and paths.

### 6. `open_file` -> `ide_open_file`
Enable `ide_open_file` under **Settings > Tools > Index MCP Server > Exposed Tools** if necessary. Open a known file first without navigation, then at a known 0-based line/column, and verify JetBrains navigates correctly.

### 7. `open_project` -> `ide_open_project`
Enable `ide_open_project` under **Settings > Tools > Index MCP Server > Exposed Tools** if necessary.

This test is state-changing, so do it only when the user explicitly allows opening another project. Use an existing absolute project directory. Verify that JetBrains opens it and that the result reports the project ready or still indexing. Do not close any existing project as part of this test.

Remember that `ide_open_project` requires at least one project to already be open because the request needs a JetBrains context project. `open_project` takes:

- `path`: absolute project directory
- `auto_link`: optional, defaults to false
- `timeout_seconds`: optional, defaults to 600

## Cross-checking

Watch for:

- failures connecting to `127.0.0.1:29170`,
- disabled `ide_open_file` or `ide_open_project`,
- malformed wrapper/backend responses,
- coordinate mismatches after 1-based/0-based normalization,
- absolute versus relative path mismatches,
- JetBrains dumb/indexing mode,
- missing or duplicate search results,
- incorrect fuzzy symbol matches,
- truncation or pagination issues.

## Report

| Serena tool | Internal Index MCP tool | Result | Notes |
|---|---|---|---|
| `find_file` | `ide_find_file` | PASS / PARTIAL / FAIL | |
| `get_symbols_overview` | `ide_file_structure` | PASS / PARTIAL / FAIL | |
| `find_symbol` | `ide_find_symbol` | PASS / PARTIAL / FAIL | |
| `find_referencing_symbols` | `ide_find_references` | PASS / PARTIAL / FAIL | |
| `search_for_pattern` | `ide_search_text` | PASS / PARTIAL / FAIL | |
| `open_file` | `ide_open_file` | PASS / PARTIAL / FAIL | |
| `open_project` | `ide_open_project` | PASS / PARTIAL / FAIL | |

For every failure include the Serena tool, exact arguments, result/error, and most likely failing layer.

Do not modify repository files during this validation.
