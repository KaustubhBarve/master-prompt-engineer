# Unknowns Discovery Protocol: Pre-Build and Mid-Build Elicitation

## Contents
- Source and Verification Status
- Core Concept: Map vs. Territory
- The Four Unknowns
- When to Reach for This Layer
- The 8 Techniques (Pre-Build, Mid-Build, Post-Build)
- How This Comes Together: A Worked Walkthrough
- Integration With This Skill

---

## Source and Verification Status (read this first)

This file is built from Anthropic's official blog post "A field guide to Claude Fable 5: Finding
your unknowns," published July 6, 2026 on claude.com/blog, written by Thariq Shihipar (member of
technical staff, Anthropic). The content below was checked against the live article directly, not
taken from a secondary summary.

That is a different provenance situation than Tier 5 (`production-patterns.md`), which draws on an
unverified, community-maintained archive. Everything here traces to a named Anthropic author on an
Anthropic-owned domain. The technique names below (Blind Spot Pass, Interview, References, and so
on) are the article's own terms, kept intact because precise terminology matters more here than
elsewhere in this skill. Every explanatory paragraph and every example prompt is original writing
for this skill's actual domains (equity research, finance newsletters, resume work, hackathon
builds, creative writing), not reproduced from the source, which is framed around agentic coding
with Claude Code and Claude Fable specifically.

One correction worth recording: this file was first drafted from a third-party YouTube video
summary of the article rather than the article itself. Checked against the primary source, that
summary undercounted the pre-build phase (it named 4 techniques; the article names 5 and the
summary had dropped "Implementation Plan" entirely), split the mid-build phase into two techniques
where the article names one ("Implementation Notes," which folds starting a fresh context into the
same technique rather than treating it as a separate step), and used the label "Show, Don't Tell"
for what the article itself calls "References." The structure below follows the verified article.

The article is framed around Claude Fable 5, a model released after most training data cutoffs. If
a future Claude reading this skill file doesn't recognize "Claude Fable," that's expected; treat it
as a specific agentic-coding-oriented model name. Everything below applies to any target AI capable
of holding a multi-turn conversation about its own uncertainty, not just Fable.

Note on origin: the known/unknown vocabulary itself predates this article. It echoes the widely
cited 2002 framing from former U.S. Secretary of Defense Donald Rumsfeld (known knowns, known
unknowns, unknown unknowns), and a four-part version has circulated in later commentary too. I'm
not certain of the precise origin of that fourth category, so treat this article's contribution as
applying the lens to AI-assisted building, not as coining the taxonomy itself.

---

## Core Concept: Map vs. Territory

Every prompt is a map. It's a compressed representation of a task, written before the AI has
touched the actual problem. The territory is where the work actually happens: the real codebase,
the real dataset, the real reader, the real set of edge cases. A map is never the territory. It's
always missing something, and the gap between the two is what this file calls an unknown.

When an AI hits a gap the map didn't cover, it has to guess. Guesses compound. On a long, multi-step
build (a financial model, a hackathon prototype, a long-form novel outline) a wrong guess made early
gets built on top of, and unwinding it later costs far more than resolving it up front would have.
The claim underneath this whole file: time spent finding unknowns before and during a build is
cheaper than time spent fixing the output of a build that ran on unstated assumptions.

This changes what "good prompting" means for anything beyond a single short answer. A single
LinkedIn post has a small territory, get the tone and structure close and the job is done. A
financial model, a hackathon submission, or a research report has a large territory, and no single
prompt, however well engineered by Tiers 1-5 above, closes that gap in one shot. It closes through
iteration, not through writing a longer prompt.

---

## The Four Unknowns

Before reaching for a technique, diagnose which kind of gap is actually in front of you.

| Type | What it means | How it shows up if ignored |
|---|---|---|
| Known Knowns | What's explicitly stated in the prompt or spec | N/A, this is the prompt itself |
| Known Unknowns | Gaps you're aware you haven't resolved | The AI guesses instead of asking, and guesses wrong |
| Unknown Knowns | Standards you'd recognize instantly but never thought to write down | Output is "correct" but misses an unstated standard |
| Unknown Unknowns | Gaps you don't know exist, usually because the domain itself is unfamiliar | An entire category of consideration gets missed |

Known Unknowns are the easiest to close (ask a direct question). Unknown Knowns need a different
move (show the AI something and react to it, rather than describe it from a blank page). Unknown
Unknowns are the hardest and the most expensive to miss, since you can't ask a direct question about
something you don't know you should be asking about. Each technique below targets one of these four.

---

## When to Reach for This Layer

