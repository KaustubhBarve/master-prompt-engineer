# Ultra-Master Technique Library - Deep Reference

## TECHNIQUE DECISION TREE (START HERE)

```
Is the target model an inference-optimized reasoning model?
(o1, o3, Claude Extended Thinking, Gemini Thinking Mode)
├── YES → Zero-shot only. No CoT. Concise constraints. Pure structure. XML/delimiters.
└── NO → Continue below.

Is token economy / latency critical?
├── YES → Use Chain-of-Draft (CoD). 70-90% token reduction, 48-76% latency reduction.
└── NO → Use verbose CoT for complex reasoning tasks.

Is this a multi-step task with high compound-error risk?
├── YES → Add Contrastive Sequential Processing.
└── NO → Continue.

Is this a highly specific or novel problem?
├── YES → Add Step-Back Prompting (abstract first, solve second).
└── NO → Continue.

Is this long-form document generation?
├── YES → Use Skeleton-of-Thought (skeleton → parallel section fill).
└── NO → Continue.

Does this require tool use or iterative action-observation?
├── YES → Use ReAct + Chain of Verification.
└── NO → Continue.

Is quality/safety/accuracy non-negotiable?
├── YES → Add Constitutional Self-Critique loop.
└── NO → Standard output spec + quality gate is sufficient.
```

---

## TIER 1: FOUNDATIONAL TECHNIQUES

### 1. Chain-of-Thought (CoT) - Standard Verbose

**Mechanistic basis:** Forces the model to generate intermediate reasoning tokens before the final answer token. The intermediate tokens occupy context positions that causally influence final answer probability. Bypasses the model's tendency to "shortcut" to a plausible-sounding answer without working through the logic.

**Performance data:** Commonly cited range is +15-40% accuracy on complex math and logic reasoning tasks on standard (non-reasoning) models, benchmark-dependent. Source: Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (arXiv:2201.11903, 2022); zero-shot variant ("Let's think step by step") from Kojima et al. (arXiv:2205.11916, 2022). A specific "19-point MMLU-Pro" figure appeared in an earlier version of this file; it could not be traced to a verifiable source during a 2026-07-16 audit and has been removed rather than repeated unverified.

**Implementation:**
```
Before giving your final answer, work through your reasoning step by step.
Show each logical step clearly. Only provide your final answer after
completing the full reasoning chain.
```

**Claude-specific (allows native reasoning):**
```
Think through this carefully within <thinking></thinking> tags,
then provide your answer outside those tags.
```

**Self-Consistency CoT (high-stakes accuracy):**
```
Solve this problem using three independent reasoning approaches.
Show each approach fully. If they reach the same answer, that is your answer.
If they diverge, identify the error in the inconsistent approach and correct it.
```
Source: Wang et al., "Self-Consistency Improves Chain of Thought Reasoning in Language Models"
(arXiv:2203.11171, 2022) - sample multiple reasoning paths, take the majority answer.

**Critical contraindication:** NEVER use on o1, o3, Claude Extended Thinking, or Gemini Thinking Mode. These models execute internal CoT via RL training. External CoT instructions cause processing conflicts, inflate tokens, and degrade output quality.

---

### 2. Chain-of-Draft (CoD) - Token-Compressed Reasoning

**Mechanistic basis:** Standard CoT forces the model to "think in full sentences" - an anthropomorphic fallacy that degrades efficiency. CoD instructs the model to generate concise, high-signal intermediate steps (maximum 5 words each) using symbolic, mathematical, or abbreviated notation. This reduces autoregressive generation distance between problem and answer while preserving the logic scaffold.

**Performance data:** Source: Xu, Xie, Zhao & He, "Chain of Draft: Thinking Faster by Writing Less" (arXiv:2502.18600, Feb 2025). The paper's own headline result: CoD matches or surpasses CoT accuracy while using as little as ~7.6% of the tokens on the tasks tested (roughly a 92% reduction in the best case). Independent third-party benchmarking across GPT-4o and Claude 3.5 Sonnet found a wider practical range depending on task - commonly 68-92% token reduction and 48-76% latency reduction - which is the range this skill uses as a realistic production estimate rather than the paper's best-case headline number. No fine-tuning required in either case.

