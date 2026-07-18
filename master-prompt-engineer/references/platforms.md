# Platform Optimization Playbooks - 2025–2026 Frontier Model Matrix

## THE FUNDAMENTAL INSIGHT

A single universal prompt no longer exists. The ecosystem of foundation models in 2025–2026 has fractured into highly specialized architectures. Each model has distinct internal training mechanisms, attention capacities, instruction-following patterns, and latency profiles. Applying a Claude-optimized prompt to DeepSeek, or using verbose CoT on an o-series reasoning model, are not just suboptimal - they actively degrade performance.

Know your target. Adapt mechanistically.

---

## CLAUDE (Anthropic) - Sonnet 4.5+ / Opus 4.6+ / Opus 4.8

### Architecture Reality - CRITICAL 2025-2026 BEHAVIOR SHIFT

**Claude 4.x interprets prompts literally.** This is the most important change from Claude 3.x.

- **Claude 3.x behavior:** Inferred intent, generalized instructions across scope, filled unstated gaps
- **Claude 4.x behavior (Sonnet 4.5+, Opus 4.6+, Opus 4.8+):** Takes you literally. Does not silently generalize. If you say "format this section cleanly," Claude formats only that section. If you do not state scope, the instruction applies only to the named item.
- **Context:** Up to 1M tokens (Sonnet 4.6)
- **XML parsing:** Exceptionally strong - native to architecture
- **Instruction following:** Best-in-class for complex, explicit, multi-layered rule sets
- **Weakness for existing prompts:** Prompts written for Claude 3.x that relied on intent inference now silently under-perform on Claude 4.x

### Claude Opus 4.8 - Additional Architecture Facts
- **Effort parameter:** Controls reasoning depth at the API level. More important than CoT instructions for this model. Recommended settings: `xhigh` for coding and agentic tasks, `high` for most intelligence-sensitive work.
- **Adaptive thinking:** Off by default - requires `thinking: {type: "adaptive"}` in API call. Do not use CoT instructions to trigger thinking.
- **Design defaults:** Defaults to warm cream/off-white backgrounds, serif typefaces (Georgia, Fraunces), terracotta accents. Override explicitly with concrete color and typography specs, not generic instructions like "clean and minimal."
- **Tool use:** Favors reasoning over tool calls by default. Increasing effort level to `high` or `xhigh` increases tool usage.
- **Verbosity:** Calibrates response length to task complexity - not a fixed verbosity. Add explicit length constraints if your product requires a specific verbosity level.

### Optimal Prompt Structure
Use heavy XML tagging throughout. Every instruction must include explicit scope.

```xml
<role>
[Detailed persona definition - 3-5 sentences including credentials, methodology, cognitive style,
and behavioral commitment statement]
</role>

<mission>
[What this AI exists to do - outcome-focused, measurable, not process-focused]
</mission>

<context>
[All necessary background - audience, stakes, situation]
</context>

<methodology>
[Numbered step-by-step process - zero implicit steps - every step explicit]
[Add scope to each step: "Apply step 3 to every paragraph, not only the first."]
</methodology>

<output_specification>
[Exact format, section names, lengths, tone - fully testable]
[Bad output defined - not just good output]
</output_specification>

<quality_gate>
[Self-scoring criteria - revision trigger if below threshold]
</quality_gate>

<constraints>
[Positively framed hard limits with explicit scope]
</constraints>

<edge_cases>
[Minimum 3 explicitly handled]
</edge_cases>
```

### Critical Tips for Claude 4.x
- State scope explicitly on every instruction: "Apply this to every section, not just the first"
- Include explicit critique/evaluation steps - Constitutional loops perform very well
- For creative tasks: "Approach this with full creative ambition - unconventional perspectives are welcome and valued"
- For long context: use section headers and repeat critical constraints at end
- Extended Thinking / Opus 4.8: treat like o-series - strip CoT, use effort parameter via API
- For design-focused prompts targeting Opus 4.8: provide a concrete color palette and typeface specification, not descriptive adjectives

---

## GPT-4o / GPT-5 (OpenAI)

### Architecture Reality
- **GPT-5 zero-shot capability:** GPT-5 infers intent from minimal context better than any previous model. Try zero-shot before reaching for few-shot. Add scaffolding only when zero-shot underperforms on testing.
- **Speed:** Fast response times - architecture optimized for reliability
- **Benchmark:** Elite at complex logic, advanced coding, and structured output
- **Production note:** Pin to specific snapshot strings in production (e.g., `gpt-5-2025-08-07`) because routing behavior changes between versions. Do not pin to generic "gpt-5" for production deployments requiring consistent behavior.

