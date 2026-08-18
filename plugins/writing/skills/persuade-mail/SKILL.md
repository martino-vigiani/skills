---
name: persuade-mail
description: Write a marketing or persuasion email that actually moves the reader to act — cold outreach to new prospects, or persuading an existing client (upsell, renegotiation, holding a price while meeting them halfway). It leads with the reader's world, proves value with concrete specifics instead of hype, and hands the reader a clear choice, all in a warm peer-to-peer voice that never talks down or oversells. Then it runs a humanize pass so the result reads like a real person, not an AI. Use this whenever the user wants to write, draft, sharpen, or "make more convincing / more powerful" an outreach or sales or client-persuasion email, says a draft "sounds weak / too soft / doesn't sell / too salesy / too aggressive", wants a cold email or pitch to a lead, or needs to convince a client on price, scope, or a proposal. Trigger even if they don't say "marketing" — any request to make an email land harder and win a yes belongs here. Default language Italian, auto-detecting the brief's language. Not for internal status updates, transactional notices, or pure de-AI-ifying with no persuasion goal (that's humanize).
---

# Persuade-mail

Turns a brief (or a weak draft) into an email that wins a yes. The persuasion comes from **value made concrete and a reader who feels respected and in control** — never from hype, pressure, or looking down on them. A superior or salesy tone makes people defend themselves instead of agreeing; the whole craft here is the opposite move.

The actual writing runs on a **Sonnet** subagent, which writes naturally for this. You orchestrate: gather the goal, delegate with the playbook, present the result. When money or a relationship is on the line, add the test loop in step 3 — that's where the quality actually comes from.

## Why this exists

Most "marketing" emails fail two ways. They **boast** (adjectives, superlatives, "revolutionary", "best-in-class") — claims with no proof, which read as weak and make the reader distrust. Or they **condescend** (explaining the obvious, "as you surely know", implied "you don't get it") — which makes the reader dig in. A powerful email does neither. It opens on the reader's situation, proves value with specifics they can check, and gives them a clean choice. The reader ends up persuaded because the argument is good and they were treated as a peer, not because they were pushed.

The full craft — the persuasion architecture, the phrasing patterns, the anti-condescension and anti-hype rules, and the templates for cold outreach vs client persuasion — lives in `references/persuasion-playbook.md`. That file is the substance; this file is the workflow.

## Workflow

### 1. Pin the goal (the load-bearing step)
Before writing, get clear on four things, from the conversation, a brief, or a draft the user points at:
- **The reader and their world** — who receives this, what they care about, what problem or goal is live for them right now. Persuasion starts from their side, so this can't be blank.
- **The one outcome** — what single action should this email produce (a reply, a call booked, a yes to a proposal, a price accepted). One email, one ask.
- **The proof you actually have** — concrete facts, numbers, names, results, before/after. These are the engine; the email is only as strong as the specifics behind it.
- **Scope** — cold outreach to a new prospect, or persuasion of an existing client. They use different templates (see the playbook).

If the reader or the one ask is genuinely missing and you can't infer it, ask once — those two are load-bearing. Everything else, proceed with sensible defaults.

**Never invent proof.** If the brief lacks a concrete number or result the email needs, use a bracketed placeholder like `[dato: es. −83% di byte serviti]` and flag it to the user, rather than fabricating a statistic. A made-up specific is worse than none: it's the one thing that destroys the credibility the whole email runs on.

The dangerous fabrication isn't the wild claim — you'd catch that. It's the **plausible derived number**: a day count you reasoned your way to, a share of something that "must be roughly right", a range that sounds like market rate. Those slip through because they feel like inference rather than invention. When the email's argument rests on numbers, get them from the source before writing: spawn a read-only agent over the repo, the docs, the ledger, the mail archive, and make it return every claim as `evidenza: <path or command> → <observed value>`, tagging anything it couldn't confirm. Then write only from what came back. In practice the numbers that survive verification are also the ones that persuade, because they're specific in a way invention never is.

