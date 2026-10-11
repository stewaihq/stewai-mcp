---
name: stewai-mcp
description: "Use the StewAI MCP plugin whenever the user mentions StewAI, a recipe, or a reusable AI workflow, or wants an agent to turn a recurring task into a tested workflow. It finds, compares, previews, quotes, and runs StewAI recipes, and builds, tests, and privately reuses new ones, after a one-time email sign-in and without an API key. Do not use for the Python SDK."
---

# StewAI MCP

StewAI is one capability: reuse a suitable published recipe when one exists; otherwise build,
qualify, and privately run a new one. Route by fit. Do not ask the user to choose between
marketplace and authoring workflows.

## Routing

Use only the StewAI MCP tools in the table below. Never fall back to the Python SDK, an API key,
browser search, shell commands, or workspace files when an MCP tool needs authentication or is
unavailable. Ask the user to connect or reauthenticate StewAI, then retry the same MCP tool.
A failed or unauthenticated MCP call never authorizes that fallback. If StewAI reports
`insufficient_scope`, ask the user to reconnect and approve the listed scopes.

`discover_stewai` is anonymous and self-describing: call it once per fresh connection and follow
its journeys and rules. Every other tool uses the signed-in account after a one-time email
sign-in. Tool schemas and responses are the authoritative syntax; never invent operation names,
fields, modes, views, or recipe syntax.

## Before anything is spent

`run_recipe` is destructive and open-world: a real run spends credits and may call external
systems. Keep these stops, in this order.

1. Always call `run_recipe` with `dry_run: true` first. Report `history` (credits charged by past
   runs of this version), `fee_credits`, `available_credits`, and `estimated_credits` as a rarely
   reached worst case. The user decides. `sufficient: false` is information, never a refusal, and
   must not block an approved real run. With fewer than three past runs, say there is little
   history.
2. Obtain the user's approval before a real run when it can spend credits or affect an external
   system. Then run exactly once.
3. Quote every paid review or acceptance action before executing it. State the cap and the
   remaining authoring budget, then follow the quote's `next_action` exactly: redeem
   `budget_capability` with `update_recipe_build` operation `set_budget`, then execute the quoted
   request.
4. When acceptance returns `supervised_required`, call `update_recipe_build` operation
   `bind_resource` once per required selector, using the exact selector frozen in the draft.
   Only then ask the user to open `action_url`. After approval, resume the same activity with
   `resume_activity`.
5. Prefer selected failing cases while repairing; do not pay repeatedly for a complete suite.
6. Never retry after insufficient credits for a fee, a spent or expired capability, exhausted
   authoring budget, an inaccessible reference, or a stable permanent error.

## Find, compare, run, read

1. Call `search_recipes` with a plain problem statement and, when known, semantic input and
   output labels such as `Contract text` and `Risk note`. `results` are direct fits;
   `related_results` are adjacent recipes, so say so. `no_suitable_match` is an answer: follow
   its `next_action` instead of promoting a weak result. `authorization_required` means a lane
   was not searched, not that nothing exists.
2. When more than one result plausibly fits, preview and compare at least the top two on
   contract fit (`output_contract_state`), `evidence_note`, cost `history`, duration, and
   current access. Do not invent an overall quality score.
3. `search_recipes` returns an opaque `recipe_ref`.
   The preview returns a refreshed `recipe_ref`; pass that preview-issued reference to `run_recipe`.
   References are account-bound and expire: search again after expiry, tenant switching, or a
   revision change, and never substitute a recipe ID.
4. Preview input handles are the only accepted run input keys. On `invalid_inputs`, repair each
   issue by its `path`, `hint`, and `limit`; the response never echoes submitted values.
5. Every `run_recipe` call, including a dry run, requires a valid `idempotency_key`. A dry run
   does not consume its key; use a fresh key for the real run. Replaying a consumed key with the
   same request returns the same run.
6. Poll `get_run` at its supplied interval. `progress` shows the phase and active steps;
   `updated_at` is only the record's time, so an unchanged value is not a stuck run. After a
   terminal status keep polling while billing is `pending`. Billing comes from `get_run`, never
   from results.
7. Call `get_results` without a `view`, or with `view:"declared_outputs"`, only when the status is
   `done` and `results_available` is `true`.
   If that response is `result_unavailable` or `result_too_large`, say that no declared output
   payload is available.
8. To explain a run or diagnose a failure, page through `view:"steps"` by `next_cursor`, add
   `detail:"full"` for parameters and normalized results, and use `view:"step"` plus `step_id`
   for one exact step. While `provisional` is true the run is unfinished, so never report a
   provisional snapshot as a failure. Respect `redaction_reasons`.

Owned normal runs expose a bounded sanitized step trace. Public-open recipes and private
recipes owned by the signed-in account expose their full sanitized trace; execute-only public
recipes expose only their interface. A recipe being public never grants access to another
user's run. Never ask for recipe source, credentials, provider payloads, or authoring
acceptance-run traces.

