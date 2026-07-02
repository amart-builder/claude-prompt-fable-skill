# prompt-fable

A Claude Code skill that turns a messy brain-dump into a prompt, system prompt, or agent spec engineered specifically for Claude Fable 5, using Anthropic's official Fable 5 prompting guide.

Type `/prompt-fable` with your rough idea. The skill figures out what you're really asking for, engineers the prompt from Anthropic's official directive modules, shows it to you for approval, then runs it (or hands you the artifact if it's destined for another harness).

## Why a Fable-specific version

Anthropic's [official guidance for prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) has one core insight: Fable 5 needs less prompt, not more. One brief instruction steers a whole class of behavior, and over-prescriptive prompts written for older models actively degrade its output. So this skill subtracts before it adds, then composes from a small menu of official modules based on what you're building:

- **One-shot task prompt**: lean, most modules off.
- **System prompt for an interactive agent**: behavior modules (brevity, checkpoints, boundaries).
- **Autonomous long-run spec**: autonomy, verification cadence, progress-grounding, memory.

The reference file stores Anthropic's recommended prompt snippets verbatim (effort levels, grounding progress claims, anti-early-stopping, subagent delegation, the send_to_user pattern, the reasoning-extraction refusal trap, and more), so the skill builds from the source instead of paraphrasing it from memory.

## Install

Copy the folder into your Claude Code skills directory:

```bash
git clone https://github.com/amart-builder/claude-prompt-fable-skill.git
mkdir -p ~/.claude/skills/prompt-fable
cp -R claude-prompt-fable-skill/SKILL.md claude-prompt-fable-skill/reference ~/.claude/skills/prompt-fable/
```

New Claude Code sessions will pick it up automatically. Invoke with `/prompt-fable`.

## How it works

1. **Understand and classify.** Reads your dump, works out the real goal, and classifies the artifact (one-shot prompt, agent system prompt, or autonomous-run spec).
2. **Clarify only if it changes the output.** At most one round of questions, with sensible defaults pre-selected. A clear dump passes straight through.
3. **Engineer from the official playbook.** Opens with the why (the larger task, who it's for, what the output enables), picks only the modules the artifact class needs, and cuts anything Fable 5's defaults already cover.
4. **Show it and wait.** The full prompt or spec goes in a code block, every time, and nothing runs until you approve. This gate is the point: you get a reusable asset and you see how it was built.
5. **Run and self-check.** After approval it runs the prompt, checks the result against your original intent, and fixes anything off before showing you.

## Files

- `SKILL.md`: the workflow and the module menu.
- `reference/fable-5-official-guidance.md`: Anthropic's Fable 5 prompting guidance with their recommended prompt language quoted verbatim, with a source link.

## Attribution

The directive snippets in the reference file are Anthropic's own recommended prompt language, quoted from [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5). The workflow around them is original.
