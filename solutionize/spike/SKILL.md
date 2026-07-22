---
name: spike
description: Produce a concise current-state narrative
disable-model-invocation: true
metadata:
  dependents: solutionize
---

## Writing principles

**Voice and tone:**
- Active voice only. Name who acts (the user, a worker, an external system)—not which module "handles" something
- Plain prose first. Module and function names belong in the story only when they are the clearest way to say it
- Do not open beats with a module as the subject ("A registrant saves facts in [BusinessInformationComponent](…)")—say what the user or system does, then anchor the beat with a link

**Narrative flow:**
- Start at the recognizable entry point (user-facing feature, API, or main job)
- Follow work through layers that matter **for this specific issue**, no further
- Proceed chronologically: who does what, what runs first, how outputs become inputs downstream
- One **main beat** per paragraph (or short sequence). Each beat is a skimmable step in the story; the reader should grasp the flow without clicking anything

**Beat anchors:**
- At each main beat, embed **one** markdown link on the word or short phrase that best names the action or decision point in plain English—e.g. "When the user [submits](/Users/…/business_information_component.ex#L35) the business information step of campaign registration…"
- The link label is whatever reads naturally in the sentence (often a verb or short phrase), not a module name, file path, or `Module.function` string
- Surrounding context stays unlinked: "business information step" and "campaign registration" do not need links unless that beat is specifically about them
- Do not litter the paragraph with links for every helper, callback, or symbol you could name

**Sentence-level clarity:**
- Each sentence answers one of: **who does what**, **in what order**, or **what data shape** moves between stages
- Introduce implementation detail **in context** as the story reaches it; do not front-load a call-chain catalog

**Paragraph structure:**
- **Maximum 3 sentences per paragraph.** Longer paragraphs signal you haven't distilled the idea
- Use white space generously—it helps readers absorb each beat before moving forward
- Start each paragraph with a new actor or stage in the flow

## Anti-patterns to avoid

**Scope and framing:**
- No preamble ("In this section we'll see…"). Jump straight into the story
- Do not write a spec or design document. You are writing a **narrative**, not documentation—keep it tight and digestible
- Do not overscope. Trace the path from entry point through affected layers **for this specific issue**; do not catalog the entire system
- No future-state language. If a sentence drifts toward "should" or "could," rewrite it as present behavior or a current constraint
- No implementation recommendations, options, or proposed changes. You are describing **what exists**, not planning what could exist
- No diagrams, tables, or visual aids
- No abstract glosses ("the system checks X"). Say who checks X and when

**Narrative structure:**
- Do not use backticks for code references — markdown links only (see **Code references**)
- Do not open deep (inside a private helper or a single schema field) and trace upward
- Do not write a paragraph per function, each sentence starting with `[function_name](…)` chained to the next—collapse internal steps into plain English unless a beat truly warrants its own anchor
- Do not use narrative shortcuts: "flows," "feeds," "lands in," "wires through," "hands off." Use concrete verbs and say who does what

**Language:**
- Precise over poetic. Every word should earn its place
- Active voice. State who does what; avoid passive constructions that hide agency

## Code references

Use markdown links only for **beat anchors**—not for every module, function, or file you mention in passing.

**Link targets** — resolve the file with realpath before writing:

```bash
realpath lib/path/to/file.ex
```

Use that output as the URL. Append `#L…` when the anchor should jump to a specific definition or clause. Do not use relative paths in link URLs.

**Labels** — whatever fits the sentence: usually a verb or short phrase ("submits," "derives campaign type," "builds TCR params"). Do not default to module names, bare function names, namespaces, arity, or file paths as labels.

**Outside lib/** (tests, priv/, scripts) — same rule: natural phrase in the sentence; URL points at the resolved absolute path (file-name-only labels are fine when the beat is clearly about that file, e.g. [structure_test.exs](…)).

---

## Workflow

**Do not modify code, tests, configuration, or project assets.** This is purely investigatory and collaborative.

1. **Read to understand:** Study all relevant modules, call sites, tests, tickets, and documentation until you can trace the path clearly.
2. **Write the story:** Follow the principles, anti-patterns, and **Code references** above. Write linearly from entry point to the layers relevant to the issue. Plain English throughout; one natural-language anchor link per main beat.
3. **Review and iterate:** Re-read against each anti-pattern. Cut link sprawl, module-as-subject openings, and passive sentences. Repeat until the story reads like a tight walkthrough with skimmable anchors, not a symbol index.
