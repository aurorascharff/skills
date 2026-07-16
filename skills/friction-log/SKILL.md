---
name: friction-log
description: Document agentic developer experience friction encountered during a development flow — captures where agents get stuck, which also surfaces DX issues for developers. Use when asked to "log friction", "document a pain point", "write a friction log", or when the agent hits confusion, dead ends, unclear docs, or unexpected blockers. Produces a structured friction log in markdown.
license: MIT
metadata:
  author: aurorascharff
  version: '1.0.0'
---

# Friction Log

Document agentic developer experience friction as it's encountered during a development task — friction that blocks agents also surfaces DX issues for developers. Treat friction logging as equally important as task completion.

## Instructions

When invoked:

1. **Identify the output file** — use the argument if provided. Otherwise, name the log after the task with the `fl-` prefix: `fl-<feature>.md`. If this run scaffolds a new `fl-<feature>/` folder (see #7), the log lives **inside** that folder as `fl-<feature>/fl-<feature>.md`. If no scaffold was created, write `fl-<feature>.md` in the current working directory. Keeping the log next to the code it describes makes the artifact self-contained: zipping or sharing the `fl-` folder ships the log with it. The `fl-` prefix makes the file recognisable as a friction log even when it has been moved out of its folder.
2. **Identify the task** from the user's first message. The user's first message IS the task. Do not ask clarifying questions before starting unless you are genuinely blocked.
3. **Identify the input** — if the task involves an existing codebase, include it as **Input** in the log header. Use the URL the user provided; otherwise use the local path.
4. **Record the prompt** — write the user's initial request verbatim into the log before doing any work.
5. **Record the model and harness** — at the top of the log, record the model name (e.g. Claude Sonnet 4.6) and the harness you are running under (e.g. Claude Code, Cursor, DX Agent). Only ask the user if both are genuinely unknown to you.
6. **Read any URLs in the prompt first** — if the user's prompt contains URLs (GitHub repos, documentation links, reference files), read their content BEFORE starting the task. Do not skip URLs or silently proceed without reading them.
7. **Name any scaffolded repo after the task** — if the task involves creating a new app or repo, use a `fl-` prefix followed by the feature or API being tested (e.g. `fl-catcherror`, `fl-instant-nav`). The prefix makes clear the repo was created as part of a friction log run. Do not use generic names like `my-app` or `test-app`. Write the friction log file **inside** this folder using the same prefix (`fl-<feature>/fl-<feature>.md`), not in the parent directory.
8. **Log friction as you encounter it** — write to the file in real time, not at the end. Each friction entry must say what you expected, what actually happened, what you tried, and how it resolved (or that it didn't). The resolution is as valuable as the friction itself.
9. **Do not initiate questions to the user mid-run.** Push uncertainty into the log as friction with the right source tag (`[web search]`, `[docs]`, `[training data]`, `[sandbox]`) showing what you fell back on, and keep working. Only stop and ask if you are truly blocked AND your harness supports interactive prompts — most agentic harnesses do not. When in doubt, write the friction down and keep going.
10. **Watch for out-of-band context the harness may inject mid-run.** Some harnesses surface user messages typed into the running session (chat-thread replies, queued Slack messages, etc.) as system messages at the start of a turn. Treat each item as either:
    - An **instruction** ("also check headers", "use Turbopack instead") — incorporate it into the current run and adjust any in-progress log entries to reflect it.
    - A **direct question** ("did you check the headers?", "what version are you on?") — answer it concisely via whatever reply mechanism your harness provides (e.g. a `slack_reply`-style tool), BEFORE continuing other work.
    Log the exchange verbatim at the point in time it arrived, using `> **User:** …` and `> **Agent:** …` blockquotes.
11. **Cite your sources** — every log entry must end with a source tag indicating how you obtained the information. See [references/agent-behavior.md](references/agent-behavior.md) for the full list.
12. **Action Items are the most important part of the log.** Every friction point (🟡 or 🔴) MUST produce at least one action item. Do NOT leave placeholders like `[To be filled]` — write concrete, specific action items. Each item MUST have an indented `Context:` line below it with the specific friction encountered. Split them into three subsections:
    - **Docs** — fixable with better documentation, clearer callouts, or updated examples. `🔧` prefix.
    - **Framework** — requires a code change: error messages, warnings, scaffold defaults, tooling, agent-layer infrastructure. `🔧` prefix.
    - **DX / Research** — open questions or investigations worth pursuing. `🔍` prefix.
13. **Build failures are friction, not stopping points.** When a build or test fails, log it as a 🔴, read the error output carefully, try to fix it, and rebuild. A failed build is often the most valuable part of the log. Cap retries — three attempts is usually enough; if it still fails, write what you learned and move on. Do NOT brute-force the same failing command.
14. **Track cumulative build time.** If the harness exposes per-build duration (or you can time it cleanly), keep a running total across all build attempts and write it in the header as `**Build time:** Xs`. This captures total time lost to builds — more builds from bad DX means higher build time. If you cannot time builds, omit the field rather than guess.
15. **Tool timeouts are friction signal.** If a tool returns `timedOut: true` (or its equivalent), log it as 🔴, **do not retry the same command**, and switch tools or approach. Repeated timeouts on the same command are pure budget waste.
16. **When done, write the log, then `cat` it to stdout, then stop.** After the file is finalized, print its full contents (e.g. `cat fl-<feature>.md` or `cat fl-<feature>/fl-<feature>.md`) so the user sees the log inline in the terminal — do not just point at the file path. You may also mention that the log can be pasted into <https://agent-friction-skill.vercel.app/> for a collapsible, severity-coded view. Do not ask the user to review it, do not offer to deploy, do not ask follow-ups. **Do NOT add an empty `## Skill Feedback` section** — an empty placeholder is uglier than no section. Only append a `## Skill Feedback` section *later*, if the user reviews the log and explicitly points out a place where the skill itself caused you to behave incorrectly (use `🔁` prefix). This is not DX feedback; it is feedback for improving the skill. If the failure was caused by the task environment, it belongs in Action Items instead.
17. **The final log MUST always include every required section from the template** — Header, Prompt, Tool Timeline, Summary, Action Items (Docs + Framework + DX / Research), and Log. No exceptions. Even if the build never succeeded, write a Summary of what happened and Action Items based on the friction you observed. An incomplete log with a missing Summary or missing Action Items is a failure. `Skill Feedback` is **optional and added later** (see #16) — never include an empty version.
18. **Write the log progressively — NEVER use placeholders.** Do NOT write the full template skeleton with `[To be filled]`, `[pending]`, `[TBD]`, or empty sections. Instead:
    - At the START: write only the header (Date, Model, Harness, Task) and the Prompt section.
    - DURING the task: append Log entries as you encounter them. Rewrite the build-time field after each build.
    - At the END: write the Summary and Action Items based on everything you observed, then rewrite the full file with all sections filled in.
    - If you are running low on context or steps, STOP working on the task and immediately write the complete final log with Summary and Action Items based on what you have so far. A complete log with fewer log entries is far better than an incomplete log. Never end the log mid-sentence or without Summary + Action Items — these are mandatory regardless of how the run went.
19. **Always write a `## Tool Timeline` section.** Place it directly after `## Prompt`, before `## Summary`. List your tool calls chronologically as `- HH:MM:SS — <tool>: <short description>`. Write it yourself even if your harness also appends one — the log is read after the run, often pasted into a viewer that doesn't have access to harness-side appendices, so the timeline must be inside the file itself. Keep entries terse; this section is reference material, not narrative.

Follow [references/template.md](references/template.md) for the exact output format and [references/example.md](references/example.md) for a real example. Behavior guidance — when to log, what counts as friction, source tags — lives in [references/agent-behavior.md](references/agent-behavior.md). [references/reading-the-log.md](references/reading-the-log.md) explains how a human should read the finished log (severity legend, action-item priority, source-tag trust levels).

## Severity Levels

Use these consistently throughout the log:

- 🟢 Smooth — worked as expected
- 🟡 Minor friction — extra steps, guesswork, or searching required
- 🔴 Major friction — blocked, broken, missing, or deeply confusing
