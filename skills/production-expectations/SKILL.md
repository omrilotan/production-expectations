---
name: production-expectations
description: Use when building, changing or releasing a feature in software that agents run in production, when adding logging, monitoring, alerting, checks, dashboards or rollback to it, when a production alert or failing check arrives and nobody is watching, or when setting a project up for any of these.
---

# Production expectations

## Overview

Code says what the software does. Telemetry says what happened. Neither says what was wanted. An **expectation** says what a feature is for, in the business's terms, as a number production can be checked against. It alerts when it fails, bounds what the alerted agent may do, and tells it when the repair has worked.

Software built this way monitors and repairs itself, under expectations a person sets. Assume it runs unattended unless the project's notes say otherwise: nobody watches production and nobody is on call. The agent an expectation alerts acts within its bounds, and a person reads the record afterwards.

Passing tests show the code does what you wrote. They do not watch production. A change with no expectation has nothing and no one to notice when it goes wrong.

- **First time in this project?** If it has no expectations or checks, or its notes for agents do not say where they live, read `adopting.md` in this directory. When you arrived with a task, do only the adoption that task needs.
- **Writing or changing an expectation?** Read `expectation.md`.
- **An alert or a failing check arrived?** Read `repair.md` before acting.

## Which changes need one

A change needs an expectation when it changes what the software does for someone: a new outcome, a new promise, a new scheduled job, a changed rule.

It needs none when behaviour stays the same: a rename, a refactor, wording, styling, documentation, a dependency update. Write "no expectation: behaviour unchanged" in your report and move on. When one release bundles several changes, judge each alone.

Write one expectation for each outcome the request asks for. Add another only for the worst thing the change could newly do wrong, and only if it can be checked from data the product already keeps.

## The project's own form wins

If the project already has checks, monitors or runbooks, add to them in their shape. Do not build a second system beside them. What this skill asks for is information, not a file format: where the project's shape has no place for something (the bounds, who is alerted), put it where the project keeps such things, such as the runbook. Leave a shared runner as it is, and record what it lacks.

## A change that needs one is done when it has all six

| # | Required | Check |
| --- | --- | --- |
| 1 | **Expectation** | Written before the code, from the request |
| 2 | **Outcome event** | One structured event each time the feature achieves what it is for, and one when it declines to |
| 3 | **Switch** | A flag or a release id, so the change can be turned off without a diagnosis |
| 4 | **Bounds** | What the alerted agent may do alone, and what it must ask for |
| 5 | **Alert route** | Something runs the expectation on its schedule, and a failure reaches the agent it names, or a person through a route the project already has |
| 6 | **First run** | Run against production before the change is switched on and after, with both values recorded |

**"Not done" is a statement, not a stop.** When the project cannot give you one of the six (nothing runs on a schedule, a failure reaches nobody), do what the request asks, release if it says to, and make the first line of your report "Not done: " and the gap. Write `alerts: "none yet"` and who has to set it up. Never name an agent or a route that does not exist, and do not wire up a route that wakes someone unless you were asked to: propose it.

A route that exists and reaches nobody is a gap too: an address that goes to no inbox, a channel nobody reads. Row 5 is not done. The gap goes in the first line even when the report mentions it further down.

Proposed values and `approved_by: "none"` do not make a change "not done". List them as open decisions.

**With no traffic since the release**, the two values in row 6 show only that the runner works. Rehearse the path on a copy of production as well, and say which of the two you did. A rehearsal never writes to production's run log.

**When you are not allowed to read production**, give the owner the exact read-only command, counts only, and record the value as not run. Do not look for another way in.

## Output

- One JSON object per event: `event`, `level`, ids, `release`, `flag`, and the trace or request id.
- Name events by outcome (`shipment.queued`, `shipment.skipped`), with ids and counts. No names, addresses or message text. The project's privacy rules outrank anything here.
- Every release has an id that events carry. If the release tool gives none, make one before you build, and write it and the time of the release in the record.
- Other agents read this stream. Write events so one can be understood without the code.

## Alerts

- Only three things alert: a failed expectation, a missed run, and a new unhandled error. Everything else stays in the stream.
- An alert carries its evidence: the name, the value and the expected value, `means`, failing since, the last release and flag, and sample ids.
- A person is brought in by the expectation, with a view built for that incident. No standing dashboard is needed for this.

## The record

Run values, findings and open decisions go in the project, where its notes for agents say, and are committed with the change. Your final reply is not a record. The next agent will not read it.

Say in your report whether what now runs in production is committed.

## Common mistakes

| Mistake | Instead |
| --- | --- |
| "Tests pass, released" | Tests are not an expectation. Add one, and run it against production |
| "The first real confirmation will be the next payment" | Then something has to check the next payment. That is the expectation, on a schedule |
| An expectation for a rename or a typo | "No expectation: behaviour unchanged" |
| A new table or log of people, added so an expectation can count | Check from data the product already keeps, or ask |
| A second system beside the project's own checks | Add to theirs |
| Found bad rows, mentioned them in the reply | Write them in the record |
| Named an alert target that does not exist | `alerts: "none yet"`, and report the gap first |
| Expectation restates the code ("createShipment was called") | State the outcome ("a paid order has a shipment") |
