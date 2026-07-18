# Context Engineering: Curating the Agent's Working Memory

## Contents
- Source and Verification Status
- Context Engineering vs. Prompt Engineering (and why "prompts are becoming obsolete" overstates it)
- Why Context Is Finite: Context Rot and the Attention Budget
- The Anatomy of Effective Context
- Context Retrieval: Pre-Fetch, Just-in-Time, and Hybrid
- Three Techniques for Long-Horizon Tasks
- Integration With This Skill

---

## Source and Verification Status (read this first)

This file is built primarily from Anthropic's official engineering blog post "Effective context
engineering for AI agents," published September 29, 2025 on anthropic.com/engineering, written by
Anthropic's Applied AI team (Prithvi Rajasekaran, Ethan Dixon, Carly Ryan, Jeremy Hadfield). The
article was fetched and read directly for this update, not taken from a secondary summary. That
gives it the same provenance tier as references/unknowns-discovery.md: a named source on an
Anthropic-owned domain, not the unverified community archive behind Tier 5.

Two supporting sources, also checked directly rather than recalled from memory:
- Andrej Karpathy, post on X, June 25, 2025, the origin of the "context engineering" term this
  skill's identity block already cites. Repeated here because it is the direct predecessor to
  Anthropic's own framework.
- Kelly Hong, Anton Troynikov, and Jeff Huber, "Context Rot: How Increasing Input Tokens Impacts
  LLM Performance," Chroma Technical Report, July 2025 (research.trychroma.com/context-rot). This
  is the study behind the term "context rot," which Anthropic's post also uses.

I have not independently verified Gartner's July 28, 2025 research note beyond its public summary
page. I'm treating the headline claim as accurate since a Gartner-hosted page confirms the date and
wording, but I have not read the full paywalled report, so treat any granular statistic attributed
to Gartner elsewhere in this skill with proportionate caution.

**A correction to this skill's own earlier content.** Before this update, three places in this
skill (the Deconstruct step in SKILL.md, the KV Cache Prefix Engineering row in SKILL.md, and a
line in references/platforms.md) stated that middle-of-context attention loss becomes significant
"past 3,000 tokens." A fourth location (references/platforms.md) and a fifth (references/anti-
patterns.md) separately claimed degradation becomes significant "past ~64K tokens." Neither number
could be traced to an actual source while building this file. Both the Chroma report and Anthropic's
own post describe the degradation as continuous and dependent on model and task, not a cliff at any
single token count. All five locations have been corrected to the gradient framing used below. If a
future edit of this skill reintroduces a specific number, it should carry a citation next to it.

---

## Context Engineering vs. Prompt Engineering

Worth stating precisely, because the premise "prompts are becoming obsolete" gets it half right.
Anthropic's own post frames context engineering as a progression from prompt engineering, not a
replacement for it. Writing a clear, well-organized system prompt does not stop being real work. It
becomes one input that a larger discipline has to manage, alongside tool definitions, retrieved
documents, conversation history, and anything persisted to memory between turns.

Where the "obsolete" framing has a real point: for a single-turn task, prompt quality is close to
the entire game, and no amount of context architecture substitutes for an unclear instruction. But
once an AI is running in a loop, calling tools, accumulating results, and operating over an extended
session, the thing that determines output quality stops being "was the instruction phrased well" and
becomes "does the model's working memory, at this specific step, still contain the right subset of
everything that has happened so far." A perfectly worded prompt at turn one does not prevent that
working memory from filling up with stale tool output by turn forty.

This skill's identity block already carries Karpathy's framing for this: the LLM as a CPU, the
context window as RAM, prompt text as the lowest layer, and context architecture as the operating
system deciding what gets loaded into that RAM at each step. The addition this update makes is
turning that framing into something actionable rather than a memorable analogy: the sections below
describe what the operating system's job actually consists of.

Practically, for this skill's own output: a single LinkedIn post, newsletter section, or research
report is still substantially a Tier 1-4 prompt engineering problem, because nothing persists across
turns that needs curating. An agentic build, a coding assistant, or anything meant to run across a
session too long to fit in one context window is a context engineering problem on top of a prompt
engineering one, and the generated prompt needs to say so explicitly rather than assuming a good
instruction is sufficient on its own.

---

## Why Context Is Finite: Context Rot and the Attention Budget

