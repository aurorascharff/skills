# How to read a friction log

The output of this skill is structured so a reader can extract the most important findings in under a minute and drill into detail only where it matters.

For a rendered, collapsible view, paste the markdown into <https://agent-friction-skill.vercel.app/>.

## Read order

1. **Header** — date, model, harness, framework/version, build time, task. Confirms what was attempted, under what tooling.
2. **Prompt** — verbatim user request. Anchors the log: everything below describes the attempt to fulfil this.
3. **Tool Timeline** — collapsed by default. Skim only if you want to see the exact sequence of tool calls.
4. **Summary** — 2–4 sentences. What went well, the biggest pain point, blast radius. This alone often answers "is there anything I need to do?"
5. **Action Items** — the synthesized takeaways. Split into three buckets:
   - **Docs** (🔧) — fix with better documentation.
   - **Framework** (🔧) — requires a code change (error messages, scaffold defaults, tooling).
   - **DX / Research** (🔍) — questions or experiments worth running. Not yet actionable.
6. **Log** — chronological narrative. Open only when an action item is unclear and you need to see how the agent got there.

## Severity legend

| Marker | Meaning |
| ------ | ------- |
| 🟢 | Worked as expected. Confirms good DX — keep an eye on these as regression baselines. |
| 🟡 | Minor friction. Extra steps, guesswork, doc-hunting. Resolvable but slowed the agent down. |
| 🔴 | Major friction. Blocked, broken, missing, or deeply confusing. Highest-value entries — every 🔴 should have a corresponding action item. |

## Action-item icons

| Icon | Subsection | What to do |
| ---- | ---------- | ---------- |
| 🔧 | **Docs** | File a docs PR or rewrite the relevant guide. Usually the cheapest fix per unit of impact. |
| 🔧 | **Framework** | File a framework issue / open a PR. Typically error-message clarity, scaffold defaults, missing CLI flags, or missing machine-readable hooks (lockfiles, JSON output). |
| 🔍 | **DX / Research** | Open a question for follow-up. These aren't actionable yet — they're hypotheses worth validating before committing engineering time. |

Every action item must have an indented `Context:` line. If you find one that doesn't, the agent skipped a rule — flag it under **Skill Feedback**.

## Source tags

Every log entry ends with a tag indicating how the agent got the information. Tags help you judge how much to trust each line:

| Tag | What it means |
| --- | ------------- |
| `[agents.md]` | Read from a bundled `AGENTS.md` in `node_modules` or repo root. High trust. |
| `[docs]` | Read from official online documentation. High trust. |
| `[url]` | Fetched from a specific URL the user supplied or that the agent followed. Trust matches the source. |
| `[error output]` | Direct observation from a build error, runtime error, or command output. High trust — this is reality. |
| `[sandbox]` | Same as `[error output]` but from a sandbox tool in a hosted harness. |
| `[web search]` | Hit a search result. Medium trust; pages can be outdated. |
| `[training data]` | Agent relied on prior knowledge without verifying. Lowest trust — treat as a guess that needs confirmation, especially for any API past the model's cutoff. |
| `[skill]` | Read from a SKILL.md loaded for this run. High trust within that skill's scope. |

If a 🔴 entry has only `[training data]` as the source, treat the surrounding fix attempts as unconfirmed.

## What to act on first

In rough priority order:

1. Every 🔴 with a 🔧 Framework action item — these block real users.
2. Every 🟡 with a 🔧 Framework action item that mentions an *error message* — small text changes, high impact.
3. 🔴 / 🟡 with 🔧 Docs items — usually cheap to land.
4. 🔍 DX / Research items — schedule for a future investigation pass.

🟢 entries don't need action but are useful as regression baselines: if a future run downgrades a 🟢 to 🟡, something changed.

## Out-of-band exchanges

Some harnesses let the user interject mid-run (chat-thread replies, queued Slack messages). When that happens, the exchange is logged verbatim at the point in time it arrived, using blockquotes:

```markdown
> **User:** Did you check the response headers?
> **Agent:** Yes — they're below.
```

These mark places where the agent's path changed in response to a user nudge.

## Skill Feedback

`## Skill Feedback` is an optional section that does **not** appear in fresh logs — the agent is instructed not to include an empty placeholder. It gets appended only if the user, after reviewing the log, points out a specific place where the **skill itself** caused the agent to behave incorrectly: missing instructions, ambiguous rules, wrong default behaviors. The entries (🔁-prefixed) are inputs for improving this skill, not for the framework being tested.

If a problem was caused by the task environment (a misleading error message, a missing doc), it belongs in **Action Items**, not Skill Feedback.
