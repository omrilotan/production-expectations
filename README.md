# Production expectations

A skill for coding agents that build software to monitor and repair itself, under expectations a person sets.

**Status: a draft.** I use it with my hobby projects. Read the [limits](#limits) before relying on it.

## The idea

Code says what the software does. Telemetry says what happened. Neither says what was wanted.

An expectation says what a feature is for, in the business's terms, as a number production can be checked against: "a paid order has a shipment" is a count of paid orders without one, and the count should be 0. Every feature ships with at least one. When it fails it alerts an agent. It says what that agent may do alone and what it must ask a person for. When it is back at its expected value, the repair is finished.

A person writes or approves the expectation, and the agent it alerts may not change it.

The skill assumes the software runs unattended: nobody watches production and nobody is on call. My hobby projects have no operations team and no on-call shifts. They have to take care of themselves.

## What is in the skill

| File | Read when | What it holds |
| --- | --- | --- |
| [`SKILL.md`](skills/production-expectations/SKILL.md) | Building, changing or releasing a feature | The six things a change needs before it is done, what the software outputs, and what alerts |
| [`expectation.md`](skills/production-expectations/expectation.md) | Writing an expectation | The form, with every field required |
| [`repair.md`](skills/production-expectations/repair.md) | An alert or a failing check arrives | Evidence first, one switch, the bounds, repairing rows apart from code, the five ways a repair ends, the record to leave, when to wake someone, and what to do when nobody can be woken |
| [`adopting.md`](skills/production-expectations/adopting.md) | The first time in a project | Eleven things to settle once, the first expectations, and the section to leave in the project's notes for agents |

An expectation, as the skill asks for it:

```js
export default {
	name: "paid_order_has_shipment",
	means: "A customer paid and nothing is on its way to them.",
	count: (db, now) => unshipped(db, now - 15 * MINUTE).length,
	expected: 0,
	sample: (db, now) => unshipped(db, now - 15 * MINUTE).map((order) => order.id),
	every: "15 minutes",
	change: { release: "r42", flag: "fast_checkout" },
	alerts: "repair-agent",
	may_do_alone: ["turn the flag off", "revert the release"],
	must_ask_first: ["release new code", "change production data"],
	bring_a_person_when: ["still failing after one repair", "no evidence explains it"],
	can_wait: "until 09:00",
	owner: "Dana",
	approved_by: "none",
};
```

`may_do_alone` and `must_ask_first` are the dial. A project starts narrow and widens them as repairs prove right.

## Using it

In Claude Code, install it as a plugin:

```
/plugin marketplace add omrilotan/production-expectations
/plugin install production-expectations@omrilotan
```

For other agents, copy `skills/production-expectations/` into the skills directory your agent reads. For Claude Code that is `~/.claude/skills/` for every project, or `.claude/skills/` inside one.

A symbolic link to a clone of this repository works too, and keeps the skill current. The project's formatter then leaves the skill's files alone, and the project's own repository holds a link, not the text.

The first time in a project, ask the agent to set the project up to work with the skill and to build nothing yet. It follows `adopting.md`.

## Licence

[Unlicense](LICENSE). Public domain.
