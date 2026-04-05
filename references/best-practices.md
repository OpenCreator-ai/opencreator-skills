# Workflow Design Best Practices

Use this file when deciding whether to use a template or build from scratch, and when designing workflow graph structure on OpenCreator.

For API operation rules (run, poll, results), see `references/api-workflows.md` instead.

## Template-First Strategy

Prefer a template when:

- The request sounds like a common use case (UGC, ecommerce, product image, style transfer)
- The main differences are prompt text, reference media, or a few model settings
- Speed to first successful result matters more than custom graph design

Prefer building from scratch when:

- The required branching logic differs materially from any available template
- The workflow needs uncommon node combinations
- The output contract is custom enough that patching a template would be riskier than rebuilding

## Graph Design Principles

- Start from the final asset the user wants, then work backward to the minimum graph
- Prefer explicit input nodes and explicit terminal outputs over hidden assumptions in a giant prompt
- Separate creative generation, control inputs, and final formatting so failed runs are easier to diagnose
- Keep node responsibilities narrow enough that weak output can be traced to one stage

## Quality Patterns

- Use templates to get a strong baseline quickly, then patch only the nodes that must change
- Use the default output node when the platform marks one, unless the user asked for a different terminal
- Surface model names and credit usage in result summaries when that helps explain quality or cost

## Anti-Patterns

- Changing many nodes at once when only one stage is suspect
- Skipping the parameters API because "the node IDs probably did not change"
- Trusting template defaults instead of asking the user
- Treating a queued task as a platform outage before verifying input shape
