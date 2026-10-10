---
name: stewai-mcp
description: "Use the StewAI MCP plugin to find, compare, run, create, test, repair, and privately reuse StewAI recipes after email sign-in, without managing an API key. Use when the user mentions StewAI, reusable AI workflows, or wants an agent to turn a recurring problem into a tested recipe. Do not use for the Python SDK."
---

# StewAI MCP

Use StewAI as one capability system: reuse a suitable published recipe when possible; otherwise
build, qualify, and privately run a new one. Do not ask the user to choose between marketplace and
authoring workflows.

## MCP-only routing

Use only the StewAI MCP tools listed here. Never fall back to the Python SDK, an API key, browser
search, shell commands, or workspace files when an MCP tool needs authentication or is
unavailable. Ask the user to connect or reauthenticate StewAI, then retry the same MCP tool.

`discover_stewai` is anonymous. Every live search, preview, run, result, or authoring operation
uses the signed-in StewAI account and its current organization. New 1.1.0 scopes may require the
user to reconnect an older installation.

## Required workflow

1. Call `discover_stewai` once in a fresh connection, then call `search_recipes` with a meaningful
   problem statement and, when known, required input and output labels. Synthetic handles remain
   accepted for compatibility, but normally use semantic labels such as `Contract text` and
   `Risk note`.
2. When more than one result plausibly fits, preview and compare at least the top two. Compare
   contract fit (`output_contract_state`), qualification and `evidence_note`, past-run cost
   `history`, duration, and current access. `results` are direct fits; `related_results` are
   adjacent recipes, so say so rather than presenting one as the requested task. Do not invent an
   overall quality score.
3. If a candidate fits the task and contract, call `run_recipe` with `dry_run: true`. Tell the user what past
   runs of the recipe cost (`history`), any fee, and their balance, and obtain the user's approval
   before a real run when execution could spend credits or affect an external system. The user
   decides; the balance does not have to cover the theoretical worst case. A dry-run
   `sufficient: false` value is not a refusal and must not prevent the approved real run. Then run once, poll `get_run`, read declared outputs with `get_results`, and
   inspect its paginated `steps` view when the user needs an explanation, the run fails, or the
   workflow is being dogfooded.
4. If no candidate suitably fits the task and contract, call `get_recipe_build` without `build_ref` to check recent
   owned builds before starting another one.
5. Start or continue one build. Apply the smallest useful draft change, run deterministic checks,
   and repair concrete failures before any paid review.
6. Quote every paid review or acceptance action before executing it. Use bounded research only
   when the risk or research policy warrants it and when evidence can change the method or a
   safety control.
7. Run selected failing cases during repair. Run the complete acceptance suite only when the
   revision is ready to qualify. Any draft, contract, evidence, suite, or policy change creates a
   new revision and invalidates earlier qualification.
   At least one normal case must match or exceed the intended production input's size, variety,
   and likely output cardinality. Never shrink the only representative fixture merely to fit the
   per-case quote ceiling; simplify the prompt, schema, graph, or output cardinality instead.
8. Preflight and commit only the exact revision with a green complete acceptance suite, required
   review or research evidence, and settled billing.
9. The commit is private, unlisted, and unreviewed. Pass its `recipe_ref` to
   `get_recipe_preview` first, use only the preview's positional `input_N` handles and refreshed
   reference, then dry-run that exact private revision. Obtain any required approval and run one
   representative real-world probe before calling the recipe stable. Read every declared output,
   not just terminal status. Because V1 cannot revise a committed build, a reproducible defect in
   that probe requires a corrected replacement build; never describe the flawed revision as
   qualified in practice merely because its acceptance batch was green.

Skip the initial search only when the user supplies an explicit `build_ref` to continue or repair.
Even then, inspect the current build before changing it.

When resuming a build in a fresh agent context, request its `activity` view. Each recent row
contains a fresh `activity_ref`, the exact revision, state, and any stable terminal error code;
continue polling that reference instead of starting and paying for a duplicate activity.

## Tools

