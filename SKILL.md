---
name: prompt-fable
description: >-
  Fable 5 edition of /prompt: turns a messy brain-dump into a prompt, system prompt,
  or agent spec engineered specifically for Claude Fable 5 using Anthropic's official
  Fable 5 prompting guide, shows it for approval, then runs it (or hands it off).
  Use when the user types /prompt-fable, asks for "a Fable prompt," wants a prompt or
  system prompt tuned for Fable 5 / Mythos 5, or wants the system prompt or spec written
  for an agent, long-running pipeline, or subagent that Fable 5 will power (not when they
  just want the agent built). Once invoked, run the full workflow
  including the approval gate; do not shortcut straight to doing the task. Do NOT
  auto-trigger on ordinary terse requests, and prefer plain /prompt when the target
  model isn't Fable.
---

# Prompt-Fable: engineer prompts for Claude Fable 5

Same spine as `/prompt` (**clarify → engineer → show and get the OK → run → verify**),
but the engineering step follows Anthropic's official Fable 5 guide instead of generic
technique. All official directive text lives in `reference/fable-5-official-guidance.md`;
read it before engineering. The guide's core insight: **Fable 5 needs less prompt, not
more.** One brief instruction steers a whole behavior class; over-prescriptive prompts
written for older models actively degrade its output. Subtract before you add.

## Workflow

### 1. Understand the dump and classify the artifact
Work out the real goal, deliverable, and audience. Then classify, because the Fable
playbook differs by shape:
- **A. One-shot task prompt** (write/analyze/build once) → lean prompt, most modules off.
- **B. System prompt / skill for an interactive agent** → behavior modules (brevity,
  checkpoints, boundaries) matter most.
- **C. Autonomous long-run spec** (pipeline, overnight run, subagent) → autonomy,
  verification, progress-grounding, and memory modules matter most.

### 2. Clarify (only if it changes the output)
Same rule as `/prompt`: ask only when a missing detail would change the artifact, via
AskUserQuestion, max 4 questions, one round, defaults as first options. Two extra
Fable-specific checks worth settling (by default or by question):
- **Effort level**, if the prompt runs via API or a harness that sets it: default `high`;
  `xhigh` only for capability-sensitive work; `medium`/`low` for routine volume work.
- **Refusal exposure**: if the task is anywhere near offensive security or bio/life
  sciences, warn that Fable 5's classifiers may refuse and recommend an Opus 4.8 fallback.

### 3. Engineer it from the official playbook
Open `reference/fable-5-official-guidance.md` and compose from its modules. Rules:

- **Give the reason, not only the request.** Every artifact opens with the why: the
  larger task, who it's for, what the output enables. (This skill's judgment call: it's
  the highest-leverage module, so it's the only always-on one.)
- **Right-size hard.** Pick only the modules the artifact class needs (menu below); a
  simple type-A prompt may need none. Never paste the whole reference in.
- **Brief beats enumerated.** Steer behavior classes with one short instruction; don't
  list every case. Delete any instruction Fable 5's defaults already cover.
- **Aim high.** If the dump undersells the task, say so: Fable 5 does its best work at
  the top of the difficulty range, scoping and executing hours-to-days-scale work.
- **Never ask it to show its reasoning in the response.** That can trigger the
  `reasoning_extraction` refusal. Reasoning visibility comes from adaptive-thinking
  summaries or a send_to_user tool, never "explain your thinking as you go."
- Keep what still applies from `/prompt`: XML structure for real inputs, explicit
  positive output format, escape clause, checkable "done," no fake-expert personas,
  no stakes/tips/threats, scan for contradictions.

**Module menu** (verbatim blocks in the reference; reference heading in parentheses):

| Module | A: one-shot | B: agent system prompt | C: autonomous run |
|---|---|---|---|
| Reason/context opener (Give the reason) | always | always | always |
| Act-don't-overplan (Longer turns by default) | if ambiguous | usually | always |
| Anti-overengineering | code tasks | code agents | code runs |
| Brevity / lead-with-outcome (Strong instruction following) | if long output | usually | in final report |
| Checkpoint, when to pause (Strong instruction following) | rarely | usually | always |
| Boundaries, assess vs act (State the boundaries) | rarely | usually | if it touches state |
| Ground progress claims | no | if long turns | always |
| Anti-early-stopping block (Rare cases of early stopping) | no | rarely | always |
| Subagent delegation (Parallel subagents) | no | if it has subagents | if it has subagents |
| Verifier-subagent cadence (Scaffolding recommendations) | no | rarely | always |
| Memory file instructions (Construct a memory system) | no | if recurring | if recurring |
| Context-budget reassurance (Rare cases of context-budget concern) | no | no | if harness shows counts |
| send_to_user tool + elicitation | no | no | if UX needs verbatim mid-run output |

### 4. Show it, get the OK, then run
1. One plain-English line on what it does and the key Fable-specific choices
   ("opens with the why, grounds progress claims, verifier subagent every phase").
2. The full prompt/spec in a code block: mandatory, every time. It is the asset being
   approved. If it's a system prompt destined for another harness, also note recommended
   effort level and any fallback advice next to the block, not buried in it.
3. **Stop and wait for approval.** End the turn: "Run this, or tweak it first?" Never
   skip the gate unless the user explicitly said to run without pausing.
4. On approval: if it's a prompt for *this* session, run it, silently self-check against
   the dump, fix, then show. If it's a system prompt/spec for another harness or agent,
   deliver the final artifact and where to put it.
5. Offer one refinement.

## Rules
- **Always show, always wait.** The approval gate is the skill.
- **Subtraction is a feature.** If the user hands over an existing prompt to "Fable-ify,"
  the main move is usually cutting over-prescription, then adding the few modules that fit.
- **Official text stays verbatim.** When a module from the reference applies, use
  Anthropic's wording as the base; adapt lightly, don't paraphrase from memory.
- **Right altitude.** Smallest high-signal set; no contradictions; positive phrasing.
