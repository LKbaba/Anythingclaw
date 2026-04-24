## Development Environment
- OS: Windows 10.0.19045
- Shell: Git Bash
- Path format: Windows (use forward slashes in Git Bash)
- File system: Case-insensitive
- Line endings: CRLF (configure Git autocrlf)

## Codex MCP Guide

Codex is an autonomous coding agent by OpenAI, integrated via MCP.

Workflow: Claude plans architecture → delegate scoped tasks to Codex → review results
- `codex` tool: start a session with prompt, sandbox, approval-policy
- `codex-reply` tool: continue a session by threadId for multi-turn tasks
- Pass project context via `developer-instructions` parameter
- Recommended: sandbox='workspace-write', approval-policy='on-failure'

Prerequisite: `npm i -g @openai/codex`, OPENAI_API_KEY configured
