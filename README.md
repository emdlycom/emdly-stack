# emdly-stack

Open-source collection of reusable AI agent skills for [emdly.com](https://emdly.com).

`emdly-stack` contains community-shared `.md` skill files that give AI agents clear instructions for performing specific tasks, workflows, and operations. Each skill is a plain Markdown document with a stable identity, making skills easy to share, version, reuse, and integrate into different agent workflows.

## Access your skills anywhere

Skills published on emdly can be consumed in multiple ways:

* **CLI** — install skills directly into your project with `npx @emdly/cli add owner/skill`
* **Raw URLs** — fetch any skill directly as Markdown, with support for both latest and pinned versions
* **MCP** — connect the entire emdly catalog to AI agents and let them search, load, and install skills automatically
* **REST API** — programmatically discover, retrieve, publish, and version skills
* **Markdown / LLM endpoints** — access emdly content in machine-readable formats

For example, a skill can be installed directly into a project:

```bash
npx @emdly/cli add owner/skill
```

Or accessed through its raw URL:

```text
https://emdly.com/raw/{owner}/{skill}.md
```

Agents can also connect to the emdly MCP server and discover relevant skills dynamically instead of manually searching for them.

## Built for the agentic development ecosystem

The goal of `emdly-stack` is to make useful agent skills **portable, discoverable, and reusable**.

Instead of keeping useful agent instructions hidden inside individual projects, developers can share them with the community and make them available to other agents and developers.

Skills can be versioned, reviewed, and reused across different projects and AI coding environments.

Contributions are welcome. If you have a useful skill, workflow, or set of agent instructions, add it to the stack and share it with the community.

**Discover, share, and reuse AI agent skills with emdly.**
