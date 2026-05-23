# Cursor

#technology

Cursor is an AI-native code editor built on VS Code. It embeds large language models into the editing workflow—inline completion, chat, multi-file edits, and autonomous agents—while keeping the familiar VS Code extension ecosystem, keybindings, and project layout.

---

## Resources

- **Deep Dives**
	- [Cursor Docs: Get Started](https://cursor.com/docs)
	- [Cursor Docs: Agent](https://cursor.com/docs/agent/overview)
	- [Cursor Docs: Rules](https://cursor.com/docs/context/rules)
	- [Cursor Docs: MCP](https://cursor.com/docs/context/mcp)

- **Docs & References**
	- [Cursor Documentation](https://cursor.com/docs)
	- [Cursor Changelog](https://cursor.com/changelog)
	- [Cursor Forum](https://forum.cursor.com/)

---

## Core Concepts

- **Tab (inline completion)**: Context-aware multi-line suggestions as you type, informed by open files and recent edits.
- **Chat**: Ask questions about the codebase, explain errors, or draft snippets in a side panel without leaving the editor.
- **Composer / Agent**: Multi-step, multi-file changes driven by natural language; the agent can search, edit, run terminal commands, and iterate until the task is done.
- **Codebase indexing**: Embeddings over the project so the model can retrieve relevant files and symbols instead of relying only on what is open.
- **Rules**: Persistent instructions in `.cursor/rules` (or project/user rules) that shape tone, conventions, and guardrails for every session.
- **Skills**: Reusable, markdown-defined playbooks the agent can load for specialized workflows (e.g. PR babysitting, SDK integration).
- **MCP (Model Context Protocol)**: Connect external tools and data sources (browsers, APIs, internal services) so the agent can act beyond the repo.
- **Context control**: `@` mentions for files, folders, docs, and symbols; helps keep prompts focused and within context limits.
- **Privacy modes**: Settings that control whether code is stored or used for training—important for work and client repos.

---
