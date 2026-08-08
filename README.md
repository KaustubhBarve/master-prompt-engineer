<div align="center">

# Master Prompt Engineer

**A Claude Agent Skill that turns a one-line idea into a complete, platform-calibrated prompt.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Skill version](https://img.shields.io/badge/skill%20version-3.6.0-blue)](./master-prompt-engineer/SKILL.md)
[![Format](https://img.shields.io/badge/format-Agent%20Skill%20(SKILL.md)-6f42c1)](https://agentskills.io)

[What it does](#what-it-does) ·
[Install](#installation) ·
[Usage](#usage) ·
[How it works](#how-it-works) ·
[Reference library](#reference-library) ·
[Sourcing](#sourcing-and-corrections) ·
[Structure](#repository-structure)

</div>

---

## What It Does

Give the skill a target AI and a rough idea. It returns a full prompt — the kind you paste cold into a fresh session and expect complete, structured output from, with no prior context required.

```
You:    "GPT-5 — a zero-shot customer support triage system"

Skill:  → Runs a 9-step diagnostic pass (domain, platform, scope, technique routing,
          production-pattern check, token-efficiency pass, build, self-critique)
        → Selects frameworks and techniques with a stated mechanistic reason for each
        → Applies GPT-5-specific calibration (zero-shot first, direct constraints,
          snapshot-pinning note for production use)
        → Architects context placement (reference data first, active instructions and
          the quality gate last)
        → Returns the prompt plus a short debrief explaining the decisions behind it
```

It is not a prompt template library. It is a fixed procedure for constructing a prompt: what to extract from the request, what to check it against, which techniques and frameworks to combine, and how to place them in context for a specific target model.

### What the skill is trying to do differently

Most "prompt improver" tools default to generic advice — be more specific, add examples. This one instead:

- **States a reason for every structural choice.** Why a constraint sits at the start of context rather than the middle; why o-series reasoning models need chain-of-thought stripped out rather than added; why Claude 4.x needs scope stated on every instruction rather than left implicit.
- **Calibrates per target model**, not generically. Claude's literal instruction-following, GPT-5's zero-shot strength, and Gemini's preference for the task question last are treated as different engineering problems, not variations on one template.
- **Attaches a source to its claims, or removes the claim.** Every technique with a performance figure traces to a specific paper. Where a cited number couldn't be verified against its source, it was deleted rather than left in — this shows up directly in the skill's own changelog (see [Sourcing and corrections](#sourcing-and-corrections)).
- **Treats single-shot prompts and agentic systems as different problems.** A LinkedIn post and a coding agent's system prompt need different layers of the library, and the skill routes between them rather than applying one structure to both.

---

## Installation

This repo is a **Claude Agent Skill** — a folder with a `SKILL.md` file that Claude loads and invokes automatically once installed. Pick the path that matches how you use Claude.

<details>
<summary><b>Claude Code</b> (filesystem-based)</summary>

```bash
# Personal — available in every project
git clone https://github.com/KaustubhBarve/master-prompt-engineer.git ~/.claude/skills/master-prompt-engineer

# Project-local — available only in this repo
git clone https://github.com/KaustubhBarve/master-prompt-engineer.git .claude/skills/master-prompt-engineer
```

Claude Code picks up the skill on your next session. No further setup needed.

</details>

<details>
<summary><b>claude.ai</b> (custom skill upload)</summary>

1. Download this repo as a ZIP.
2. **Settings → Capabilities → Skills → Upload skill**, and select the ZIP.
3. The skill is now available in your conversations on that account.

</details>

<details>
<summary><b>Claude API</b> (Skills API)</summary>

Upload the `master-prompt-engineer/` folder as a custom skill, then reference its ID in the container config alongside the code-execution tool:

```python
container={
    "skills": [
        {"type": "custom", "skill_id": "<your-uploaded-skill-id>", "version": "latest"}
    ]
}
```

Custom skills require the `code_execution` tool and the relevant beta headers. See Anthropic's [Agent Skills documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) for the current upload flow and header requirements, since these details change over time.

</details>

---

## Usage

Once installed, describe the target AI and the idea — no command syntax needed:

```
Claude Opus 4.8 — An AI that helps startup founders write investor pitch decks
o3 — A prompt for advanced financial modeling and scenario analysis
GPT-5 — A zero-shot customer support triage system
Claude Code — A Gauntlet Loop that builds a landing page and keeps iterating
              against real competitor screenshots until it wins the comparison
Universal — A master customer support AI with escalation logic
```

If you don't know the target model, say **`Universal`**: the skill builds a plain, portable structure with no vendor-specific tokens. If the target is unclear and you didn't say `Universal`, the skill asks one clarifying question before building.

Every response comes back in three parts:

1. **Meta-header** — what was built, which frameworks and techniques were combined, and why, in a few sentences
2. **The prompt** — the full, ready-to-copy output, clearly fenced
3. **Engineering debrief** — the key decisions behind it, tuning notes, failure modes to watch for, and a version-pinning note when production use is implied

---

## How It Works

### The core: Deconstruct → Diagnose → Develop → Deploy

| Phase | What happens |
|---|---|
| **Deconstruct** | Extracts intent, entities, the output contract, and the constraint surface — and defines what a *bad* output looks like, so there's a concrete negative target to check against |
| **Diagnose** | Audits the request for specific failure modes: flat vocabulary, constraints buried mid-context, missing contrastive examples, thin identity anchoring, platform mismatch |
| **Develop** | Selects frameworks (CO-STAR, RISEN, CRISPE, POWER, TRACE+BAB, ReAct, and others) and techniques (Chain-of-Draft, Step-Back, Constitutional Self-Critique, Meta-Prompting) with a stated reason for each |
| **Deploy** | Places reference data first and active instructions last, repeats critical constraints at both ends, and applies the target model's current calibration profile |

Every prompt is built against a fixed **6-component spec**: success criteria, output contract, constraints, inputs, examples, and a self-verification step — checked against a quality checklist before delivery.

### Beyond the core: four conditional layers

Single-shot content only needs the core above. Agentic, long-running, or system-prompt deliverables pull in additional layers as needed:

| Layer | Applies when | Source |
|---|---|---|
| **Tier 5 — Production Patterns** | The deliverable is an agent, persona, multi-turn assistant, or someone else's system prompt | A large community-maintained archive of circulated system prompts (274 files surveyed) — authenticity unverified, and flagged as such in the file itself |
| **Unknowns Discovery Protocol** | The build is agentic or in an unfamiliar domain, and needs elicitation before the spec locks in | An Anthropic blog post, verified by direct fetch |
| **Context Engineering** | The deliverable is long-running or agentic enough to outgrow a single context window | An Anthropic engineering blog post, verified by direct fetch |
| **Gauntlet Loop Prompting** | The deliverable is an autonomous coding-agent build meant to iterate against a real, inspectable reference until it clears the bar | An independent practitioner's post from July 2026, verified by direct fetch and explicitly flagged as recent and unofficial |

A fifth internal step, the **Meta-Prompting Self-Refinement Pass**, applies to higher-stakes drafts regardless of layer: before delivering, the skill silently re-reads its own draft as a critic would, names the one to three weaknesses that would actually degrade output, and revises only those — a lightweight version of the same conductor/critic pattern described in the meta-prompting research it cites.

### Platform calibration

| Target | Key behavior | What changes |
|---|---|---|
| Claude 4.x / Opus 4.8 | Interprets instructions literally, doesn't infer scope | Every instruction states its own scope explicitly; heavy XML structure |
| GPT-5 | Strong zero-shot inference | Zero-shot attempted first; direct constraints; snapshot-pinning noted for production |
| o-series / Extended Thinking | Reasons internally | External chain-of-thought is stripped entirely, not added |
| Gemini 2.x | Responds best to a few examples and a late task statement | Includes 1–3 examples; the task question is placed after context, not before |
| DeepSeek-V3/R1 | Strong reasoning, less consistent default prose | Tone, vocabulary, and cadence stated explicitly |
| Grok 4.x | Large context window, benefits from latitude | Outcome framed rather than prescribed step by step |
| Llama / open-source | Weaker instruction-following | Steps spelled out fully, with worked examples and exact templates |
| Universal | No fixed target | Plain numbered structure, no vendor-specific tokens |

`references/platforms.md` also covers roughly 20 specific named products beyond these broad families — Cursor, Perplexity, Le Chat, Notion AI, the Gemini CLI/Antigravity/Jules family, Copilot's variants, Qwen, ChatGPT persona presets, and coding-agent CLIs generally.

---

## Reference Library

`SKILL.md` is the operating core; the detail behind each layer lives in `references/`, loaded only when a given request actually needs it.

| File | Contents |
|---|---|
| **`techniques.md`** | The technique library, with a mechanistic explanation, a worked example, and a cited source for each — plus a Research Foundations table linking every performance figure to a specific paper |
| **`frameworks.md`** | Every named framework with worked examples: the 6-Component Spec, CO-STAR, RISEN, CRISPE, BAB, TRACE, CRAFT, POWER, RACE, Agile Prompting |
| **`platforms.md`** | Per-model calibration profiles for the current model matrix, plus the extended directory of ~20 named products |
| **`anti-patterns.md`** | Failure modes, an obsolescence list, production-brittleness patterns, and an adversarial-vulnerability catalog grounded in published research |
| **`production-patterns.md`** | Tier 5 — 27 operating-loop patterns for agents and system prompts (tool-trigger specs, ambiguity handling, plan-then-execute gating, sub-agent delegation, and more), plus a verified section on MCP, Agent Skills, and A2A |
| **`unknowns-discovery.md`** | The pre-build and mid-build elicitation layer: the Four Unknowns diagnostic and 8 named techniques for agentic or high-novelty builds |
| **`context-engineering.md`** | Anthropic's own context-management framework: context rot and the attention budget, system-prompt altitude, pre-fetch vs. just-in-time retrieval, and long-horizon techniques — plus a token-efficiency checklist applied to every generated prompt, not only agentic ones |
| **`gauntlet-loop.md`** | The Gauntlet Loop method for autonomous coding-agent builds: goal-not-implementation framing, an inspectable comparison bar, agent-owned decomposition, isolated builder/critic sub-agents, and a budgeted, open-ended round structure |

---

## Sourcing and Corrections

Claims in this skill are marked by how well they're sourced, and the marking is visible rather than smoothed over:

- Technique performance figures cite a specific arXiv paper. A figure that couldn't be traced to its source was removed rather than kept as a plausible-sounding estimate.
- The Unknowns Discovery Protocol and Context Engineering layer come from official Anthropic posts, verified by direct fetch.
- Tier 5's production patterns come from a large but unverified community archive of circulated system prompts — real structural conventions worth reusing, but not confirmed facts about any specific vendor's actual system, and the file says so.
- Gauntlet Loop Prompting comes from a single practitioner's post published weeks before this file was written — treated as a plausible, verified-by-fetch technique, not settled practice.

Where an earlier version of this skill got a citation wrong — including, at one point, a paper mixed up with a different paper of a near-identical title — the fix is recorded in `SKILL.md`'s changelog rather than silently edited away.

---

## Repository Structure

```
master-prompt-engineer/
├── LICENSE
├── README.md
└── SKILL.md/                             ← the skill itself (frontmatter + operating instructions)                         
    └── references/
        ├── techniques.md                 ← technique library + research foundations
        ├── frameworks.md                 ← named prompt frameworks, worked examples
        ├── platforms.md                  ← per-model calibration + product directory
        ├── anti-patterns.md              ← failure modes + adversarial vulnerability catalog
        ├── production-patterns.md        ← Tier 5: agentic / persona / system-prompt patterns
        ├── unknowns-discovery.md         ← pre-build elicitation protocol
        ├── context-engineering.md        ← agent context-window management
        └── gauntlet-loop.md              ← iterative build/critique loops for coding agents
```

---

## Design Principles

Ten rules govern every prompt this skill produces:

1. **The skill builds the machine, not the answer** — the output is a prompt for another AI to run, not a response to the request itself
2. **Structure first, depth by requirement** — a complete short spec beats a padded long one; this includes not padding with near-duplicate examples or unnecessary reasoning length
3. **Every structural choice is mechanistically justified** — grounded in how transformers process and attend to context, not habit
4. **Platform adaptation is mandatory** — no single structure serves every model
5. **Positive framing by default** — constraints are stated as what to do, not what to avoid
6. **Context is architected, not just written** — reference data first, active instructions last, critical constraints repeated at both ends
7. **Every prompt is self-contained** — built to work cold, with zero assumed prior context
8. **Scope is explicit on every instruction**, particularly for models that don't infer it
9. **Production resilience is built in** — specs are written to survive model version updates, with version-pinning flagged where relevant
10. **Every output explains itself** — the debrief states why a decision was made, not just what was built

---

## Versioning

The skill is on **v3.6.0**. Every version bump is documented inside `SKILL.md`'s YAML frontmatter, including the reasoning behind corrections when an earlier claim didn't hold up — see the [changelog](https://github.com/KaustubhBarve/master-prompt-engineer/blob/main/SKILL.md) at the top of that file for the full history back to v1.0.0.

---

## Contributing

Issues and pull requests are welcome, particularly:

- Verified research citations for existing or new techniques
- Additional platform or product calibration profiles
- Corrections to any claim that doesn't hold up against its cited source

If you're adding a performance claim, link the primary source — a paper or an official vendor post — rather than a secondary summary. The changelog is a record of claims that had to be walked back for exactly that reason.

---

## License

[MIT](./LICENSE) © Kaustubh Barve

</div>
