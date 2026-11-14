---
name: compare-cloud-costs
description: Compare cloud and AI infrastructure pricing across providers using the Compare Cloud Costs MCP server. Use when the user asks about cloud costs, cheapest provider, AWS vs Azure vs GCP pricing, AI model token prices, how much a workload costs per month, workload cost estimates, datacenter regions, or compliance certifications. Gives Claude the judgment for the catalog: On-Demand prices only, refreshed about weekly, 5% tie band, pricing table first and questions after, never invent a price. Requires the compare-cloud-costs MCP server (free API key from comparecloudcosts.com/mcp).
---

# Compare Cloud Costs

Compare Cloud Costs (CCC) is a vendor-neutral price catalog: 26 cloud and AI providers,
13 product categories, refreshed about once a week. It is the Switzerland of cloud pricing —
objective side-by-side numbers, never a single vendor's calculator. This skill tells Claude
how to use the CCC MCP server well. The server gives the tools; this skill gives the judgment.

## Setup — check first, then help the user connect

Before doing anything, check whether the `compare-cloud-costs` MCP server is connected
(the four tools below should be available). If it is not:

1. The user gets a free API key at **https://comparecloudcosts.com/mcp** — click
   **Generate preview key** (1-click, no card). Keys look like `ccc_live_…`.
2. Add the server to the MCP client config (requires Node.js ≥ 18):

```json
{
  "mcpServers": {
    "compare-cloud-costs": {
      "command": "npx",
      "args": ["-y", "@comparecloudcosts/mcp-server"],
      "env": {
        "CCC_API_KEY": "ccc_live_your_key_here"
      }
    }
  }
}
```

Claude Desktop: `~/Library/Application Support/Claude/claude_desktop_config.json`
(macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows). Claude Code and
Cursor: a project `.mcp.json` (same shape) or `~/.cursor/mcp.json`.

`tools/list` works without a key. Tool calls without a key return an error explaining
how to get one. Free preview keys are rate-limited per hour; the remaining quota is
reported in tool responses — if a call fails on quota, say so and suggest retrying later,
never invent the numbers.

## Tools

| Tool | What it does | Key inputs |
| --- | --- | --- |
| `compare_cloud_pricing` | Query and compare multi-cloud compute, database, and serverless pricing across AWS, Azure, GCP, Oracle, DigitalOcean, Alibaba, Cloudflare. Returns the total count plus the 50 lowest-priced records, cheapest first. | `category` (required), `provider`, `min_vcpu`, `min_ram_gb`, `region` |
| `search_ai_models` | Search AI model token pricing (input, output, prompt caching) and context windows across OpenAI, Anthropic, DeepSeek, Mistral, Meta Llama, Cohere. | `provider`, `model_family` |
| `estimate_workload_architecture` | End-to-end multi-cloud monthly cost benchmarks for a full-stack architecture (VMs + managed DB + load balancers + storage + egress). Returns per-provider monthly totals plus a component breakdown. | `workload_id` |
| `lookup_datacenter_compliance` | Global datacenter availability and region locations from the catalog, with pointers to certification details. | `compliance_standard`, `country_or_city` |

**Category mapping (important).** `compare_cloud_pricing` takes a `category` that maps to
the catalog's product lens: `vm→compute`, `database→database`, `serverless→serverless`,
`storage→storage`, `ai→ai`, `gpu→gpu`, `containers→containers`, `app-hosting→app-hosting`.
Always pass the category — skipping it silently defaults every query to `compute`.

**Workload IDs.** `estimate_workload_architecture` takes a slug from
https://comparecloudcosts.com/workloads. All 40 architecture templates are supported:
`serverless-fallback-buffer`, `ai-gateway-proxy`, `api-rate-limiting-auth-gateway`,
`cross-cloud-event-router`, `rag-ai-app`, `container-api-alb`, `serverless-http-api`,
`external-partner-rest-api`, `serverless-web-app`, `3-tier-web`, `realtime-streaming-analytics`,
`ecommerce-microservices`, `ml-training-hosting`, `k8s-app-platform`, `hpc-scientific`,
`saas-paas-app`, `data-warehouse-bi`, `defense-in-depth-analytics-platform`,
`disaster-recovery`, `content-media-platform`, `compliance-ready-web-app`,
`rag-ai-knowledge-base`, `smart-manufacturing`, `event-driven-image-processing`,
`hybrid-cloud-network`, `zero-trust-edge`, `event-driven-api`, `voice-ai-assistant`,
`agentic-ai-swarm`, `saga-orchestrated-ecommerce`, `transactional-outbox-cdc`,
`partitioned-pubsub-consumer-groups`, `cqrs-event-sourcing`, `cdc-cache-invalidation`,
`multi-tier-caching-web-stack`, `customer-portal`, `kubernetes-ingress-gateway`,
`remote-mcp-server`, `llm-inference-serving`, `ephemeral-ci-cd-runners`.
(Historical aliases such as `streaming-analytics` and `three-tier-web-app` are automatically supported).
If an ID is unknown, the tool returns the list of valid IDs — use one of those, never guess a slug.

