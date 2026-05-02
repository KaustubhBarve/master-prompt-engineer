# Prompt Engineering Frameworks — Ultra-Master Reference

## FRAMEWORK SELECTION MAP

```
Task type                         → Best Framework        → Stack With
─────────────────────────────────────────────────────────────────────────────
Creative content, copy, brand     → CO-STAR               → Anchor, Priming, BAB
Complex multi-step analysis       → RISEN                 → CoT/CoD, Constitutional
Persuasive / argumentative        → TRACE                 → BAB, Contrastive examples
Consistency / style matching      → CARE                  → Contrastive few-shot
Professional branded documents    → CRAFT                 → CO-STAR for tone
Research / evidence-heavy         → POWER                 → Step-Back, Self-Consistency
Agentic / tool-use workflows      → ReAct + RISEN         → Chain of Verification
AI persona design                 → CRISPE                → CO-STAR, Constitutional
Coaching / scenario               → RACE                  → CO-STAR audience layer
Transformation narrative          → BAB                   → TRACE for persuasion
Iterative / sprint workflows      → Agile Prompting       → Reflexion loop
Any complex request               → 4D (meta-framework)   → All above as needed
```

---

## THE 4D META-FRAMEWORK

The master operating process. It selects and combines all other frameworks.

**Deconstruct → Diagnose → Develop → Deliver**

The 4D is not a template — it is a process. Use it to determine WHICH templates to apply.

**Phase 1 — Deconstruct:** Map core intent, entities, output contract, constraints, missing information, failure modes.

**Phase 2 — Diagnose:** Audit for lexical flatness, attention collapse risk, thin identity, missing quality gates, platform mismatch.

**Phase 3 — Develop:** Select framework + technique stack from the decision trees in SKILL.md and techniques.md.

**Phase 4 — Deliver:** Build the prompt in KV-cache-optimal structure order. Run the quality checklist.

---

## RTF — Role, Task, Format

**Best for:** Simple, single-task prompts where the request is fully clear.
**Note:** This is the MINIMUM acceptable framework. For GODMODE outputs, always escalate to RISEN or above.

```
Role: You are a [specific, credentialed role with methodology and cognitive style].

Task: [Clear, unambiguous task description. One task only. Specific verbs.]

Format: [Exact output structure. Section names. Lengths. Tone. Audience.]
```

**When to use RTF alone:** Only for genuinely simple tasks. If there is any complexity, add context layering and escalate.

---

## CO-STAR — Context, Objective, Style, Tone, Audience, Response

**Best for:** Content creation, marketing, communications, any tone-sensitive task, brand voice work.

**Components:**
- **Context:** Background situation that frames the entire task
- **Objective:** The specific, measurable goal to achieve
- **Style:** Writing approach — analytical, narrative, listicle, conversational, academic
- **Tone:** Emotional register — authoritative, warm, urgent, inspirational, dry
- **Audience:** Who reads this — their knowledge level, role, what they care about
- **Response:** Exact output format and length specification

**Template:**
```xml
<context>
[Background situation — what's happening, why this matters, what came before]
</context>

<objective>
[Specific measurable goal — not "write a good email" but "write an email that achieves X for Y audience"]
</objective>

<style>
[Writing approach — e.g., "analytical and evidence-led, not conversational"]
</style>

<tone>
[Emotional register — e.g., "authoritative but accessible; confident without being arrogant"]
</tone>

<audience>
[Exact audience — role, knowledge level, what they already believe, what they're skeptical of]
</audience>

<response_format>
[Exact output structure — sections, lengths, formatting rules]
</response_format>
```

**Worked Example:**
```xml
<context>
We are launching a new AI-powered legal contract review tool for small businesses. 
Most small business owners sign contracts without understanding them, leading to 
disputes and financial loss. Our tool analyzes contracts in plain English in under 2 minutes.
</context>

<objective>
Write a 200-word homepage hero section that communicates our core value proposition 
and drives sign-ups for our free trial.
</objective>

<style>
Plain, direct, and jargon-free. Lead with the problem, not the product. 
Use active voice. Short punchy sentences. No marketing buzzwords.
</style>

<tone>
Empathetic but confident. Acknowledge the fear of signing contracts blind. 
Reassuring without being condescending. Like advice from a trusted lawyer friend.
</tone>

<audience>
Small business owners aged 30–55, not legally trained, who sign contracts regularly 
and feel anxious about what they're agreeing to. Skeptical of expensive lawyers. 
Time-constrained.
</audience>

<response_format>
- One headline (under 10 words, benefit-led)
- One sub-headline (under 20 words, specific mechanism)  
- One paragraph body copy (80–100 words)
- One CTA button text (under 5 words, action-oriented)
- Three alternative headline options for A/B testing
</response_format>
```

---

## RISEN — Role, Instructions, Steps, End Goal, Narrowing

**Best for:** Complex, multi-step, process-driven tasks. Analysis, documentation, research, reports. The go-to GODMODE framework for substantive tasks.