**Example transformation:**
- Standard CoT: "Jason started with 20 lollipops. After giving some away, he has 12 left. Subtracting 12 from 20 gives 8. Therefore he gave away 8 lollipops."
- CoD: `20 - x = 12 → x = 8`

**Implementation:**
```
When working through this problem, use concise draft notation only.
Each intermediate step must be 5 words or fewer - use symbolic, mathematical,
or abbreviated notation. Do not explain steps in full sentences.
Final answer should be clear and complete.
```

**When to use:** Production deployments, cost-sensitive environments, API integrations, any task where speed matters and symbolic reasoning applies.

---

### 3. Few-Shot Learning

**Mechanistic basis:** Example input-output pairs in context shift the probability distribution toward the demonstrated pattern. The model pattern-matches rather than following abstract instructions, which is often more reliable for format and style consistency.

**Standard few-shot:**
```
Here are examples of the output format I require:

Example 1:
Input: [input A]
Output: [ideal output A]

Example 2:
Input: [input B]
Output: [ideal output B]

Now apply exactly this pattern to:
Input: [actual task]
```

**Contrastive few-shot (Ultra-Master upgrade):**
```
Here is a correct example and a flawed example. Learn from the contrast.

CORRECT EXAMPLE:
Input: [input]
Output: [ideal output - explain why this is correct]

FLAWED EXAMPLE:
Input: [same input]
Output: [bad output - explain exactly what is wrong and why]

Apply the correct pattern to:
Input: [actual task]
```

**Tips:** 2–3 examples suffice for most tasks. More than 5 adds cost without proportional gains. Diverse examples prevent overfitting. Always show edge cases if they matter.

---

### 4. Role Prompting

**Mechanistic basis:** Role tokens shift the model's generative distribution toward domain-specific training data. A well-specified role narrows the probability space and pulls token selection from higher-quality, domain-accurate regions of the pre-training distribution.

**Weak vs. Ultra-Master role definition:**
- Weak: "You are an expert."
- Ultra-Master: "You are a principal data scientist with 12 years of experience building production ML systems at scale. You specialize in feature engineering, model interpretability, and translating technical complexity into business-accessible language. You approach every problem by first establishing the business question, then working backward to the data requirements, and you are known for catching assumptions that others miss."

**Persona blending (for hybrid expertise):**
```
You operate as a synthesis of two disciplines:
- The analytical rigor of a [role A]: [specific quality]
- The communication clarity of a [role B]: [specific quality]
When these perspectives conflict, [resolution rule].
```

**Motivational priming (add to role block):**
Research shows compound emotional stimuli shift model output toward higher-effort training distributions. High-stakes framing receives disproportionately large attention weights, anchoring instruction representation.
```
This task requires your absolute highest standard of work.
The analysis you produce will directly inform a critical decision.
Approach this with the full depth of your expertise.
```

---

## TIER 2: ADVANCED REASONING ARCHITECTURES

### 5. Contrastive Sequential Processing

**Mechanistic basis:** Standard few-shot shows the model what to do. Contrastive processing shows the model what to do AND what the specific failure looks like. The attention mechanism computes semantic distance between correct and flawed formulations, mapping the logical boundary. This dramatically reduces cascade errors in multi-step deduction.

**Advanced implementation with triplet extraction:**
```
I will show you a problem, a correct reasoning path, and a flawed reasoning path.

PROBLEM: [problem]

CORRECT PATHWAY:
Step 1: [correct step]
Step 2: [correct step]
Step 3: [correct conclusion]
Why this is correct: [explanation]

FLAWED PATHWAY:
Step 1: [flawed step - identical to correct]
Step 2: [where the error enters]
Step 3: [wrong conclusion]
Error identified: [Entity A] was incorrectly related to [Entity B] when the correct relation is [Entity C to Entity D]

Now apply the correct reasoning pattern to:
[actual problem]

Trigger phrase: "Let me identify the correct and incorrect pathways before reaching my conclusion."
```

---

### 6. Step-Back Prompting

**Mechanistic basis:** Highly specific or complex prompts cause attention collapse - the model fixates on idiosyncratic surface details and accesses niche, potentially low-quality training data. Step-back forces the model to first generate a higher-level abstraction (the underlying physics, principles, or patterns), drawing attention to broad, structurally sound training representations before engaging the specifics.

