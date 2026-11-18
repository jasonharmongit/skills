---
name: writing-for-agents
description: Writing documents for agents. Use when creating or editing skills, or modifying AGENTS.md or CLAUDE.md.
metadata:
  # Based on writing-for-agents by Matt Pocock
  author: jasonharmon
  dependents: create-or-edit-skill, fix-skill, refine-base-rules, refine-code-review-rules
---

## DRY

Do not repeat the same instructions multiple times throughout a skill or document. Just because a point relates to a subject, it doesn't mean that point needs to be made every time that subject is mentioned. If the same point is scattered/restated multiple times across the document, consider consolidating those points to a single, concise location.

## Information hierarchy

A document has two content types:

- **Steps**: the ordered actions the agent does.
- **Reference**: the definitions, rules, and facts that the agent reads on demand.

For each piece of content, decide where it sits in the **information hierarchy**. The three tiers rank content by how soon the agent needs it:

1. **In-file step**: the top tier. What the agent does, in order.
2. **In-file reference**: content the agent reads on demand. Many documents keep all their rules in one flat set at this tier. That is correct, not a problem.
3. **Deferred reference**: content in a separate file. A **context pointer** reaches it. A pointer is a line in context that names the content and gives the condition to read it. The agent loads the content only when the pointer fires. The file can be a sibling in the same folder, or an external file that any document points to.

**Progressive disclosure** is the arrangement these tiers give you. The agent reads the main file first, and it reads the rest in stages, only when it needs them. To build the arrangement, defer content: move it out of the main file and behind a pointer. The main file then stays legible.

To choose what to defer, look at the **branches**. A branch is a distinct case that the document handles. Keep the content that all branches need. Defer the content that only some branches reach. In a document with steps, reference that belongs in a deferred file hides the steps beside it. The agent then attends to those steps unreliably.

Keep each concept in one place in the file. Put its definition, rules, and caveats under one heading. The agent then reads the parts together.

Length is also a failure mode. A document can be too long, even when each line is unique and relevant. Use the hierarchy: defer reference behind pointers, and split the document by branch. Each path then carries only what it needs.

## Leading words

A **leading word** is a compact concept that is already in the model's pretraining, such as _lesson_, _fog of war_, or _tracer bullets_. The agent thinks with the word while it runs the document. Repeat the word as a token, never as a sentence. Its meaning then builds up across the uses, and the word anchors a large area of behaviour with few tokens.

You can coin your own word if you define it clearly. But a new word recruits no priors, so you pay in definition tokens what a pretrained word gives free. Look for an existing word first.

A leading word anchors twice:

- **Execution**, in the body. The agent reaches for the same behaviour each time it reads the word. In flat reference, the word also focuses attention on one class of thing to look for.
- **Invocation**, in a pointer. Put the same word in your prompts, your documents, and your codebase. The agent links that shared language to the content, and it reaches the content more reliably.

Look for the restatements that every document accumulates. A set of three terms at three sites, or a pointer that spends a sentence on one idea, can collapse into one token: "fast, deterministic, low-overhead" → _tight_ (a _tight_ loop).

## Phrasing

State the target behavior positively. Reserve prohibitions for hard guardrails that cannot stand as a positive alone.

## Pruning

- Check each line for **relevance**. A line loses relevance in two ways: it never bears on the task, because it is exposition or a branch you should defer. Or it goes stale, because the behaviour it describes changes.
- Look for **no-ops**, sentence by sentence. An instruction the model already obeys by default uses tokens for no reason.
- Use strong leading words: _be thorough_ is a no-op when the agent is already thorough, and the fix is a stronger word, such as _relentless_.

**Usually cut** (the agent already knows how):

- Generic investigation ("find the entry point", "read the tests", "trace the code")
- Standard tooling habits ("use aliases", "copy-pasteable blocks")
- The reason an instruction exists, when the instruction alone is enough
- Environment facts the agent can look up (`package.json` scripts, directory layout, `--help`)

**Usually keep** (specific to the document, or set by the user):

- Hard rules on behaviour that the agent breaks otherwise
- Non-obvious repo bootstrap (supervisor trees, env flags, paths that fail silently)
- The output shape the user expects (numbered workflow, before/run/after sections)
- Scope limits ("one happy path", "one basic scenario")
- Project conventions agents get wrong (fixtures unavailable outside test, never `ecto.reset` without approval)
