# Chalk Voice — Writing Principles

Optimise for the reader, not the writer.

Whatever you're writing — a commit, an issue, a PR description, a docs page — the reader is trying to do one of four things: acquire cognition, acquire action, apply cognition, apply action.
[Diataxis](https://diataxis.fr) names these quadrants (explanation, tutorial, reference, how-to) and they apply recursively at every level, from a whole docs site down to a paragraph inside a commit body.

This file has three parts:

1. **Universal principles** — apply everywhere, regardless of quadrant.
2. **The four quadrants** — what each is for and how the voice differs.
3. **Artefacts as compositions** — which quadrants each chalk artefact (commit, issue, PR, docs page) actually occupies.

The existing chalk skills (`chalk:commit`, `chalk:pr`, `chalk`) inherit from this file — they stop reciting principles and just name their quadrant(s).

## Universal principles

These apply to all writing in this plugin's voice.

### Concrete over abstract

Ground the reasoning in specific scenarios, code snippets, data, real identifiers — not placeholders.
Abstract explanations are harder to follow and easier to misinterpret.

Not this:
> Updated the type system to better represent logical types, improving the separation between compile-time and run-time representations.

This:
> A logical type is one of: Mono (null, scalar, listy, struct), Maybe (nullable mono), or Poly (set of monos).
> Previously, VectorType represented physical types — the compile-time type and run-time type could differ because many physical representations map to one logical type.
> For physical representations, we now exclusively use Arrow's `Field` class.

**Identifiers stay; numbers leak.**
Class names, method names, data structures, callsites, error codes are anchored to the codebase and stay accurate as instances of a bug recur.
Magnitudes, sizes, timings, thread counts, log timestamps are anchored to one reproduction and go stale the next time the same class of bug occurs with different inputs.
The first belongs in the commit body; the second belongs in the issue.

Sometimes a magnitude *is* part of the architecture ("the snap write lock is held for at most one record's worth of work") — the test isn't "is there a number?" but "is the number a property of the system, or a property of one log?"

Not this (pinned to one log dump):
> DETACH of an external-source database leaked ~96kb of live-index allocator memory whenever the leader's persister was mid-`importTx` when the close ran.

This (the shape):
> DETACH of an external-source database could race the leader's persister against `LiveIndex.close()` — the persister kept mutating `LiveIndex` after `close()` returned, freeing buffers under in-flight writes.

Both are concrete.
The first names `96kb` and `importTx` — facts true of one log dump that won't recur identically.
The second names `LiveIndex.close()` and "the persister" — structural identifiers that stay accurate across reproductions.

### Line formatting depends on the destination

Where prose ends up shapes how to break lines.
No 80-character wrapping in either case.

**Files that get `git diff`'d — sentence per line.**
Docs pages, code comments, READMEs, any markdown checked into the repo.
Sentence-per-line keeps diffs minimal — a one-sentence edit touches one line — and makes structure easier to scan in plain source.

**Everything else — paragraph per line.**
Commit messages, issue bodies, issue comments, chalk comments, PR descriptions, progress sections — anything read primarily after rendering (GitHub, `gh`, commit views) rather than as a tracked source file.
GitHub Flavored Markdown renders single newlines as soft `<br>` breaks, so sentence-per-line fragments the rendered output into staccato lines instead of paragraphs that wrap to the viewport.
Put each paragraph on a single line, separate paragraphs with a blank line, and let the rendering wrap.

The split is `git diff`, not the artefact.
A docs page goes sentence-per-line because reviewers read its diff.
A commit message goes paragraph-per-line because nobody diffs it — they read it rendered in `git log`, GitHub, or a PR commit list.

### Lead with the problem or context

Set up *why this matters* before the solution.
The reader needs the situation before the fix makes sense.
This applies across quadrants — a how-to opens with the goal, an explanation opens with what needs explaining, a reference block sits under the concept it references.

### Be concise — but keep the reasoning

A terse note with no *why* is just as unhelpful as a verbose one.
When the reasoning is complex but the change is simple, say so.
"Simple change in the end: ..." helps the reader calibrate.

### No marketing fluff

"Powerful", "seamless", "blazing-fast", "revolutionary" — cut them.
State the capability; let the reader draw the conclusion.

### Discover project conventions, don't impose them

Version markers, changelog shapes, file layout, cross-link style — these vary between projects.
Read the docs README, existing pages, and recent commits before writing, and match what's there.

## The four quadrants

Diataxis's axes: **acquisition vs application** × **cognition vs action**.

| | Acquire | Apply |
| --- | --- | --- |
| **Cognition** | Explanation | Reference |
| **Action** | Tutorial | How-to |

### Explanation — acquire cognition

The reader wants to *understand*.
Mental models, why-this-way, decisions, tradeoffs, dead ends, invariants.

**Voice:**
- Discursive.
  Grounded in concrete examples that ground abstractions.
- Lead with the problem or the concept you're explaining.
- Capture the *why*, not the *what* — the reader can see the what elsewhere (the diff, the config, the code).
- Teach the mental model when the change introduces or reshapes a concept.
  Don't assume the reader has the model you built up while implementing.

**What to capture:**
- **The problem first**: why this matters, before what you did.
- **Decisions and tradeoffs**: why X over Y, what constraints drove the choice.
- **Counter-intuitive findings**: anything that surprised you or would surprise a developer familiar with the project.
- **Dead ends**: what didn't work and why — this prevents re-investigation.
  Include *why* the wrong approach seemed right, not just that it was wrong.
- **Scope boundaries**: what was explicitly out-of-scope and why, so nobody re-opens a question you already closed.
  If there's a natural "next up", say so.
- **The mental model**: when the change introduces or reshapes a concept, explain the model — what the abstractions are, how they relate, why they're shaped this way.

**What to omit:**
- Obvious details self-evident from the diff, the code, or the issue description.
- Play-by-play of mechanical steps ("then I ran the tests", "then I edited the file").
- The journey of how you got there — optimise for the reader, not the writer.
- Incident-specific data (leak sizes, timestamps, magnitudes, thread counts) pulled from one reproduction's logs — the issue carries that, the commit body shouldn't restate it.

After drafting, re-read each paragraph adversarially and ask:
- Could a reader infer this from the diff?
- Am I defending a choice the reader would accept on sight?
- Am I describing what this change doesn't do?
- Am I speculating about how users will behave?
- Would this sentence still be accurate if the same class of bug recurred with different inputs? (If no, it's reproduction detail.)

Cut anything that answers yes.
The chalk thread and `Refs` trailers carry the journey; the body carries the destination.

**Examples.**

Not this (restates the diff):
> Made TemporalBounds fields volatile and added a snapshot method that reads both atomically.

This (captures the decision):
> Volatile over lock — simpler, lower contention, sufficient for read-mostly pattern.
> Immutable TemporalBounds + AtomicReference would be cleanest but requires changing every call site that mutates bounds — out of scope for a bugfix.

Not this (mechanical play-by-play):
> Investigated the flaky test. Found it only failed in the full suite. Added logging to narrow it down. Discovered a race condition in TemporalBounds.intersect().

This (what matters to the next person):
> Initially suspected a test ordering issue since it only failed in the full suite — red herring.
> The full suite just increases thread contention enough to trigger a race in TemporalBounds.intersect(), which reads validFrom and validTo non-atomically.

Not this (obvious from the diff):
> Added a new `resolveTimeout` parameter to the `QueryConfig` class and threaded it through to the executor.

This (why it exists):
> Queries against the Kafka log can stall indefinitely when a partition leader is mid-election.
> A per-query timeout lets callers fail fast rather than blocking on a cluster event they can't control.

Not this (states the fix without the problem):
> This PR changes UPDATE to not create new rows when all values remain the same.

This (problem first, with a concrete example and explicit scope):
> UPDATE was creating duplicate rows even when no values actually changed.
> Uses type-strict equality — `UPDATE docs SET a = 1.0 WHERE _id = 1` on a doc with `{:a 1}` *will* create a new record because `1 ≠ 1.0`.
> PATCH is out of scope of this PR (see #5030).

### How-to — apply action

The reader has a goal and wants to *achieve it*.
Deployment, configuration, integration, a specific operational task.

**Voice:**
- Goal stated up top: "To do X, ...".
- Numbered, imperative steps.
- Terse — no narrative padding.
- Concrete identifiers, not placeholders (`my-kafka`, not `cluster-name`).
- Prerequisites and assumptions stated before the steps, not mid-flow.
- A closing "you should now see …" check, so the reader knows whether it worked.

How-tos don't teach — they assume the reader already has enough cognition to recognise the steps.
If the reader needs to *understand* something first, that's an explanation, and it belongs in a different section (or a different page).

### Reference — apply cognition

The reader is working and needs to *look something up*.
A CLI flag, an API signature, a config key, a SQL clause.

**Voice:**
- Neutral, exhaustive, structured.
- Tables, definition lists, grammar productions.
- No narrative, no "we"/"you".
- Alphabetised or structurally ordered (not narratively ordered).
- Complete — every flag, every field, every case.
- **Keep rationale out of the body.** This is the one quadrant where chalk's usual "always capture the why" does not apply inline. 
  The *why* — tradeoffs, motivation, upgrade story — belongs in an adjacent explanation section or changelog block, not next to the definition. 
  A reader looking up a flag wants the semantics, not the story behind it.

Reference is unforgiving: if it's incomplete, the reader gets burned.
Better to generate it from the source of truth (schema, CLI help, spec) than to hand-write and drift.

Watch for rationale that drifts in unnoticed — phrases like *"This exists for compatibility with..."*, *"These functions are provided so that..."*, *"This was added because..."* are explanation quadrant sentences wearing reference clothing.
Strip them, or relocate them to the adjacent changelog or explainer.

### Tutorial — acquire action

The reader is new and wants to *learn by doing*.
A guided first experience.

**Voice:**
- Gentle pacing, small wins, hand-holding.
- "You should see X" after each step, so the reader can confirm progress.
- Friendly and reassuring — confidence-building.
- Single working path — no branching ("or you could do Y"), no optionality.
- Ends with a clear next step ("now that you've done X, try Y").

Tutorials are the hardest quadrant to get right.
Explanation can be shortened; reference can be incomplete; a bad tutorial *strands the reader* and they don't come back.

## Artefacts as compositions

Most things you write aren't one quadrant — they're a composition.
Each paragraph, section, or code block does one quadrant's job; don't blur them.

**Commit bodies** are **explanation** artefacts, end-to-end.
The diff is the code change, not a reference section inside the commit.
Open with the problem (explanation framing), deliver the reasoning (explanation payload).

**Issue descriptions** are **explanation** artefacts, with reference-shaped evidence embedded (log excerpts, stack traces, block dumps).
The evidence is illustrative material inside the explanation; it's not a separate reference section.

**PR descriptions** are **explanation** artefacts.
Context, decisions, tradeoffs, dead ends, scope — all explanation.
Usage examples, test-plan checklists, and commit lists have a reference *shape* but they're illustrations embedded in explanation, not standalone reference.

**Docs pages** play all four.
Each page leans toward one quadrant at the top level; sections within it may hit other quadrants.
A how-to page often has a short explanation intro (what this setup is for), numbered steps (how-to), a commented config block (reference), and a closing section on failure modes (explanation).
Each section does one quadrant's job; they don't blur together.

The test isn't "where does this page live?"
It's "what is *this paragraph/section* for?"

## A note on the skills

The chalk skills (`chalk:commit`, `chalk:pr`, the core `chalk` skill, and eventually `chalk:docs`) don't restate the rules in this file.
They name the quadrant(s) they cover and inherit the voice.

This keeps guidance in one place and lets the skills stay short.
