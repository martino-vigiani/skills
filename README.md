# skills

Claude Code skills and subagents pulled out of a working repo and cleaned up so they run somewhere else.

Four independent plugins. Install one, install all four, install none — nothing here depends on anything else here.

```
/plugin marketplace add martino-vigiani/skills
/plugin install core-agents@martino-vigiani
/plugin install optimize-app@martino-vigiani
/plugin install swift-quality@martino-vigiani
/plugin install writing@martino-vigiani
```

After the `add`, `/plugin marketplace` lists what's inside before you commit to anything.

## Read this before you install

**Most of it is written in Italian.** The agent definitions in `core-agents`, the `optimize-app` skill and its references, the two Swift agents, and `humanize` are all Italian prose. `persuade-mail` and the SwiftUI 27 reference are English.

That's a choice, not an oversight. These are written the way I'd say them, and the sharp lines are sharp because I wrote them in my own language. Translated, they'd read like every other skill pack. Claude reads Italian fine and answers you in whatever language you're working in, so in practice this affects you when you open the files to edit them, not when you use them.

## What they cost you

Skills and agents are not free: a description sits in the context window on every turn whether you use it or not. Numbers from `claude plugin details`:

| Plugin | Always loaded | When it fires |
|---|---|---|
| `core-agents` | ~469 tokens | agent context, only when you spawn one |
| `optimize-app` | ~380 tokens | ~5,200 tokens |

`swift-quality` and `writing` are in the same range. If you don't write Swift, don't install `swift-quality` — that's the whole reason these are four plugins instead of one.

## `core-agents`

Four general-purpose subagents: `fixer` and `fixer-quick` for edits, `explore` for read-only lookup, `verifier` for adversarial checking.

The part worth having is how `fixer` treats proof. It has an explicit ladder, top down:

1. A measurement it can take by itself — a command, a script, an HTTP request, a simulator screenshot.
2. A measurement that needs an environment but not a person — a simulator to configure, a network profile, seeded data. Costs more, still its job.
3. Proof that needs your hands. Last resort. If it gets there it has to **say why the first two weren't enough** and ask for one precise gesture, not "try it and tell me".

The common failure is jumping straight to 3 because it's easier. That spends a person's time to save the agent's, and the answer arrives hours later.

Two more rules on top. **The proof has to be able to fail**: break the thing the fix protects on purpose and show that something complains. If nothing complains you don't have proof, you have an impression. And **the report is always delivered** — failed, partial, stopped halfway, doesn't matter. Ending a run without one costs the caller a round trip.

Both fixers refuse to start without an explicit list of files they may touch and an explicit list they may not. If the fix turns out to need a file outside the list, they stop and say so instead of reaching for it. Neither ever commits.

`verifier` exists for the symmetric reason: an agent sent to "check" confirms almost every time. This one is sent to **refute**. It reads the real code rather than the snippet you pasted, hunts for the caller or the edge case that contradicts the claim, and returns CONFIRMED / REFUTED / PARTIAL with `file:line`. It's also told not to manufacture doubt: if it searched and found no counter-evidence, it says confirmed and stops.

## `optimize-app`

A performance pass run the way someone who pays the bill would run it.

**A number you didn't measure doesn't exist.** This skill ends in a table of percentages, which is exactly where lying is easiest — you estimate −30% because you removed a pointless decode and nobody checks. Three rules follow:

- **Baseline before touching anything.** No line of code changes until a measurement of the current state is written to disk, in `.perf/<slug>/baseline.md`, with commit hash, machine and device at the top. On disk, not in memory, because your context gets summarized long before the work ends.
- **Every row of the final table has two real numbers**, taken the same way on the same machine — or it says `not measured` and what it would take. Never an estimate dressed as a measurement.
- **A change that doesn't improve a number gets thrown away**, even if it's cleaner. Cleanliness is a different job.

Around that: measure each thing at least three times and record median and spread, because on a dev machine the noise is comfortably ±15% and one run can't tell an improvement from luck. Measure the real thing, not its stand-in — the simulator doesn't have the phone's GPU, localhost doesn't have the host's latency, a warm cache isn't a cold start.

There's an inventory step before any optimizing, and it has to record **what it didn't look at**. The classic failure is optimizing the same file three times because it was already open, and shipping a "complete" pass that never saw half the app.

Then rounds: audit with read-only subagents on disjoint areas, rank by measured cost over risk, apply **one change at a time and re-measure**. Five changes and a 20% drop tells you nothing about which of the five worked. Stop when two consecutive rounds move nothing by 5%, or when what's left is smaller than the measurement noise, or when the next step is a decision that isn't yours to make.