Pull this layer in when at least one of these is true:
- The deliverable is agentic or multi-step (a coding build, an automation, a financial model
  spanning many sheets, a research report built across several passes)
- The domain is one you don't have deep, current expertise in (a new sector for the newsletter, a
  new design surface, a new subject area for a hackathon)
- The build will span more than one AI session, so context needs to be deliberately carried forward
- Getting it wrong is expensive to unwind (a locked-in architecture, a submitted report, a shipped
  product)

Skip it for single-shot, low-ambiguity content, the same class of task Tier 5 already recommends
skipping production patterns for. A LinkedIn caption doesn't need a blind spot pass.

---

## The 8 Techniques

### Pre-Build

#### 1. Blind Spot Pass
**Closes:** Unknown Unknowns.
**What it is:** Before writing a full spec, ask the AI to name what a domain expert would flag that
you haven't thought of, given who you are and what you already know. The value is in surfacing
categories of question you wouldn't have known to ask, not in answering questions you'd already
have thought to raise yourself.
**When to use:** Any domain outside your established expertise.
**Adapted example:**
```
I'm building "Setu," an eligibility-matching tool for Indian government welfare schemes, aimed at
rural and semi-literate users. My background is finance and equity research, not civic-tech or
accessibility design. Do a blind spot pass: tell me what an experienced designer for low-literacy,
low-connectivity users would immediately flag that I likely haven't considered, before I lock in
the architecture. Ask me about my current assumptions first so your answer is targeted, not generic.
```
**When NOT to use:** Domains where you already have deep, current expertise. Running this on an
equity research report structure you've built a dozen times adds a step with nothing left to find.

#### 2. Brainstorm & Prototype
**Closes:** Unknown Knowns.
**What it is:** Ask for several genuinely different directions before committing, then react to
what you see rather than trying to describe a preference you haven't formed yet. Fastest when the
criteria are visual, structural, or otherwise easier to recognize than to specify in words.
**When to use:** Design decisions, structural decisions, anything where "I'll know it when I see
it" is an honest description of your own certainty.
**Adapted example:**
```
I don't yet know which interaction pattern is right for the Setu eligibility checker: a form, a
chat flow, or a card swipe. Mock all three as static HTML with placeholder data first. I'll tell
you which one to build for real once I've clicked through them.
```
**When NOT to use:** When the output format is already fixed and non-negotiable (a regulatory
filing template, a locked newsletter structure). There's nothing to react to if the shape can't
change.

#### 3. Interview
**Closes:** Known Unknowns.
**What it is:** Instead of trying to pre-answer every open question yourself, ask the AI to
interview you one question at a time, prioritized by how much each answer would actually change
the build. This surfaces gaps you already half-know exist but haven't resolved.
**When to use:** After a brainstorming pass, once you have a rough direction but before locking in
the full spec.
**Adapted example:**
```
Before I finalize the hackathon pitch, walk me through what's still open, one question at a time.
Weight your questions by how much each answer would reshape the submission (which data sources get
cited, which slides even need to exist) over anything that's just a wording preference.
```
**When NOT to use:** Time-boxed situations where a good-enough default beats a slower, more
accurate one, or when there are genuinely no open questions left.

#### 4. References
**Closes:** Unknown Knowns, specifically ones too complex or unfamiliar to describe in words.
**What it is:** Point the AI at an actual artifact rather than describing what you want from
scratch. Source material (an existing file, site, or document) carries more usable structure than a
written description or even a screenshot, since the AI can read the underlying structure directly.
**When to use:** Whenever something similar to what you want already exists somewhere you can
point to.
**Adapted example:**
```
For the Working Capital Cycle explainer, don't work from a description of what I want. Go read the
actual markup and interaction code behind dupont-analysis.vercel.app and carry over the same
structural approach, not just the visual style.
```
**When NOT to use:** When no comparable reference exists and forcing one in would anchor the output
to something irrelevant.

#### 5. Implementation Plan
**Closes:** A mix of Known Unknowns and Unknown Knowns, surfaced before they get expensive.
**What it is:** Before full execution, ask for a plan that leads with the decisions most likely to
change (data structure, assumptions, anything user-facing) and pushes mechanical, low-risk work to
the bottom. Review and correct the plan, not the finished output.
**When to use:** Any build substantial enough that redoing it after the fact would be costly.
**Adapted example:**
```
Before building out all 21 sheets, give me a plan that front-loads whatever is most likely to
change: growth rate assumptions, the peer set, working capital day counts. The formula wiring and
formatting can wait until the end since that part won't need a second look from me.
```
**When NOT to use:** Short, single-pass tasks where reviewing a plan takes longer than redoing the
task would.

