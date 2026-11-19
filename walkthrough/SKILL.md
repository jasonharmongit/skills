---
name: walkthrough
description: Invoke only when walkthrough is mentioned by name. Walks the user through the user story flow of a branch's changes in chat, stage by stage, with the side effects each stage causes.
metadata:
  author: jasonharmon
---

Answer in chat only. Do not edit code or write files.

## Scope

- **Branch:** committed, staged, unstaged, and untracked changes. Untracked files show only in `git status --short`; read each new file in full.
- **Side effects:** everything a stage changes outside its own process. This includes database writes, external API calls, enqueued jobs, PubSub broadcasts, cache writes or busts, and notifications (email, Slack, webhooks). Reads are not side effects; mention one only when it decides where the user goes or what they see. If the user names specific systems, cover each one at every stage where it acts.
- **Altitude:** high level. Name each action and its effect. Skip parameters, payload fields, helper names, and validation detail unless they change what the user sees or what gets stored, sent, or charged.

## Steps

1. Run `git diff "$(git merge-base HEAD origin/main)"` (no `...HEAD`, so uncommitted changes are included) and `git status --short`, then read the changed and new files. Trace the call chain from each new entry point (route, LiveView, controller, API action, worker) until you know every side effect along the path.
2. Pick the primary user path. Note any feature flag or condition that gates it, and what the old path still does. Other entry points that join this path get one sentence in the stage where they join it.
3. Hunt for loose ends: risks in the changes, such as unwired code, external calls without idempotency, side effects that cannot be retried after a charge or send, and states the user cannot leave.
4. Write the story in the shape and voice below.
5. Check each stage against the code: call order, which system does what, where the user goes next, and that each link lands on the line where that beat runs in that order (not a helper definition elsewhere).

## Shape

- **Lead:** an unlabeled opening paragraph of one or two sentences. State what the flow does end to end, the gate, what the old path still does, and the commit state of the changes (for example "all uncommitted"). The gate and old path appear only here.
- **Stages:** one `##` heading per stage, in execution order, named in a few plain-English words (not module names). Under it, short paragraphs of at most three sentences. Start each paragraph with a new actor or beat. Each beat says who acts, what changes in which system, and what the user sees or where they go next. Include the failure path only where it is new or notable.
- **Loose ends:** a final `## Loose ends` heading with up to three bullets from step 3. Each bullet states one problem and its consequence, with one anchor link on the code at fault. Leave this out only if step 3 found nothing.

## Voice

- Active voice with a concrete actor: the user, a worker, Twilio, the database. Do not make a module the subject ("`OrderController` handles…"); say what the user or system does.
- Name the system in the sentence that changes it: "The worker inserts the message row and enqueues a delivery job, then Twilio sends the SMS."
- Use concrete verbs. Avoid "flows," "feeds," "lands in," "wires through," "hands off," and "handles."
- Introduce detail as the story reaches it. Do not front-load a call-chain catalog.

## Links

- Give each main beat **one** markdown link on the word or short phrase that names the action, for example "When the user [confirms](/abs/path/order_controller.ex#L42) the order…". The reader should follow the story without clicking.
- Link only changed or new code a reader needs to review that beat. When a side effect runs through unchanged code, name the system and the effect with no link.
- The label reads naturally in the sentence, usually a verb or short phrase. Never a module name, function name, or file path. Do not use backticks for code references.
- Resolve each target with `realpath` and append `#L…` for the line where that beat runs in the call chain: the call site, `with` step, branch, or pipeline step the sentence describes. Do not default to the helper's definition. Link a definition or dispatch clause only when the beat is that entry (for example a route action or a type-dispatch clause).
- Use working-tree line numbers so uncommitted changes link correctly.