The report has a **Not done** section that is never omitted, and if the total is small it says so. "−8% on startup, the rest was already fine" is a legitimate result. References with concrete commands for iOS/SwiftUI, web/PWA, and PHP/Node backends. It doesn't commit — the branch is the handoff.

## `swift-quality`

`swift-test-expert` and `e2e-tester`, plus a SwiftUI reference for iOS 27.

Both agents know the trap that wastes the most time: **a scheme with no test action makes `xcodebuild test` exit 0 without running anything**, and it looks green. A run with zero tests executed is not proof. They check the executed count and say so when it's zero.

`swift-test-expert` starts from one idea: a suite exists to tell the truth about the code, and any shortcut that turns it green without making the code right destroys it silently — it keeps running, keeps passing, protects nothing. So it will not loosen an assertion, stretch a timeout, or comment out an `#expect` to get a pass. When the red is the code's fault, the output is the **bug**, with `file:line` and the sequence that produces it, plus a proposed fix it does not apply. When the red is the test's fault, fixing it is the right work — and the report never mixes the two cases. A test it pauses gets named out loud, because a silently skipped test is indistinguishable from a passing one.

`e2e-tester` picks the paths itself. That's the point of it: an agent handed a list of tests to write is a typist, and choosing what deserves an end-to-end is the hard part. It ranks candidates by whether the user notices the breakage before you do, whether it broke once already (it greps the fix commits), whether it crosses a boundary — network, disk, cache, database, state surviving a relaunch — whether it's irreversible, and how many states actually interact. Pure logic gets sent back to unit tests, and it says so rather than pretending that's a downgrade.

Both do **mutation testing** on everything they write: break the protected code on purpose, confirm the test goes red, restore. A test you haven't seen fail you haven't verified. They also flag the specific false positive where an end-to-end stays green with the rule it protects disarmed, because the path produces the same side effect some other way — that test gets replaced, not kept because it runs.

Neither commits, neither touches production code except for the mutation, which is always restored, and both check `git diff --stat` at the end to prove only test files moved.

The `swiftui-whats-new-27` skill is Apple's published SDK 27 guidance, not mine — I packaged it. It's here for one thing it catches: `@State` became a macro in SDK 27, views that compiled before now fail with "used before being initialized", and the obvious fix (reordering the init assignments) compiles and is wrong at runtime.

## `writing`

`humanize` rewrites text so it sounds like a person instead of a model. Style only, never content: same facts, same numbers, same structure, same register, same language. It works out the type and the language on its own, keeps a formal client email formal, doesn't warm up technical documentation, and never swaps accented characters for ASCII. The rewrite runs on a Sonnet subagent against a rules file; the orchestrating model checks what comes back for lost facts and re-runs if it finds any.

`persuade-mail` writes a mail that moves the reader — cold outreach, or convincing a client on price or scope. It opens on the reader's situation, proves value with checkable specifics, and gives one clear ask. The humanize pass runs in the same shot.

The rule I'd keep even if you take nothing else from it: **never invent proof**. The dangerous fabrication isn't the wild claim, you'd catch that one. It's the plausible derived number — a day count you reasoned your way to, a percentage that "must be roughly right". Those slip past because they feel like inference. So when the argument rests on numbers, they get pulled from the source first and tagged with where they came from, and anything unconfirmed stays a visible placeholder.

For anything that moves real money there's an optional loop: one subagent plays the recipient in character and reports whether they'd accept or counter and with what figure, another hunts arrogance, self-pity, salesman prose and contradictions, and checks the arithmetic the reader will do. Re-judge with a **fresh** persona each round — one that has already read a version rates the next on improvement instead of merit.

## What you won't find here

Some of the skills I use daily aren't here because they're **not mine**. If you're looking for those:

- [`impeccable`](https://github.com/pbakaus/impeccable) — Paul Bakaus
- [`app-store-review`](https://github.com/JustinPerea/app-store-review-skill) — Justin Perea
- [`make-interfaces-feel-better`](https://github.com/jakubkrehel/make-interfaces-feel-better) — Jakub Krehel
- [`taste-skill`](https://github.com/Leonxlnx/taste-skill) — Leonxlnx

Others are missing because outside my setup they'd have nothing to run against: mail triage, session close-out, deploy preflight. They assume a database, a set of scripts and an issue tracker that don't exist on your machine. Publishing them would be handing you a shell.

## License

MIT. If one of these is useful and you improve it, I'd like to hear about it: martino@subralabs.com

Martino Vigiani