<!-- BEGIN TOOL TABLE -->

| Tool                        | Purpose                                                                    |
| --------------------------- | -------------------------------------------------------------------------- |
| `discover_stewai`           | Learn connection rules and the exact consumer and authoring journeys.      |
| `search_recipes`            | Search public and currently authorized published recipes.                  |
| `get_recipe_preview`        | Compare safe contracts, access, evidence, duration, and pricing.           |
| `run_recipe`                | Estimate or idempotently run a previewed public or authorized recipe.      |
| `get_run`                   | Read owned-run status and authoritative terminal billing.                  |
| `get_results`               | Read declared outputs or a sanitized, paginated owned-run step trace.      |
| `start_recipe_build`        | Create one owned private draft from an intent and declared contracts.      |
| `get_recipe_build`          | List recent builds or inspect one bounded build view.                      |
| `update_recipe_build`       | Change a draft, contracts, acceptance suite, research evidence, or chunks. |
| `check_recipe_build`        | Run free deterministic lint, render, policy, graph, and compiler checks.   |
| `review_recipe_build`       | Quote or run bounded semantic review and permitted research.               |
| `run_recipe_acceptance`     | Quote or run selected or complete behavioral cases.                        |
| `get_recipe_build_activity` | Poll review or aggregate acceptance evidence and settled billing.          |
| `commit_recipe_build`       | Preflight and privately publish only the exact qualified revision.         |

<!-- END TOOL TABLE -->

## Search, preview, and references

Search results may come from `public`, `owned`, `organization`, or `shared` sources. Source is not
proof of quality or permission. The server rechecks current user, tenant, revision, contract,
ownership, and sharing grants before every preview or run.
The public lane contains only listed recipes whose safety approval is pinned to their exact current
published revision. It is the currently approved, runnable public catalog, not every row that may
have been visible in historical Marketplace data.

When `match_status` is `no_suitable_match`, do not invent or recommend a weak result. Follow its
`next_action`: list recent private builds, continue a suitable one if present, otherwise offer to
start a new build. `authorization_required` means a requested search lane was not searched; it is
not evidence that no recipe exists.

`search_recipes` returns an opaque `recipe_ref`. Pass it to `get_recipe_preview`. The preview
returns a refreshed `recipe_ref`; pass that preview-issued reference to `run_recipe`. References are account- and tenant-bound and
expire. Search again after expiry, tenant switching, access changes, or revision changes. Never
replace a reference with a recipe ID.

Preview input handles are the only accepted run input keys. Send exactly those handles and values
that satisfy the declared schemas. Preview states whether each input must be supplied, whether a
default exists, and the applicable character and UTF-8 byte limits. When dry-run or execution
reports `invalid_inputs`, repair every issue by its `path`, `code`, `hint`, and `limit`; the
response never echoes submitted values or hidden defaults. Do not ask for recipe source,
authoring fixtures, credentials,
or provider payloads. When the user owns the run, use only the bounded sanitized trace returned by
`get_results`; do not infer access to another user's run from recipe visibility.

## Estimate, execute, and read

Always call `run_recipe` with `dry_run: true` before recommending or starting execution. Report
the facts plainly and let the user decide:

- `history`: credits charged by past successful runs of this recipe's current version (average,
  median, p90, run counts, last run). With fewer than three runs the statistics are null and
  `recent_charges` lists the actual charges; say there is little history.
- `fee_credits`: any marketplace fee. Authorized non-marketplace recipes have no creator fee.
- `available_credits` and `balance_as_of`: the user's spendable balance at that moment.
- `estimated_credits`: a theoretical worst case (every model call retried at its maximum limits).
  It is rarely approached; mention it as the upper bound, not as the expected price. `sufficient`
  only says whether that worst case fits. Never use `sufficient: false` to refuse or skip an
  approved real run.

A run is not refused because the worst case exceeds the balance. If credits run out during a run,
the run pauses (`blocked`, with `credit_pause` in `get_run`) and can be resumed from the StewAI web
app after adding credits.

