# Named Framework Library - Worked Examples

This file was referenced from SKILL.md but did not exist in earlier versions of this skill.
It now contains full worked examples for every framework named in the FRAMEWORK TOOLKIT table.
A framework is the **macro-shape** of a prompt (which sections exist, in what order). A
technique (see `techniques.md`) is what happens **inside** a section. Frameworks and techniques
stack: pick one framework as the skeleton, then layer in 2-4 techniques for the reasoning process.

---

## CO-STAR - Creative, Content, Communication

**Shape:** Context, Objective, Style, Tone, Audience, Response format.

**Why this order:** Context and Objective come first because they are the highest-leverage tokens
for narrowing the output distribution before any stylistic instruction is applied. Style/Tone/
Audience come next because they modulate the same content differently. Response format comes last
so it sits closest to generation (highest positional attention for a well-defined final constraint).

**Worked example - LinkedIn thought-leadership post generator:**

```
# Context
You are writing for a first-generation MBA graduate targeting equity research and investment
analyst roles in the Indian market. The reader has just published a sector note and wants a
LinkedIn post that drives profile visits from hiring managers at boutique research firms.

# Objective
Produce one LinkedIn post that converts a technical sector note into an accessible narrative
hook, drives comments from finance professionals, and signals the author's analytical rigor.

# Style
Plain-English financial storytelling. Short sentences. One central metaphor. No jargon without
a one-line translation immediately after it.

# Tone
Confident, curious, understated - never salesy, never using superlatives ("game-changing",
"revolutionary").

# Audience
Indian equity research analysts, boutique fund PMs, and MBA hiring managers on LinkedIn.

# Response format
150-200 words. Opening hook line (no more than 12 words). 3 short paragraphs. One closing
question that invites comments. No hashtags inside the body; hashtags listed separately after.
```

**When NOT to use:** Purely technical or code-generation tasks - CO-STAR's tone/audience axes add
no value when there is no human reader to persuade or delight.

---

## RISEN - Role, Instructions, Steps, End goal, Narrowing

**Shape:** Role, Instructions, Steps (numbered), End goal, Narrowing (explicit exclusions/scope).

**Why this order:** RISEN front-loads Role because complex multi-step analytical tasks benefit
most from a stable evaluative lens applied consistently across every step. Steps are numbered and
sequential to prevent the model from collapsing multi-stage reasoning into a single pass. Narrowing
closes the prompt because exclusion boundaries are most effective as the last thing read before
generation - a form of KV-cache prefix engineering (see `techniques.md`, Technique 16).

**Worked example - Equity research variance analysis:**

```
# Role
You are a sell-side equity research analyst covering Indian auto OEMs, specializing in
volume-mix-price decomposition of quarterly revenue variance.

# Instructions
Decompose the YoY revenue variance for the given quarter into volume, mix, and price
contributions. Flag which driver explains the majority of the variance.

# Steps
1. Extract reported revenue, unit volumes, and average selling price (ASP) for the current and
   prior-year comparable quarter.
2. Calculate the volume effect: (current volume - prior volume) x prior ASP.
3. Calculate the price effect: (current ASP - prior ASP) x current volume.
4. Calculate the mix effect as the residual: total variance - volume effect - price effect.
5. Rank the three effects by absolute contribution.

# End goal
A three-line variance bridge stating which driver dominated and by how much, in the RK Advisory
newsletter format (bold one-line headings, YoY/QoQ shortforms, one decimal on percentages).

# Narrowing
Do not comment on stock price target or valuation - this task is variance decomposition only.
Do not use segment-level data if only consolidated figures are provided; state the limitation.
```

**When NOT to use:** Single-turn creative or conversational tasks - the Steps section adds
overhead without benefit when there is no genuine multi-stage computation or reasoning to sequence.

---

## CRISPE - Capacity/Role, Insight, Statement, Personality, Experiment

**Shape:** Capacity and Role, Insight (background/context the AI needs), Statement (the actual
task), Personality (behavioral/tonal traits), Experiment (request for multiple variations).

**Why this order:** CRISPE is optimized for persona and assistant design, where the "Experiment"
step - asking for N variations - is what lets a designer compare personality calibrations side by
side before committing to one. This makes CRISPE the right framework specifically when the deliverable
IS a persona or system prompt, not content produced by one.

**Worked example - Designing a finance-newsletter voice assistant persona:**

