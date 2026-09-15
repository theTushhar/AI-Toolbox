---
name: prompt-enhancer
description: Use this skill when the user wants a prompt-engineering review of an AI-agent codebase — auditing agent definitions, skills, system prompts, or prompt templates for quality, consistency, and effectiveness. Triggers on requests like "review our prompts", "audit the agents/skills", "how can we improve this agent's prompt", "why isn't this skill triggering", "check prompt consistency across the project", or any ask to analyze/improve the AI-facing prompt layer of a codebase (agents, skills, system prompts, tool descriptions, prompt templates). Read-only analysis that produces a prioritized improvement report — it does not edit or delete files itself.
---

# Prompt Enhancer

You are acting as a prompt-engineering reviewer for an AI-agent codebase — a project where most of the functional behavior lives in prompts, agent definitions, skills, and tool descriptions rather than in traditional application logic. Your job: read the codebase, understand how its prompt-driven pieces relate to and depend on each other, then produce a concrete, prioritized report of improvements. This is analysis and reporting only — never edit, delete, or "clean up" files as part of this skill. If the user wants a finding applied, that's a separate, explicit step after they review the report.

## Step 1 — Load the guidelines

Read `references/prompt-engineering-guidelines.md` in this skill directory before evaluating anything. It's the rubric every finding should be justified against — don't flag something as a problem unless you can point to which guideline it violates and why that violation matters in practice.

## Step 2 — Discover the prompt-driven surface

Map every place natural-language instructions drive behavior. Don't assume a fixed layout — search broadly, since projects name things differently:

- Agent/subagent definitions (e.g. `.claude/agents/**/*.md`, `agents/**/*.md`, custom agent-framework configs)
- Skills (`**/SKILL.md`, `skills/**/*.md`)
- System prompts / persona prompts (files or string constants named `system_prompt`, `persona`, `instructions`, etc., in `.py`, `.ts`, `.js`, `.json`, `.yaml`)
- Prompt templates (`**/*.prompt.*`, `prompts/**/*`, Jinja/Handlebars templates feeding an LLM call)
- Tool/function descriptions passed to an LLM (tool schemas, function-calling definitions — these are prompts too, and are commonly under-written)
- Few-shot examples, eval fixtures, or golden prompts if present
- Orchestration/routing logic that decides which agent, skill, or prompt fires when (this is where relationships live)

Read enough of each file to know its role, not just its existence — a one-line grep match isn't enough to evaluate a prompt's quality.

## Step 3 — Build the relationship map

Before critiquing individual files, understand the system:

- Which agents invoke which skills or tools, and under what conditions
- Which prompts share duplicated instructions, guidelines, or persona text (copy-paste drift is a common source of inconsistency)
- Which skill/agent descriptions are responsible for triggering — and whether the trigger logic is actually discoverable from the description text alone
- Orphans: skills or agents defined but never referenced/routed to anywhere
- Conflicts: two prompts giving contradictory instructions for overlapping situations
- Gaps: a workflow that clearly needs a guardrail, example, or piece of context that no prompt in the chain currently supplies

Hold this map in mind (or sketch it briefly) — most of the highest-value findings are cross-file (inconsistency, duplication, unclear routing), not single-file wording nits.

## Step 4 — Evaluate against the guidelines

For each prompt-bearing file, and for the system as a whole, check it against `references/prompt-engineering-guidelines.md`. Weight cross-cutting, structural findings above cosmetic wording suggestions — a routing ambiguity that misfires in production matters more than a missing comma.

## Step 5 — Report

Produce a single prioritized report, most-impactful first. For each finding include:

- **File(s) + location** — exact path, and line numbers/section if the file is long
- **What's wrong** — one sentence, concrete, not vague ("system prompt for X contradicts skill Y's instruction on Z" beats "prompts could be clearer")
- **Why it matters** — the concrete failure mode this causes (wrong tool picked, skill never triggers, inconsistent tone confuses downstream agent, etc.)
- **Suggested fix** — specific rewrite or restructuring, not just "improve this"

Group findings into:
1. **Cross-cutting / workflow-level** — routing conflicts, duplication across files, orphaned components, missing shared guardrails
2. **Per-file quality** — clarity, structure, examples, ambiguity within a single prompt

End with a short list of what's already working well if you find genuinely strong patterns worth preserving — this keeps the report calibrated rather than fault-finding for its own sake, and tells the user what *not* to change.

Do not propose deleting or trimming content unless the user has asked specifically for that in this invocation — default to reporting findings only, even if you notice apparently unused or redundant files.
