---
name: serena-index-wrapper-test
description: Validate the five Serena tools in this fork that delegate internally to JetBrains Index MCP. Use when testing plugin installation, wrapper correctness, path/position handling, or diagnosing Serena-to-Index integration failures.
---

# Test Serena's Index MCP wrappers

Validate the architecture:

```text
VS Code agent -> Serena MCP -> Index MCP -> JetBrains project index
```

Do not call direct `ide_*` tools. They should not be exposed by this plugin.

## Preparation

1. Confirm the `serena-index` MCP server is connected.
2. If no project is active, call `activate_project` with the current VS Code workspace root.
3. Confirm these Serena tools are available:
   - `find_file`
   - `get_symbols_overview`
   - `find_symbol`
   - `find_referencing_symbols`
   - `search_for_pattern`

## Wrapper tests

### 1. `find_file` -> `ide_find_file`

Search for a known source filename. Record the input, returned paths, correctness, and errors.

### 2. `get_symbols_overview` -> `ide_file_structure`

Choose a real source file and request its overview. Verify meaningful top-level classes, functions, methods, types, or constants are returned.

### 3. `find_symbol` -> `ide_find_symbol`

Take a symbol from the previous test and find it by name. If supported, constrain the search to its file. Verify the returned definition and location.

### 4. `find_referencing_symbols` -> `ide_find_references`

Choose a symbol with known usages and retrieve its references. Verify representative references against the source code.

### 5. `search_for_pattern` -> `ide_search_text`

Search for a distinctive string or code pattern. Verify matching files/content and any path filtering behavior.

## Cross-checking

After each Serena call, use VS Code built-in reads/search only as a verification step. The built-ins are not the primary test path.

Watch for:

- failures connecting to `127.0.0.1:29170`,
- malformed wrapper/backend responses,
- 0-based versus 1-based line/column conversion errors,
- absolute versus relative path mismatches,
- missing or duplicate results,
- incorrect fuzzy symbol matches,
- wrong overload/reference resolution,
- truncation or pagination issues,
- mismatches between Serena arguments and Index MCP semantics.

## Report

Finish with this table:

| Serena tool | Internal Index MCP tool | Result | Notes |
|---|---|---|---|
| `find_file` | `ide_find_file` | PASS / PARTIAL / FAIL | |
| `get_symbols_overview` | `ide_file_structure` | PASS / PARTIAL / FAIL | |
| `find_symbol` | `ide_find_symbol` | PASS / PARTIAL / FAIL | |
| `find_referencing_symbols` | `ide_find_references` | PASS / PARTIAL / FAIL | |
| `search_for_pattern` | `ide_search_text` | PASS / PARTIAL / FAIL | |

For every failure include the Serena tool, exact arguments, returned error/result, and most likely failing layer: VS Code plugin, Serena wrapper, Index MCP client, Index MCP server, or JetBrains project/index state.

Do not modify code during this validation unless explicitly asked.
