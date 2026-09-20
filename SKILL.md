---
name: loop
description: "Create or update scheduled work in Codex. Choose a Scheduled Task for independent runs or a Scheduled Message for follow-ups that need the current conversation, then define the work, cadence, reporting, stopping, and input rules. Use for recurring work, monitoring, and scheduled continuation, not programming loops."
---

# Loop

Turn a request into a working schedule with the right context and a durable prompt. Infer details from the conversation and available project or schedule metadata. Ask only for missing information that materially changes the workflow.

This skill sets up or updates a schedule. A scheduled run should execute the saved work, not invoke this setup workflow and create another schedule.

## First decide where each run belongs

Ask internally: **Could a new conversation perform the next run correctly from the saved prompt and explicitly available inputs alone?**

| Answer | Choose | What each run needs |
| --- | --- | --- |
| Yes | **Scheduled Task** | A self-contained prompt and accessible sources or project files. Each run can start fresh. |
| No | **Scheduled Message** | The current conversation's evolving progress, decisions, previous findings, or unresolved questions. Return to this conversation. |

Use the user's names, Scheduled Task and Scheduled Message, when explaining the choice. The app or tools may call the latter a heartbeat or a scheduled task inside a chat.

Stable instructions discussed here can be included in a standalone prompt; their origin in this conversation does not make the work context-dependent. Similarly, comparing against a baseline in an accessible file or service can work with fresh runs. When the required baseline, decisions, or progress live only in this conversation, choose a Scheduled Message. Do not introduce an external state store merely to force fresh runs.

Honor an explicit destination preference. Otherwise, state the inferred choice and the concrete reason in one sentence. If the dependency is unclear and changes the route, ask one focused question about the needed context rather than asking the user to understand scheduler internals.

## Fill only the material gaps

Infer each of these before asking. They are decision criteria, not a mandatory questionnaire.

| Question | Infer or establish |
| --- | --- |
| **What should Codex do each time?** | The target, sources, time window, permitted actions, and expected result. Resolve references such as "this PR" or "the report" from the conversation. Ask only when the target or scope remains ambiguous. |
| **How often should it run?** | The interval or calendar schedule, relevant time of day, and timezone. Reuse an established cadence and the known user or runtime timezone. Ask when missing timing would change the usefulness of the result; do not invent an arbitrary polling frequency. |
| **What change is important enough to report?** | An observable event or threshold, such as completion, a failure, a new actionable finding, or a decision needed. For a monitor, default to reporting meaningful changes and staying quiet on unchanged or non-actionable checks. A requested recurring report or reminder should still be delivered as requested. |
| **When should it stop?** | A completion condition, terminal failure, deadline, run limit, or explicit cancellation. Infer the natural end of a finite follow-up. An ongoing service such as a daily briefing can continue until stopped; say so. Ask when the distinction between finite and ongoing work is unresolved. |
| **When should it ask me for input?** | Missing access or essential data, an unresolved choice that changes the result, a requested action outside existing authorization, or a user-specified escalation condition. Continue routine work within the agreed scope without repeated confirmation. |

When questions are needed, bundle only the unresolved ones in concise, task-specific language. Use an asynchronous input tool when available and continue independent preparation. Do not ask the five questions again when the conversation already answers them. State material assumptions briefly; wait for an answer when it is necessary to choose or authorize the workflow.

## Write the prompt that will actually run

Save clear, cohesive prose that describes the work itself. Include the resolved target and sources, what to inspect or do on each run, how to recognize a reportable result, when to stop, and when to request input. Keep the timing in the schedule fields except when a time window or deadline affects the work.

