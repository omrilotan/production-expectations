# Adopting this in a project

Read the first time you work in a project that has no expectations, or whose notes for agents do not say where they live. Do it once. The result is a short section in the project's notes for agents (`CLAUDE.md`, `AGENTS.md`, or the README if it has neither), so no later agent decides any of this again.

## Settle these, in the project's own terms

| Question | Look for | If there is nothing |
| --- | --- | --- |
| **Where do expectations live?** | Existing checks, monitors, health routes, runbooks. Use them | `expectations/`, one file each |
| **What runs them on a schedule?** | The platform's scheduler: a cron trigger, a scheduled CI job, a worker | Record the gap. Do not invent a scheduler the platform lacks |
| **Where does a failure go?** | An agent that can be triggered, an issue tracker, the owner's email | `alerts: "none yet"`, and who has to set it up |
| **Who is the owner?** | The person who approves changes to what is wanted | Ask. This cannot be guessed |
| **Where is the record?** | The project's docs | `docs/expectations.md` |
| **What is a release id, and where does the time of a release come from?** | Version metadata, a deploy log, a commit | Make ids and record them |
| **What can be switched off without a release?** | Flags, config, routes | Reverting is the only switch. Say so |
| **What may never be stored or logged?** | The project's privacy and security rules | Ids and counts only |
| **Can an alert be acknowledged?** | A way to stop repeats while the expectation keeps running and failing | Record the gap. Without it a disputed alert repeats until its owner decides |
| **Where does the owner look first?** | A channel, an inbox, a file | Ask |
| **Is anybody on call?** | A pager, a rota, any route that wakes a person | Nobody is. Write that in the notes, and put the choice to the owner: with narrow bounds, harm stands until they next look. Ask what an agent may do alone when harm keeps growing and nobody can be reached |

Ask the owner what you cannot find. Propose the rest and mark it as a proposal.

A route that wakes a person, such as a pager, is not yours to wire up. Propose it.

## When you arrived with a task

Settle only what the task's own expectation needs: where expectations live, the runner and its schedule, the record, and the section in the project's notes. Then do the task. The first expectations for the rest of the product are a separate job. Propose it in the record and do not start it.

## Then write the first expectations

Not one for every feature. Write one for each thing the product exists to do: the two or three promises that, if broken quietly, would matter most. Take them from the README, the public promises (privacy, terms) and the owner. Run each against production and record the values. Rows that already fail are findings for the owner, not rows for you to repair. Start each expectation from the day of adoption (`change: { release: "adopted <date>" }`), so it begins at its expected value.

## Releasing

Adoption does not release anything unless the request says so. Say when the schedule will start (often with the next release), and what the first scheduled run will report.

The run log is production's output: keep it out of version control. The values from a run you made by hand go in the record.

## Before launch

When the product is not live yet, write expectations for what runs today (a waiting list, a sign-up, a promise already published), and list in the record the ones to write with each feature still to come.

## What to leave alone

- Existing code that works. Adoption adds expectations. It does not rewrite features to emit events. If the request does ask for that, list every renamed event in the record: searches and alerts on the old names stop matching.
- Existing checks and runners. Add to them in their shape, and record what they lack.
- Anything the project's own rules forbid. They outrank this skill.

## The section to leave in the project's notes

```md
## Expectations

Every change to what this does for someone ships with an expectation (skill: production-expectations).

- They live in: …
- They run: … (schedule and what runs it)
- A failure goes to: …
- Owner: …
- The record and incidents: …
- Release ids and times come from: …
- Switches: …
- An alert is acknowledged by: …
- The owner looks first at: …
- On call: … (and what an agent may do alone when nobody can be reached)
- Never stored or logged: …
- Not set up yet: …
```

Finish by reporting what is not set up yet, first.
