# Turborepo

#technology

Turborepo is a high-performance build system for JavaScript and TypeScript monorepos. It speeds up builds and tasks by caching, parallelizing, and intelligently orchestrating work across multiple packages and apps in a repo.

---
## Resources

- **Deep Dives**  
	- [Turborepo Docs: Build System](https://turbo.build/repo/docs)  
	- [Turborepo with Docker Guide](https://turbo.build/repo/docs/guides/tools/docker)  

- **Tools & CLI**  
	- [Turbo CLI Reference](https://turbo.build/repo/docs/reference/cli)  

---

## Core Concepts

- **Monorepo Structure**: Organize code in `/apps`, `/packages` folders, each with own `package.json`.
- **Caching**: Stores previous task results to avoid redundant work on rebuilds.
- **Incremental Builds**: Only rebuild what changed and affected downstream dependencies.
- **Task Pipelines**: Define `turbo.json` to control task dependencies and run order.
- **Pruning for Docker**: `turbo prune` trims repo to only relevant files for Docker builds, optimizing layers and cache.
- **Remote Caching & Cloud**: Share build cache across machines to speed up CI and developer workflows.
- **Parallel Execution**: Runs tasks in parallel where possible to maximize CPU use.

---
