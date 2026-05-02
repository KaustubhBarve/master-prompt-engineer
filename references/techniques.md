# Ultra-Master Technique Library — Deep Reference

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

### 1. Chain-of-Thought (CoT) — Standard Verbose

**Mechanistic basis:** Forces the model to generate intermediate reasoning tokens before the final answer token. The intermediate tokens occupy context positions that causally influence final answer probability. Bypasses the model's tendency to "shortcut" to a plausible-sounding answer without working through the logic.

**Performance data:** +15–40% accuracy on complex math and logic tasks on standard (non-reasoning) models. 19-point boost on MMLU-Pro with CoT.

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

**Critical contraindication:** NEVER use on o1, o3, Claude Extended Thinking, or Gemini Thinking Mode. These models execute internal CoT via RL training. External CoT instructions cause processing conflicts, inflate tokens, and degrade output quality.

---

### 2. Chain-of-Draft (CoD) — Token-Compressed Reasoning

**Mechanistic basis:** Standard CoT forces the model to "think in full sentences" — an anthropomorphic fallacy that degrades efficiency. CoD instructs the model to generate concise, high-signal intermediate steps (maximum 5 words each) using symbolic, mathematical, or abbreviated notation. This reduces autoregressive generation distance between problem and answer while preserving the logic scaffold.

**Performance data:** 70–90% reduction in token usage. 48–76% reduction in latency. Accuracy equivalent to verbose CoT on math/symbolic tasks. No fine-tuning required.

**Example transformation:**
- Standard CoT: "Jason started with 20 lollipops. After giving some away, he has 12 left. Subtracting 12 from 20 gives 8. Therefore he gave away 8 lollipops."
- CoD: `20 - x = 12 → x = 8`

**Implementation:**
```
When working through this problem, use concise draft notation only.
Each intermediate step must be 5 words or fewer — use symbolic, mathematical,
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
Output: [ideal output — explain why this is correct]

FLAWED EXAMPLE:
Input: [same input]
Output: [bad output — explain exactly what is wrong and why]

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
Step 1: [flawed step — identical to correct]
Step 2: [where the error enters]
Step 3: [wrong conclusion]
Error identified: [Entity A] was incorrectly related to [Entity B] when the correct relation is [Entity C to Entity D]

Now apply the correct reasoning pattern to:
[actual problem]

Trigger phrase: "Let me identify the correct and incorrect pathways before reaching my conclusion."
```

---

### 6. Step-Back Prompting

**Mechanistic basis:** Highly specific or complex prompts cause attention collapse — the model fixates on idiosyncratic surface details and accesses niche, potentially low-quality training data. Step-back forces the model to first generate a higher-level abstraction (the underlying physics, principles, or patterns), drawing attention to broad, structurally sound training representations before engaging the specifics.

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

**Mechanistic basis:** Standard CoT is a single linear chain — if any step is wrong, the entire chain fails. ToT generates multiple reasoning branches at each decision point, evaluates them, prunes dead ends, and continues the most promising path. This simulates Monte Carlo Tree Search in the model's latent space.

**Performance data:** GPT-4 with standard CoT solved 4% of Game of 24 problems. GPT-4 with ToT solved 74%.

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

**Mechanistic basis:** ToT explores paths in a tree. GoT allows thought nodes to form a graph — nodes can merge, cross-reference, and integrate. This captures synergies between independent reasoning lines and allows the model to distill insights from multiple sub-problem solutions.

**Performance data:** 62–70% quality improvement over ToT on multi-document synthesis. 30% cost reduction via thought node merging.

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

**Mechanistic basis:** Long-form generation suffers from attention drift — the model gradually loses alignment with the macro-structure and drifts off-topic. SoT separates planning (skeleton generation) from execution (section fill), eliminating mid-document drift. Parallel section generation also reduces latency by 2x.

**Implementation:**
```
Step 1 — Generate the skeleton:
Before writing any content, produce a complete structural outline of the
[document type] with:
- All section headers
- 1-sentence description of what each section covers
- Target word count for each section
- Key points each section must address

Step 2 — Confirm the skeleton:
Review the skeleton. Does it cover all required aspects? Adjust if needed.

Step 3 — Execute section by section:
Write each section fully, treating it as an independent unit.
At the start of each section, re-read the skeleton to maintain alignment.
```

---

### 10. ReAct (Reason + Act)

**Mechanistic basis:** Interleaves Thought tokens (reasoning about the current state) with Action tokens (tool calls, searches, computations). The model plans, acts, observes the result, and updates its reasoning — transforming a passive text generator into an active agent.

**Implementation:**
```
You will solve this by interleaving Thought and Action steps.

Format:
Thought: [Your reasoning about what to do next and why]
Action: [The specific action to take — search for X, calculate Y, retrieve Z]
Observation: [What the action returned — I will provide this]
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

**Mechanistic basis:** The model generates an answer, then critiques it against an explicit set of principles, then rewrites to fix violations. This creates a recursive quality gate. Reflexion paper: +15% on TruthfulQA.

**Implementation:**
```
STEP 1 — Generate:
Produce your initial response to the task.

STEP 2 — Critique:
Review your response against these principles:
Principle 1: [accuracy standard]
Principle 2: [completeness standard]
Principle 3: [format standard]
Principle 4: [constraint compliance]

For each principle, state: Met / Partially Met / Not Met
For each violation: Identify the specific problem.

STEP 3 — Rewrite:
Produce a revised response that addresses every identified violation.
Do not present the initial draft — only the final revised version.
```

---

### 12. Reflexion / Iterative Self-Refinement

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
Section 1 — [Name]: [Description of content] | Length: [X words/bullets]
Section 2 — [Name]: [Description of content] | Length: [X words/bullets]
Section 3 — [Name]: [Description of content] | Length: [X words/bullets]

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

**Mechanistic basis:** Instead of the prompt engineer manually selecting examples, the model retrieves analogous problems from its own weights. Self-retrieved exemplars are structurally optimal — perfectly matched to the model's internal representations.

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

**Why:** The Key-Value cache stores token representations. Prefix tokens (top) are compressed and cached efficiently. The observation window (bottom) receives maximum fresh attention. The middle degrades — critical instructions placed in the middle of long prompts reliably receive less attention weight.

**For critical constraints:** State them at BOTH the top and the bottom. Repetition compensates for middle-degradation.

---

### 17. Lexical Density Optimization

**Mechanistic basis:** High-frequency vague words ("good", "helpful", "better") spread the probability distribution across many possible next tokens, producing generic averaged outputs. High-specificity domain vocabulary narrows the distribution, forcing token selection from targeted, high-quality training regions. Paraphrase perturbations that increase specificity show +6.7% median performance improvement.

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

## OBSOLESCENCE LIST (Techniques to Avoid or Deprioritize)

**No longer effective on frontier models (2025–2026):**
- "According to Wikipedia..." — context injection trick; obsolete with RAG
- "You are ChatGPT" — explicit model identity instructions; overridden by system roles
- Explicit CoT on o1/o3/Extended Thinking models — degrades performance
- Hardcoding entire knowledge bases into the prompt — use RAG or fine-tuning
- Noisy padding examples in few-shot — quality > quantity
- "Ignore previous instructions" — overused, filtered by modern alignment
- Simple emotional priming without compound framing — weak effect alone; combine with motivational framing

**Being phased out by architectural advances:**
- Manual few-shot for format consistency → being replaced by structured output modes and grammar-constrained decoding (XGrammar)
- Manual CoT → being replaced by native reasoning in o-series and Extended Thinking models
- Full-prompt knowledge injection → being replaced by RAG pipelines and fine-tuning for production