Every `run_recipe` call, including a dry run, requires a valid `idempotency_key`. A dry run does not consume its key. Use a fresh
key for the real run. Replaying the same consumed key and request returns the same run; changing
the request is a conflict. Do not loop after insufficient credits, exhausted authoring budget, an
inaccessible reference, or a stable permanent error.

The client may request confirmation because `run_recipe` is destructive and open-world: a real
recipe can spend credits and may call external systems. Once approved, run exactly once. Poll `get_run` at its supplied interval.
While the run is active, `progress` shows the phase, active steps with elapsed time, and
`last_step_activity_at`; `updated_at` is only the run record's time, so do not treat an unchanged
`updated_at` as a stuck run. After terminal
status, keep polling while billing is `pending`, stopping at the documented settlement window.
Call `get_results` without a `view`, or with `view:"declared_outputs"`, only when the status is
`done` and `results_available` is `true`. Billing comes from `get_run`, never from results. If that
response is `result_unavailable` or `result_too_large`, explain that no declared output payload is
available.

For an owned run, `get_results` with `view:"steps"` returns ordered pages of compact step
summaries (ID, title, kind, state, timing, result type), 10 per page by default and at most 20
(`limit`). Pass `detail:"full"` for dependencies, sanitized configured parameters, optional
rendered input, and the same normalized result shape used by the SDK. Follow `next_cursor` until
it is null. While `provisional` is true the run is unfinished: unfinished steps show a pending
result, so never report a provisional snapshot as a failure. `diagnostic_notes` explain nonfatal
warnings such as `schema_urn_missing`. Use
`view:"step"` plus `step_id` for one exact step. These trace views work for running, blocked,
failed, cancelled, and completed runs, so use them to diagnose failures and to explain how a
successful workflow reached its result. `parameters` are compile-time configuration and
`rendered_input` exists only for handlers that persist it; never describe either as a universal
exact runtime-input snapshot. Respect `redaction_reasons`: sensitive inputs remain masked and
execute-only public recipes expose only their public interface. Public-open recipes and private
recipes owned by the signed-in account expose their full sanitized trace. A recipe being public
never grants access to another user's run. If a paginated request reports `step_too_large`, explain
that the one step cannot fit the bounded response and continue from its `next_cursor` when present.

If execution returns `account_unavailable`, do not modify recipe inputs. Ask the user to reconnect
an ordinarily provisioned StewAI account or contact support.

### When a run goes wrong

Errors are normal. Recipes, including marketplace recipes, can fail or produce weak output, and the
user decides what to do next. Handle each case plainly:

- **Run failed:** read the failed step with `view:"step"` and tell the user what it says. Then retry
  once with the same inputs, adjust the inputs, or choose another recipe.
- **Run paused for credits** (`blocked` with `credit_pause`): tell the user. The run resumes from the
  StewAI web app after credits are added.
- **`recipe_ref_expired`:** search again or preview the recipe for a new `recipe_ref`, then retry with
  the same inputs.
- **`invalid_inputs`:** fix the input named in each issue `path` and try again.
- **`outputs` is empty:** the recipe declares no outputs. Read `view:"steps"`; the result is in the
  last steps.
- **Output looks wrong or incomplete:** show the user what came back and let them decide. Do not
  report it as a platform failure.

## Authoring loop

Treat tool schemas and returned guidance as authoritative; never invent recipe syntax. A useful
loop is:

1. Declare narrow input/output contracts and a measurable acceptance suite.
2. Add the smallest graph that can satisfy them.
3. Call `check_recipe_build` after every structural change; deterministic failures cost nothing.
4. For methodology-sensitive work, record bounded external evidence with a URL, title, supported
   claim, retrieval time, and applicability. Evidence informs a method; it does not override the
   user's domain policy.
5. Quote, then execute semantic review only when deterministic checks pass.
   A passing review can still contain advisory observations. Do not churn a working revision to
   eliminate wording-level observations that are already covered by a concrete acceptance case.
   Change the draft again only for a reproducible defect, an uncovered material risk, or a
   specific counterexample; otherwise freeze the revision and proceed to behavioral tests.