**Implementation:**
```
Before attempting to solve this problem, first step back and identify:
1. What domain or field does this problem belong to?
2. What are the fundamental principles that govern this domain?
3. What general class of problem is this an instance of?

Once you have established the underlying principles, apply them to solve
the specific problem presented.
```

**Paired with analogical prompting:**
```
Before solving this, generate an analogous problem from a related domain
that shares the same underlying structure. Solve the analogy first,
then apply those principles to the original problem.
```

---

### 7. Tree-of-Thought (ToT)

**Mechanistic basis:** Standard CoT is a single linear chain - if any step is wrong, the entire chain fails. ToT generates multiple reasoning branches at each decision point, evaluates them, prunes dead ends, and continues the most promising path. This simulates Monte Carlo Tree Search in the model's latent space.

**Performance data:** Source: Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (arXiv:2305.10601, NeurIPS 2023). On the paper's Game of 24 task, GPT-4 with standard CoT solved 4% of problems; GPT-4 with ToT solved 74%. This is a single-benchmark result, not a general-purpose accuracy multiplier - treat it as evidence the technique class works, not as a transferable percentage for other tasks.

**Implementation (single-prompt version):**
```
For this problem, generate THREE different initial approaches. Label them Approach A, B, and C.

For each approach:
- Briefly outline the reasoning path (2-3 sentences)
- Identify the strongest and weakest aspects
- Rate its likelihood of success (high/medium/low)

Select the highest-rated approach. Continue developing it through all required steps.
At each major decision point, pause and evaluate: "Is this path still optimal? Should I backtrack?"

If you backtrack, explain why and which alternative you are now pursuing.
```

**Use case:** Complex planning, multi-step logical puzzles, architectural decisions.

---

### 8. Graph-of-Thought (GoT)

**Mechanistic basis:** ToT explores paths in a tree. GoT allows thought nodes to form a graph - nodes can merge, cross-reference, and integrate. This captures synergies between independent reasoning lines and allows the model to distill insights from multiple sub-problem solutions.

**Performance data:** Source: Besta et al., "Graph of Thoughts: Solving Elaborate Problems with Large Language Models" (arXiv:2308.09687, AAAI 2024). The paper's own reported result is on a **sorting** task, not multi-document synthesis (corrected 2026-07-16 - an earlier version of this file misattributed the task domain): GoT improved sorting quality by 62% over ToT while reducing cost by more than 31%, with the advantage growing as list length increased. Treat "multi-document synthesis" below as an applied analogy this skill draws from that result, not as something the original paper tested.

**Implementation:**
```
Decompose this problem into [N] independent sub-problems:
Sub-problem 1: [X]
Sub-problem 2: [Y]
Sub-problem 3: [Z]

Solve each sub-problem independently using chain-of-thought reasoning.

Once all sub-problems are solved:
1. Identify where the solutions reinforce each other
2. Identify where they conflict and resolve the conflict
3. Merge the solutions into a unified, integrated answer
4. Identify any emergent insights that only appear when the solutions are combined
```

---

### 9. Skeleton-of-Thought (SoT)

**Mechanistic basis:** Long-form generation suffers from attention drift - the model gradually loses alignment with the macro-structure and drifts off-topic. SoT separates planning (skeleton generation) from execution (section fill), eliminating mid-document drift. Parallel section generation also reduces latency. Source: Ning et al., "Skeleton-of-Thought: Prompting LLMs for Efficient Parallel Generation" (arXiv:2307.15337, 2023) - the paper reports up to ~2.4x average speedup across the models tested, with some models reaching higher; this skill uses a conservative "~2x" as the practical planning figure.

**Implementation:**
```
Step 1 - Generate the skeleton:
Before writing any content, produce a complete structural outline of the
[document type] with:
- All section headers
- 1-sentence description of what each section covers
- Target word count for each section
- Key points each section must address

Step 2 - Confirm the skeleton:
Review the skeleton. Does it cover all required aspects? Adjust if needed.

Step 3 - Execute section by section:
Write each section fully, treating it as an independent unit.
At the start of each section, re-read the skeleton to maintain alignment.
```

---

### 10. ReAct (Reason + Act)

