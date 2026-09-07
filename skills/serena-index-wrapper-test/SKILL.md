---
name: serena-index-wrapper-test
description: Validate the five Serena tools in this fork that delegate internally to JetBrains Index MCP. Use when testing plugin installation, wrapper correctness, path/position handling, or diagnosing Serena-to-Index integration failures.
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

## Coordinate-system rule

This is critical when comparing Serena with direct Index MCP output:

- Index MCP reports line/column positions as **1-based**.
- Serena's public position model is **0-based**, matching its existing LSP-oriented APIs and line-based tools.
- Therefore Index `line: 34` and Serena `start_line: 33` are the **same location** and must be treated as a PASS, not an off-by-one failure.
- Normalize before comparing: `serena_line == index_line - 1` and, where applicable, `serena_column == index_column - 1`.
- Only report a conversion bug when the normalized coordinates still disagree with the actual source location.

Do not recommend removing Serena's `-1` conversion merely because the raw numbers differ by one.

## Wrapper tests

### 1. `find_file` -> `ide_find_file`

Search for a known source filename. Record the input, returned paths, correctness, and errors.

Remember that Serena preserves its filename-mask semantics locally while Index MCP uses a query-oriented file search. Compare the final returned files, not just the raw argument shape.

### 2. `get_symbols_overview` -> `ide_file_structure`

Choose a real source file and request its overview. Verify meaningful top-level classes, functions, methods, types, or constants are returned.

### 3. `find_symbol` -> `ide_find_symbol`

Take a symbol from the previous test and find it by name. If supported, constrain the search to its file. Verify the returned definition and location after normalizing Index 1-based coordinates to Serena 0-based coordinates.

### 4. `find_referencing_symbols` -> `ide_find_references`

Choose a symbol with known usages and retrieve its references. Verify representative references against the source code. Normalize coordinates before comparing.

### 5. `search_for_pattern` -> `ide_search_text`

Search for a distinctive string or code pattern. Verify matching files/content and any path filtering behavior. Normalize coordinates before comparing.

## Cross-checking

After each Serena call, use VS Code built-in reads/search only as a verification step. The built-ins are not the primary test path.

Watch for:

- failures connecting to `127.0.0.1:29170`,
- malformed wrapper/backend responses,
- coordinate mismatches **after** normalizing Index 1-based to Serena 0-based positions,
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

For every failure include the Serena tool, exact arguments, returned error/result, normalized coordinate comparison where relevant, and most likely failing layer: VS Code plugin, Serena wrapper, Index MCP client, Index MCP server, or JetBrains project/index state.

Do not modify code during this validation unless explicitly asked.
