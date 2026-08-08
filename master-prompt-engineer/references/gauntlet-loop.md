# Gauntlet Loop Prompting: Iterative Build/Critique Loops for Coding Agents

## Contents
- Source and Verification Status (read this first)
- What a Gauntlet Loop Is
- Prerequisite: An Actual Agentic Harness
- The Method, Step by Step
- Building a Gauntlet Loop Prompt (template)
- Cost Discipline for Gauntlet Loops
- Relationship to This Skill's Other Tiers
- A Naming Collision Worth Knowing About
- Integration With This Skill

---

## Source and Verification Status (read this first)

This file documents a technique that is, as of this writing, a few weeks old. Treat it accordingly:
directionally useful and worth engineering into a prompt, but a community-popularized practice from
one practitioner's public writing, not confirmed vendor documentation from Anthropic, OpenAI, or
anyone else.

**Primary source, checked by direct fetch, not a secondary summary:** Matt Shumer, "How to Run a
Gauntlet Loop: The Prompting Method Behind Claude of Duty," published on his own site
(somethingbig.ai/gauntlet-loop) on July 27, 2026. Shumer coined the name that same day, after a
demo he posted two days earlier - a browser-based first-person shooter built in Three.js by an
agentic coding session - went viral. The prompt and roughly 55,000 lines of generated code are
open-sourced (github.com/mshumer/Claude-of-Duty).

**Honesty caveats worth carrying into any prompt built from this file:**
- Shumer's original post claimed the build was "one-shotted." He corrected this publicly the next
  day to "zero-shot," meaning one initial prompt kicked off a long unsupervised agentic session
  (many hours, a large number of subagent calls) rather than a single model response. The method's
  own value is in that distinction: it is a technique for structuring a long autonomous run, not for
  getting a complex build out of one inference call.
- The source material names Claude Opus 5 as the model behind the original demo. This skill's own
  product-knowledge grounding (see references/platforms.md and the product-self-knowledge skill for
  Claude's current model lineup) may not list that exact name with full confidence, since the claim
  comes from a July 27, 2026 community source rather than Anthropic's own documentation. Don't
  present the specific model name as confirmed fact if precision matters to the user; do present the
  loop structure itself with confidence, since that is what was actually verified.
- Anecdotal cost/scale figures circulating alongside this method (specific dollar or token totals
  from individual builders) are self-reported and unbenchmarked. Never repeat a specific number from
  this space as if it were a controlled measurement.
- Two adjacent but genuinely different open-source projects also use the word "gauntlet" for coding-
  agent work: `agent-gauntlet` and `old-coder`/`pi-gauntlet` are validation-loop runners (tests,
  linters, evidence reports) that put an agent's *output* through checks before you review it. That
  is a real and useful practice, but it is not this one. See "A Naming Collision Worth Knowing
  About" below before assuming which one a user means.

---

## What a Gauntlet Loop Is

In one paragraph: give a lead agent a goal and a concrete, inspectable bar to measure against - not
an adjective like "great" or "production-ready," but something the agent can actually look at and
compare its own output to. Let the lead agent decide how to split that goal into the smallest pieces
that can be judged independently. For each piece, spawn a builder and a separate critic in a fresh
context window. The critic never sees the builder's reasoning, only the goal, the bar, and the real
artifact - it makes a blind comparison, and if the artifact loses, it names the single biggest gap
and sends the piece back. That cycle repeats with no fixed number of rounds; it continues until the
work clears the bar or the user manually stops the run.

The mechanism this is exploiting is the same one Tier 5's Pattern 14 (Professional Objectivity /
Anti-Sycophancy) already names for a single model: an agent that graded its own homework tends to
explain why its choices were reasonable, because it remembers making them. A fresh critic with no
access to that reasoning has nothing to defend and can only compare what is actually in front of it.

---

## Prerequisite: An Actual Agentic Harness

This does not reproduce in a normal chat window. The source is explicit about this, and the
mechanism explains why: the loop needs a model that can open files, run code, render or execute the
result, inspect what it actually produced, and spawn other agents into their own clean context
windows for the critic role. Claude Code and Codex-style CLIs are named examples; the defining
property is tool use plus sub-agent spawning, not any specific product.

If the user's target is a single-turn chat prompt, a one-off script, or content that can't be
mechanically inspected and compared (an opinion piece, a first-draft brainstorm), a Gauntlet Loop is
the wrong tool - route through the ordinary Tiers 1-4 instead, or Tier 5 without this file if the
deliverable is agentic but not iterative-by-comparison in this specific sense.

---

## The Method, Step by Step

**1. State the goal, not the implementation.** Describe the destination. Do not prescribe the
architecture, the list of subsystems, or the exact steps. The source's own game prompt did not
mention a renderer design or a system list; it named the target quality bar and left the how to the
agent. Recent frontier models are good at decomposing an open-ended goal on their own; prescribing
every step in the prompt substitutes the prompt-writer's judgment for the model's, which is exactly
the failure mode this skill's DIAGNOSE step already calls "over-engineered branching logic" in
references/context-engineering.md.

