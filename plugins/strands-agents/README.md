# Strands Agents — Codex plugin

A [Codex plugin](https://developers.openai.com/codex/plugins/build) providing a skill for building AI agents with the [Strands Agents SDK](https://strandsagents.com) (AWS-started, community-governed open-source agent SDK for Python and TypeScript).

## What's inside

- **Skill: `building-agents-with-strands`** — when to choose Strands (agent loop with lifecycle hooks, tools/MCP, multi-agent orchestration, sessions/memory, model portability, streaming, structured output, observability), install commands, minimal usage, and copy-pasteable recipes for adding Strands to an existing **FastAPI**, **Express**, or **Next.js** app (`skills/building-agents-with-strands/references/`).

## Install in Codex

Use the `$skill-installer` in Codex with this repository's plugin directory, or follow the [Build plugins](https://developers.openai.com/codex/plugins/build) guide to add a skill-only plugin from this repo.

## Also available

The same skill ships as a [Claude Code plugin](https://github.com/tmoreton/claude-code-plugin-strands) (with the Strands docs MCP server and a `/strands-docs` command) and as a Cursor rule (`.cursor/rules/strands-agents.mdc`).

## License

Apache-2.0