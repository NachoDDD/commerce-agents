# Decisions

Settled decisions for this fork, numbered D-01, D-02, ... Each entry is dated, covers one
decision, and states the decision and the reasoning behind it. Once an entry is added,
treat it as an input to future work, not a starting position to relitigate.

## D-01 — 2026-09-13 — `allowed_hosts=["*"]` in host.py is dev-only

`examples/demo_common/host.py`'s `TrustedHostMiddleware` is set to
`allowed_hosts=["*"]`, disabling Host-header validation entirely. This was
needed in the GitHub Codespaces dev environment because a Codespace forwards
ports through a dynamic `*.app.github.dev` hostname and rewrites the Host
header, so a static localhost-only allowlist rejected forwarded requests.

Before production (Azure Container App via Microsoft Foundry, see
Environments in CLAUDE.md), this must become a narrow allowlist of the
actual production hostname(s) rather than a wildcard — a wildcard host
allowlist defeats the middleware's purpose and would leave the production
service open to Host-header attacks (cache poisoning, password-reset
poisoning, etc.). The allowlist should come from an environment variable
per CLAUDE.md's Environments rule, not be hardcoded.
