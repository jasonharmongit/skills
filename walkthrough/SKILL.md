---
name: walkthrough
description: Invoke only when walkthrough is mentioned by name. An annotated, click-through story of the branch's logical flows — stage by stage, with side effects and links on every branch-relevant action.
metadata:
  author: jasonharmon
---

Answer in chat only. Do not edit code or write files.

## Purpose

The reader uses this output like an annotated code review: follow execution order, one beat at a time, and click links to land on the line where that beat runs. Cover the primary path and notable branches; name side effects when they change data, send traffic, or enqueue work.

## Scope

- **Branch:** committed, staged, unstaged, and untracked changes. Untracked files show only in `git status --short`; read each new file in full.
- **Side effects:** everything a stage changes outside its own process. This includes database writes, external API calls, enqueued jobs, PubSub broadcasts, cache writes or busts, and notifications (email, Slack, webhooks). Reads are not side effects; mention one only when it decides where the user goes or what they see. If the user names specific systems, cover each one at every stage where it acts.
- **Altitude:** high level in prose. Name each action and its effect. Skip parameters, payload fields, helper names, and validation detail unless they change what the user sees or what gets stored, sent, or charged.

## Steps

1. Run `git diff "$(git merge-base HEAD origin/main)"` (no `...HEAD`, so uncommitted changes are included) and `git status --short`, then read the changed and new files. If diff output truncates, use the status list as the file checklist and read each path. Trace the call chain from each new entry point (route, LiveView, controller, API action, worker) until you know every side effect along the path.
2. Pick the primary user path. Note any feature flag or condition that gates it, and what the old path still does. Other entry points that join this path get one sentence in the stage where they join it.
3. Hunt for loose ends: risks in the changes, such as unwired code, external calls without idempotency, side effects that cannot be retried after a charge or send, and states the user cannot leave.
4. Write the story in the shape and voice below.
5. Check each stage against the code: call order, which system does what, where the user goes next, branch-relevant actions are linked, and each link lands on the line where that beat runs in that order (the call site or branch step, not a helper definition elsewhere unless the story enters that definition). When the story crosses a function or module boundary, confirm the last link in the caller is the call into that callee before any link inside the callee.

## Shape

- **Lead:** an unlabeled opening paragraph of one or two sentences. State what the flow does end to end, the gate, what the old path still does, and the commit state of the changes (for example "all uncommitted"). The gate and old path appear only here.
- **Stages:** one `##` heading per stage, in execution order, named in a few plain-English words (not module names). Under it, short paragraphs of at most three sentences. Start each paragraph with a new actor or beat. Each beat says who acts, what changes in which system, and what the user sees or where they go next. Include the failure path only where it is new or notable.
- **Loose ends:** a final `## Loose ends` heading with up to three bullets from step 3. Each bullet states one problem and its consequence, with one anchor link on the code at fault. Leave this out only if step 3 found nothing.

## Voice

- Active voice with a concrete actor: the user, a worker, Twilio, the database. Do not make a module the subject ("`OrderController` handles…"); say what the user or system does.
- Name the system in the sentence that changes it: "The worker inserts the message row and enqueues a delivery job, then Twilio sends the SMS."
- Use concrete verbs. Avoid "flows," "feeds," "lands in," "wires through," "hands off," and "handles."
- Introduce detail as the story reaches it. Do not front-load a call-chain catalog.
- Advance one stack frame at a time when the story crosses layers: caller first, then the next call inside the callee, then inside new modules when the story steps in.

## Links

- Link every branch-relevant action: writes, enqueues, external calls, status transitions, queries that gate UX, and new or changed control flow. On the primary path, link each entry point once even when that file is unchanged. Omit boilerplate and unchanged plumbing deeper in the stack unless the branch changes behavior there.
- Order links in **execution order**, one callee entry per beat. Pattern: caller invokes → link the invocation in the caller; callee loads or branches → link that line in the callee; callee invokes another module → link that call site; only then link lines inside the next module. Never skip a call site and jump straight to a line inside the callee (for example an HTTP client inside a helper the context calls).
- Use markdown links on the word or short phrase that names the action, for example "The worker [calls the context](/abs/path/worker.ex#L17)…". The reader should follow the story without clicking.
- Prefer links on changed or new code. When a side effect runs only through unchanged code, name the system and the effect with no link unless it is an entry point on the primary path.
- The label reads naturally in the sentence, usually a verb or short phrase. Never a module name, function name, or file path. Do not use backticks for code references.
- Resolve each target with `realpath` and append `#L…` for the line where that beat runs: the call site, `with` step, branch, or pipeline step the sentence describes. Link a definition or dispatch clause only when the beat is that entry (for example a route action or a type-dispatch clause).
- Use working-tree line numbers so uncommitted changes link correctly.