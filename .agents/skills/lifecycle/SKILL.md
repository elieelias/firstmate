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
   - **Agentico route** (section 4) when its conditions hold.
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

Agentico (DoorDash's agentic-orchestrator) builds features: research, design, roadmap, phase plans, implementation, and its own final review.
The captain chose two checks: agentico's final review, then this home's no-mistakes pipeline before any pull request.
The captain approves agentico's gates (research, design, roadmap, questions) in agentico's desktop app, never through this session.

### When to use it

Use it for an authorized **feature** spec, and for each feature from an approved app plan, when all of these hold:

- `agentico api GET /api/v1/health` succeeds (the agentico app or `agentico server` is running);
- `agentico api GET /api/v1/readiness` lists the project's repository under `workspace.repositories[].name`.

If either fails, tell the captain in one line what is missing and use section 2's "Otherwise" path instead; never start agentico's server yourself.

Talk to agentico only through its helper, one bare call per command: `agentico api <METHOD> <path> ['<json body>']`.
Never read its discovery file or token, and never edit its worktrees or state files.
Agentico's own supervisor skill (`api-reference.md` and `recipes.md` in its installed skills) owns the exact fields; read it when a call is refused.

### Hand off a feature

1. Choose the pipeline: `medium` when the spec is detailed and the approach is obvious from the existing code, otherwise `large`; `moonshot` only when the captain asks.
2. Create the feature, with the spec as `description` (shorten to the decisions and acceptance criteria if it exceeds 10000 characters), the backlog item id as `idempotency_key`, and `manual_publish` on so agentico never opens a pull request itself:
   `agentico api POST /api/v1/features '{"name":"<title>","description":"<spec>","repos":["<repo>"],"pipeline":"<medium|large>","checkpoints":{"manual_publish":true},"idempotency_key":"<item-id>"}'`
3. Dispatch its setup with `POST /api/v1/features/<feature_id>/actions/setup '{}'`, read the feature until setup is done, then start it with `POST /api/v1/features/<feature_id>/actions/start '{}'`.
4. Record the agentico feature id in the backlog item note, and tell the captain in one line that the feature is running in agentico and its gates will appear in the agentico app.

### While it runs

On every heartbeat while an agentico feature is open, and whenever the captain asks, read `GET /api/v1/features/<feature_id>`:

- an open `review_gate` or `need_user_input`: tell the captain once that agentico is waiting in its app;
- a `failure`: relay it and ask whether to retry, rewind, or stop;
- `CodeReady`: hand the result to no-mistakes, below.

Never answer agentico's gates, questions, or permission requests yourself.

### Second check: no-mistakes

When the feature reaches `CodeReady`:

1. Read the repository's worktree path with `GET /api/v1/features/<feature_id>/repositories/<repo>/path`.
2. Dispatch a ship task in `no-mistakes` mode through section 7, with the spec in `## Captain's intent` and this in `## Firstmate spec`:
   > The implementation already exists, built and reviewed by agentico in `<worktree path>`.
   > Do not reimplement it. Bring it into your ship branch with `git fetch <worktree path> HEAD` and the fetched commits, rebased onto the base branch if needed.
   > Then follow your delivery path; fix only what the pipeline finds.
3. Never use agentico's own `publish` or `merge` actions; the pull request comes from no-mistakes, and merge authority stays with the captain as usual.
4. After the pull request lands, ask the captain before removing agentico's worktrees with `POST /api/v1/features/<feature_id>/actions/cleanup '{}'`.

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
