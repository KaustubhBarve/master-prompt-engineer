# Platform Optimization Playbooks — 2025–2026 Frontier Model Matrix

## THE FUNDAMENTAL INSIGHT

A single universal prompt no longer exists. The ecosystem of foundation models in 2025–2026 has fractured into highly specialized architectures. Each model has distinct internal training mechanisms, attention capacities, instruction-following patterns, and latency profiles. Applying a Claude-optimized prompt to DeepSeek, or using verbose CoT on an o-series reasoning model, are not just suboptimal — they actively degrade performance.

Know your target. Adapt mechanistically.

---

## CLAUDE (Anthropic) — Sonnet 4 / Opus 4

### Architecture Reality
- **Context:** Up to 1M tokens (Sonnet 4)
- **Benchmark:** Elite qualitative analysis (9.0/10), exact tone matching, identifying subtle patterns
- **XML parsing:** Exceptionally strong — uses XML tags natively in its training and architecture
- **Instruction following:** Best-in-class for complex, multi-layered rule sets
- **Weakness:** Defaults to "playing it safe" creatively — needs explicit permission to push into bold/unconventional territory

### Optimal Prompt Structure
Use heavy XML tagging throughout. Claude's attention mechanism is tuned to recognize XML tags as structural delimiters.

```xml
<role>
[Detailed persona definition — 3–5 sentences including methodology and cognitive style]
</role>

<mission>
[What this AI exists to do in this context]
</mission>

<context>
[All necessary background — audience, stakes, situation]
</context>

<methodology>
[Numbered step-by-step process]
</methodology>

<output_specification>
[Exact format, section names, lengths, tone]
</output_specification>

<quality_gate>
[Self-scoring criteria and revision instruction]
</quality_gate>

<constraints>
[Positively framed hard limits]
</constraints>

<edge_cases>
[Explicit handling of ambiguous/incomplete inputs]
</edge_cases>
```

### Critical Tips
- Include explicit critique/evaluation steps — Claude performs well with Constitutional review loops
- For creative tasks: add "Approach this with full creative ambition — unconventional perspectives are welcome and valued"
- For long context: use section headers and repeat critical constraints at end
- Extended Thinking mode (when available): treat like o-series — strip CoT, use concise constraints only

---

## GPT-4o / GPT-5 (OpenAI)

### Architecture Reality
- **Speed:** Average 1.2s response time — fastest among frontier models
- **Benchmark:** Highly reliable; elite at complex logic and advanced coding
- **Strengths:** Code generation, structured output, consistent instruction following
- **Preference:** Directness and clarity over elaborate scaffolding

### Optimal Prompt Structure
Use markdown headers and numbered sections. GPT models respond well to clear visual structure without XML overhead.

```
## Role
[Persona definition — specific, credentialed]

## Context
[Background situation and audience]

## Task
[Clear task description — direct, no padding]

## Instructions
1. [Step]
2. [Step]
3. [Step]

## Output Format
[Exact specification]

## Constraints
[Positively framed limits]

## Quality Check
[Self-evaluation criteria]
```

### Critical Tips
- Direct, unadorned constraints work best — avoid elaborate meta-commentary
- CO-STAR framework works naturally with GPT-4o for content tasks
- For code tasks: "Write clean, production-ready, commented code. Include error handling and edge case coverage."
- Avoid padding and filler text — GPT models prefer precision over verbosity

---

## o1 / o3 — OpenAI INFERENCE-OPTIMIZED REASONING SERIES

### Architecture Reality
TOTAL PARADIGM INVERSION required. These models are explicitly RL-trained to execute deep latent cognitive processing BEFORE generating visible output. They already reason internally — CoT is an internal process, not a prompt-requested one.

- **Internal reasoning:** Executes before any visible output token
- **Zero-shot supremacy:** Extrapolates from pure rulesets without examples
- **Few-shot risk:** Examples can cause the model to over-index on syntax patterns over logic
- **API:** Must use Responses API with `store=true` for multi-turn coherence
- **Message role:** Use `developer` message, not `system` (higher hierarchical priority)

### Optimal Prompt Structure
```
[No elaborate scaffolding. No headers. Pure constraint specification.]

[Identity in one sentence maximum]
[Clear task statement]
[Specific constraints as a numbered list]
[Output format specification — exact]
[Quality criteria — explicit]

[If markdown is needed: add "Format your response using markdown headers and structure."]
```

### What to NEVER Include for o-Series
- "Think step by step"
- "Explain your reasoning"
- "Work through this methodically"
- "Show your thought process"
- Elaborate few-shot examples
- Multi-step chain-of-thought instructions
- Any verbose cognitive scaffolding