Anthropic's post gives a structural reason context cannot be treated as free just because context
windows keep getting larger. Transformer attention relates every token in the context to every other
token, so the number of relationships the model is tracking scales with the square of the token
count, not linearly with it. Stretch that mechanism across a longer sequence and each individual
relationship gets a thinner slice of the model's attention. Training data compounds this: models see
far more short sequences than long ones during training, so they arrive with less specialized
capacity for reasoning across very distant parts of a context, independent of window size.

The Chroma study supplies the empirical half of this. Testing 18 frontier models, including Claude
4, GPT-4.1, Gemini 2.5, and Qwen3, on retrieval and reasoning tasks of varying length, the
researchers found accuracy declining as input length grew, on tasks as basic as locating a repeated
string, and found no single model that held up best across every task type tested. That last point
matters for this skill: it rules out picking one universal number and applying it across platforms,
which is exactly what this skill's retired "3,000 tokens" and "64K tokens" claims tried to do.

Anthropic's own phrase for the shape of this decline is worth carrying forward exactly, since it is
short, precise, and directly contradicts the false precision this skill previously stated: a
"performance gradient rather than a hard cliff." Capability does not switch off at a threshold. It
erodes gradually, unevenly across models, and more sharply on tasks that require synthesizing
information across distant parts of the context than on tasks that only need to locate one clearly
marked fact.

**What this means for prompt construction (the practical guidance is unchanged; the justification
underneath it is now correct):** reference material and identity or persona content belongs near the
start of a prompt, the active task instruction and quality gate belong near the end, and genuinely
critical constraints earn a repeat at both ends. That guidance survives this correction because it
responds to a real, well-evidenced tendency toward degraded mid-context attention, not because of any
specific token count this skill previously asserted.

---

## The Anatomy of Effective Context

Anthropic's operating principle across every component of context: keep only what is actually
earning its place, and treat "minimal" as a quality bar rather than a length target. A long,
well-justified system prompt can satisfy this. A short, vague one can fail it just as easily.

**System prompts and the right altitude.** The failure mode this skill's existing DIAGNOSE step
already names is under-specification: vague roles, thin identity anchoring, high-frequency words
that spread the model's probability mass too wide. Anthropic's post names the failure mode at the
opposite end explicitly, which this skill had not previously called out on its own: engineers
overcorrect into rigid, hardcoded branching logic trying to nail down every possible case in advance,
which becomes brittle and expensive to maintain as new edge cases surface. The target sits between
the two: concrete enough that the model has a real signal to act on, general enough that it can still
apply good judgment to situations the prompt did not anticipate.

**Tools.** A tool is a channel through which an agent pulls new information into its context at
runtime, which makes tool design a context budget decision, not just an interface decision. The
practical test Anthropic proposes: if the person designing the toolset cannot say with confidence
which tool a given situation calls for, the model calling those tools is unlikely to guess correctly
either. Overlapping or poorly scoped tools do not just risk the wrong call, they burn context on
resolving ambiguity the tool design should have removed in the first place. This connects directly to
the tool-trigger specification pattern already documented in references/production-patterns.md
(Tier 5); this section adds the reasoning for why that pattern matters beyond interface clarity.

**Examples.** Few-shot examples remain genuinely worth including, but Anthropic's post pushes back on
a specific overuse pattern: trying to cover every conceivable edge case by listing dozens of
near-duplicate examples. A small set of examples chosen to be genuinely different from each other,
each illustrating a distinct facet of the desired behavior, teaches the model more per token than a
long list that mostly repeats the same lesson with small variations.

---

## Context Retrieval: Pre-Fetch, Just-in-Time, and Hybrid

Anthropic's post describes two broad strategies for getting information into an agent's context, and
a third that blends them.

**Pre-fetch.** An embedding-based retrieval step runs before inference and loads relevant material
directly into the prompt. This is fast and predictable, but it commits to a guess about what will
matter before the agent has actually engaged with the task, which risks pulling in material that
turns out to be irrelevant, or missing material the pre-fetch step did not anticipate.

**Just-in-time retrieval.** Rather than loading data up front, the agent keeps lightweight pointers
(file paths, saved queries, links) and uses tools to pull the underlying data into context only at
the moment it is actually needed. Claude Code is the example Anthropic gives: it explores a codebase
with commands like glob and grep, and inspects large datasets with head and tail, rather than
loading entire files or full data objects into its window. This is closer to how a person actually
works with a large body of information: nobody memorizes an entire archive, they build an index (a
folder structure, a search tool, a set of bookmarks) and pull the specific item they need when they
need it. A useful side effect the post highlights: metadata alone often carries a signal before any
content is even read. A file called test_utils.py sitting inside a tests folder tells an agent
something different than a file with that same name sitting inside src/core_logic, purely from where
it lives.

