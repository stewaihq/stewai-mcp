# StewAI

**Give your agent superpowers.**

Keep the agent you already use. StewAI sits beside it with ready-to-run recipes for research
synthesis, document extraction, executive summaries, data quality review, classification, and
policy checks. Your agent finds the recipe, shows you what it needs, what it returns, and what
past runs cost, and runs it only after you approve. The results come back into the
conversation.

If nothing fits, your agent can build a private recipe, test it, and keep it for next time.

One email sign-in. No API key.

## What it feels like

1. **Ask.** "Check whether each citation in this draft supports its claim."
2. **See it first.** Inputs, outputs, past-run cost, and your balance, before anything runs.
3. **Approve.** The recipe runs once. Progress is visible while it runs.
4. **Use the result.** Declared outputs come back to your agent, with a step trace where the
   recipe allows it.

## Try asking

- "Find a recipe that pulls obligations, owners, dates and renewal terms out of these
  contracts, and quote it before running."
- "Turn these meeting notes into decisions and action items."
- "Review this dataset for missing values, duplicates, and conflicting definitions."
- "Nothing fits my weekly operating brief. Build a private recipe, test it, and keep it."

## Good to know

- Search says no match rather than pushing a weak recipe at you.
- A dry run always comes first. You decide whether a real run is worth its credits.
- Recipes your agent builds stay private and unlisted. The plugin does not submit them to the
  public marketplace, set prices, or approve safety.
- Recipes can fail or return weak output, like any workflow. Your agent shows you what came
  back, and you choose what to do next.

## What the plugin connects to

The plugin connects to `https://api.stewai.com/mcp` and signs you in through your browser with
OAuth. It sends the inputs you provide to the recipe you approve. It never exposes recipe
source, credentials, or provider payloads, and it never asks you for an API key.

## Links

- Website: https://stewai.com
- Documentation: https://docs.stewai.com
- Privacy: https://stewai.com/privacy
- Terms: https://stewai.com/terms
- Imprint: https://stewai.com/imprint
- Support: support@stewai.com

## License

Copyright 2026 Christian Mueller. This plugin package is licensed under Apache-2.0; the full
text is in `LICENSE`. The license covers the manifest, MCP configuration, skill, README, and
icons only. StewAI itself, its services, Python SDK, and marketplace data remain subject to
their own terms, and the StewAI name and logo stay protected.