**Mechanistic basis:** Interleaves Thought tokens (reasoning about the current state) with Action tokens (tool calls, searches, computations). The model plans, acts, observes the result, and updates its reasoning - transforming a passive text generator into an active agent. Source: Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (arXiv:2210.03629, 2022).

**Implementation:**
```
You will solve this by interleaving Thought and Action steps.

Format:
Thought: [Your reasoning about what to do next and why]
Action: [The specific action to take - search for X, calculate Y, retrieve Z]
Observation: [What the action returned - I will provide this]
Thought: [Your updated reasoning based on the observation]
...continue until...
Final Answer: [Your complete response]

Begin with your first Thought.
```

**Chain of Verification (add-on for adversarial accuracy):**
```
Before integrating any observation into your main reasoning, spawn a
verification sub-query: "Is this observation consistent with what I already know?
What would be wrong with this if it were a hallucinated result?"
Only incorporate observations that pass this verification check.
```

---

## TIER 3: QUALITY ENGINEERING

### 11. Constitutional Self-Critique Loop

**Mechanistic basis:** The model generates an answer, then critiques it against an explicit set of principles, then rewrites to fix violations. This creates a recursive quality gate. Source: Bai et al., "Constitutional AI: Harmlessness from AI Feedback" (arXiv:2212.08073, Anthropic, 2022) - the paper trains this generate-critique-revise loop into the model itself via RLAIF; using the same loop as an in-context prompting pattern (as below) is this skill's application of that idea, not a result the paper measured directly. **Correction (2026-07-16):** an earlier version of this file attributed "+15% on TruthfulQA" to "the Reflexion paper." This was a misattribution on two counts - the Reflexion paper's own headline result is 91% pass@1 on HumanEval versus GPT-4's 80% (Shinn et al., arXiv:2303.11366, 2023; see item 12 below), not a TruthfulQA figure, and it is a different paper from Constitutional AI. The fabricated number has been removed rather than corrected in place, since no verifiable source for a "+15% TruthfulQA" figure could be found for either paper.

**Implementation:**
```
STEP 1 - Generate:
Produce your initial response to the task.

STEP 2 - Critique:
Review your response against these principles:
Principle 1: [accuracy standard]
Principle 2: [completeness standard]
Principle 3: [format standard]
Principle 4: [constraint compliance]

For each principle, state: Met / Partially Met / Not Met
For each violation: Identify the specific problem.

STEP 3 - Rewrite:
Produce a revised response that addresses every identified violation.
Do not present the initial draft - only the final revised version.
```

---

### 12. Reflexion / Iterative Self-Refinement

**Mechanistic basis:** Source: Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" (arXiv:2303.11366, NeurIPS 2023). The agent verbally reflects on task feedback and stores that reflection in an episodic memory buffer rather than updating model weights, then uses it on the next attempt. The paper's headline result is 91% pass@1 on HumanEval, versus 80% for the GPT-4 baseline it compares against - a coding benchmark, not a general-purpose "self-refinement always helps" guarantee. Self-reflection's effect is also task-dependent: at least one later replication found it helped on TruthfulQA-style factual recall but hurt on HotpotQA-style multi-hop reasoning, so this technique is best applied where the task rewards revision rather than assumed universally beneficial.

**Implementation:**
```
After producing your response, conduct a self-evaluation:

Quality check:
- Accuracy: Does every factual claim hold up to scrutiny?
- Completeness: Does this fully address the request?
- Constraints: Have all format and content requirements been met?
- Tone: Does the voice and register match the specification?

Score each dimension 1–5. If any score is below 4, rewrite that section.
Present only the final, revised version.
```

---

### 13. Output Contract Specification

**Implementation template:**
```
Your output must conform exactly to this contract:

STRUCTURE:
Section 1 - [Name]: [Description of content] | Length: [X words/bullets]
Section 2 - [Name]: [Description of content] | Length: [X words/bullets]
Section 3 - [Name]: [Description of content] | Length: [X words/bullets]

FORMATTING:
- Use [markdown/plain text/XML]
- Headings: [H2/H3/bold]
- Lists: [bullet/numbered/prose]
- Total length: [X–Y words]

TONE: [Register description]
PERSPECTIVE: [First/second/third person]
AUDIENCE: [Who reads this and what do they already know]

SUCCESS CRITERIA (self-evaluate before delivering):
A 5/5 response: [describe ideal output]
A 3/5 response: [describe acceptable but weak output]
A 1/5 response: [describe unacceptable output]

If your assessment is below 4/5 on any criterion, revise before delivering.
```