**Hybrid.** Most agents built for production settings mix the two: enough pre-fetched up front to
start fast, with room to explore further at the agent's own discretion once it understands the task
better. Anthropic's post ties the right mix to how dynamic the domain is, and specifically names
legal and finance work, two domains that overlap heavily with this skill's own recurring output, as
better suited to leaning on pre-fetch, since the underlying reference material changes on a slower
cycle than, say, a live codebase.

---

## Three Techniques for Long-Horizon Tasks

These apply once a task's real token requirements exceed a single comfortable context window.
Anthropic frames the relevant range as work spanning tens of minutes to multiple hours of continuous
agent activity, using large codebase migrations and extended research projects as examples. This
skill's own multi-session builds (a full financial model, a hackathon build like Setu, a long-form
manuscript) sit in the same category.

**Compaction.** Before a conversation hits its context limit, its contents get summarized at high
fidelity and a fresh context window starts from that summary instead of the full history. Anthropic's
own implementation inside Claude Code keeps architectural decisions, open problems, and implementation
detail, and drops repetitive tool output that no longer needs to be seen in full. The skill worth
naming here: tune the summarization step by first checking it captures everything that mattered
across real, complex agent traces (favor over-including at first), then trim what turns out to be
noise once the pattern of what gets missed becomes clear, rather than guessing at the right cutoff
from the start. One of the lowest-risk places to start trimming is old tool call results themselves;
once a tool has already returned a result deep in the history, the agent rarely needs to see that raw
output a second time.

**Structured note-taking (agentic memory).** The agent writes notes to a location outside its context
window as it works, a running file, a dedicated memory store, and reads that record back in at later
points rather than relying on everything staying inside one continuous window. This buys persistent
memory at very little ongoing token cost. Anthropic's illustration is an agent playing Pokemon across
thousands of game steps: with no explicit instruction to keep notes, it develops its own running log
of training progress and a self-built map of explored territory, then reads that log back after a
context reset and continues a multi-hour session without losing the thread. For a prompt this skill
generates, this is the specific behavior to spell out whenever the deliverable is meant to keep
running longer than one context window comfortably allows.

**Sub-agent architectures.** Instead of a single agent carrying the full history of a project, bounded
sub-agents take on specific pieces of work inside their own clean context windows, potentially using
tens of thousands of tokens internally while they explore, and hand back only a distilled result
(Anthropic's post puts a typical figure at one to two thousand tokens) to whichever agent is
coordinating the overall task. This shares ground with Pattern 8 already documented in references/
production-patterns.md (Sub-Agent Delegation With Concurrency Safety), which focuses on when running
sub-agents in parallel is safe. What this section adds is the reason to delegate at all from a context
standpoint: the exploratory cost of a sub-agent's work never touches the coordinating agent's own
window, not merely its critical path.

**Choosing between the three:** compaction fits tasks that need to preserve a continuous back-and-
forth; structured note-taking fits iterative work organized around clear milestones; sub-agent
delegation fits research or analysis where genuinely independent threads can run in parallel. Nothing
prevents combining more than one inside a single build.

---

## Integration With This Skill

Two ways this layer applies, following the same pattern already established for the Unknowns
Discovery Protocol above.

1. **Into the generated prompt.** When Step 6 (Production Pattern Check) flags the deliverable as
   agentic, long-running, or explicitly multi-session, check whether the target agent's own context
   will realistically outgrow a single comfortable window during actual use. If it will, the
   generated prompt should name which of the three long-horizon techniques above the target agent
   should use, not just describe what the agent is supposed to accomplish. An agentic prompt that
   never addresses its own context management will degrade in exactly the way this file describes,
   regardless of how well the rest of the instruction is written.
2. **Into how this skill builds every prompt, agentic or not.** The "nothing without earning its
   place" principle and the system-prompt altitude framing apply to ordinary Tier 1-4 output too, not
   only to agent-specific work. Pair the existing DIAGNOSE step's under-specification checks (lexical
   flatness, thin identity anchoring) with an explicit check for the failure mode named here on the
   other end: rigid, over-engineered branching logic that a genuinely capable model does not need
   spelled out to that degree.

**When this layer is not worth reaching for:** single-session, single-shot deliverables (a
newsletter, a LinkedIn post, a standalone research report) rarely accumulate enough turns for
compaction or sub-agent isolation to earn their overhead. Applying this layer to a five-minute task is
the same over-engineering failure mode already flagged for Tier 5 and the Unknowns Discovery
Protocol.