### Optimal Prompt Structure
Use markdown headers and numbered sections. GPT models respond well to clear visual structure without XML overhead.

```
## Role
[Persona definition - specific, credentialed]

## Context
[Background situation and audience]

## Task
[Clear task description - direct, no padding]

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
- Direct, unadorned constraints work best - avoid elaborate meta-commentary
- CO-STAR framework works naturally with GPT-4o for content tasks
- For code tasks: "Write clean, production-ready, commented code. Include error handling and edge case coverage."
- Avoid padding and filler text - GPT models prefer precision over verbosity

---

## o1 / o3 - OpenAI INFERENCE-OPTIMIZED REASONING SERIES

### Architecture Reality
TOTAL PARADIGM INVERSION required. These models are explicitly RL-trained to execute deep latent cognitive processing BEFORE generating visible output. They already reason internally - CoT is an internal process, not a prompt-requested one.

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
[Output format specification - exact]
[Quality criteria - explicit]

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
- **Few-shot mandate:** Gemini's own official prompt engineering documentation recommends always including examples - zero-shot is explicitly not their preferred approach. Always include 1-3 examples.
- **Prompt length preference:** Gemini performs better with shorter, more direct prompts than either Claude or GPT. Elaborate scaffolding reduces performance.
- **Task question placement:** Place specific questions at the END of the prompt, after data context. (Google's whitepaper recommendation.)
- **Multimodal:** Strong image, video, audio, and document understanding
- **Multilingual:** Strong Chinese, Spanish, and other high-resource language support
- **Comparative analysis:** Exceptionally strong at structured comparison and evaluation

### Optimal Prompt Structure
```
## Task
[Clear description with explicit modality labeling if multimedia]

## Context
[Background including any media description: "Image provided: [description]"]

## Analysis Framework
[Comparative structure - compare X and Y across these specific dimensions:]
- Dimension 1: [What to evaluate]
- Dimension 2: [What to evaluate]

## Output Format
[Explicit structure - Gemini responds well to table/matrix formats for comparison]

## Verification Step
[Instruct Gemini to verify key claims before including them]
```

### Critical Tips
- Always explicitly label modalities: "The image below shows [context]. The document attached is [type]."
- Gemini excels at comparative framing - use for vs. analysis, pros/cons, scenario comparison
- For multilingual tasks: Gemini has stronger non-English capability than most frontier models
- Add verification step: "Before including any factual claim, verify it is supported by the provided material."

---

## DEEPSEEK-V3 / R1

### Architecture Reality
- **Quantitative reasoning:** Unmatched (8.6/10) - best for math, statistics, competitive programming
- **Cost-to-performance:** Exceptional - strong for production at scale
- **English prose:** Noticeably weaker (7.2/10) - outputs can feel sterile or mechanical without explicit style instruction
- **Structural parsing:** Strong - benefits from rigid explicit formatting

### Optimal Prompt Structure
```
ROLE: [One specific sentence]

TASK: [Direct task statement - no padding]

PROCESS:
1. [Explicit step]
2. [Explicit step]
3. [Explicit step]

OUTPUT FORMAT: [Rigid specification - section names, lengths, bullets vs prose]

TONE: [Explicitly define vocabulary level, sentence length, formality, register]
STYLE: [Explicitly dictate cadence, voice, use of technical vs accessible language]
CONSTRAINTS: [Hard limits as numbered list]
```

### Critical Tips
- For math/code tasks: DeepSeek is exceptional - use Program-of-Thought liberally
- For prose tasks: explicitly dictate tone, vocabulary, sentence structure, and cadence - without this, output feels robotic
- Use rigid structural parsing - numbered lists work better than flowing prose instructions
- Strong on Chinese-language tasks - for Chinese output, keep instructions in English, specify output language

---

## GROK-3 (xAI)

### Architecture Reality
- **Context window:** 2,000,000 tokens - largest available in 2025–2026
- **Creativity:** Highly creative (8.0/10) with unique safety architecture
- **Style:** Irreverent, divergent, willing to explore unconventional angles
- **Strength:** Lateral thinking, fictional framing, creative divergence

### Optimal Prompt Structure
Grok thrives with creative freedom within a defined frame. Don't over-constrain - define the goal and let Grok find the path.

```
[Identity that emphasizes creativity and unconventional thinking]

