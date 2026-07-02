# Prompting Claude Fable 5: Official Anthropic Guidance

Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5
(fetched 2026-07-02). The quoted blocks below are Anthropic's own recommended prompt language,
verbatim. Use them as drop-in modules when engineering a Fable 5 prompt. **Apply what fits, not all of it.**
Everything here applies equally to Claude Mythos 5 (same underlying model).

## What Fable 5 is for

- Takes on problems previously too complex, long-running, or ambiguous for prior models.
  Best results come from assigning it your *hardest* unsolved problems; testing only on
  simple workloads undersells it. It still performs reliably on straightforward tasks.
- Improvements over Opus 4.8: long-horizon autonomy (multi-day runs), first-shot correctness
  on complex well-specified problems, vision on dense/technical images, enterprise output
  (financial analysis, spreadsheets, slides, docs), code review and debugging recall
  (outside the cybersecurity domains the safety classifiers cover), navigating ambiguity,
  and dispatching/managing parallel subagents.
- NOT for offensive cybersecurity or biology/life-sciences work: safety classifiers can
  return `stop_reason: "refusal"`, and benign work near those domains can also trigger them.
  For API pipelines, configure fallback to Claude Opus 4.8 for declined requests.
- API notes: adaptive thinking only, summarized-only thinking output, no extended thinking
  budgets, and a `refusal` stop reason.

## Effort is the primary dial

Effort trades intelligence vs latency vs cost. **`high` is the default for most tasks;
`xhigh` for the most capability-sensitive work; `medium`/`low` for routine work.** Lower
effort on Fable 5 still often exceeds `xhigh` on prior models. Reduce effort if a task
completes but takes longer than necessary, or for a quicker interactive feel.

## Longer turns by default

Hard tasks can run many minutes per request at higher effort; autonomous runs can extend
for hours. Adjust client timeouts and progress UX; prefer checking on runs asynchronously
over blocking. To keep Fable 5 from overplanning on ambiguous tasks:

```text
When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.
```

## Anti-overengineering (higher effort can over-deliver)

```text
Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements: do the simplest thing that works well. Avoid premature abstraction and half-finished implementations. Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.
```

## Strong instruction following: brief beats enumerated

One short instruction steers a whole behavior class; you do not need to list every case.
This is the core migration insight: **prompts written for older models are usually too
prescriptive for Fable 5 and can degrade output.** Cut before you add.

Brevity / lead-with-outcome module:

```text
Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did you find": the thing the user would ask for if they said "just give me the TLDR." Supporting detail and reasoning come after. Being readable and being concise are different things, and readability matters more.

The way to keep output short is to be selective about what you include (drop details that don't change what the reader would do next), not to compress the writing into fragments, abbreviations, arrow chains like A → B → fails, or jargon.
```

Checkpoint module (when to stop and ask):

```text
Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide. If you hit one of these, ask and end the turn, rather than ending on a promise.
```

## Ground progress claims (long runs)

Nearly eliminated fabricated status reports in Anthropic's testing:

```text
Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.
```

## State the boundaries (unrequested actions)

Fable 5 can occasionally take unrequested actions (drafting an email nobody asked for,
defensive git-branch backups). Define what it should and should not do:

```text
When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state (restarts, deletes, config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
```

## Parallel subagents

Fable 5 dispatches subagents more readily than prior models. Encourage delegation, say
when it's appropriate, and prefer async communication over blocking on each subagent.
Long-lived subagents that keep context across subtasks save time and cost:

```text
Delegate independent subtasks to subagents and keep working while they run. Intervene if a subagent goes off track or is missing relevant context.
```

## Memory system

Fable 5 performs particularly well when it can record and reference lessons from prior
runs. A Markdown file is enough:

```text
Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.
```

Bootstrap from history:

```text
Reflect on the previous sessions we've had together. Use subagents to identify core themes and lessons, and store them in [X]. Make sure you know to reference [X] for future use.
```

## Anti-early-stopping (autonomous pipelines)

Deep into long sessions Fable 5 can rarely end a turn with a statement of intent ("I'll
now run X") without the tool call, or ask permission it doesn't need. Pair this with the
checkpoint module above so the model knows when pausing IS appropriate. For autonomous runs:

```text
You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking "Want me to…?" or "Shall I…?" will block the work. For reversible actions that follow from the original request, proceed without asking. Offering follow-ups after the task is done is fine; asking permission after already discussing with the user before doing the work is not. Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ("I'll…", "let me know when…"), do that work now with tool calls. End your turn only when the task is complete or you are blocked on input only the user can provide.
```

## Context-budget reassurance

In very long sessions Fable 5 can suggest a new session or trim its own work, usually
when the harness shows a remaining-token countdown. Avoid surfacing counts; if you must:

```text
You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits. Continue the work.
```

## Give the reason, not only the request

Fable 5 performs better when it understands intent; context lets it connect the task to
relevant information instead of inferring intent. Template:

```text
I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request].
```

## Readability addendum (agentic / long conversations)

```text
Terse shorthand is fine between tool calls (that's you thinking out loud, and brevity there is good). Your final summary is different: it's for a reader who didn't see any of that.

If you've been working for a while without the user watching (overnight, across many tool calls, since they last spoke), your final message is their first look at any of it. Write it as a re-grounding, not a continuation of your working thread: the outcome first, then the one or two things you need from them, each explained as if new. The vocabulary you built up while working is yours, not theirs; leave it behind unless you re-introduce it.

When you write the summary at the end, drop the working shorthand. Write complete sentences. Spell out terms. Don't use arrow chains, hyphen-stacked compounds, or labels you made up earlier. When you mention files, commits, flags, or other identifiers, give each one its own plain-language clause. Open with the outcome: one sentence on what happened or what you found. Then the supporting detail. If you have to choose between short and clear, choose clear.
```

## send_to_user tool (long async agents)

For agents whose UX needs verbatim mid-task messages (deliverables, numbers, direct
answers), define a client-side tool; tool inputs are never summarized:

```json
{
  "name": "send_to_user",
  "description": "Display a message directly to the user. Use this for progress updates, partial results, or content the user must see exactly as written before the task finishes.",
  "input_schema": {
    "type": "object",
    "properties": {
      "message": { "type": "string", "description": "The content to display to the user." }
    },
    "required": ["message"]
  }
}
```

Defining it is not enough; pair with elicitation language, and don't route narration
through it:

```text
Between tool calls, when you have content the user must read verbatim (a partial deliverable, a direct answer to their question), call the send_to_user tool with that content. Use send_to_user only for user-facing content, not for narration or reasoning.
```

## Scaffolding recommendations

- **Start at the top of your difficulty range.** Assign harder tasks than you would to
  prior models; have Fable 5 scope, ask clarifying questions, and execute.
- **Make self-verification explicit in long-run prompts.** Fresh-context verifier
  subagents outperform self-critique: `Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.`
- **Refactor existing prompts and skills.** Older, over-prescriptive instructions can
  degrade Fable 5 output; try removing them and compare against default behavior.
- **Never instruct Fable 5 to echo, transcribe, or explain its internal reasoning in the
  response.** That can trigger the `reasoning_extraction` refusal category and elevated
  fallbacks. If reasoning visibility is needed, read summarized `thinking` blocks from
  adaptive thinking and use send_to_user for progress.
