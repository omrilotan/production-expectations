# Repairing against an expectation

Read when an alert or a failing check arrives. The expectation that alerted you is your brief: it says what is wrong, what you may do alone, and when you are finished.

## In order

**Look for an open record first.** An alert repeats. If an incident file for this expectation is open, read it before anything else. When nothing has changed but the value, add the new value and the time to it, commit, and stop. Start again only when the evidence no longer fits the record, or its owner has decided something.

1. **Evidence first.** Start from the alert's value, a log event, a trace or a row. Reproduce the fault from it, and find the release that introduced it. That may be neither the last release nor the one the expectation names.
2. **Count everything the fault touched.** The alert's value is a floor. Rows still inside the grace period are affected too.
3. **Stop the damage with one switch, the narrowest, and only one the evidence points to.** Turn the flag off. With no flag, revert the faulty release, unless later releases would be lost with it. Never revert a release the evidence clears. This holds even when the expectation's bounds allow a revert: the bounds are an upper limit, and the cautions here still apply.
4. **Read the bounds.** Do what `may_do_alone` lists. Prepare what `must_ask_first` lists, and ask. If the expectation declares no bounds, these apply:
   - Alone: turn a flag off, revert the faulty release, re-run a check or a job, acknowledge an alert.
   - Ask first: release new code, change production data, anything that cannot be undone.
5. **Repair what is already wrong.** A revert restores the code, not the rows the fault wrote. The expectation keeps failing until those are repaired. That repair is a change to production data: follow the bounds. Prepare it as a script that lists by default and writes only when told to.
6. **Fix forward** with a test that fails without the fix. Release it only if the bounds allow, with a release id, from a commit. A fix inherits the expectation that caught it and needs no flag of its own.
7. **Check again**, and name how it ended (below).
8. **Leave the record** (below).
9. **If no expectation caught the fault**, add one with the fix.

Gaps you inherit (no scheduler, a runner with no run log, events with no release id) go in the record for the owner. You do not rebuild them during an incident.

## How it ends

| Ending | When | What you do |
| --- | --- | --- |
| **Cleared** | The expectation is at its expected value | Record and stop. A decision may still be open, such as turning the flag back on |
| **Waiting** | Damage is stopped, the rest is prepared, a person has a decision to make | Record and stop. With narrow bounds this is the normal end of a fault that wrote bad rows |
| **Blocked** | Damage is still growing and nothing in your bounds stops it | Wake a person now with one line and where the record will be, then prepare everything. When nothing wakes anyone, see below |
| **Unexplained** | The evidence shows what happened and not why, or shows nothing | No fix and no switch. Count, prepare the data repair, record. Wake a person if the value rises |
| **Disputed** | A release deliberately produces the wanted outcome later or differently than the expectation allows | Do not revert and do not edit. Acknowledge the alert, draft the change to the expectation for its owner, record |

**Blocked, and nothing wakes anyone.** When the project has no route that wakes a person, do not invent one, and do not widen your bounds because nobody is there to ask. Prepare everything. Open the record with the count that is still rising, how fast it rises, and the one decision that stops it. Leave a pointer where the owner looks first, and stop. The harm stands until the owner looks: say so in those words, and propose the wider bounds that would have let you stop it.

**Fault or dispute?** It is a fault when nothing will ever produce the outcome the expectation wants. It is a dispute when something will, on purpose, later or differently. A count that rises in a dispute is not new harm.

A cleared expectation proves the rows are repaired. It does not prove the fix while the flag is off and no traffic reaches the fixed code. Rehearse the fixed path on a copy of production, and say that is what you did.

## The expectation is not yours to change

Do not edit the expectation that alerted you: not its query, its expected value, its grace period, its bounds or its schedule, and do not delete or skip it. The same goes for the runner that checks it. An expectation loosened by the agent it alerts no longer says what is wanted.

- **Acknowledging is allowed.** If the project can acknowledge an alert, so that it stops repeating while the expectation keeps running and stays failing, you may do that alone. With no such thing in the project, the alert keeps repeating and the record says so.
- **You may draft a change** as a patch or a proposal for the owner. You do not apply it.
- **An owner's instruction counts when it says what the expectation should now say.** "Preorders are exempt until their release date" is the owner changing it: apply exactly that, record their words, and name them in `approved_by`. "Make it stop" and "do whatever it takes" say nothing about what is wanted. Acknowledge, and bring them the decision.
- If it is missing information, say so to its owner in the record and leave it. With no owner named, the owner is the person the project says to page.

## Bounds are not yours to widen

Bounds are the project's dial. Its owner widens them in the expectation as repairs prove right. You do not widen them in the moment because it is 03:00 and the fix looks safe.

## The record, and bringing a person in

One file per incident: where the project's notes for agents say, or `incidents/<date>-<expectation>.md`, dated by the clock the expectation runner uses. Commit it on the main branch. Prepared code, the fix and the repair script, goes on a branch, never in the tree the release tool ships. The command you give the person may run it from that branch. Build branches and copies of production in a temporary directory. The file is both the record for the next agent and the view for the person. In this order:

1. **The decisions needed**, in the order they should be applied: each as one question, its options, what each costs, and the exact command for the option you recommend. Describe the others. Mark the decision that clears the expectation.
2. **What failed**, in the words of `means`, how many are affected, and how it ended.
3. **The evidence**: the expectation's value against the releases, sample ids, the events or trace.
4. **What you did**, and what you prepared and did not do.
5. **The state you left**: flags, the release and its id, data, the expectation's value now, and the gaps you found.

## Waking someone

In this order:

1. **Wake now** when new cases are still arriving and nothing in your bounds stops them. This outranks `can_wait` and a general request not to be woken. With no route that wakes anyone, see "Blocked, and nothing wakes anyone" above.
2. **Do not wake** a person who already knows about this failure and has said to wait, unless it has got worse since they said so.
3. **Wake** when the harm will still stand past the time `can_wait` names.
4. Otherwise leave the file, and a pointer to it where the project's notes say its owner looks first, and stop.
