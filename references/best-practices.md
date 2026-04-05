# Workflow Design Best Practices

Use this file when the user asks for a workflow that is stable, high quality, reusable, or production-ready on OpenCreator.

This file is about choosing the right workflow shape and operator behavior, not about low-level API syntax.

## Core Principles

- Start from the final asset or text the user wants, then work backward to the minimum graph that can produce it.
- Prefer template reuse over greenfield workflow creation when the job is a known pattern.
- Prefer explicit input nodes and explicit terminal outputs over hidden assumptions inside one giant prompt.
- Separate creative generation, control inputs, and final formatting so failed runs are easier to diagnose.
- Ask the user for real runtime inputs instead of trusting template defaults.

## Template-First Strategy

Prefer a template when:

- The request sounds like a common OpenCreator use case such as UGC, ecommerce, product image generation, or style transfer
- The main differences are prompt text, reference media, or a few model settings
- Speed to first successful result matters more than custom graph design

Prefer building from scratch when:

- The required branching logic differs materially from any available template
- The workflow needs uncommon node combinations
- The output contract is custom enough that patching a template would be riskier than rebuilding

## Runtime Correctness Rules

- Always fetch runtime parameters before a run
- Always ask the user about each returned input node
- Never assume `default_value` matches the user's real intent
- Always map `inputs` as `node_id -> string value`
- Never wrap runtime values in an extra object layer
- Never call results before status is `success`

## User Interaction Rules

- Speak in business language, not technical field names
- Reuse information the user already gave you and only ask for missing pieces
- When multiple assets are needed, ask for them in a short numbered list
- If the output is media, return the media directly when possible instead of only sharing a raw URL

## Reliability Patterns

- For fast text or image workflows, poll about every 10 seconds
- For video or heavier multi-node workflows, poll about every 30 seconds
- If a task sits in `queued` too long, inspect the submitted input shape before changing the workflow itself
- If a copied template returns graph integrity errors, switch templates before investing in deep repair
- If media inputs are local files, upload them to a service that produces direct file URLs before running

## Quality Patterns

- Use templates to get a strong baseline quickly, then patch only the nodes that must change
- Keep node responsibilities narrow enough that weak output can be traced to one stage
- Use the default output node when the platform marks one, unless the user asked for another terminal output
- Surface model names and credit usage in result summaries when that helps explain quality or cost

## Debugging Playbook

Use this order when a run misbehaves:

1. Check whether the task was started with the correct `flow_id`
2. Check whether parameters were fetched immediately before the run
3. Check whether the `inputs` keys exactly match returned `node_id` values
4. Check whether the `inputs` values were sent flat instead of nested
5. Check `status`, `error_message`, and `node_statuses`
6. Change the smallest possible thing and rerun

## Anti-Patterns

- Running a public template directly without copying it first
- Skipping the parameters API because "the node IDs probably did not change"
- Trusting template defaults instead of asking the user
- Showing `node_id` or internal field names to the user
- Changing many nodes at once when only one stage is suspect
- Treating a queued task as a platform outage before verifying input shape