These cause probability collisions with the model's internal reasoning, inflate tokens, and degrade output quality.

### Multi-Turn Configuration
```javascript
// Always use Responses API, not Chat Completions
const response = await openai.responses.create({
  model: "o3",
  previous_response_id: previousResponseId, // maintains internal reasoning trace
  input: userMessage,
  store: true
});
```

---

## CLAUDE EXTENDED THINKING MODE

### Architecture Reality
Identical to o-series from a prompting perspective. Extended Thinking executes internal deliberative reasoning before visible output. External CoT instructions interfere with this process.

### Treat exactly like o-series:
- Concise, constraint-only prompts
- Zero CoT directives
- Zero-shot by default
- XML tags still work for structural separation (unlike o-series)
- Let the thinking happen internally

---

## GEMINI 2.x / ULTRA (Google)

### Architecture Reality
- **Multimodal:** Strong at image, video, audio, document understanding
- **Multilingual:** Strong Chinese, Spanish, and other high-resource language support
- **Comparative analysis:** Exceptionally good at structured comparison and evaluation
- **Strengths:** Multi-document synthesis, comparative reasoning, verification tasks

### Optimal Prompt Structure
```
## Task
[Clear description with explicit modality labeling if multimedia]

## Context
[Background including any media description: "Image provided: [description]"]

## Analysis Framework
[Comparative structure — compare X and Y across these specific dimensions:]
- Dimension 1: [What to evaluate]
- Dimension 2: [What to evaluate]

## Output Format
[Explicit structure — Gemini responds well to table/matrix formats for comparison]

## Verification Step
[Instruct Gemini to verify key claims before including them]
```

### Critical Tips
- Always explicitly label modalities: "The image below shows [context]. The document attached is [type]."
- Gemini excels at comparative framing — use for vs. analysis, pros/cons, scenario comparison
- For multilingual tasks: Gemini has stronger non-English capability than most frontier models
- Add verification step: "Before including any factual claim, verify it is supported by the provided material."

---

## DEEPSEEK-V3 / R1

### Architecture Reality
- **Quantitative reasoning:** Unmatched (8.6/10) — best for math, statistics, competitive programming
- **Cost-to-performance:** Exceptional — strong for production at scale
- **English prose:** Noticeably weaker (7.2/10) — outputs can feel sterile or mechanical without explicit style instruction
- **Structural parsing:** Strong — benefits from rigid explicit formatting

### Optimal Prompt Structure
```
ROLE: [One specific sentence]

TASK: [Direct task statement — no padding]

PROCESS:
1. [Explicit step]
2. [Explicit step]
3. [Explicit step]

OUTPUT FORMAT: [Rigid specification — section names, lengths, bullets vs prose]

TONE: [Explicitly define vocabulary level, sentence length, formality, register]
STYLE: [Explicitly dictate cadence, voice, use of technical vs accessible language]
CONSTRAINTS: [Hard limits as numbered list]
```

### Critical Tips
- For math/code tasks: DeepSeek is exceptional — use Program-of-Thought liberally
- For prose tasks: explicitly dictate tone, vocabulary, sentence structure, and cadence — without this, output feels robotic
- Use rigid structural parsing — numbered lists work better than flowing prose instructions
- Strong on Chinese-language tasks — for Chinese output, keep instructions in English, specify output language

---

## GROK-3 (xAI)

### Architecture Reality
- **Context window:** 2,000,000 tokens — largest available in 2025–2026
- **Creativity:** Highly creative (8.0/10) with unique safety architecture
- **Style:** Irreverent, divergent, willing to explore unconventional angles
- **Strength:** Lateral thinking, fictional framing, creative divergence

### Optimal Prompt Structure
Grok thrives with creative freedom within a defined frame. Don't over-constrain — define the goal and let Grok find the path.

```
[Identity that emphasizes creativity and unconventional thinking]

[Task framing as creative challenge or scenario, not rigid instruction set]

[Desired outcome — what success looks like — not exactly how to get there]

[Core constraints — only the non-negotiable ones]

[Permission to explore unconventional approaches: "Lateral thinking, unexpected angles, and divergent approaches are encouraged"]
```

### Massive Context Leverage
```
With your 2M token context window, I am providing the following extensive reference material.
Use all of it to inform your response — don't just skim the beginning.

[Inject months of logs, full books, entire codebases, etc.]
```

### Critical Tips
- Complex role-play and fictional framing work especially well
- Gradual escalation technique is effective for nuanced exploration
- Leverage the massive context window — Grok can process and synthesize more reference material than any other available model

---

## MISTRAL LARGE 2