### When a run goes wrong

Errors are normal. Recipes, including marketplace recipes, can fail or produce weak output, and
the user decides what happens next.

- **Run failed:** open the failed step with `view:"step"` and relay what it says. Then retry once
  with the same inputs, adjust the inputs, or choose another recipe.
- **Paused for credits** (`blocked` with `credit_pause`): tell the user. The run resumes from the
  StewAI web app after credits are added. Do not change the inputs.
- **`recipe_ref_expired`:** search or preview again for a fresh reference, then retry.
- **`invalid_inputs`:** fix the input named in each issue `path` and try again.
- **`outputs` is empty:** the recipe declares no outputs; the result is in the last steps.
- **`account_unavailable`:** ask the user to reconnect a provisioned StewAI account or contact
  support.
- **Output looks wrong or incomplete:** show what came back and let the user decide. Do not
  call it a platform failure.

## Build when nothing fits

1. Call `get_recipe_build` without `build_ref` to list recent owned builds before starting
   another one. Skip the initial search only when the user supplies an explicit `build_ref`;
   even then, look at the current build before changing it.
2. Start one build with narrow input and output contracts and a measurable acceptance suite,
   then add the smallest recipe that satisfies them.
3. Before choosing syntax, request `view:"guidance"` and follow its `authoring_rules`. Request
   `view:"capabilities"` to learn which handlers are draft-authorable and which can qualify
   through unattended acceptance. Structurally valid is not the same as
   `automatic_acceptance_eligible`; explain the difference instead of calling a draft invalid.
4. Apply the smallest useful change with `update_recipe_build`, call `check_recipe_build` after
   every structural change, and repair concrete failures before any paid review.
5. Quote and run semantic review only once deterministic checks pass. Quote and run the smallest
   selected case set that reproduces a failure; run the complete suite only when the revision is
   ready to qualify. Any change creates a new revision and invalidates earlier qualification.
6. Poll review and acceptance through `get_recipe_build_activity`. When resuming in a fresh
   context, request the `activity` view and keep polling its fresh `activity_ref` instead of
   starting and paying for a duplicate.
7. Commit only the exact revision with a green complete suite, required evidence, and settled
   billing. The commit is private, unlisted, and unreviewed. Preview its `recipe_ref` before any
   dry run, and never claim that commit listed, priced, or published the recipe.

## Tools

<!-- BEGIN TOOL TABLE -->

| Tool                        | Purpose                                                                    |
| --------------------------- | -------------------------------------------------------------------------- |
| `discover_stewai`           | Learn connection rules and the exact consumer and authoring journeys.      |
| `search_recipes`            | Search public and currently authorized published recipes.                  |
| `get_recipe_preview`        | Compare safe contracts, access, evidence, duration, and pricing.           |
| `run_recipe`                | Estimate or idempotently run a previewed public or authorized recipe.      |
| `get_run`                   | Poll owned-run status, progress, and authoritative terminal billing.       |
| `get_results`               | Declared outputs or a sanitized, paginated owned-run step trace.           |
| `start_recipe_build`        | Create one owned private draft from an intent and declared contracts.      |
| `get_recipe_build`          | List recent builds or open one bounded build view, including guidance.     |
| `update_recipe_build`       | Change a draft, contracts, acceptance suite, research evidence, or chunks. |
| `check_recipe_build`        | Free deterministic lint, render, policy, and compiler checks.              |
| `review_recipe_build`       | Quote or run bounded semantic review and permitted research.               |
| `run_recipe_acceptance`     | Quote or run selected or complete behavioral cases.                        |
| `get_recipe_build_activity` | Poll review or aggregate acceptance evidence and settled billing.          |
| `commit_recipe_build`       | Preflight and privately publish only the exact qualified revision.         |

<!-- END TOOL TABLE -->

## Boundaries

<!-- BEGIN NOT-AVAILABLE -->

The plugin does not expose recipe source, authoring acceptance-run traces, provider payloads,
credentials, raw exceptions, or unrestricted research evidence. Owned normal runs expose a
bounded sanitized step trace, not a source export or a raw runtime-input log. It cannot publish
to the marketplace, approve safety, set public pricing, or claim creator compensation.
Authoring qualification is parameter-sensitive: deterministic steps, short pauses, and literal
or step-backed vector search can qualify automatically; literal HTTPS ingest and the approved
read-only Brave Search integration use the conditional route. Resource-bound or effectful steps
such as document, email, export, long pauses, subrecipes, managed agents, and other integrations
use the returned supervised handoff. Unknown modes fail closed and must be repaired. For a
subrecipe, pass the `recipe_ref` returned by search or preview as `child_recipe_ref`; StewAI
resolves and pins the runnable revision privately.

<!-- END NOT-AVAILABLE -->

## Related local skill

Use `stewai` only when the user explicitly asks for the Python SDK with an API key.