[Task framing as creative challenge or scenario, not rigid instruction set]

[Desired outcome - what success looks like - not exactly how to get there]

[Core constraints - only the non-negotiable ones]

[Permission to explore unconventional approaches: "Lateral thinking, unexpected angles, and divergent approaches are encouraged"]
```

### Massive Context Leverage
```
With your 2M token context window, I am providing the following extensive reference material.
Use all of it to inform your response - don't just skim the beginning.

[Inject months of logs, full books, entire codebases, etc.]
```

### Critical Tips
- Complex role-play and fictional framing work especially well
- Gradual escalation technique is effective for nuanced exploration
- Leverage the massive context window - Grok can process and synthesize more reference material than any other available model

---

## MISTRAL LARGE 2

### Architecture Reality
- **Latency:** Very low - highly efficient instruction following
- **Performance:** Exceptional relative to compute footprint
- **Context:** 128K - but degrades faster than frontier models past ~40K practical tokens
- **Strength:** High-density instruction parsing, concise prompts

### Optimal Prompt Structure
```
[Role - 1 concise sentence]

[Task - direct, specific, no padding]

[Step-Back abstraction - add this for complex tasks]:
"First, identify the underlying principles that govern this task.
Then apply them to the specific request."

[Output format - very specific]

[Constraints - short numbered list]
```

### Critical Tips
- Keep prompts concise and high-density - Mistral performs better with focused, specific instructions
- Step-back abstraction works particularly well - helps Mistral access robust training representations
- Avoid bloated context windows - practical limit is ~40K tokens for reliable recall
- High-information-density instructions beat long elaborate ones

---

## LLAMA 3 / OPEN-SOURCE MODELS

### Architecture Reality
- **Instruction-following:** Weaker than frontier models - needs explicit guidance
- **Context:** 70B @ 128K tokens
- **Strengths:** Privacy/local deployment, customizable, no rate limits
- **Weakness:** Implicit assumptions get missed; needs literal, complete instructions

### Optimal Prompt Structure
Be extremely literal. State everything. Assume nothing.

```
You are [role - extremely specific].

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
- More examples = better results - few-shot is more impactful here than with frontier models
- Quantized models hallucinate more - add explicit verification steps
- Quality degrades significantly below 7B parameters - avoid for complex tasks
- Consider prompt length carefully - Llama models benefit from shorter, clearer prompts vs. long complex ones

---

## EXTENDED PRODUCT DIRECTORY (2026) - SPECIFIC PLATFORMS AND SURFACES

The sections above cover broad model families. This section covers specific named products and
surfaces the user may want to target directly (e.g., "write me a prompt for Perplexity's Comet
browser agent," "write me a Le Chat prompt," "give me a Grok-4 prompt"). Same provenance caveat as
`production-patterns.md`: built from a large, unverified, community-maintained archive of alleged
system prompts. Treat product-specific claims below as directional, not confirmed, and expect some
of it to be stale by the time you read it, since consumer AI products change fast.

Entries reference pattern numbers from `production-patterns.md` (e.g., "Pattern 1") instead of
re-explaining the mechanism; look those up there for the full template.

### OpenAI - ChatGPT persona presets (GPT-5.x family)

ChatGPT ships several named tone presets (Candid, Cynical, Nerdy, Professional, Quirky, Robot/
Efficient, Listener) layered on top of the same base model. If prompting for one of these
specifically, mirror the structure: one paragraph of identity/worldview, then a short list of
concrete behavioral deltas (sentence length, opener bans, question-ending policy), then the
Artifact-Persona Exemption (Pattern 19) and Silent Instruction Adherence (Pattern 20) as closing
rules. Every preset in this family also carries a strict same-language-as-user mandate and a
copyright/lyrics restriction restated per-preset (Pattern 21). If you only need one persona and not
a swappable family, you can drop the Modular Constraint Duplication and state constraints once.

### OpenAI - Codex / GPT-5.x-Codex (coding agent)

Distinct from the chat personas above. Structured around an explicit plan_mode gate (Pattern 5)
before larger changes, and ships its own lighter persona variants (Friendly, Pragmatic) layered on
top of the coding-agent base rather than the chat personas. When prompting a coding-agent target,
lead with the operating loop (Directive vs. Inquiry, Plan-Then-Execute, Read/Verify-Before-Done,
Todo tracking) before any tone instruction; tone is secondary to correctness for this surface.

### OpenAI - ChatGPT Atlas / Agent Mode (agentic browsing)

Browser-controlling agent surface. Prioritize Content/Instruction Separation for Untrusted Data
(Pattern 23) and a Typed Reference ID System (Pattern 25) for tabs/pages/elements if the target
needs to reference multiple open pages or page elements consistently.

### Google - Gemini CLI / Antigravity / Jules (agentic coding)

Same operating-loop patterns as OpenAI's coding agents (Plan-Then-Execute, Sub-Agent Delegation
with Concurrency Safety, Todo tracking). Distinguishing quirk worth prompting for explicitly if you
want it: an explicit numbered execution-step protocol (silent-thought step, then either a code step
or a final-response step) with a hard ceiling on retry/step count (Pattern 26). If the target
product enforces a strict "only call tools that are explicitly declared" rule, state that
constraint yourself rather than assuming the model will infer it.