- **Scheduled Task:** Make the prompt self-contained. Include necessary instructions and concrete links, identifiers, paths, or skill names. Do not rely on "as discussed above" or assume the originating conversation or prior run will be available. If comparing against previous results is necessary, identify the accessible baseline and how it will be maintained.
- **Scheduled Message:** Describe the ongoing objective and what progress to resume. Use the current conversation's latest state and corrections; recheck changing facts at the source. Carry forward prior findings and pending input so the same unchanged result or unresolved question is not reported repeatedly.
- **Reporting:** Define what evidence qualifies as meaningful and what the report should contain. Suppress repeated alerts for an unchanged condition unless reminders were requested. A failed check is not evidence that nothing changed; report an actionable failure or request the missing input.
- **Stopping and input:** Check the stop condition before starting new work. When it is met, use the scheduler's supported pause or stop operation for the matching schedule, and report the outcome once. Merely writing "done" does not disable future runs. Ask once per unresolved issue and suspend dependent actions; pause the schedule if no useful work can continue until the user replies. Resume only after the blocking input is resolved.
- **Repeated actions:** For work that changes external state, check whether the intended action already succeeded before repeating it. If the result of a write is uncertain, inspect the destination before retrying and ask for input when it cannot be established. A monitoring request alone does not authorize unrelated edits or messages to other people.

Keep reportability rules in the saved prompt. Keep app-level mute or unmute preferences in the scheduler's notification setting, following the active tool schema.

## Create or update the schedule

Discover the current `automation_update` tool first and read its schema. Use its supported scheduling workflow; do not emulate a schedule with sleeping loops, shell cron, or handwritten automation directives. Do not show raw recurrence rules to the user.

The current Codex app tool maps Scheduled Task to `kind: "cron"` and Scheduled Message to `kind: "heartbeat"`. Follow the active tool's destination and authorization requirements. If it requires explicit standalone intent and the conversation establishes only a generic recurring request, prepare the concrete Scheduled Task prompt and timing first, then ask the single destination question needed by that tool. Explain that the requirement comes from the scheduling tool. Do not silently replace a context-dependent message with a standalone job.

For a Scheduled Task, use `list_projects` when the tool needs a project and resolve the intended project from context. Do not invent project identifiers. For a Scheduled Message, attach it to the current conversation using the tool's supported default or a verified identifier. Preserve existing model and reasoning settings for updates; use available defaults where supported rather than asking unnecessary configuration questions.

When updating existing scheduled work, identify the matching schedule by its name, prompt, and destination. With the current desktop tool, inspect `$CODEX_HOME/automations/*/automation.toml` (defaulting to `~/.codex` when `CODEX_HOME` is unset) to resolve its ID, then use the tool to view and update it. Preserve unrelated fields and prefer updating the matching schedule over creating a duplicate. Resolve an ambiguous match before changing it.

Review whether the saved prompt can execute with the selected context, sources, tools, and stop rules. A small read-only dry run can resolve a real uncertainty; do not perform extra external actions just to test scheduling. Once the scheduling request and necessary details are established, create or update it without adding a blanket approval step. If the user asked only for a draft, deliver the draft.

Check the tool result. Distinguish an active schedule from a suggestion awaiting user action or a failed creation. If the required scheduling capability is unavailable, provide the completed prompt and human-readable schedule, explain the specific limitation, and say that it has not been scheduled. Do not substitute a different destination without authorization.

Finish briefly with the chosen type, what it will do, its cadence and timezone, what it will report, and when it will stop or ask for input. Include the returned schedule link or native result when available.

## Examples of the routing decision

- "Every weekday at 9 AM, start a new task summarizing the last day's commits in this project." → **Scheduled Task**. The project and saved reporting instructions are sufficient for each run.
- "Check this deployment every ten minutes until it finishes, using the troubleshooting decisions we made here." → **Scheduled Message**. The next check continues the current investigation. Report completion, failure, or required input, then stop or pause as appropriate.
- "At 8 AM each day, generate a fresh status report from the project tracker and its saved baseline." → **Scheduled Task**. The comparison state is available outside this conversation.
- "Keep checking this." → Infer the target and context dependency from the conversation; ask only for unresolved timing, reporting, stopping, or input rules that change the workflow.

For current product behavior or changed tool terminology, consult the [official scheduled-work documentation](https://learn.chatgpt.com/docs/automations). The active tool schema determines which operations are available in this environment.