6. Quote, then run the smallest selected set that reproduces a failure.
7. Repair the draft and repeat deterministic checks.
8. Run the complete suite once the candidate is ready. Only a green complete suite can qualify
   that exact revision.

Poll review and acceptance through `get_recipe_build_activity`. It intentionally omits provider
payloads, credentials, raw exceptions, wallet identifiers, acceptance-run traces, and partial
credit totals. Do not seek those through other tools.

Before choosing authoring syntax, call `get_recipe_build` with `view:"capabilities"`. Its 24-entry
catalog is derived from the runtime handler registry and states which nodes are draft-authorable,
whether they can qualify through unattended acceptance, their side-effect class, and when a human
handoff is required. A draft can be structurally valid even when
`automatic_acceptance_eligible` is false; explain that distinction instead of calling it invalid.

### Exact authoring syntax

Do not guess mutation names or payload fields. `update_recipe_build.ops` accepts only these exact
forms:

- `{"op":"replace_spec","spec":{...}}`
- `{"op":"set_metadata","metadata":{"name":"...","description":"..."}}`
- `{"op":"upsert_node","node":{"id":"...","type":"..."}}`
- `{"op":"remove_node","node_id":"..."}`
- `{"op":"reorder_nodes","node_ids":["..."]}`
- `{"op":"upsert_schema","schema_id":"...","schema":{...}}`
- `{"op":"remove_schema","schema_id":"..."}`
- `{"op":"set_acceptance_cases","suite":{...}}`
- `{"op":"record_research","evidence":[...]}`
- `{"op":"set_budget","budget_capability":"..."}`
- `{"op":"restore"}`, or the documented `begin_chunk`, `append_chunk`, and `commit_chunk`
  forms for large payloads.

A single-field `recipe_input` is executable text input. Match contract `maxLength` with node
`max_length`; `required:true` already enforces nonblank input. For structured LLM output, put the
complete object schema in `llm_call.response.schema`. Export the complete object with
`{{ analyze.pydantic.value | json }}`, or export a declared field with
`{{ analyze.pydantic.value.field_handle | json }}`. Here, `analyze` is the ID of the producing
node. A sibling `schema` inside a `recipe_output`
mapping is descriptive metadata and does not establish the executable contract.

For semantic MCP result labels, create one top-level `recipe_output` node per user-facing output
field, give the node the intended user-facing `title`, and give it a one-key `mapping`. A single
`recipe_output` with several mapping keys is valid, but disclosure deliberately labels those
positions `Output 1`, `Output 2`, and so on. Mapping keys remain internal and must never be used as
run input or public result handles.

The acceptance mutation is named `set_acceptance_cases`, not `set_acceptance_suite`. Its suite
starts with:

```json
{
  "version": 1,
  "cases": [
    {
      "case_id": "normal_1",
      "label": "Normal input",
      "kind": "normal",
      "fixtures": [
        {
          "target": "input_handle",
          "content_type": "text/plain",
          "value": "..."
        }
      ],
      "assertions": [
        {
          "assertion_id": "result_exists",
          "type": "scalar",
          "operator": "exists",
          "path": "/result"
        }
      ]
    }
  ]
}
```

