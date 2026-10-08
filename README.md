# Council

A decision council for [Claude Code](https://code.claude.com). You bring a decision or a plan, four agents argue it out, and a Judge rules against **your** goals, not against generic best practice.

```
/council:decide Should I quit my job to freelance, or keep it and freelance on weekends?
```

## How a run works

1. **Situation first.** Claude frames the question, lists what is known and what is missing, and adds at least one *third option* if your question is binary. You answer up to 3 questions once.
2. **Tipping gap.** If a missing number would flip the decision, you choose: stop and find it out first, or get a provisional verdict that says exactly at which value it tips.
3. **Three pleas in parallel:**
   - **Believer**: the strongest honest case *for* the option
   - **Skeptic**: weak spots, untested assumptions, self-deception, weighted critical / medium / minor
   - **Investor**: money, time, opportunity cost
4. **Witness round.** Each role asks you one question. You answer once, briefly.
5. **Rebuttal.** Believer and Skeptic attack each other's claims and must concede at least one point.
6. **Verdict.** The Judge decides every disputed point, checks one touchstone per role (the Skeptic's killer question, the Believer's condition, the Investor's "only worth it if") and gives you the smallest next step (max. 2 hours). Fixed format, max. 330 words.
7. **Saved** to `council/decisions/YYYY-MM-DD_<slug>.md`, with the full debate.
8. **Loop closed.** Claude asks "What do you do now?" and records your answer. After 7 days comes the step check (did you do it, what came of it), at the revisit date the review (was the verdict right?).

About 6 subagent calls per run.

## Install

In Claude Code:

```
/plugin marketplace add notXperium/council
/plugin install council@notxperium
```

Then, in any project: `/council:decide <your decision>`. You can also just describe a decision you are weighing; the skill triggers on its own.

## Your context

On the first run the Council creates `council/context.md` in your project and fills it with you in one short round: situation, goals, success criteria, limits, strengths, known risks. Every role measures against this file. The better it is, the less generic the verdict.

Keep it current. Outdated goals give wrong verdicts. If you add new facts and an old verdict was waiting for exactly that info (`revisit-when:`), the Council tells you.

## Files in your project

```
council/
├── context.md            # who you are, what you want (only you edit the facts)
└── decisions/
    └── 2026-10-08_freelance-or-job.md
```

Nothing is committed or pushed unless you ask.

## Language

The plugin is written in English. The Council answers in the language you write in.

## Principles

- **No invented numbers.** Missing numbers are marked `MISSING` and asked for.
- **No simulated customers.** A role pretending to be your customer is the Council talking to itself. Real data points come from you, with a source.
- **No legal, tax or investment advice.** Instead you get the concrete question for your tax advisor or lawyer.
- **Small next step.** "Version 1 is live" beats "perfectly planned".

## Origin

Built as part of a personal AI system and used for real decisions since September 2026. v4 added the situation step, the witness round and the closed loop after the first runs showed that most bad verdicts came from a badly posed question, not from bad arguments.

## License

MIT