---

## TIER 4: META-LEVEL AND EMERGING TECHNIQUES

### 14. Analogical Prompting (Auto Few-Shot)

**Mechanistic basis:** Instead of the prompt engineer manually selecting examples, the model retrieves analogous problems from its own weights. Self-retrieved exemplars are structurally optimal - perfectly matched to the model's internal representations.

**Implementation:**
```
Before tackling this problem, perform an analogical retrieval:
1. Identify a structurally similar problem from a related domain that you have seen before
2. Briefly describe that analogous problem and its solution
3. Explain the structural parallels between the analogy and the current problem
4. Apply the analogous solution structure to the current problem

Then solve the problem using the framework your analogy revealed.
```

---

### 15. Program-of-Thought (PoT)

**Performance data:** Source: Gao et al., "PAL: Program-aided Language Models" (arXiv:2211.10435, 2022), the paper this technique is drawn from (published under the name PAL). On GSM8K, PAL using Codex reached 72.0% top-1 accuracy, a 15-point absolute improvement over PaLM-540B using CoT (65.6%); on a harder variant with larger numbers (GSM-Hard), the gap widened to as much as ~40 points. An earlier version of this file cited a flat "~12% accuracy gain," which understated and oversimplified a result that is closer to 15 points on the paper's primary benchmark and considerably higher on arithmetic-heavy variants - corrected 2026-07-16.

**When to use:** Any task involving non-trivial arithmetic, data manipulation, statistical computation, or any domain where exact calculation matters.

**Implementation:**
```
For the computational portions of this task, write executable Python code
rather than performing calculations in natural language.

Format:
```python
# [Clear comment explaining what this computes]
[code]
result = [final computed value]
print(result)
```

Then use the computed result in your natural language response.
This ensures computational accuracy that natural language arithmetic cannot guarantee.
```

---

### 16. KV Cache Architecture (Structural Technique)

**This is a structural technique, not a content technique.** It governs WHERE you place different content types in a long prompt.

**Rule:**
- TOP of prompt: All background, context, reference documents, persona definition
- BOTTOM of prompt: Active task instructions, critical constraints, output specification, activation phrase

**Why:** The Key-Value cache stores token representations. Prefix tokens (top) are compressed and cached efficiently. The observation window (bottom) receives maximum fresh attention. The middle degrades - critical instructions placed in the middle of long prompts reliably receive less attention weight.

**For critical constraints:** State them at BOTH the top and the bottom. Repetition compensates for middle-degradation.

---

### 17. Lexical Density Optimization

**Mechanistic basis:** High-frequency vague words ("good", "helpful", "better") spread the probability distribution across many possible next tokens, producing generic averaged outputs. High-specificity domain vocabulary narrows the distribution, forcing token selection from targeted, high-quality training regions. **Correction (2026-07-16):** an earlier version of this file cited "+6.7% median performance improvement" for paraphrase perturbations that increase specificity. This figure could not be traced to a verifiable paper during a targeted search and has been removed. The mechanistic argument above is well-supported by the broader prompt-sensitivity literature (LLM outputs are demonstrably sensitive to phrasing specificity), but this skill no longer attaches an unsourced number to it - if you need a defensible statistic for a specific claim, verify it against the current literature rather than reusing this one.

**Implementation:** Conduct a lexical audit of your prompt. Replace every vague word with a specific one.

| Vague | Specific |
|-------|----------|
| "Write a good analysis" | "Write a systematic causal analysis identifying root mechanisms" |
| "Be helpful" | "Provide actionable recommendations with implementation steps" |
| "Explain clearly" | "Explain for an audience of non-specialist executives who make decisions based on summaries" |
| "Cover the main points" | "Cover the 5 most consequential factors, ranked by impact magnitude" |

---

### 18. Polyglot / Multilingual Prompting

**Mechanistic basis:** Foundation model pre-training data is overwhelmingly English. Complex reasoning, code architecture, and structural frameworks are entangled with English syntax in the latent space. Translating a complex English prompt directly degrades performance by forcing reasoning through lower-parameter linguistic representations.