**2. Give it a real, inspectable bar.** This is the load-bearing part of the whole method. "Make it
amazing" gives a critic nothing to compare against. A bar is something the critic can put side by
side with the actual output: real screenshots of a comparable shipped product, a reference codebase,
a test suite with a pass threshold, a latency or accuracy target, a set of exemplar paragraphs whose
clarity should be matched. If no natural bar exists, make finding one part of the task: instruct the
lead agent to propose a concrete comparison and justify why it plays the same role a reference
artifact would, before starting the loop.

**3. Let the agent own the decomposition.** Tell the lead agent to split the goal into the smallest
pieces that can be improved and judged separately, and to decide that split itself rather than
working from a list handed to it. "Make the game better" is too large a target for a critic to judge
usefully; "does this one tree read as favorably as the reference tree" is a target an agent can
actually attack and a critic can actually score. This is the same instinct behind Tier 5 Pattern 7
(Todo/Progress-List Tracking): break large ambiguous work into inspectable units before acting on it.

**4. Separate builder and critic completely.** The critic gets a fresh context window: the goal, the
bar, and the real artifact - never the builder's rationale or history. It behaves like a blind
A/B tester, not a reviewer reading a defense. When the artifact loses the comparison, its only job is
to name the single largest remaining gap and route the piece back to the builder. This directly
extends Tier 5 Pattern 8 (Sub-Agent Delegation With Concurrency Safety) and Pattern 14
(Professional Objectivity): the isolation is what makes the critique load-bearing rather than
decorative.

**5. Do not cap the round count - cap the resource instead.** The source's guidance is to keep
looping rather than stopping at a pre-set number of rounds, since a sufficiently high bar almost
always leaves another gap to close. That is in tension, on its face, with Tier 5 Pattern 26
(Execution Loop Guardrails: bounded steps, declared tools only) - reconcile it the way the source
itself actually does, not by ignoring either side: state an explicit **resource** budget (a wall-
clock ceiling, a token or dollar ceiling, or both) rather than a round-count ceiling. That preserves
the open-ended spirit ("keep going until it wins or you decide to stop") while still giving the run a
guardrail that prevents an unreachable bar from consuming unbounded compute. See "Cost Discipline for
Gauntlet Loops" below - this is the single most important place to apply it.

**6. Make progress visible without interrupting the run.** Have the lead agent maintain a simple,
self-updating status artifact (a live HTML page, a markdown workbench file) that shows what's
happening: screenshots, test results, a running log of what each round changed. This lets the user
check in from anywhere without breaking the agent's flow by interrupting it every few minutes to ask
for a status update, which itself costs a context switch and tokens.

**7. Optional - a smoothing pass.** When several pieces have been independently improved in
parallel, they can drift out of sync with each other even if each one individually improved. After a
wave of parallel loops, one fresh agent can inspect the whole result and reconcile inconsistencies -
not redesign anything, just make the parts read as one coherent artifact. This is genuinely optional;
the core of the method is steps 1-6.

---

## Building a Gauntlet Loop Prompt

The pattern below is this skill's own adaptation of the method - written fresh for this file, not
copied from the source prompt - built to also satisfy the token-efficiency checklist in
references/context-engineering.md. Fill in the bracketed sections; keep everything else as close to
this length as the task allows. Minimal is a feature here, not a shortcut: the shorter the prompt,
the more the lead agent's own judgment does the work, which is the entire premise of step 1 above.

```
GOAL: [the destination, described concretely, with no implementation detail]

BAR: [a real, inspectable comparison target - screenshots, a reference repo, a test suite,
a latency/accuracy number, exemplar prose. If none exists yet, your first job is to propose
one and state in one sentence why it plays that role, before starting the loop.]

Decide your own decomposition. Split this goal into the smallest pieces you can improve and
judge independently. Don't wait for me to hand you that list.

For each piece: build it, then hand it to a fresh sub-agent with no memory of your reasoning -
only the goal, the bar, and the actual artifact. That sub-agent makes a blind comparison against
the bar and, if you lose, names the single biggest gap. Fix that gap. Loop again.

BUDGET: stop when you clear the bar, or when you hit [a stated time/token/dollar ceiling] -
whichever comes first. No fixed round count otherwise; keep going as long as there's a real gap
left to close.

Keep critic context minimal: goal + bar + artifact only, not the accumulated history of every
prior round. Maintain one running status file (screenshots/results/log) instead of asking me for
updates - I'll check it myself.

Use ultracode / your highest effort setting for this run if your harness exposes one.
```

**A worked micro-example (goal substitution only, not a different structure):** for "build a
CLI tool that migrates a Postgres schema safely," the bar might be a curated set of real-world
migration edge cases (a maintainer's own past incident postmortems, or a small suite of intentionally
gnarly schemas) rather than screenshots; the decomposition the lead agent finds on its own might turn
out to be per-migration-type (adding a column, renaming with data preserved, splitting a table)
rather than per-file. The structure travels; the substance of goal and bar do not.

---

## Cost Discipline for Gauntlet Loops