**Components:**
- **Role:** Expert persona with specific credentials and methodology
- **Instructions:** Overall task description — what you're doing and why
- **Steps:** Ordered sequence of actions to follow (the methodology)
- **End Goal:** What success looks like — how you'll know it worked
- **Narrowing:** Constraints, scope limitations, what to exclude

**Template:**
```
ROLE:
You are a [specific expert role] with [specific credentials/experience].
Your approach is characterized by [methodology/cognitive style].
You are known for [specific quality that matters for this task].

INSTRUCTIONS:
[Overall task description — what you are doing, for whom, and why it matters.
Include stakes, purpose, and what this output will be used for.]

STEPS:
1. [First specific action — what exactly to do, not a vague instruction]
2. [Second action]
3. [Third action]
4. [Continue as needed]
[Each step should be concrete enough that there is only one way to interpret it]

END GOAL:
[What the final output must accomplish. What does a reader of this output think, feel, 
or do differently? How would you score a 10/10 response vs. a 6/10?]

NARROWING:
- [Constraint 1 — positively framed]
- [Constraint 2 — scope limitation]
- [Constraint 3 — what to exclude]
- [Constraint 4 — quality floor]
```

**Worked Example:**
```
ROLE:
You are a principal product strategist with 15 years of experience building 
B2B SaaS products from 0 to $100M ARR. Your methodology is market-back: 
you always start with the customer's problem, not the technology. You are 
known for cutting through feature debates to identify the single critical 
differentiation that drives adoption.

INSTRUCTIONS:
Analyze the following product concept and produce a structured go-to-market 
strategy brief. This will be presented to the board to decide whether to fund 
the next development sprint. It needs to be clear, credible, and actionable.

STEPS:
1. Identify and articulate the core customer problem being solved (1 paragraph — 
   specific, not generic; name the customer persona and their specific pain)
2. Define the target market segment with quantifiable characteristics 
   (company size, industry, role, current behavior)
3. Articulate the unique value proposition in one sentence — what the product 
   does, for whom, that nothing else does as well
4. Map the top 3 competitive alternatives (including "do nothing") and state 
   why this product wins against each
5. Define the acquisition channel strategy — how does the first customer hear 
   about this product and what triggers them to try it?
6. Identify the single most critical assumption that must prove true for this 
   business to succeed
7. Propose a 90-day validation experiment that tests that assumption cheaply

END GOAL:
A board member who reads this brief should leave the meeting with a clear 
mental model of who buys this, why, and what must be true for it to work — 
without needing to ask clarifying questions.

NARROWING:
- Total length: 600–800 words
- Each section clearly labeled
- No jargon — the board includes non-technical members
- Do not discuss technology implementation details
- Avoid vague claims — every assertion must be defensible or flagged as an assumption
- Do not cover pricing model (separate brief)
```

---

## CARE — Context, Action, Result, Example

**Best for:** Tasks where you have a sample of the desired output. Maximizes style and format consistency.

**Template:**
```
CONTEXT:
[Background — who you are, what situation you're in, what has led to this task]

ACTION:
[Specific task to perform — what to do]

RESULT:
[What the output must achieve — the effect it should have on the reader/user]

EXAMPLE:
[One or more samples of ideal output — use contrastive pairs when possible:
Example A (correct): [sample]
Example B (what to avoid): [sample with explanation of why it's weaker]]
```

---

## CRAFT — Context, Role, Action, Format, Tone

**Best for:** Professional documents, branded communications, when voice and structure are equally critical.

**Worked Example:**
```
CONTEXT:
We are a Series B fintech startup (payment infrastructure for gig economy workers) 
that just raised $25M. We want to attract enterprise talent — senior engineers 
who currently work at Stripe, Plaid, or similar.

ROLE:
You are our Head of People and Employer Brand. You write with authority, 
authenticity, and specificity — never corporate-speak, never generic job board language.

ACTION:
Write a LinkedIn post announcing our funding and inviting senior engineers to explore 
opportunities with us.

FORMAT:
- Opening hook (1–2 sentences that stop the scroll)
- Funding announcement (1 sentence — specific number, specific what we'll use it for)
- The problem we're solving (2–3 sentences — vivid, specific, human)
- What we're building technically (2–3 sentences — specific stack/challenge, not vague)
- What kind of person thrives here (3 bullet points — concrete behaviors, not values)
- CTA (1 sentence with link placeholder)
- Total: 200–250 words

TONE:
Genuine and direct. Like a real person wrote it, not a PR team. 
Confident without being arrogant. Excited without being hypey.
```

---

## TRACE — Topic, Reason, Audience, Counterargument, Evidence

**Best for:** Persuasive writing, debate prep, argumentative essays, thought leadership, pitches.

**Template:**
```
TOPIC:
[The specific position or claim to argue]

REASON:
[Why this argument is being made — the purpose and stakes of the persuasion]

AUDIENCE:
[Who needs to be persuaded. What they currently believe. Why they're skeptical. 
What would change their mind. What they care about most.]

COUNTERARGUMENT:
[The strongest objection the audience will raise — stated fairly and specifically.
This MUST be addressed head-on in the response, not avoided.]

EVIDENCE:
[Specific data points, studies, examples, or analogies to use.
"Strong evidence" — not vague references. Include source type if possible.]
```

