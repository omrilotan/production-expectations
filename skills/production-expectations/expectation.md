# The expectation

Every piece of information below is required, in this form or in the project's own. The values are an example: use the project's names.

```js
export default {
	name: "paid_order_has_shipment",
	// What is wanted, and who is hurt when it fails. No implementation words.
	means: "A customer paid and nothing is on its way to them.",
	// One number from production. Here: orders paid more than 15 minutes ago
	// that have no shipment. The 15 minutes is a grace period.
	count: (db, now) => unshipped(db, now - 15 * MINUTE).length,
	expected: 0,
	// Ids of the rows that fail, for the alert's evidence.
	sample: (db, now) => unshipped(db, now - 15 * MINUTE).map((order) => order.id),
	every: "15 minutes",
	// The id the release tool reports, or the one you made. Until you have it, "next".
	change: { release: "r42", flag: "fast_checkout" },
	// An agent, or a person until there is one.
	alerts: "repair-agent",
	may_do_alone: ["turn the flag off", "revert the release"],
	must_ask_first: ["release new code", "change production data"],
	bring_a_person_when: ["still failing after one repair", "no evidence explains it"],
	// How long the harm in `means` may stand before a person is woken, in the owner's local time.
	can_wait: "until 09:00",
	// Who may change this. Never the agent it alerts.
	owner: "Dana",
	approved_by: "none",
};
```

## Writing one

- **Write `means` first, from the request.** If you cannot say who is hurt, you do not yet know what the feature is for.
- **Prefer a count that should be 0** over a rate. It works at any traffic, and one failure is visible.
- **Check from data the product already keeps.** Do not add a table, a column or a log of people so that an expectation can count. If it cannot be checked without new stored data, say so and ask.
- **`approved_by` stays "none"** until a person or a different agent has read it. The agent that wrote the code is the wrong one to judge its expectation.

## Decisions that are not yours

`can_wait`, the bounds and the owner are the business's to decide. If the request does not give them and nobody can be asked now, write your proposal marked as one (`can_wait: "proposed: until 09:00"`), and list it under open decisions in the record. Do not write a guess as if it were decided, and do not wake someone to ask.

- **Bounds are the project's dial.** Start narrow. The owner widens them as repairs prove right.
- **`can_wait` governs waking only.** The expectation still runs and still fails in the record. If the project's only route to a person wakes them, propose the route and do not wire it.
- **When nobody is on call**, `may_do_alone` is all that happens until the owner next looks. Ask of each expectation whether anything in its `may_do_alone` would reduce the harm in `means`. Turning a new feature off rarely does: it puts back the state the feature was built to end. When nothing would, the failure waits for the owner. List that as an open decision, and propose the wider bound that would help, such as "release a fix that has a failing test" or "run the job again". Do not write the wider bound as decided. `can_wait` then says how long the owner may take to look, not when to wake them.
- **When the worst failure cannot be watched** (a job that deletes the wrong row leaves nothing to count), say so under open decisions. Do not add stored data to make it countable.
- **When every switch causes the failure it guards** (turning off a deletion job breaks the promise to delete), leave the switch out of `may_do_alone`, list what is safe instead, such as running the job again, and say why.

## Rows from before the change

- **Run it against production before the change is switched on**, once its migrations are applied. Rows that already fail are a finding for the record.
- **Scope it to what the change is responsible for.** Start it from the release, using the release time the platform records, so it begins at its expected value and a new failure shows.
- **Scope the code the same way.** New code does not act on rows from before its release unless the request says so. A retry that sweeps up last week's failures sends last week's messages.
- **You do not repair old rows on your own.** That is a change to production data. The exception is a request that asks for exactly that, such as a clean-up job: the request is the yes. Say in the first lines of your report what will be changed or deleted, and when. Its expectation then counts old rows too, after one interval of the job from the release.

## When your change makes an existing expectation wrong

A change that alters what is wanted (preorders ship on the release date, not at once) leaves the old expectation failing for ever. Do not edit it yourself. Draft the new wording, get its owner's yes, and ship the two together. If the owner cannot be asked, do not switch the change on.

## Scheduled work

An outcome expectation with a grace period catches a job that has stopped. Add a record that the job ran, and an expectation on it, only when nothing else would notice. Whether the runner itself is alive has to be checked from outside it.

## The runner

If the project has none: one command that runs every expectation, prints one JSON line each (`at`, `name`, `value`, `expected`, `ok`, `sample`), appends the line to a run log, and exits non-zero when any fails. It runs at the shortest `every`. The run log is what gives an alert its "failing since".
