# StewAI

StewAI gives agents protected access to reusable AI workflows. The plugin can show fixed examples
without sign-in. Live marketplace search, recipe previews, estimates, execution, and final results
use a one-time email sign-in. You do not manage an API key.

## Public tools

- `discover_stewai` shows fixed workflow examples without sign-in.
- `search_recipes` searches the live marketplace after email sign-in.
- `get_recipe_preview` shows safe inputs, outputs, and historical credit bands for an opaque
  recipe reference. These bands are not input-aware estimates or affordability checks.
- `run_recipe` estimates credits or starts a previewed recipe with idempotent execution.
- `get_run` reads owned-run status and, after terminal settlement, the authoritative charged
  credits. Execution-capable connections also receive the current available account balance.
- `get_results` reads available final author-declared outputs from a completed owned run.

Agents must use `run_recipe(dry_run=true)` before recommending execution. Search and preview
history never replace that estimate. Terminal billing can remain pending briefly; clients poll at
the supplied interval for up to five minutes without exposing partial charges.

If execution returns `account_unavailable`, reconnect with an ordinarily provisioned Free account
or contact StewAI support. This account-state error is distinct from invalid recipe inputs and is
never repaired by weakening tenant ownership checks.

This repository currently produces a validated distribution artifact. Directory installation is
not yet published, and the archive is not advertised as a one-click install. Verified publication
and installation instructions will accompany the public directory release.

## Links

- Website: https://stewai.com
- Privacy: https://stewai.com/privacy
- Terms: https://stewai.com/terms
- Imprint: https://stewai.com/imprint
- Support: support@stewai.com

## License boundary

Copyright 2026 Christian Mueller.

The plugin manifest, MCP configuration, `stewai-mcp` skill, plugin README, and plugin icons are
licensed under Apache-2.0. The complete license text is in `LICENSE`. This distribution license
does not relicense StewAI, its backend, its Python SDK, or marketplace data, which remain subject
to their existing terms. The StewAI name and logo remain protected, and no additional trademark
permission is granted.