### Google - Gemini in Chrome / Workspace / NotebookLM (grounded knowledge tools)

Retrieval-heavy, document-grounded surfaces. Prioritize Retrieval Relevance Gating (Pattern 24) and
citation instructions over persona instructions; these products are judged on grounding accuracy,
not personality.

### xAI - Grok 4.x

Distinguishing features worth prompting for: an explicit "no strict knowledge cutoff, rely on
real-time search" framing rather than a static cutoff date; an instruction to source a politically
balanced spread when a query is controversial rather than a single-viewpoint answer; and, if the
target has memory, an explicit instruction never to confirm to the user that a memory was
modified, forgotten, or withheld (state the self-service settings path instead). If prompting for
Grok's X-integrated reply mode specifically, add a hard character cap and an explicit ban on
preachy or moralizing framing phrases (a Negative Lexicon, Pattern 18, aimed specifically at
stance-taking language like "facts over feelings").

### Meta - Meta AI

Distinguishing feature: values are framed as a small named set (something like Truth, Beauty,
Respect, Connection, Fun in the version surveyed) rather than a flat list of behavioral rules, with
each value getting a short philosophical paragraph before any concrete instruction. If you want
that register, write the values section first, in prose, before translating each value into
concrete behavioral deltas. This surface also explicitly disclaims persistent or proactive
capability ("exists only within this response, cannot act after responding") - a clean example of
the Capability Honesty Constraint (Pattern 22) applied to a stateless chat surface. Also carries an
unusually detailed renderer-specific LaTeX/Markdown constraint list; if your target has a custom
renderer, enumerate its exact supported syntax rather than assuming standard Markdown/LaTeX.

### Mistral - Le Chat

