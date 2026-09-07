---
name: serena-index-workflow
description: Use when working in VS Code with the Serena Index agent plugin. Explains project activation, which Serena tools are backed by JetBrains Index MCP, and how to choose Serena versus VS Code built-in tools.
---

# Serena Index workflow for VS Code

This plugin exposes **one MCP server to the agent: Serena**.

JetBrains Index MCP is an internal backend used by selected Serena tools. Do not look for or call a separate direct `ide_*` MCP server.

## Start by activating the workspace

The Agent Plugins runtime starts MCP processes from the installed plugin directory rather than from the open VS Code workspace. Therefore, when Serena has no active project, call `activate_project` with the current workspace root before using project-scoped tools.

If Serena already reports the correct active project, do not activate it again.

## Index-backed Serena discovery and navigation tools

Prefer these Serena tools for source-code discovery before broad file reads or raw text search:

- `get_symbols_overview` -> internal `ide_file_structure`
- `find_symbol` -> internal `ide_find_symbol`
- `find_referencing_symbols` -> internal `ide_find_references`
- `find_file` -> internal `ide_find_file`
- `search_for_pattern` -> internal `ide_search_text`
- `open_file` -> internal `ide_open_file`
- `open_project` -> internal `ide_open_project`

These are Serena tool names. The underlying `ide_*` calls are implementation details and are not agent-visible tools in this plugin.

`open_file` accepts a project-relative path plus optional Serena-style 0-based `line` and `column`. It converts those coordinates to Index MCP's 1-based navigation parameters.

`open_project` accepts an absolute filesystem path plus optional `auto_link` and `timeout_seconds`, opens the target in JetBrains, and waits for indexing. The current Serena project provides the Index MCP request context.

## Other Serena tools

Use Serena's remaining project, memory, symbolic editing, and refactoring tools when they fit the task. Capabilities that have not yet been wrapped through Index MCP remain Serena-native for now.

Use VS Code built-in file/search/edit/shell tools when:

- the target is non-code or generated content,
- the content is not indexed or parseable,
- Serena cannot express the operation cleanly,
- or a raw text operation is genuinely the correct abstraction.

Do not call `initial_instructions` as a routine startup step. The Serena `vscode` context and this skill already contain the integration guidance.

## Runtime expectations

Serena itself is launched from the fork with:

```text
uvx -p 3.13 --from git+https://github.com/Areo-RGB/serena-main serena start-mcp-server --context=vscode
```

The Index-backed wrappers expect the local JetBrains Index MCP service at:

```text
http://127.0.0.1:29170/index-mcp/streamable-http
```

`ide_open_file` and `ide_open_project` are opt-in/disabled-by-default Index MCP tools, so enable them in **Settings > Tools > Index MCP Server > Exposed Tools** before using Serena `open_file` or `open_project`.

`ide_open_project` requires at least one project to already be open in JetBrains so the MCP request has a context project.

If an Index-backed wrapper fails because the backend is unavailable, report that clearly and use an appropriate Serena-native or VS Code fallback rather than repeatedly retrying the same failing call.