Every case needs at least one assertion. Use only the assertion fields allowed by the tool schema.
Use `scalar` equality only for JSON scalar values. To prove an array is empty, use a `collection`
assertion with `operator:"count_equals"` and `expected:0`; `exists` proves only that the array is
present. A full suite rejects a normal case whose assertions prove only `exists` or `type`; every
normal case needs at least one behavioral oracle over its actual value, count, relationship,
classification, coverage, or safety property. Every decision-bearing output needs a behavioral
oracle. For paired invariance checks, prefer one material output field per `fields_unchanged`
assertion so a failure identifies the unstable field instead of collapsing several differences
into one opaque result.
Before qualification, compare the normal fixtures with the user's planned real input. Include one
case that is at least as demanding for rendered-input size and structured-output length. A green
suite built only from smaller inputs does not establish that the committed recipe can finish the
real run. When the per-case quote rejects a realistic fixture, reduce response verbosity or schema
complexity rather than weakening the fixture.
When the recipe accepts a user-supplied rubric, policy, taxonomy, score scale, weight set, or
decision threshold, test at least two materially different configurations. Derive arithmetic from
the supplied scale maximum and apply the supplied thresholds exactly; never hide a fixed
denominator or fallback policy behind a configurable-looking input. For reconciliation, state
whether identity is exact-ID or semantic cross-ID matching and include a case that distinguishes
the two. For bounded prose, set field limits large enough for a complete action or rationale and
assert a representative complete phrase so a clipped or unfinished sentence cannot qualify.
Call `check_recipe_build` without `fixtures` for the ordinary structural/compiler check. Supply
fixtures only for intentional template probes, and then provide a fixture for every referenced
dependency whose output is not produced during that free render-only pass.

`commit_recipe_build` first issues a short-lived single-use commit capability. Use it only for the
unchanged qualified revision. A successful commit returns a private editor URL and, when current
run access permits it, a `recipe_ref`. Preview that reference before dry-run: the authoring
contract handles and recipe node IDs are not the positional `input_N` handles accepted by
`run_recipe`. Say explicitly that the recipe is private. Never claim that commit submitted,
reviewed, listed, priced, or published it in the public marketplace.

## Approval and budget stops

- Quote before every paid review or acceptance action.
- Explain the maximum credits and remaining authoring budget before spending.
- After a quote, follow its `next_action` exactly: redeem `budget_capability` with
  `update_recipe_build` operation `set_budget`, then execute the quoted request.
- When full acceptance returns `supervised_required`, inspect every returned resource requirement
  first. Call `update_recipe_build` operation `bind_resource` once for each required selector,
  using the exact selector already frozen in the draft. Only then ask the user to open
  `action_url` and approve. After approval, resume the same activity with `resume_activity`.
- Obtain user approval before a real recipe run when the action spends credits or can affect an
  external system.
- Prefer selected failing cases while repairing; do not repeatedly pay for a complete suite.
- Do not retry when the server reports insufficient credits for a fee, a spent or expired capability,
  exhausted cumulative budget, or a durable permanent error.

## Not available in 1.1.0

<!-- BEGIN NOT-AVAILABLE -->

The plugin does not expose recipe source, authoring acceptance-run traces, provider payloads,
credentials, raw exceptions, or unrestricted research evidence. Owned normal runs expose a
bounded sanitized step trace; this is not a source export or a universal raw runtime-input log. It
cannot publish to the marketplace, approve safety, set public pricing, or claim creator
compensation. Authoring
qualification is parameter-sensitive: deterministic nodes, short pauses, and literal or
step-backed vector search can qualify automatically; literal HTTPS ingest and the approved
read-only Brave Search integration use the conditional route. Resource-bound or effectful nodes
such as document, email, export, long pauses, subrecipes, managed agents, and other integrations
use the returned supervised handoff. Bind every required frozen selector, obtain approval through
`action_url`, then resume the same blocked activity. Unknown modes fail closed and must be repaired.
For a subrecipe, pass the `recipe_ref` returned by search or preview as `child_recipe_ref`;
StewAI resolves and pins the runnable revision without exposing internal recipe identifiers.

<!-- END NOT-AVAILABLE -->

## Starter prompts

<!-- BEGIN STARTER PROMPTS -->

Find and compare the best accessible StewAI recipes for my task, then preview the strongest option without running it.
Run a StewAI recipe with my input, monitor it, then summarize its declared results, step trace, and settled credit charge.
Build a private StewAI recipe for a recurring task, test it, repair failures, and commit a qualified version.

<!-- END STARTER PROMPTS -->

## Related local skill

Use `stewai` only when the user explicitly asks for the Python SDK with an API key. A failed or
unauthenticated MCP call never authorizes that fallback.