### Mid-Build

#### 6. Implementation Notes
**Closes:** Unknown Unknowns that only surface once work is actually underway, no matter how good
the pre-build phase was.
**What it is:** Two parts, done together. First, once a plan is approved, start a clean session and
hand over only the finished plan and artifacts, not the whole messy planning conversation, so the
build runs against a clear spec instead of a cluttered context window. Second, have the AI keep a
running notes file logging any point where it had to deviate from the plan because of something the
plan didn't anticipate.
**When to use:** Any multi-step build, especially one spanning more than one sitting.
**Adapted example:**
```
Once building starts from the approved plan, maintain a running implementation-notes.md. When
something comes up that the plan didn't anticipate, default to the more conservative choice, add a
short entry under a "Deviations" heading explaining what happened and why, then keep working
instead of stopping to check in.
```
**When NOT to use:** Work short enough to finish in a single pass, where there's no context worth
preserving and no deviation worth logging.

### Post-Build

#### 7. Pitch/Explainer
**Closes:** The gap between what got built and what a reviewer needs to understand quickly enough
to approve it.
**What it is:** Package the plan, the prototype, and the implementation notes into a single
artifact built specifically to get buy-in fast, leading with the demonstration rather than the
process.
**When to use:** Anything needing sign-off from someone who wasn't in the build conversation, a
hackathon judge, a professor, a mentor, a reviewer.
**Adapted example:**
```
Take everything from this build (the prototype, the scheme-database research, the implementation
notes) and turn it into a single page a hackathon judge can absorb in under two minutes. Open with
what the tool actually does for a user, not with how it was built.
```
**When NOT to use:** Internal work with no external reviewer, where packaging is pure overhead.

#### 8. Quiz
**Closes:** The gap between what the AI built and what you can actually explain and defend
yourself.
**What it is:** After a substantial build, ask the AI to quiz you on what it did, not just
summarize it. The goal is confirming you remain the person who can explain the work under
questioning, not just the person who approved it.
**When to use:** Before presenting, submitting, or defending anything you didn't write line by
line yourself, especially in front of a professor or panel likely to probe details.
**Adapted example:**
```
Quiz me on what actually changed in this model: the key formulas, why each assumption was picked,
and what happens to the output if the working capital days number moves. I need to defend every
figure live when I present this, so I'm not moving on until I can answer cleanly.
```
**When NOT to use:** Trivial changes, or builds you wrote yourself and already understand fully.

---

## How This Comes Together: A Worked Walkthrough

Take the Setu hackathon submission as an example, since it sits well outside established expertise
(equity research) and inside a genuinely unfamiliar one (civic-tech UX for low-literacy users).
That mismatch is exactly the situation this layer is built for.

Start with a Blind Spot Pass, since the domain itself is new: ask what an accessibility-focused
civic-tech designer would flag that a finance background wouldn't surface on its own. That pass
will likely raise questions about literacy level, connectivity assumptions, and trust signals for a
first-time government-scheme user, none of which show up naturally from a finance lens.

Once those categories of question exist, move to Brainstorm & Prototype for the parts that are
genuinely a matter of taste, like the eligibility-checker interaction pattern, and react to mocked
options rather than trying to specify the "right" one from a blank page.

Use Interview to close whatever is left open and known-unresolved (which schemes to launch with,
how eligibility criteria get sourced and verified, what the judging rubric actually rewards), then
Implementation Plan to get a lightweight structure to review before the full build, front-loaded
with the decisions most likely to need a second look.

Once the plan is approved, move to a fresh session for the actual build and keep an Implementation
Notes log, since a hackathon build under time pressure will hit edge cases (a scheme with unusual
eligibility logic, a data field the source database doesn't actually provide) that no amount of
upfront planning fully anticipates.

Finish with a Pitch/Explainer packaging the prototype and the verified-data research for the
judges, and a Quiz pass before presenting, so every claim about how the eligibility logic works can
be defended live, under questioning, without hesitation.

---

## Integration With This Skill

This layer doesn't replace the 6-Component Spec Framework or Tiers 1-5 above. It runs before them
(deciding what should go into the spec) and sometimes gets engineered into them (an agentic build's
Constraints or Verification components can explicitly instruct the target AI to run one or more of
these 8 techniques against the end user, rather than silently guessing through its own unknowns).
See Phase 0 Step 6 and the Framework Toolkit's agentic/tool-use row in SKILL.md for where this
connects to the rest of the skill.