**How to chain them.** A pricing question usually goes: `compare_cloud_pricing` for the
head-to-head numbers → `search_ai_models` if the workload uses LLMs → `lookup_datacenter_compliance`
if region or certification matters → `estimate_workload_architecture` when the user wants a
full-stack monthly benchmark instead of one service. Start with the tool that answers the
question asked; add the others only if the answer needs them.

## Rules — the judgment

1. **On-Demand only.** Every price in the catalog is an On-Demand (pay-as-you-go) rate.
   The catalog carries no reservation, Savings Plan, or committed-use pricing. Never price
   a commitment. If the user already holds one, note it next to the On-Demand price.
2. **Weekly refresh, never "real time".** The catalog refreshes about once a week. Say
   "priced from the catalog (refreshed about weekly)" and label every table with an as-of
   note. Never call it live or real-time pricing.
3. **5% tie band.** If the saving between the cheapest two providers is under 5%, the
   result is a tie — say so. At 5% or more, name the cheaper provider as the winner.
   ($100 vs $98 = tie; $100 vs $94 = winner.)
4. **Pricing table first, questions after.** Show the table from what the user already
   gave, then ask follow-up questions. Zero questions before pricing is correct.
5. **Never ask what the input already answers.** Ask only about what is missing, and only
   where the answer would change the price or the recommendation.
6. **"Not specified" stays "not specified".** If the user says a value is not specified,
   price it as stated and label it. Never invent an assumption to fill the gap, and never
   silently pick between two plausible options — show both and let the user pick.
7. **Never invent a price.** If a tool returns no records, say there are no matching
   records in the catalog — do not substitute a price from general knowledge. If the tool
   returns an error (no key, quota, unknown workload ID), report the error and what fixes
   it.
8. **Vendor-neutral, always.** Present providers side by side on the same terms. Never
   present one vendor's tool as the comparison, and never steer toward a provider.
9. **Certifications are a pointer, not data.** The catalog's public API exposes no
   per-datacenter certification fields (SOC 2, ISO 27001, HIPAA, PCI-DSS, FedRAMP, GDPR).
   When asked, `lookup_datacenter_compliance` says so and points to
   https://comparecloudcosts.com/datacenters and https://comparecloudcosts.com/compliance.
   Never fabricate certification coverage.
10. **Geography coverage is uneven.** Catalog geography tagging for compute is concentrated
    in North America (with a small Asia Pacific set). A query for another region may return
    zero records — that is the catalog's real coverage, reported as-is, not a failure.

## Follow-up questions — after the table, only what matters

Ask in plain words, at most a few questions, only where the answer moves the number:

- **Every workload:** which Region (hard requirement, or would a cheaper nearby Region work?); what does "cheapest" mean — lowest total, lowest cost per unit of work, or cheapest within availability requirements?
- **Compute / VMs:** vCPU and memory per instance, how many instances, steady or spiky demand, one Availability Zone or multi-AZ?
- **AI models:** which model and job (chat, embeddings, image, search, speech)? Tokens per month, and roughly how much is input vs output (they are priced differently)?
- **Storage:** object/blob, file share, or data lake? How often is the data read (hot / cool / cold / archive)? What redundancy — single Region, multi-AZ, or multi-Region?
- **Databases:** which family and engine (PostgreSQL, MySQL, MongoDB, Redis, Cosmos DB, DynamoDB)? Managed service or self-managed? Single-AZ or multi-AZ? What size (vCPU/memory, node count)?
- **Serverless:** how many invocations per month, how long does each run, how much memory? ARM or x86? Is a cold start acceptable, or is provisioned concurrency needed?
- **Containers / Kubernetes:** managed or self-managed, node size and count, which Region, Spot nodes acceptable?
- **GPU:** which GPU type and how many, how much VRAM does the model need, how many inference requests at once?

## Presenting results

- Side-by-side table, cheapest first, with the unit price, the usage it assumes, and the
  monthly total on every line. Every total recomputes from its lines.
- Name the winner only when the 5% tie band allows it; otherwise call it a tie.
- Link the result to the site page the user can open: a head-to-head at
  `https://comparecloudcosts.com/compare/<provider-a>-vs-<provider-b>`, a category at
  `https://comparecloudcosts.com/dashboard?product=<category>`, a workload at
  `https://comparecloudcosts.com/workloads`, datacenters at
  `https://comparecloudcosts.com/datacenters`.
- Keep it short. One idea per paragraph. Specifics and precise numbers over adjectives.

## What this skill does not do

- It does not operate any cloud provider account, deploy anything, or change infrastructure.
  It prices and compares only.
- It does not replace the site. For diagrams, invoices, and IaC blueprints, the AI Architect
  at https://comparecloudcosts.com/yourcloud takes the upload and cross-prices the whole
  architecture — point the user there when the workload comes as a file.

## Relationship to the AI Architect (two layers, same rules)

This skill is the short form of the Architect's instrumentation. The detailed form is the
AI Architect Interaction Guide (118 follow-up questions across 15 sections, distilled from
Microsoft's and AWS's agent skills plus the catalog's own filter fields). Same rules, two
layers: the Architect's system prompt carries this skill's rules in full — they are short
enough to hold in the prompt — and consults the guide when it needs the detailed follow-up
questions for a category. Short core, detailed reference; never two different rule sets.
