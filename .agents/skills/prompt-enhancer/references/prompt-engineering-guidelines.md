# Prompt Engineering Guidelines

Rubric for evaluating prompts, agent definitions, skills, and tool descriptions in an AI-agent codebase. Use these as the justification for every finding — cite the specific guideline a file violates rather than flagging vague "could be better" issues.

## 1. Clarity and directness

- Instructions should be explicit and unambiguous. Vague guidance ("be helpful", "write good code") produces inconsistent behavior — flag it and suggest the concrete rule it should have been.
- Say what to do, not just what to avoid. A prompt full of "don't"s without the corresponding "instead, do X" leaves a gap the model fills unpredictably.
- One instruction, one place. If the same rule appears reworded in three files, that's a maintenance hazard, not thoroughness — flag it as duplication, not redundant reinforcement.
- Front-load the most important constraints. Models weight early and late context more heavily than the middle of a long prompt ("lost in the middle").

## 2. Structure

- Long prompts should use clear sectioning (headers, XML-style tags, or numbered steps) so the model can locate the relevant instruction instead of treating the whole prompt as an undifferentiated blob.
- Keep a consistent structural convention across sibling files (all agent definitions formatted the same way, all skills using the same section order). Inconsistent structure across similar files is itself a finding.
- Separate stable identity/role instructions from task-specific or turn-specific instructions, so the former doesn't need to be rewritten every time the latter changes.

## 3. Examples (few-shot)

- Concrete examples of desired input/output resolve ambiguity that abstract instruction cannot. A prompt describing a nuanced judgment call (tone, format, edge-case handling) without a single example is a strong candidate for improvement.
- Examples should cover edge cases and the boundary between "do this" and "don't do this," not just the happy path — a single easy example teaches less than one hard one.
- Watch for stale examples that no longer match current instructions elsewhere in the same file (a sign the prompt was edited without updating its examples).

## 4. Reasoning and chain-of-thought

- For tasks requiring judgment (classification, multi-step decisions, tool selection), check whether the prompt gives the model room/instruction to reason before answering, versus forcing an immediate terse output that skips deliberation.
- Reasoning instructions should specify *what to reason about* (relevant factors, tradeoffs) — "think step by step" alone is weaker than naming the actual decision criteria.

## 5. Role and context framing

- A defined role/persona should be doing real work — shaping tone, priorities, or domain assumptions — not decorative ("You are a helpful assistant" that has no bearing on subsequent instructions).
- The prompt should give the model enough *why* behind constraints that it can generalize to unstated edge cases, not just a checklist it can only follow literally.

## 6. Agent and skill descriptions (routing/triggering)

This project's agents and skills are selected largely by their description text, so description quality is a first-class concern, not metadata:

- The description must state *when* to use the thing, in concrete trigger terms (phrases, situations, keywords a router or model would actually match against) — not just what it is.
- Equally important: state when *not* to use it, or how it differs from a similarly-named sibling, if overlap exists. Two agents/skills with similar descriptions and no differentiation will misroute.
- Descriptions that are accurate summaries but poor triggers (e.g., describe internals instead of invocation conditions) are a common, easy-to-miss failure mode — check specifically for this.
- If a skill/agent is never referenced by any routing logic, description, or documentation, it's effectively unreachable — flag as an orphan (report only; don't remove it yourself).

## 7. Tool and function descriptions

- Tool descriptions are prompts read by the model, not just interface documentation for humans. Vague parameter descriptions or missing usage guidance cause misuse (wrong args, wrong tool chosen) just as much as a bad system prompt does.
- Check that tool descriptions state constraints (when a param is required, valid ranges/formats, side effects) explicitly rather than assuming the model will infer them from a name.

## 8. Consistency across the system

- Tone, terminology, and formatting conventions should match across prompts that a user or model will encounter as part of one continuous experience. Drift (one agent formal, another casual; inconsistent terms for the same concept) reads as incoherence.
- Shared constraints (safety rules, output format rules, escalation rules) that are copy-pasted across many files should be flagged as candidates for centralization — the risk is that a future edit updates one copy and not the others, leaving contradictory instructions live simultaneously.
- Watch for two files giving genuinely contradictory instructions for the same situation — this is a critical-severity finding since behavior becomes nondeterministic (order/model-dependent).

## 9. Length and signal density

- Longer isn't better. Padding a prompt with caveats, hedges, or restated instructions dilutes the signal of the instructions that actually matter and wastes context budget.
- Prefer trimming/tightening suggestions over additive ones when a prompt is already covering the needed ground but verbosely.
- Boilerplate that doesn't change model behavior (generic disclaimers, restated obvious facts) is a candidate to flag, even though this skill doesn't remove it itself.

## 10. Guardrails and failure modes

- Check whether prompts that can take consequential or irreversible actions (deletions, sends, external calls) have explicit confirmation/scoping instructions, or whether that's left implicit and hoping the model infers caution.
- Check for missing handling of the "unhappy path" — what the prompt says to do when its assumptions don't hold (ambiguous input, tool failure, missing context) versus only specifying the golden path.

## 11. Testability and evaluation

- A prompt with no way to tell if it's working (no examples of success/failure, no stated acceptance criteria) is harder to improve safely later — note this as a structural gap where relevant, especially for prompts governing high-stakes or frequently-changed behavior.
