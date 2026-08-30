---
name: stewai-mcp
description: "Use the StewAI MCP plugin to discover workflow examples, search and preview live recipes after email sign-in, estimate credits, run recipes, and retrieve final declared results without managing an API key. Use when the user mentions the StewAI plugin, StewAI MCP, or wants keyless access to an existing StewAI recipe. Do not use for Python SDK or recipe-authoring work."
---

# StewAI MCP

Use StewAI recipes through the plugin without installing a package or managing an API key.

## MCP-only routing

Use only the StewAI MCP tools listed below. Never fall back to the Python SDK, an API key, a
browser search, shell commands, or workspace files when an MCP tool needs authentication or is
unavailable. Ask the user to connect or reauthenticate StewAI, then retry the same MCP tool.

## Access model

`discover_stewai` is anonymous and returns fixed workflow examples. Every live marketplace,
preview, execution, status, or result request starts with a one-time email sign-in.

## Tools

<!-- BEGIN TOOL TABLE -->

| Tool                 | Auth          | Purpose                                                                 |
| -------------------- | ------------- | ----------------------------------------------------------------------- |
| `discover_stewai`    | none          | Show fixed examples of what StewAI can automate.                        |
| `search_recipes`     | email sign-in | Search the live marketplace with a meaningful query.                    |
| `get_recipe_preview` | email sign-in | Preview safe labels, synthetic inputs, outputs, and historical pricing. |
| `run_recipe`         | email sign-in | Estimate credits or run a previewed recipe idempotently.                |
| `get_run`            | email sign-in | Read caller-owned status and terminal billing settlement.               |
| `get_results`        | email sign-in | Read available final author-declared outputs for a completed owned run. |

<!-- END TOOL TABLE -->

## Recipe references and inputs

`search_recipes` returns an opaque `recipe_ref`. Pass that reference to `get_recipe_preview`. The
preview returns a refreshed `recipe_ref`; pass that preview-issued reference to `run_recipe`. A
reference is bound to the signed-in account and expires after one hour. Search again when a
reference is stale. Never replace it with a recipe ID.

The preview names accepted inputs as `input_1`, `input_2`, and so on. Send only those synthetic
handles to `run_recipe`.

## Estimate, execute, and read

Search and preview return a `pricing` object based only on the recipe's historical average across
successful runs. Its `credit_band` is context, not an input-aware estimate or affordability
check. Always call `run_recipe` with `dry_run: true` before recommending or starting execution.

Every `run_recipe` call, including a dry run, requires a valid `idempotency_key`. Call it with
`dry_run: true` to receive `estimated_credits`, `available_credits`, and `sufficient` without
executing or consuming that key. For a real run, provide a fresh key. Reusing a consumed key with
the same request returns the same run. Reusing it with changed inputs is a conflict.

If `run_recipe` returns `account_unavailable`, the signed-in identity does not have a runnable
StewAI workspace. Do not change or retry the recipe inputs. Ask the user to reconnect with an
ordinarily provisioned Free account or contact StewAI support for account repair.

The client may ask the user to confirm a real run because `run_recipe` is marked destructive and
open-world. Use `get_run` to check status. After the run is terminal, continue polling `get_run` at
its supplied interval while `billing.state` is `pending`, stopping after the 300-second settlement
window. A settled response reports the authoritative run-tree charge; tokens allowed to run also
receive the account's current available balance and its timestamp. Do not interpret that balance
as this run's isolated before/after effect. If billing becomes `unavailable`, explain that the
charge could not be verified through the connector.

Call `get_results` only when the status is `done` and `results_available` is `true`. A successful
result contains only the final author-declared outputs as `{label, value}` entries. Billing never
comes from `get_results`. If the response is `result_unavailable` or `result_too_large`, explain
that no declared result payload is available; do not look for internal run data instead.

## Not available

<!-- BEGIN NOT-AVAILABLE -->

This plugin does not provide a recipe specification, graph, nodes, edges, step ids, internal
prompts, models, parameters, code, output schemas, execution contracts, or run traces. No tool
named `get_recipe`, `get_recipe_spec`, `list_recipes`, `export_recipe`,
`get_execution_contract`, or `get_run_steps` exists. There is no browse or list-all mode.
Search requires a real query.

<!-- END NOT-AVAILABLE -->

## Starter prompts

<!-- BEGIN STARTER PROMPTS -->

What can StewAI help me automate?
Show me examples of StewAI workflows.
Find a live StewAI recipe for contract review.
This starts a one-time email sign-in before StewAI searches the live marketplace.

<!-- END STARTER PROMPTS -->

## Route related requests

Use `stewai-create-recipe` when the recipe does not exist yet or needs changing. Use `stewai` when
the user explicitly asks to work through the Python SDK with an API key. A failed or unauthenticated
MCP call never authorizes that fallback.