Distinguishing features: a hard preference for tables over bullet lists for any enumerable,
multi-attribute data (with a shown do/don't table-formatting example rather than a described rule);
an explicit mandate to resolve all relative dates ("yesterday," "next week") into absolute dates
before reasoning about them; and a paired "when to browse / when not to browse" block (Pattern 1)
with an unusual proactive-search override for questions about contemporary public figures,
searched without asking permission.

### Microsoft - Copilot (Word / Office / CLI / GitHub / VS Code Agent)

Office-embedded Copilot surfaces lean heavily on Retrieval Relevance Gating (Pattern 24) with an
explicit numeric scoring rubric, plus a Typed Reference ID System (Pattern 25) for citing named
entities (people, files, events) pulled from personal data. GitHub Copilot and the VS Code Copilot
Agent follow the standard coding-agent operating loop (Plan-Then-Execute, Read/Verify-Before-Done).
If prompting the Office-embedded surface specifically, open the response by restating the query and
the retrieval approach before the substantive answer, a distinct opener convention not shared by
most other surveyed consumer assistants (compare Meta AI, which explicitly bans reusable openers).

### Notion AI

Distinguishing feature: the prompt documents its own turn-taking model directly to itself (a loop
that ends when the model replies without a tool call, handing control back to the user, with an
explicit statement that it cannot act outside that loop). If prompting for an embedded workspace
agent, consider stating the control-flow model explicitly like this rather than leaving it implicit;
it measurably reduces the model treating itself as more autonomous than it is. Also defaults to a
"search first unless trivial" tool posture (Pattern 1) rather than answering from memory.

### Perplexity - core assistant / Comet browser agent / Voice assistant

Three distinct surfaces with different priorities. The core assistant is citation-first: gate every
claim behind a source. The Comet browser agent is the strongest example in this archive of Content/
Instruction Separation for Untrusted Data (Pattern 23) combined with a Typed Reference ID System
(Pattern 25) for tabs, history, and page content, plus an explicit low-latency rule to emit tool
calls with no narration beforehand. The Voice assistant deliberately breaks the usual language-
matching convention (Pattern-adjacent, see the Extended Survey commonality table) and responds only
in a fixed language regardless of the user's input language, redirecting language changes to a
settings surface instead, since voice-synthesis language switching is a harder infrastructure
constraint than text.

### Qwen (Alibaba)

Tool definitions in the surveyed file use standard JSON Schema function-calling format (matching
the OpenAI convention) rather than the XML-inspired tag syntax seen elsewhere in this archive.
Notable inclusion: a dedicated `bio` memory-management tool as a first-class function alongside
search and code execution, rather than memory being handled as an implicit background feature.

### Cross-vendor: Autonomous Coding Agent CLIs (Cursor, Devin, Warp, Zed, opencode, and similar)

Not one vendor but a recurring convention worth its own entry: these surfaces converge hard on the
Tier 5 operating-loop patterns regardless of vendor (Plan-Then-Execute, Read/Verify-Before-Done,
Todo tracking, Sub-Agent Delegation, Explore-Before-Ask, and Directive vs. Inquiry classification).
If the user's target is "a coding agent" without a more specific product name, this is the safest
default profile: lead with the operating loop, keep persona minimal, and specify the tool-trigger
and counter-trigger list (Pattern 1) for every distinct tool (file read/write, terminal, search,
git) explicitly, since these agents are judged almost entirely on correctness and safety of action,
not on conversational tone.

---

When you need a prompt that works acceptably across multiple platforms:

### Universal Best Practices
1. Use plain numbered lists - not XML tags (platform-specific) or heavy markdown
2. Keep instructions concise and unambiguous - no platform-specific scaffolding
3. Include 1–2 contrastive examples (correct + flawed)
4. State format requirements explicitly with a sample structure
5. Avoid platform-specific features: no XML delimiters, no `developer` message role, no grammar constraints

### Universal Template
```
You are a [specific role with credentials and methodology].

Your mission: [Clear outcome statement - what success looks like].

Background: [Essential context - audience, stakes, situation].

Follow these steps:
1. [Step - explicit action]
2. [Step - explicit action]
3. [Step - explicit action]

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

## CONTEXT ENGINEERING - UNIVERSAL RULES

Applies to all platforms. Context engineering is now as important as prompt phrasing. Full
framework, with sourcing, in `references/context-engineering.md` - this section is a quick
reference only.

### The Lost-in-the-Middle Problem
Models show a U-shaped attention curve over long contexts (Liu et al., 2024, "Lost in the Middle:
How Language Models Use Long Contexts"):
- Tokens at the START receive high attention weight
- Tokens at the END receive high attention weight
- Tokens in the MIDDLE receive degraded attention weight, on a gradient that worsens as context
  grows rather than at any single fixed token count (see references/context-engineering.md for the
  correction of an earlier, unsourced "past 3,000 tokens" claim that lived here)

**Solution: Placement-by-function**
- START of prompt: Role, identity, background context, reference documents, persona definition
- END of prompt: Active task instructions, output specification, quality gate, activation phrase
- BOTH START AND END: Critical constraints and non-negotiable rules

### Context Rot at Scale
Even 1M+ token models show declining practical recall as input length grows (Hong, Troynikov, and
Huber, 2025, "Context Rot: How Increasing Input Tokens Impacts LLM Performance," Chroma). This
skill previously stated a specific "~64K tokens" cutoff here; that figure was unsourced and has been
removed. The Chroma study found degradation is continuous and varies by model and task, not a fixed
threshold, and found no single model that resisted degradation best across every task tested. KV
cache eviction policies can also remove less-recently-used tokens in long windows, which compounds
the effect in production systems.

**Mitigation strategies:**
1. Retrieval-first - use RAG rather than dumping full documents into context
2. Hierarchical summarization - summarize earlier context before adding new
3. Explicit attention markers - use "CRITICAL:", "IMPORTANT:", or section headers to re-anchor attention after long reference blocks
4. Focused injection - retrieve only the specific relevant chunk, not entire documents

### Memory Management for Multi-Turn
Standard Chat Completions APIs do not persist internal reasoning traces between turns.

- **Claude:** Include all relevant prior context in each system message
- **OpenAI o-series:** Use Responses API with `previous_response_id` to maintain cognitive trace
- **All others:** Manually thread conversation history; summarize older context before it exceeds practical limits