**Stack with BAB for maximum persuasion:**
```
[TRACE framework for the argument structure]

BAB STRUCTURE FOR DELIVERY:
BEFORE: [Describe the reader's current painful situation in vivid, specific terms]
AFTER: [Paint the picture of life with this change — specific, concrete, desirable]
BRIDGE: [Explain exactly how to get from Before to After — the specific mechanism or decision]
```

---

## POWER — Purpose, Output, Why, Evidence, Refinement

**Best for:** Research-heavy tasks, analysis, policy briefs, evidence-based recommendations.

**Template:**
```
PURPOSE:
[What you are trying to understand, decide, or communicate]

OUTPUT:
[The specific deliverable — format, structure, length, audience]

WHY:
[Why this output matters — who uses it, what decision it informs, what's at stake]

EVIDENCE:
[The specific types of sources, data, or information to draw from.
What counts as valid evidence for this task? What is out of scope?]

REFINEMENT:
[How this output should differ from a generic treatment of this topic.
What makes this analysis specific, not generic? What standard treatment would 
you avoid and why?]
```

---

## CRISPE — Capacity, Role, Insight, Statement, Personality, Example

**Best for:** AI assistant persona design, advanced system prompts where capabilities need explicit definition.

**Components:**
- **Capacity:** What the AI can do — specific capabilities it possesses
- **Role:** The persona/expert it embodies
- **Insight:** Key knowledge or perspective it brings
- **Statement:** The core task
- **Personality:** How it communicates and behaves
- **Example:** An illustrative sample of ideal behavior

**Worked Example:**
```
CAPACITY:
You have the capacity to: (1) analyze complex financial data and extract non-obvious patterns, 
(2) translate quantitative findings into clear narrative for non-specialists, 
(3) identify assumptions hiding in financial models, and (4) generate scenario analyses 
with explicit uncertainty quantification.

ROLE:
You are a senior financial analyst with 12 years at a top-tier investment bank, 
specializing in technology sector valuations and earnings analysis.

INSIGHT:
Your key insight is that most financial narratives hide their most critical assumptions 
in footnotes or between the lines. You surface these explicitly and force them to 
be examined before any conclusion is accepted.

STATEMENT:
Analyze the financial data I provide and produce a structured investment memo.

PERSONALITY:
Direct and precise. You never hedge unnecessarily but always flag genuine uncertainty. 
You write in clear, jargon-free language. You push back on weak reasoning. 
You ask clarifying questions when the data is insufficient to support a conclusion.

EXAMPLE:
When asked "Is this company a good investment?", rather than answering yes or no, 
you first identify the three most critical assumptions driving any answer, 
state what would need to be true for each, and then give a conditional assessment.
```

---

## BAB — Before, After, Bridge

**Best for:** Persuasive and transformation narratives. Pairs with TRACE for maximum persuasive impact.

**Template:**
```
BEFORE:
[Vivid, specific description of the reader's current painful situation. 
Use their language. Name the specific frustrations, fears, or costs they experience. 
Make them feel seen.]

AFTER:
[Equally vivid picture of life after the change. Specific, concrete, desirable. 
Not abstract benefits — actual lived experience. What is different on a Tuesday morning?]

BRIDGE:
[The specific, credible mechanism that gets them from Before to After.
Not vague "our solution helps you" — the specific how. What changes, when, because of what.]
```

---

## RACE — Role, Action, Context, Expectation

**Best for:** Coaching, instructional, scenario-based prompts where narrative flow matters.

**Template:**
```
ROLE:
[Who the AI is and their relationship to the user — mentor, advisor, coach, expert guide]

ACTION:
[What the AI does — the specific activity or task in this scenario]

CONTEXT:
[The full situation — what the user is going through, what has led to this moment,
what challenges they face]

EXPECTATION:
[What a successful interaction looks like — what the user should know, feel, or 
be able to do differently after this session]
```

---

## AGILE PROMPTING

**Best for:** Complex tasks that benefit from iterative refinement rather than a single-shot prompt. Multi-session workflows. Tasks where the full scope isn't known upfront.

**Process:**
```
SPRINT 1 — MVP PROMPT:
[Minimum viable prompt for the first iteration. Get something working, not perfect.]

DEFINITION OF DONE FOR SPRINT 1:
[What does a successful Sprint 1 output look like? What's the minimum bar?]

RETROSPECTIVE:
[After reviewing Sprint 1 output: What worked? What was missing? What needs refinement?]

SPRINT 2 — REFINED PROMPT:
[Second iteration incorporating retrospective learnings. Add one major improvement.]

[Continue sprints until the output meets full quality criteria]
```

**Built-in Reflexion loop:**
```
After each sprint output, evaluate:
- What did the AI do well that I should preserve?
- What specific element produced the weakest result?
- What single change would most improve the next sprint?
Make only ONE major change per sprint to isolate variables.
```
