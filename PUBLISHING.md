# Publishing the Compare Cloud Costs skills repo

Draft layout is ready in this folder. Nothing here is on GitHub yet — pushing is a
Rodrigo step (the Muse GitHub App cannot create repos).

## Decisions to confirm before pushing

1. **GitHub org: `comparecloudcosts` — DECIDED Nov 6.** Rodrigo picked `comparecloudcosts`
   over `rodrigo-orzari`. **Verified Nov 6 23:05 PT: the org does NOT exist yet —
   github.com/comparecloudcosts is a 404, so the name is free.** He must create the org
   first (GitHub Settings → Organizations → New organization, free for public repos),
   then create the `skills` repo inside it. Fallback if he declines: `rodrigo-orzari/skills`
   — replace `comparecloudcosts` with `rodrigo-orzari` in README.md, `.claude-plugin/plugin.json`
   (`repository`), and `.claude-plugin/marketplace.json` (`owner.name`).
2. **License: MIT assumed.** `LICENSE` names Cosell Plus LLC (the company behind CCC,
   per the site footer). Confirm MIT, or swap the file and the `license` field in
   `plugin.json`.
3. **Version: `1.0.0`.** Bump `version` in `plugin.json` on every published change —
   marketplace users only receive updates when it changes.

## Steps

1. Create the public repo (empty, no README — this draft supplies it):
   `gh repo create <org>/skills --public` (or create it in the browser).
2. Push this folder:
   `git init && git add -A && git commit -m "Compare Cloud Costs skills" && git branch -M main && git remote add origin git@github.com:<org>/skills.git && git push -u origin main`
3. Add discovery topics (feeds skills.sh / SkillsMP / GitHub search):
   `gh repo edit <org>/skills --add-topic agent-skills --add-topic claude-skills --add-topic codex-skills --add-topic skill-md --add-topic ai-agents`
4. Validate:
   - `claude plugin validate .claude-plugin/plugin.json` (authoritative check for the manifest)
   - `claude plugin validate .claude-plugin/marketplace.json`
   - `gh skill publish --dry-run` (validates all skills against the agentskills.io spec; needs the `skills-sh/skill-publish` extension: `gh extension install skills-sh/skill-publish`)
   - `npx skills add <org>/skills --skill compare-cloud-costs` from a clean machine — the skill must install and load.
5. Cut a release (this is what the ecosystem recognizes):
   `gh skill publish --tag v1.0.0`
6. skills.sh discovery — submit the index request (skills.sh does NOT auto-index):
   `gh issue create --repo vercel-labs/skills --title "Request to index skill: <org>/skills" --body "Repository: https://github.com/<org>/skills"` — may take days to process. `npx skills add` works immediately without it; only search/discovery waits on indexing.
7. Link it from the MCP side so the two distributions reinforce each other:
   - MCP server README (`rodrigo-orzari/ccc-mcp-server`): add a "Use with Claude" section pointing at this repo.
   - https://comparecloudcosts.com/mcp page: link the skill repo next to the key generator.

## Distribution channels — beyond Claude

Publish once (this repo), and the skill flows to many registries because the format is
one open standard. Most of them index public GitHub repos automatically; only the ones
marked "submit" need an action from you.

- **skills.sh (Vercel)** — submit index request (step 6 above). Install analytics and
  the leaderboard live here. `npx skills add` works into 22 agents.
- **SkillsMP (skillsmp.com)** — auto-indexes public repos with SKILL.md files. No action.
- **LobeHub, Claude Market, agentskill.sh, claudeskills.info, MCP Market** — aggregators
  that index public skill repos. No action; they pick it up.
- **ClawHub (clawhub.ai)** — OpenClaw's official marketplace, the only one with
  platform-level security screening (Skill Vetter / SkillScan, signed manifests,
  moderated releases). Submit the repo there for the OpenClaw audience.
- **Claude community marketplace (`claude-community`)** — the public community catalog
  next to Anthropic's curated `claude-plugins-official`. Submissions are reviewed before
  listing. This repo's `.claude-plugin/marketplace.json` is the entry point.
- **OpenAI Codex** — this repo ships `.codex-plugin/plugin.json` (same shape as the
  Claude manifest). Validate it against the Codex plugin docs before the first release —
  the file's shape is mirrored from the Claude manifest, not yet verified against Codex.
- **Agensi** — the only marketplace with creator payments (80/20 split), manual +
  automated 8-point scan. Optional; only if you ever want to monetize a skill.

- New skills: add `skills/<name>/SKILL.md`, a row to both READMEs, bump `version` in
  `plugin.json`, cut a new release (`gh skill publish --tag vX.Y.Z`).
- The skill's `description` frontmatter is the discovery surface — every install and
  every search match reads it first. Keep it trigger-rich (what it does AND when to use it).
