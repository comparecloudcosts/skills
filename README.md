# Compare Cloud Costs — Skills

Agent skills for working with [Compare Cloud Costs](https://comparecloudcosts.com) —
vendor-neutral cloud and AI pricing across 26 providers, refreshed about once a week.
The skills give your AI assistant the judgment for the catalog; the
[MCP server](https://github.com/rodrigo-orzari/ccc-mcp-server) gives it the tools.

## Skills

| Skill | What it does |
| --- | --- |
| [compare-cloud-costs](skills/compare-cloud-costs/) | Compare cloud and AI infrastructure pricing across providers with the CCC MCP server. On-Demand prices only, 5% tie band, pricing table first and questions after, never invent a price. |

## Install

**skills CLI (Claude Code, Antigravity, Cursor, and other agents):**

```bash
npx skills add comparecloudcosts/skills --skill compare-cloud-costs
```

**Claude Code plugin (installs the skill versioned under one name):**

```
/plugin marketplace add comparecloudcosts/skills
/plugin install compare-cloud-costs@compare-cloud-costs
```

The skill is invoked as `/compare-cloud-costs:compare-cloud-costs` and loads automatically
when your question matches its description (cloud costs, cheapest provider, AWS vs Azure
vs GCP pricing, AI model token prices, workload cost estimates, datacenter regions).

## Works everywhere — one skill, many agents

`SKILL.md` is one open standard ([agentskills.io](https://agentskills.io)), so this skill
runs without modification on Claude Code, OpenAI Codex, Google Gemini CLI, GitHub Copilot
(VS Code), Cursor, Windsurf, OpenClaw, Cline, OpenCode, Goose, Antigravity, and more.
Install with the skills CLI for your agent:

```bash
npx skills add comparecloudcosts/skills --skill compare-cloud-costs --agent <agent>
```

(For example `--agent codex`, `--agent antigravity`, `--agent cursor`.) Codex also
installs it as a plugin — this repo ships `.codex-plugin/plugin.json` next to the Claude
manifest in `.claude-plugin/plugin.json`.

## The MCP server (required)

The skill needs the Compare Cloud Costs MCP server connected. It is free:

1. Get a free API key at **https://comparecloudcosts.com/mcp** (Generate preview key, no card).
2. Add it to your MCP client config — see the
   [MCP server README](https://github.com/rodrigo-orzari/ccc-mcp-server#install)
   for Claude Desktop, Cursor, and VS Code. Requires Node.js ≥ 18.

```json
{
  "mcpServers": {
    "compare-cloud-costs": {
      "command": "npx",
      "args": ["-y", "@comparecloudcosts/mcp-server"],
      "env": { "CCC_API_KEY": "ccc_live_your_key_here" }
    }
  }
}
```

## Links

- Product: https://comparecloudcosts.com
- MCP server (npm `@comparecloudcosts/mcp-server`, official MCP Registry): https://github.com/rodrigo-orzari/ccc-mcp-server
- Get a free API key: https://comparecloudcosts.com/mcp
- Methodology: https://comparecloudcosts.com/methodology

## Contributing

New skills go in `skills/<name>/SKILL.md` with YAML frontmatter (`name` + `description` —
the description is what agents match on, so make it trigger-rich). Add a row to the table
above and to `skills/README.md`, then see [PUBLISHING.md](PUBLISHING.md).