### 2. Delegate the writing to Sonnet
Spawn a subagent with the Agent tool, `subagent_type: general-purpose`, **`model: sonnet`**. In the prompt:
- Tell it to first read `<ABSOLUTE_PATH_TO>/references/persuasion-playbook.md` and write the email applying it, picking the right template for the scope.
- Then, in the **same** pass, apply the humanize rules at `<ABSOLUTE_PATH_TO_HUMANIZE>/humanize/references/humanize-rules.md` so the result is persuasive AND reads human (no em dashes, no chatbot vocabulary, natural rhythm). Persuasion first, humanize second, one clean output.
- Pass the pinned goal from step 1: reader + their world, the one ask, the concrete proof (verbatim, so it can't drift or invent), language, register (Lei vs tu — a formal client relationship stays "Lei" throughout), and scope.
- Hard constraints: keep every fact/number/name exactly as given and **add no claim, statistic, or promise not in the brief**; no hype superlatives, no condescension, no fake urgency (the playbook lists these); return ONLY the email (subject line + body), ready to paste, no preamble or trailing notes.

Resolve `<ABSOLUTE_PATH_TO>` to this skill's own `references/persuasion-playbook.md`, and `<ABSOLUTE_PATH_TO_HUMANIZE>` to the sibling `humanize` skill directory in this same plugin (`plugins/writing/skills/humanize/`), so the subagent reads both rather than you re-pasting them.

### 3. Test it before you hand it over (when the stakes justify it)
For anything that moves real money or a real relationship, don't ship the first draft. Judge it, fix it, judge it again. Two agents, `model: sonnet`, run in parallel:

- **The recipient.** Give a subagent a system prompt that *is* the reader: age, trade, how they talk, what they've already said and written, what they're afraid of, what they've already conceded, and the exact numbers they have in hand. Then have them react in character — gut reaction, which phrases annoyed them, which convinced them, whether they accept or counter and **with what figure**, and the reply they'd actually send. Ask for three scores out of ten: convinced, arrogant, and *how much they now want a competing quote*. That third one catches damage the other two miss.
- **The critic.** A second subagent hunting the four failure modes with quotes and rewrites: arrogance, self-pity, salesman prose, and internal contradiction. It also checks the arithmetic (see the playbook's section on the arithmetic the reader will do). This is the one that catches "three weeks" between two dates two weeks apart.

Then revise and re-judge with a **fresh** persona agent, never the same one — an agent that has already seen a version rates the next one on improvement rather than on merit. Stop when the notes are only subtractive ("too long", "cut this line"), which means the argument is done and the rest is trimming. Three rounds is usually where it lands.

Two things this loop is for, beyond polish. It tells you **where the price actually breaks**: a persona that accepts 3.900 without countering is evidence the number could go higher, and a persona that counters tells you the landing zone. And it finds the **missing paragraph** — twice in one session the recipient's first question was about something the email never addressed, which no amount of rereading your own draft surfaces.

### 4. Present
Output the email as the **last thing in your turn**, ready to copy-paste, subject line included. No meta-commentary after it. If you left a `[dato: …]` placeholder or made a notable strategic choice (e.g. which single ask you picked), put a one or two line note *before* the email, never after — and keep it short.

## Guardrails

- **Persuasion, never deception.** Honesty is the engine here, not a constraint on it: admitting a real limit or trade-off buys more trust than relentless positivity. Fabricated proof, manufactured scarcity, and manipulative flattery all read as weak and backfire — they're banned in the playbook for a reason.
- **Respect beats superiority, always.** If the draft comes back talking down to the reader, over-explaining, or scoring points, reject it. The reader should feel understood, not managed.
- **One ask.** If the brief has three goals, either pick the strongest for this email or tell the user it needs to be split. A single clear ask converts; a menu of asks doesn't.
- **Don't translate.** Follow the brief's language; default Italian.
- The compass is a real reader deciding whether to say yes, not an impressive-sounding paragraph. When a sentence doesn't help them decide, cut it.
