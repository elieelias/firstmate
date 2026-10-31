---
name: lifecycle
description: >-
  Load at intake for every new captain request to build, change, plan, or design something in a project, before classifying ship or scout and before writing any brief.
  Sizes the request (tiny, small, feature, huge), picks how much clarification it needs, and picks the route.
user-invocable: false
metadata:
  internal: true
---

# lifecycle

This skill is the captain's standing intake policy for build work.
It decides two things before section 7 of `AGENTS.md` runs as usual: how big the request is, and how much the captain must be questioned before anyone builds.
It never relaxes a section 7 rule: project resolution, delivery mode, `yolo`, the brief contract, worktree isolation, and merge authority all still apply.

Skip this skill for pure questions, status checks, merges, recoveries, and investigations the captain asked for as reports; those follow section 7 directly.

## 1. Size the request

Pick the smallest size that honestly fits.
When two sizes fit, pick the larger one only if a wrong guess would waste real work; otherwise pick the smaller.

| Size | Signals | Examples |
| --- | --- | --- |
| **tiny** | One obvious change, no product decision left open, easy to redo. | Fix a typo, rename a field, bump a dependency, fix a bug with a clear cause. |
| **small** | One behavior change in one area, but one to five details are unstated (copy, edge cases, defaults, where it appears). | Add a filter to an existing list, add a validation rule, add one settings toggle. |
| **feature** | Several connected behaviors, new screens or data, or more than one subsystem; the captain has not described most of the decisions. | Add user profiles, add payments, add notifications. |
| **huge** | A new product or app, or a body of work that is really several features; whether or why to build it may still be open. | Build a mobile app, build an admin dashboard, start a new SaaS. |

The captain's own words override the table:

- "just do it", "quick fix", or a fully specified ask → **tiny**, whatever its size.
- "grill me", "ask me first", or "let's think this through" → at least **feature**-depth clarification.
- "office hours", "is this worth building", or "challenge this idea" → **huge** route; an idea-validation step (gstack's `/office-hours` and `/plan-ceo-review`) is not installed yet, so say so and run grilling.

State the size in one plain line when you acknowledge the request, for example "This looks like a small change, so the crewmate will ask a few questions first."
The captain can correct it in one word; accept the correction without argument.
Record the size and route in the backlog item note next to the delivery mode.

## 2. Route by size

### tiny: build directly

Run section 7 unchanged: ship task, no clarification step.

### small: grilling, then build in the same task

Dispatch one ship task.
Mark the brief captain-attended (section 3) and add this to `## Firstmate spec`:

> Before writing code, read and follow `<code-root>/.agents/skills/phases/clarify/grilling/SKILL.md` in this pane, exactly as written; "the user" in it is the captain.
> When the captain confirms you share the same understanding, put the agreed decisions in the PR description, then build.

Replace `<code-root>` with this home's absolute code root, the directory that holds `AGENTS.md`.
The grilling skill is Matt Pocock's original, unchanged; a small request naturally produces a small design tree, so never ask the crewmate to shorten or alter it.

### feature: grilling first, then a spec, then build

1. Dispatch a **captain-attended scout** (`bin/fm-brief.sh <id> <repo> --scout`, then section 3) whose `## Firstmate spec` says:

   > Read and follow `<code-root>/.agents/skills/phases/clarify/grilling/SKILL.md` in this pane, exactly as written; "the user" in it is the captain.
   > When the captain confirms you share the same understanding, write the decisions as this scout's report in the spec format below.

   Append the spec format from section 5 to that `## Firstmate spec`.
2. Tell the captain in one line which Herdr tab to open for the conversation.
3. When the scout reports done, load `scout-completion` and relay the spec's summary and open questions.
   A finished spec recommends building; it does not authorize it.
4. When the captain authorizes the build:
   - **Agentico route** (only when section 4 says it is enabled): hand the spec to agentico as the feature description.
   - **Otherwise:** if the spec describes one shippable change, promote the scout in place through `bin/fm-promote.sh`.
     If it describes several independently shippable slices, propose a split into backlog items with dependencies, one ship task per slice, each brief carrying the spec's relevant decisions in `## Captain's intent`, and dispatch after the captain agrees.

### huge: grilling, then a spec with an app plan

1. Dispatch one captain-attended scout whose `## Firstmate spec` says:

   > Read and follow `<code-root>/.agents/skills/phases/clarify/grilling/SKILL.md` in this pane, exactly as written; "the user" in it is the captain.
   > When the captain confirms you share the same understanding, write the decisions as this scout's report in the spec format below, including its **App plan** section.

   Append the spec format from section 5 to that `## Firstmate spec`.
2. A brand-new product with no repository yet needs a project first: load `project-management` and create one before dispatch.
3. When the captain approves the app plan, file one backlog item per feature with its dependencies.
   Each feature then takes the **feature** route, and its grilling covers only the gaps the app plan left open, never questions the report already answers.

## 3. Captain-attended crewmates

A captain-attended crewmate talks with the captain directly in its own Herdr tab, so those conversations never fill this session's context.
The generated brief opens with "Work on your own; do not wait for a human", which is wrong for these tasks.
`bin/fm-brief.sh` allows adjusting sections when a task genuinely deviates, so after scaffolding:

1. Replace that opening line with:
   > You are a crewmate managed by firstmate. The captain will talk with you directly in this terminal. Ask your questions here, then wait for the captain's reply.
2. Add this to `## Firstmate spec`:
   > Before your first question, append `paused [key=interview] [at=<epoch>]: waiting for the captain in this pane` to your status file, once, using the status command from your Rules.
   > Append `resolved [key=interview] [at=<epoch>]: interview finished` when the questioning ends.
   > Questions the captain answers here are not `needs-decision` events; do not route them through firstmate.
   > Save progress into your report after every round, so nothing agreed exists only in this conversation.

Never answer the crewmate's interview questions on the captain's behalf.
A long-idle captain-attended pane is healthy while its `paused [key=interview]` declaration is open.

## 4. Agentico route

Status: **not enabled.**
This section is enabled only after the captain has run agentico on its own and asked for it to be connected.
Until then, route every feature through section 2's "Otherwise" path.

## 5. Spec format

Copy this block into the brief wherever section 2 asks for it:

> Write the spec with these sections:
> 1. **Goal**: one paragraph on what this is and why.
> 2. **Users and scenarios**: who uses it and the main situations.
> 3. **Decisions**: every settled decision, grouped by area, one line each.
> 4. **Edge cases and errors**: empty, invalid, failing, and limit states and what happens in each.
> 5. **Out of scope**: what was explicitly excluded.
> 6. **Acceptance criteria**: checkable statements that say when it is done.
> 7. **Open questions**: anything the captain chose to leave open; these are captain calls for your completion gate.
> 8. **App plan** (huge route only): the features, their order, and which depend on which, one line each.