```
# Capacity and Role
Act as a senior equity research mentor persona for an AI assistant used by MBA students
practicing sector analysis.

# Insight
The assistant will be used by students producing RK Advisory-format newsletters (six-section
structure: stock news, sector news, macro factors, company familiarity, financial statement
impact, financial model changes). Most users are early-career and prone to vague language.

# Statement
Draft the persona's system prompt: identity, first response to a vague student query, and
tone rules for correcting imprecise financial language.

# Personality
Rigorous but encouraging. Uses concrete numeric examples over abstract advice. Never says
"good job" without pointing to a specific improved data point. Short sentences, no filler.

# Experiment
Produce three variations of the opening line the persona uses when a student submits a
newsletter draft with no cited data points, ranging from most direct to most Socratic.
```

**When NOT to use:** One-off content generation tasks - CRISPE's overhead (Insight + Experiment)
only pays off when the deliverable is a reusable persona or system prompt.

---

## BAB - Before, After, Bridge

**Shape:** Before (the painful current state), After (the desired future state), Bridge (how the
AI's output gets the reader from one to the other).

**Why this order:** BAB is a persuasion primitive, not a task-instruction primitive. It works by
maximizing the semantic distance between Before and After so the Bridge (the actual content) reads
as the resolution to a felt gap. This is why it stacks with TRACE rather than replacing it - BAB
frames the emotional arc while TRACE supplies the evidentiary backbone.

**Worked example - Cold LinkedIn outreach to a hiring manager:**

```
# Before
The reader is a hiring manager at a boutique equity research firm who receives dozens of
generic "I'm passionate about markets" outreach messages from MBA students each week.

# After
The reader immediately recognizes this candidate as someone who already thinks like a junior
analyst, based on one specific, falsifiable claim about a stock or sector.

# Bridge
Write a 4-sentence LinkedIn connection request that opens with one specific data point from a
recent sector note (not a generic claim), states the ask (a 15-minute call) in one sentence,
and closes without a generic "let me know" filler line.
```

**When NOT to use:** Neutral informational or technical writing where there is no persuasion
goal - BAB manufactures an emotional gap that reads as forced outside of marketing/outreach copy.

---

## TRACE - Task, Request, Action, Context, Example

**Shape:** Task (what kind of output), Request (the specific ask), Action (what the AI should
literally do, step by step), Context (background data), Example (a model output to imitate).

**Why this order:** TRACE is built for argumentative/persuasive writing where the model needs a
concrete worked Example at the end to lock the register - persuasive tone drifts more than
analytical tone across long generations, so TRACE anchors it with a literal exemplar as the last
thing read before generation.

**Worked example - Persuasive investment thesis memo:**

```
# Task
A one-page persuasive investment memo arguing for initiating coverage on a specific stock.

# Request
Argue for a BUY rating using three supporting data points, each rebutting the most likely
bear-case objection to that point.

# Action
1. State the thesis in one sentence.
2. For each of the three data points: state the bull case, then the strongest bear objection,
   then the rebuttal in one sentence.
3. Close with the single biggest risk to the thesis, stated honestly.

# Context
[insert financial data, valuation multiples, peer comps]

# Example
"Bull case: Revenue grew 18% YoY driven by premiumization. Bear objection: raw material costs
rose 12% YoY and could compress margins further. Rebuttal: management has already passed 60% of
the cost increase through price hikes effective this quarter, per the earnings call transcript."
```

**When NOT to use:** Tasks with no adversarial or comparative structure - the rebuttal step is
wasted effort on a purely descriptive task.

---

## CRAFT - Context, Role, Action, Format, Target audience

**Shape:** Nearly identical to CO-STAR but collapses Style/Tone into Role and drops explicit
Objective in favor of Action. Optimized for professional branded documents where the Role itself
(e.g., "senior brand copywriter for a fintech company") already implies the tone.

**Worked example - Branded one-pager:**

```
# Context
A fintech startup is producing a one-page investor-facing product overview for its Series A deck.

# Role
Act as the startup's senior product marketing writer, whose house style is data-forward and
avoids buzzwords like "disruptive" or "seamless."

# Action
Write the one-pager: a problem statement, the product's mechanism in plain language, one proof
point (a metric), and a closing call-to-action line for the investor reader.

# Format
One page. Four labeled sections. No bullet lists longer than 3 items. No em dashes.

# Target audience
Series A investors evaluating 15-20 similar decks in a single week - assume low patience for
unexplained jargon and high scrutiny of any unsupported claim.
```

**When NOT to use:** Use CO-STAR instead when Style and Tone genuinely diverge from what the Role
alone implies (e.g., a technical expert asked to write in a deliberately casual voice).

---

## POWER - Purpose, Output, Work (process), Examples, Requirements

**Shape:** Purpose (why this matters), Output (the deliverable), Work (the research/analysis
process to follow), Examples, Requirements (hard constraints).

**Why this order:** POWER is built for research and evidence-heavy tasks. Purpose is stated first
specifically to give the model a criterion for judging source relevance during the Work phase -
without a stated purpose, evidence-gathering has no filter and pulls in tangential material.

**Worked example - Sector deep-dive research brief:**

```
# Purpose
Inform an investment committee decision on whether to increase portfolio exposure to Indian
two-wheeler EV manufacturers over the next two quarters.

# Output
A 600-word research brief with a clear directional recommendation.

# Work
1. Establish current market share and growth rate of the top 3 listed EV two-wheeler players.
2. Identify the two macro factors most likely to move demand in the next two quarters
   (subsidy policy changes, input cost trends).
3. Cross-check company-level guidance from the most recent earnings calls against the macro view.
4. Flag any contradiction between company guidance and macro data as the key risk.

# Examples
[Provide 1-2 examples of the target house style - e.g., a prior RK Advisory-format brief.]

# Requirements
Every quantitative claim needs a stated source or is flagged as an estimate. No forward-looking
statement without a stated confidence level (high/medium/low).
```

**When NOT to use:** Time-boxed or low-stakes questions - POWER's Work phase encodes a multi-step
research process, which is overhead when a single well-known fact answers the question.

---

## RACE - Role, Action, Context, Expectation

**Shape:** A lightweight four-slot framework: Role, Action (the task), Context (background), and
Expectation (the success bar). This is the fastest framework to fill in and is best used as a
floor, not a ceiling - a diagnostic starting point before deciding whether a heavier framework
(RISEN, CO-STAR) is actually warranted.

**Worked example - Quick internal Slack-style summary:**

```
# Role
You are summarizing for a teammate who has 90 seconds before a meeting.

# Action
Summarize the attached earnings call transcript into the 3 most decision-relevant takeaways.

# Context
The teammate cares about guidance changes and margin commentary, not historical results already
known to the team.

# Expectation
3 bullet points, each one sentence, no preamble.
```

**When NOT to use:** Anything requiring multi-step reasoning, persuasion, or precise stylistic
control - RACE has no slot for Steps, Tone, or Format, so it under-specifies complex tasks.

---

## Agile Prompting - Sprint Decomposition

**Shape:** Not a fill-in-the-blank template but a decomposition strategy: break a large generation
task into sequential "sprints," each producing one reviewable artifact before the next sprint
begins, rather than requesting the full deliverable in one shot.

**Why it works:** Long single-shot generations compound early errors - if the outline is wrong,
every downstream section inherits the mistake. Sprint decomposition inserts a human (or automated)
checkpoint between structural decisions and content generation, so errors are caught at the
cheapest possible stage.

**Worked example - Long-form equity research report:**

```
Sprint 1 - Outline only: Section headers and one-line scope for each section. Stop and wait
for approval before continuing.

Sprint 2 - Financial analysis section: Full P&L/BS/CF walk-through, using the approved outline
as the fixed skeleton. Do not alter the outline structure.

Sprint 3 - Valuation section: DCF and comps, cross-referencing figures introduced in Sprint 2
for consistency.

Sprint 4 - Executive summary: Written last, synthesizing Sprints 1-3, capped at 150 words.
```

**When to use:** Any deliverable long enough that a structural mistake would be expensive to
unwind - reports, research briefs, multi-slide decks, long technical specs. See also
`references/production-patterns.md` Pattern 5 (Plan-Then-Execute Gate), which is the agentic-tool
equivalent of this same idea.

**When NOT to use:** Short deliverables under roughly 300 words - the checkpoint overhead exceeds
the cost of a single redo.

---

## Framework Selection - Quick Reference

| Signal in the request | Framework |
|---|---|
| Human reader must be persuaded, delighted, or emotionally moved | CO-STAR, BAB, TRACE |
| Multi-step analytical or computational process | RISEN, POWER |
| The deliverable IS a persona, assistant, or system prompt | CRISPE |
| Professional document where Role already implies tone | CRAFT |
| Quick, low-stakes, single-pass task | RACE |
| Deliverable is long enough that structural errors compound | Agile Prompting (sprint decomposition), layered onto whichever framework above fits the content |
