# commerce-agents

This fork adapts the merchant agent for Spotlight Retail Group's Mirakl dropship
marketplace, scoped to the Spotlight AU banner only. Users are category buyers. Primary
use cases: sales and performance Q&A, and identifying underperforming products for
repricing. Upstream is anthropics/commerce-agents.

## Layout

- `commerce-common/commerce_common/`: what both roles share; its `__init__` lists the modules.
- `shopping-agent/core/shopping_agent/`: types, `StorefrontBackend`, config, prompt, `tools/`, gates, enrichment, executor.
- `merchant-agent/core/merchant_agent/`: the merchant equivalents, plus `changes.py` and `analysis.py`.
- `*/skills/`: five flows per role, one `SKILL.md` each.
- `*/runtime-messages-api/`: `ShoppingAgent`, `MerchantAgent`, the merchant analysis delegate.
- `*/runtime-agent-sdk/`: each agent as `ClaudeAgentOptions`, with a console.
- `*/managed-agents/`: the manifest directory (with the derived `system.md`) and the role's MCP server.
- `examples/demo_common/` and `examples/web-shared/`: what the verticals' APIs and web apps share; `examples/` is the npm workspace.
- `examples/<vertical>/`: `api/`, `data/`, `storefront-web/`, `merchant-web/`; ports 8000-8003, 3000-3003, 3100-3103.
- `plugins/commerce-builder/`: six skills, four commands; `.claude-plugin/marketplace.json` points at it.
- `docs/`: `safety.md`, `backends.md`, `deployment.md`. `scripts/`: install, demo, smoke, screenshots, check, deploy, verify.
- `tests/`: the suites that span packages (both roles on all three paths); each package keeps its own `tests/`.

`requirements.txt` installs the seven packages and their pinned dependencies (`requirements-dev.txt`
adds pytest and ruff); `scripts/install.sh` runs it.

## Design rules

- One model owns the conversation; a rule goes in a tool description, the prompt, or a skill by how often it applies.
- The static prompt and `tools[]` are the same bytes on every turn; per-request data goes in the fenced block after the breakpoint.
- UI is presentation tool calls, validated and filled in on the server, streamed as `ui` events.
- Third-party content is fenced data; writes are provenance-gated and capped in code; `checkout` charges nothing; merchant writes apply only through host approval.
- Core is domain-neutral; a vertical adds UI through `PresentationExtension` and keeps the rest to itself.
- Each mechanism is defined once, in `commerce_common` or a role core, and shared by all three paths.

## Real data

This fork intentionally uses real SRG brand, supplier and product references — the
upstream fictional-only rule does not apply to additions made here. Examples under
examples/ stay fictional and untouched. Development data is scrambled: real schema and
real identifier formats, fake values.

## Environments

Development runs in a GitHub Codespace against the direct Anthropic API with scrambled
data. Production will run as an Azure Container App with Claude reached via Microsoft
Foundry against live data. Model endpoint, authentication, data source and hosting
configuration must always come from environment variables. Never hardcode them —
promoting to Foundry has to be a configuration change, not a code change.

## Working rules

Prefer editing definition files — skills, config, guardrails — over writing Python. If a
change requires a new Python module, explain why before writing it. One working change
per commit, with messages explaining why rather than what. Show me the diff before
committing. Ask before adding any dependency.

## Decisions

Settled decisions live in DECISIONS.md. Read it before proposing any design. Treat
entries there as inputs, not starting positions.

## Conventions

- Python 3.11+, `ruff` (root `ruff.toml`), `pytest` (root `pytest.ini`), type hints, `pydantic` schemas; web apps are Next.js and TypeScript.
- Skill descriptions name the request class, without sample utterances; tool descriptions say when the tool applies; examples stay schematic.
- A change to prompt text, a tool description, a skill, or a fence notice re-derives `system.md`; `scripts/check.py` compares them.
- Prose: plain declarative sentences; one term per thing; each fact once, naming its module; each role in its own terms; a README says what a thing is, how to run it, and where its interfaces are; no history, dates, or process narrative; cut before restyling.
- A new module updates this file and its README.

## Verify

```bash
ruff check . && ruff format --check . && pytest && python scripts/check.py
python scripts/verify_all.py          # adds deploy dry runs and web builds
```