This method is a multi-agent fan-out by construction: a lead agent, plus a builder and a fresh-context
critic for every independently-judged piece, run for an open-ended number of rounds. Anthropic's own
published account of building a multi-agent research system found that agentic workflows typically
use roughly 4x the tokens of an ordinary chat turn, and that full multi-agent architectures run
roughly 15x - a cost that is worth paying for genuinely high-value, independently-decomposable work
and easy to pay without earning it otherwise. A Gauntlet Loop should be treated as squarely in the
"high-value" bucket that multiplier is meant for, and engineered accordingly rather than left to run
unmonitored:

- **State the resource budget explicitly** (step 5 above) - an unbounded loop against a deliberately
  unreachable bar is the method working as intended, which is exactly why it needs an explicit ceiling
  rather than relying on the agent to notice it should stop.
- **Keep the critic's context minimal**, per the method's own design: goal, bar, and current artifact,
  not the full history of prior rounds. This is already token-lean by construction if followed as
  written; the failure mode to watch for is a builder or lead agent that starts appending full round
  history "for context" out of caution.
- **Keep the meta-prompt itself short** (see the template above). Over-specifying architecture and
  decomposition doesn't just fight the model's own judgment (step 1); every prescribed detail is
  tokens spent in a prefix that gets re-sent or re-reasoned-about across every subsequent round.
- **Where the harness supports prompt caching, keep the goal/bar block byte-identical across rounds**
  and place any per-round dynamic content (the latest artifact, the latest critic verdict) after it.
  See "Token-Efficient Prompt Construction" in references/context-engineering.md for the mechanics and
  the evaluation this is grounded in (Lumer et al. 2026, arXiv:2601.06007) - cache invalidates on any
  change to that prefix, including the tool set, so treat the goal/bar statement as fixed for the
  run's duration.

---

## Relationship to This Skill's Other Tiers

A Gauntlet Loop prompt is not a replacement for Tiers 1-4 or Tier 5 - it's a specific operating
pattern to layer on top of them when the deliverable calls for it:

- **Framework/technique core (Tiers 1-4):** ReAct still describes the builder's own act-observe-
  reason cycle inside each round; RISEN or POWER still shapes the goal/bar statement itself if it
  needs more structure than the minimal template above provides.
- **Tier 5 (production-patterns.md):** Pattern 8 (Sub-Agent Delegation), Pattern 14 (Anti-
  Sycophancy/Professional Objectivity), Pattern 26 (Execution Loop Guardrails), and Pattern 7
  (Todo/Progress Tracking) are the specific patterns this method is a concrete instance of. If the
  user wants the underlying mechanics explained rather than just the recipe, point there.
- **Context engineering (context-engineering.md):** an open-ended Gauntlet Loop is close to the
  textbook case for the long-horizon techniques in that file - structured note-taking (the live
  status artifact in step 6 above IS this technique, applied) and sub-agent architectures (steps 3-4)
  both show up here under different names.
- **The Unknowns Discovery Protocol (unknowns-discovery.md):** choosing the bar in step 2, when no
  obvious one exists, is functionally the "References" technique from that file - replacing a
  hard-to-verbalize quality target with something concrete to react to.

---

## A Naming Collision Worth Knowing About

If a user says "use the gauntlet on this" or "run this through the gauntlet," don't assume which
practice they mean without a quick check - the word currently covers two unrelated things in
circulation:

1. **This file's practice (Gauntlet Loop):** an iterative *build* pattern - split, build, blind-
   critique against a real bar, repeat - used to produce a better artifact through many rounds.
2. **Validation-loop tooling** (`agent-gauntlet`, `old-coder`/`pi-gauntlet`, and similar): a
   *verification* pattern - run tests/linters/evidence checks on code an agent already wrote, so a
   human reviews a pass/fail report instead of the raw diff. This is closer to Tier 5 Pattern 6
   (Read/Verify Before Claim-Done) than to anything in this file.

A user asking to "build X well" or "make this as good as Y" almost always means practice 1. A user
asking to "prove this works" or "check the agent's code before I review it" almost always means
practice 2. If the phrasing is genuinely ambiguous, ask - this is exactly the situation Tier 5
Pattern 3 (Directive vs. Inquiry Ambiguity Protocol) already covers.

---

## Integration With This Skill

**Into the generated prompt.** When Phase 0 Step 6 (Production Pattern Check) identifies the
deliverable as an autonomous coding-agent build meant to run an iterative improve-against-a-bar cycle
- not just any agentic system prompt - pull this file in alongside Tier 5 and write the goal/bar/
budget structure into the generated prompt explicitly, using the template above as a starting point
rather than a fixed script.

**When this layer is not worth reaching for:** a one-off script, a single-file utility, a request
that already has a clear, narrow spec and doesn't benefit from open-ended iteration, or - the most
common miss - a deliverable the user will run in a plain chat interface rather than an agentic
harness. Forcing a multi-round critic loop onto a five-minute coding task is the same over-
engineering failure mode already flagged for Tier 5 and the Unknowns Discovery Protocol, and here it
carries a real token bill on top of the usual over-engineering cost.