**Ultra-Master rule:** Keep ALL structural, operational, and reasoning directives in English. Only target the data inputs and output format to the secondary language.

**Implementation:**
```
[English: Full role definition, methodology, constraints, quality gate]

The user's input will be in [Language]. Process it using the above methodology.
Your final output must be in [Language], following the specified format.
All internal reasoning can remain in English if that produces higher quality.
```

---

### 19. The Meta-Prompting Family

**Corrected 2026-08-08:** earlier versions of this file cited "Suzgun & Kalai 2023, arXiv:2311.11482"
for meta-prompting. Direct verification found this conflates two unrelated papers that happen to
share almost the same title. arXiv:2311.11482 ("Meta Prompting for AI Systems") is by Zhang, Yuan &
Yao - not Suzgun & Kalai - and gives a categorical formalization of meta-prompting. Suzgun & Kalai's
actual paper is arXiv:2401.12954 ("Meta-Prompting: Enhancing Language Models with Task-Agnostic
Scaffolding"). Both are real, useful, and distinct - they're now split into 19a and 19b below, with
19c and 19d covering the lineage this family descends from and its applied, product-level instance.

#### 19a. Meta-Prompting (Conductor-Expert Scaffolding)

**Mechanistic basis:** A single LM is guided by a high-level instruction to act as a **conductor**:
it decomposes a complex task into smaller subtasks, then delegates each subtask to a distinct
"expert" instance of itself operating under specific, tailored instructions, and finally integrates
and verifies the experts' outputs into one coherent result. Structurally decoupled from content
(unlike few-shot, which injects content examples) - the conductor's scaffolding logic is reusable
across tasks without content binding. Source: Suzgun & Kalai, "Meta-Prompting: Enhancing Language
Models with Task-Agnostic Scaffolding" (arXiv:2401.12954, 2024), which also extends the approach to
tool integration (e.g., a Python interpreter as one of the "experts").

**When to use:** A single complex task cleanly decomposes into distinct sub-skills (e.g., "draft,"
"fact-check," "tone-edit") that benefit from separate, focused instructions rather than one
instruction trying to do everything at once. When prompt logic needs to scale to new content types
without rewriting the scaffold.

**Implementation:**
```
You are the conductor of a team of expert assistants, all played by you. For this task:
[TASK]

Step 1: Decompose the task into 3-5 subtasks, each requiring a distinct kind of expertise.
Step 2: For each subtask, define a tailored persona and instruction set for the "expert" who
  will handle it (e.g., "Expert 1: Research Analyst - gathers and verifies facts only").
Step 3: As conductor, run each expert in sequence, feeding each expert's output forward as
  context to the next where relevant.
Step 4: As conductor, integrate all expert outputs into one coherent final result, resolving
  any contradictions between experts explicitly.
Step 5: State which expert's output you are least confident in and why.
```

#### 19b. Recursive Meta-Prompting (RMP) - Proposer-Validator-Executor Self-Refinement

**Mechanistic basis:** A tighter, more automatable loop than 19a: a **proposer** model generates a
candidate edit to a prompt, a **validator** checks the edit against a schema (so only well-formed
edits are accepted), and an **executor** model runs the resulting prompt. Source: Zhang, Yuan & Yao,
"Meta Prompting for AI Systems" (arXiv:2311.11482), which formalizes the base framework categorically
(a functor mapping task transformations to prompt transformations) and extends it to this recursive
proposer/validator/executor loop, modeling edit accumulation with a Writer monad so that applying
several edits doesn't depend on the order they're grouped in. **Emerging, not settled:** this paper
was revised to its current version as recently as 2026-08-03 - days before this update - and the
authors are explicit that functorial structure preservation does not by itself guarantee semantic
correctness or convergence. Their own reported result (Qwen-72B guided by example-free meta-prompts:
46.3% pass@1 on MATH, 83.5% on GSM8K) is a single-model data point, not a general benchmark claim.

**When to use:** Iterative refinement across multiple rounds where each round should fix a bounded,
identifiable weakness rather than a wholesale rewrite - and where you want that discipline without
manually re-deriving the critique criteria each time. This is the mechanism the skill's own META-
PROMPTING SELF-REFINEMENT PASS (see SKILL.md) is modeled on, scoped down to a single bounded cycle.

**Implementation:**
```
You are a prompt engineering specialist. I will give you a draft prompt.

Your task:
Step 1 (Proposer): Identify the 3 most significant structural weaknesses in this prompt.
Step 2 (Validator): For each weakness, confirm it is a genuine structural issue - not a
  stylistic preference - before proposing a fix. Reject and discard any proposed edit that
  would change scope or intent, not just phrasing.
Step 3 (Executor): Produce a complete revised prompt incorporating only the validated edits.
Step 4: State the mechanistic reason why each accepted change improves performance, and name
  any proposed edit that was rejected and why.

Draft prompt to refine:
[PASTE PROMPT HERE]
```

**Recursive variation (bounded improvement loop - cap the rounds explicitly):**
```
Prompt version 1: [initial draft]

Apply this improvement cycle for a maximum of 2 rounds:
Round 1: Critique the prompt against these criteria: [criteria]. Rewrite to address every critique.
Round 2: Critique the Round 1 output against the same criteria. Rewrite only if a genuine
  weakness remains - if Round 1 already satisfies every criterion, output it unchanged and say so.
Output only the final result, plus one line naming how many rounds actually changed anything.
```

**Token efficiency note:** meta-prompts that focus on structural rules rather than content examples
can be reused across hundreds of tasks without modification - more token-efficient at scale than
few-shot prompts that require new content examples per task. Always cap recursive rounds explicitly
(as above) - an unbounded critique loop is a cost and latency risk with no guaranteed payoff, per the
convergence caveat above.

#### 19c. Lineage: Automatic Prompt Engineer (APE)

**Mechanistic basis:** The propose-score-search pattern both papers above build on. APE treats an
instruction as a "program" to be optimized: an LLM proposes a pool of instruction candidates from
input-output demonstrations, each candidate is scored by evaluating another LLM's zero-shot
performance under it, and an iterative Monte Carlo search proposes semantically similar variants of
the best-scoring candidates. Source: Zhou et al., "Large Language Models Are Human-Level Prompt
Engineers" (arXiv:2211.01910, 2022), which reports APE-generated instructions beat human-written ones
on 19 of 24 instruction-induction tasks tested. Relevant here mainly as lineage - it predates and
structurally anticipates the propose-then-select loop inside both 19a and 19b - not as a technique
this skill generates prompts for directly (it requires an evaluation harness, not a single chat turn).

#### 19d. Applied Instance: Anthropic's Own Metaprompt

Worth naming because it's the same job this skill performs, at Anthropic's own hands: the Metaprompt
(Claude Cookbook, `platform.claude.com/cookbook/misc-metaprompt`) is a long multi-shot prompt loaded
with worked examples of good prompts, used to guide Claude to draft a tailored starting prompt from a
plain-language task description - explicitly framed as solving the "blank page problem." The same
mechanism powers the Console's built-in "Generate Prompt" feature (`platform.claude.com/docs`,
Console prompting tools). Two honest scope differences worth carrying into any generated prompt that
cites this precedent: Anthropic's version is explicitly single-turn only (not multi-turn), and its
own documentation states the output "is not guaranteed to be optimal" - a first draft for iteration,
not a finished spec. This skill's own output aims for the fuller 6-Component Spec rather than a
first-draft template, but the underlying mechanism - an LM generating a tailored prompt from a task
description - is the same one this whole section describes.