### Architecture Reality
- **Latency:** Very low — highly efficient instruction following
- **Performance:** Exceptional relative to compute footprint
- **Context:** 128K — but degrades faster than frontier models past ~40K practical tokens
- **Strength:** High-density instruction parsing, concise prompts

### Optimal Prompt Structure
```
[Role — 1 concise sentence]

[Task — direct, specific, no padding]

[Step-Back abstraction — add this for complex tasks]:
"First, identify the underlying principles that govern this task.
Then apply them to the specific request."

[Output format — very specific]

[Constraints — short numbered list]
```

### Critical Tips
- Keep prompts concise and high-density — Mistral performs better with focused, specific instructions
- Step-back abstraction works particularly well — helps Mistral access robust training representations
- Avoid bloated context windows — practical limit is ~40K tokens for reliable recall
- High-information-density instructions beat long elaborate ones

---

## LLAMA 3 / OPEN-SOURCE MODELS

### Architecture Reality
- **Instruction-following:** Weaker than frontier models — needs explicit guidance
- **Context:** 70B @ 128K tokens
- **Strengths:** Privacy/local deployment, customizable, no rate limits
- **Weakness:** Implicit assumptions get missed; needs literal, complete instructions

### Optimal Prompt Structure
Be extremely literal. State everything. Assume nothing.

```
You are [role — extremely specific].

Your task is to [exact task description].

You must follow these instructions in exactly this order:
1. [Action]
2. [Action]
3. [Action]

Your response must be in exactly this format:
SECTION_NAME: [content]
SECTION_NAME: [content]

Do not include any other text, preamble, or explanation outside this format.

Input you will receive: [exact description of what to expect]
```

### Critical Tips
- Include explicit stop sequences and output wrappers
- More examples = better results — few-shot is more impactful here than with frontier models
- Quantized models hallucinate more — add explicit verification steps
- Quality degrades significantly below 7B parameters — avoid for complex tasks
- Consider prompt length carefully — Llama models benefit from shorter, clearer prompts vs. long complex ones

---

## UNIVERSAL PROMPTS (Cross-Platform Portability)

When you need a prompt that works acceptably across multiple platforms:

### Universal Best Practices
1. Use plain numbered lists — not XML tags (platform-specific) or heavy markdown
2. Keep instructions concise and unambiguous — no platform-specific scaffolding
3. Include 1–2 contrastive examples (correct + flawed)
4. State format requirements explicitly with a sample structure
5. Avoid platform-specific features: no XML delimiters, no `developer` message role, no grammar constraints

### Universal Template
```
You are a [specific role with credentials and methodology].

Your mission: [Clear outcome statement — what success looks like].

Background: [Essential context — audience, stakes, situation].

Follow these steps:
1. [Step — explicit action]
2. [Step — explicit action]
3. [Step — explicit action]

Output this exact structure:
[Section name]: [what goes here]
[Section name]: [what goes here]
[Section name]: [what goes here]

Quality check: Before finalizing, verify that your response is [criteria 1], [criteria 2], and [criteria 3]. If not, revise.

Constraints:
- [Hard limit stated positively]
- [Hard limit stated positively]
```

---

## CONTEXT ENGINEERING — UNIVERSAL RULES

Applies to all platforms. Context engineering is now as important as prompt phrasing.

### The Lost-in-the-Middle Problem
Models show a U-shaped attention curve over long contexts:
- Tokens at the START receive high attention weight
- Tokens at the END receive high attention weight
- Tokens in the MIDDLE receive degraded attention weight (especially past 3,000 tokens)

**Solution: Placement-by-function**
- START of prompt: Role, identity, background context, reference documents, persona definition
- END of prompt: Active task instructions, output specification, quality gate, activation phrase
- BOTH START AND END: Critical constraints and non-negotiable rules

### Context Rot at Scale
Even 1M+ token models degrade in practical recall past ~64K tokens in real applications. KV cache eviction policies remove less-recently-used tokens in long windows.

**Mitigation strategies:**
1. Retrieval-first — use RAG rather than dumping full documents into context
2. Hierarchical summarization — summarize earlier context before adding new
3. Explicit attention markers — use "CRITICAL:", "IMPORTANT:", or section headers to re-anchor attention after long reference blocks
4. Focused injection — retrieve only the specific relevant chunk, not entire documents

### Memory Management for Multi-Turn
Standard Chat Completions APIs do not persist internal reasoning traces between turns.

- **Claude:** Include all relevant prior context in each system message
- **OpenAI o-series:** Use Responses API with `previous_response_id` to maintain cognitive trace
- **All others:** Manually thread conversation history; summarize older context before it exceeds practical limits
