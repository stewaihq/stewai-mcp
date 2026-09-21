# StewAI

StewAI gives agents protected access to reusable AI workflows. Version 1.1.0 can search and compare
published recipes available to the signed-in account, estimate and run a suitable recipe, or build,
test, qualify, and privately reuse a new one. Authentication uses a one-time email sign-in; users
do not manage an API key.

The server is self-describing: a fresh agent calls `discover_stewai` to learn the connection,
search, authoring, approval, execution, polling, and recovery sequence. Required operational
knowledge is carried by MCP tool contracts and responses rather than hidden in this README.

## Agent workflow

The plugin first searches the currently approved, runnable public catalog and currently authorized
recipes. If an existing capability fits,
the agent previews its contract, dry-runs the cost, obtains any required approval, executes it, and
returns declared results with settled billing.

If no existing recipe fits, the agent checks recent private builds before creating another one. It
then uses free deterministic checks, quoted semantic review, selected-case repair, and a complete
acceptance suite to qualify one exact revision. A successful commit creates a private, unlisted,
unreviewed revision and returns a reference that can be dry-run and executed immediately by its
creator. Recent build activity rows retain fresh poll references so another agent context can
resume an in-flight or completed review instead of paying for a duplicate.

Search returns an explicit `no_suitable_match` outcome instead of promoting a weak result. Preview
declares required/default input state and size limits, and invalid run inputs return bounded,
value-free diagnostics that identify the exact field and repair. Build inspection exposes a
runtime-derived catalog for all 24 recipe node handlers and separates draft validity from
unattended acceptance eligibility.

## Public tools

The MCP server exposes six discovery and execution tools plus eight authoring tools when the
corresponding server flags and OAuth scopes are enabled:

- `discover_stewai`, `search_recipes`, `get_recipe_preview`, `run_recipe`, `get_run`, and
  `get_results`. The result tool keeps its declared-output default and also provides paginated,
  sanitized step traces for runs paid by the signed-in account.
- `start_recipe_build`, `get_recipe_build`, `update_recipe_build`, `check_recipe_build`,
  `review_recipe_build`, `run_recipe_acceptance`, `get_recipe_build_activity`, and
  `commit_recipe_build`.

Agents must dry-run before real execution and quote every paid authoring review or acceptance
action. The dry run reports what past runs of the recipe cost, any fee, the balance, and a
theoretical worst case; the user decides whether to run. Private and organization recipes do not
incur a marketplace creator fee.

The plugin does not expose recipe source, credentials, provider payloads, raw exceptions, or
authoring acceptance-run traces. Owned normal runs of private and public-open recipes expose
sanitized step parameters, available rendered inputs, and normalized results; execute-only public
recipes retain their interface-only redaction. Oversized steps are reported individually with a
continuation cursor so later steps remain reachable. Subrecipe composition uses the supervised
handoff: pass the `recipe_ref` returned by search or preview as `child_recipe_ref`; StewAI resolves
and pins it privately. Bind the exact child recipe and revision, obtain approval, then resume the
same activity.
Version 1.1.0 does not support forking, public marketplace submission, safety approval, public
pricing, or creator compensation.

## Distribution status

Version 1.1.0 is active in the Official MCP Registry as `com.stewai/stewai`. The OpenAI and
Anthropic directory updates are in review. The tagged GitHub release provides deterministic
`stewai-plugin.zip` and `stewai-mcp.zip` downloads; both connect to the production endpoint at
`https://api.stewai.com/mcp`. Claude web, mobile, and Desktop use the remote connector and do not
need either archive.

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