---

## OBSOLESCENCE LIST - UPDATED 2026

**No longer effective on frontier models (2025-2026):**
- "According to Wikipedia..." - context injection trick; obsolete with RAG
- "You are ChatGPT" - explicit model identity overrides; filtered
- Explicit CoT on o1/o3/Claude Extended Thinking/Opus 4.8 - degrades performance
- **Minimum word count mandates for prompts** - structure beats length; padding produces generic output
- Hardcoding entire knowledge bases into the prompt - use RAG or fine-tuning
- Noisy padding examples in few-shot - quality over quantity
- Simple emotional priming without compound framing - weak effect alone
- **Claude 3.x intent-inference patterns** - Claude 4.x is literal; prompts relying on intent generalization are now a brittle failure mode

**Being phased out by architectural advances:**
- Manual few-shot for format consistency → being replaced by structured output modes and grammar-constrained decoding
- Manual CoT → being replaced by native reasoning in o-series, Extended Thinking, and Claude Opus 4.8 effort parameter
- Full-prompt knowledge injection → being replaced by RAG pipelines and fine-tuning for production
- Generic "be an expert" role definitions → being replaced by multi-sentence credentialed personas with behavioral commitments

---

## RESEARCH FOUNDATIONS - VERIFIED CITATIONS (added 2026-07-16, extended 2026-08-08)

