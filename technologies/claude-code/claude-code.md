# Claude Code

#technology

Claude Code is Anthropic’s agentic coding tool that runs in the terminal. You describe goals in natural language; it reads and edits files, runs shell commands, and uses MCP-connected tools to complete software tasks—similar in spirit to IDE agents, but optimized for CLI-first and CI/automation workflows.

---

## Resources

- **Deep Dives**
	- [Claude Code Docs: Overview](https://docs.anthropic.com/en/docs/claude-code/overview)
	- [Claude Code Docs: Common workflows](https://docs.anthropic.com/en/docs/claude-code/common-workflows)
	- [Claude Code Docs: MCP](https://docs.anthropic.com/en/docs/claude-code/mcp)
	- [Claude Code Docs: Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)

- **Docs & References**
	- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
	- [Anthropic API Docs](https://docs.anthropic.com/en/api/getting-started)
	- [Claude Code GitHub](https://github.com/anthropics/claude-code)

---

## Core Concepts

- **Terminal-native agent**: Operates from the shell in a project directory; no separate IDE required, though it pairs well with any editor.
- **CLAUDE.md**: Project-level instructions (stack, commands, conventions) loaded automatically so behavior stays consistent across sessions.
- **Tool use**: Built-in tools for file read/write, search, bash, and optional MCP servers for browsers, issue trackers, and custom APIs.
- **Permission model**: Prompts before destructive or sensitive actions (e.g. writes, network, certain commands); can be tuned for trusted environments.
- **Subagents / task delegation**: Spawns focused workers for exploration, shell, or review so the main session stays on the critical path.
- **Hooks**: Scripts triggered on lifecycle events (e.g. before/after tool use) to enforce linting, logging, or org policies.
- **MCP integration**: Same protocol as other AI tools—plug in databases, docs, or internal services without bespoke integrations per task.
- **Headless / CI usage**: Runnable non-interactively for scripts, pipelines, and automation where an API key and clear prompts are available.
- **Model selection**: Uses Anthropic Claude models; choice of model affects speed, cost, and depth of reasoning for hard refactors or debugging.

---