Every citation below was checked by direct web search against the source paper's own abstract or a
reproduction of its reported figures, not copied from a secondary summary (original pass 2026-07-16;
meta-prompting family added and re-verified 2026-08-08). This replaces several unsourced or
misattributed statistics that existed in earlier versions of this file (see the correction notes
inline above for CoT's dropped "19-point MMLU-Pro" claim, Constitutional Self-Critique's dropped
"+15% TruthfulQA / Reflexion" misattribution, Program-of-Thought's corrected GSM8K figures, Graph-of-
Thought's corrected task domain, Lexical Density Optimization's dropped "+6.7%" claim, and most
recently Meta-Prompting's paper-title mix-up - see section 19 above for the full correction). Where a
paper's own number differs from what this skill treats as a realistic production estimate, both are
stated and attributed separately rather than blended into one figure.

| Technique | Paper | arXiv | Verified figure this skill relies on |
|---|---|---|---|
| Chain-of-Thought | Wei et al., 2022 | 2201.11903 | Foundational CoT result; commonly cited +15-40% range is benchmark-dependent, not a single fixed number |
| Zero-shot CoT | Kojima et al., 2022 | 2205.11916 | "Let's think step by step" as a zero-shot reasoning trigger |
| Self-Consistency | Wang et al., 2022 | 2203.11171 | Majority-vote over independently sampled reasoning chains |
| Chain-of-Draft | Xu et al., 2025 | 2502.18600 | Matches/beats CoT accuracy at as little as ~7.6% of CoT's token count (paper's best case) |
| Tree-of-Thought | Yao et al., 2023 | 2305.10601 | Game of 24: CoT 4% vs. ToT 74% (single-benchmark result) |
| Graph-of-Thought | Besta et al., 2023 | 2308.09687 | Sorting task: +62% quality over ToT, >31% lower cost |
| Skeleton-of-Thought | Ning et al., 2023 | 2307.15337 | Up to ~2.4x average speedup via parallel section generation |
| ReAct | Yao et al., 2022 | 2210.03629 | Interleaved Thought/Action/Observation loop for tool-using agents |
| Program-of-Thought / PAL | Gao et al., 2022 | 2211.10435 | GSM8K: 72.0% (PAL+Codex) vs. 65.6% (PaLM-540B+CoT), a 15-point gap |
| Constitutional AI | Bai et al., 2022 | 2212.08073 | Generate-critique-revise loop, trained via RLAIF; helpfulness/harmlessness tradeoff, not a TruthfulQA number |
| Reflexion | Shinn et al., 2023 | 2303.11366 | HumanEval: 91% pass@1 vs. 80% for the GPT-4 baseline compared against |
| Meta-Prompting (Conductor-Expert) | Suzgun & Kalai, 2024 | 2401.12954 | Single LM as conductor delegating to tailored "expert" instances of itself; corrected 2026-08-08 from a prior mix-up with 2311.11482 (an unrelated paper, near-identical title) |
| Recursive Meta-Prompting (RMP) | Zhang, Yuan & Yao | 2311.11482 | Categorical formalization (functor) extended to a proposer-validator-executor loop, modeled as a Writer monad; revised to current version 2026-08-03, so treat as active research |
| Automatic Prompt Engineer (APE) | Zhou et al., 2022 | 2211.01910 | Propose-score-search loop; APE-generated instructions beat human-written ones on 19/24 instruction-induction tasks tested |

**What this table does not cover:** the "Lexical Density Optimization" mechanistic claim above is
left uncited because no verifiable source was found for it, and it should be treated as this
skill's own reasoning from the broader prompt-sensitivity literature rather than a specific paper's
finding. If precision matters for your use case, verify current figures directly rather than
reusing any number from this file without checking it.
