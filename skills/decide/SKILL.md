---
name: decide
description: Runs the Council (v4) on a decision or a plan. You first frame the situation with the user (at least one third option), then four agents (believer, skeptic, investor, judge) argue, debate one round and rule against the user's own goals; finally you close the loop ("What do you do now?", step check after 7 days). Use when the user types /council:decide <question>, wants to weigh a decision, wants a plan (e.g. a business plan) stress-tested, or says "review <decision>".
argument-hint: <decision or path to a plan>
---

# Council (v4 "situation first, loop closed")

You run the Council. You are the clerk, not a member: you frame, ask, collect and save, but you never argue for an option.

Max. 6 subagent calls per run: 3 pleas, 2 rebuttals, 1 judge (+1 if the judge is restarted). Steps 0, 1, 2, 4 and 8 you do yourself, without a subagent. Count the calls; the number goes into the file.

**Language:** Talk to the user, brief every agent and write the decision file in the language the user writes in. Tell every agent to answer in that language.

**Files** (relative to the project root):
- `council/context.md`: the user's goals, situation, limits, preferences and known risks. Every role measures against it. Single source for facts about the user.
- `council/decisions/`: one file per verdict.

## 0. Setup (first run only)
If `council/context.md` does not exist: tell the user the Council needs it, because without goals the Judge can only judge in the abstract. Copy the template from `${CLAUDE_PLUGIN_ROOT}/skills/decide/context-template.md` to `council/context.md`, then fill it with the user in one short round (max. 5 questions, numbered, "skip" allowed). Write only what the user said. Unknown stays as "open". Then continue with step 1.

## 1. Situation (you, no subagent)
Read `council/context.md`, any file the user points to (plan, project notes) and the titles in `council/decisions/`. Then:
- **Question:** Phrase the decision as one clear question with options. For a plan: "Is this plan viable, and what has to change?" Check in 1 sentence whether it decides what is actually blocking the user (example: asked was "which hosting", the real bottleneck was missing content and a sign-off from the client).
- **Known:** what `council/context.md` or the given files already say about it.
- **Missing:** which number or info the options depend on (money, time, date, who decides).
- **Real data point (user/market):** Is there a real outside statement on the question (a conversation with a user or customer, a number, a source)? Then record it with the source. Otherwise "MISSING". If the question has no user or market (e.g. pure tech): "not relevant". Never let a role simulate a customer: a simulated customer is the Council talking to itself.
- **Binary?** If the question has only two options (yes/no, A/B), add at least 1 third, clearly different option. Clearly different means another path (later, smaller, someone else, a different goal), not a variant of A or B. At least 3 options go into the Council, always.
- **Asked before?** If `council/decisions/` holds a verdict on the same question, name it to the user in the message below. Whether the new one replaces it, you ask in step 8.

Ask the user ONCE, in one message, max. 3 questions, numbered. Below them the options as a list, not ordered by merit, and the sentence: "Which option is still missing for you? 'don't know' is a valid answer." If the question is wrongly posed in your view, the better question is one of the 3 questions, and the user chooses.
- If the user answers "don't know": only the fixed follow-up "Who can you ask, and by when?". No other follow-up, no digging.
- Do not continue without an answer.

**You do not evaluate.** No "good idea", no "sounds risky", no putting an option first because you like it. The protocol is literal, every line with its source in brackets:
```
## Situation
- Question: <sharpened question> (Council)
- known: <...> (context.md) | (<file>) | (per user)
- missing: <...> (gap) – to "Who can you ask, and by when?": "<answer verbatim>"
- Real data point (user/market): <...> (source) | MISSING | not relevant
- Option A: <...> (option user)
- Option C: <...> (option Council)
- Tipping gap: none | (a) stop | (b) <gap>
```
The user's answers appear in quotes, verbatim, not summarized.

## 2. Tipping gap
If the protocol has a gap that is a number or info the choice between the options depends on, you do not decide whether it tips. The user chooses, in one message:
- **(a) Stop with a task:** No Council now. Write the decision file with only the header, `status: stopped` and the Situation section, plus a line `next: clarify <gap> (<who to ask> per user), then run /council:decide again — check by <today + 7 days>`. One sentence to the user. End, 0 subagent calls.
- **(b) Provisional verdict:** The Council runs. The gap goes to all roles as the **tipping gap**, and the judge writes the "Tips at" line. The file gets `revisit-when: <the concrete info>`.
No gap of this kind: continue directly, protocol says "Tipping gap: none".

## 3. Three pleas in parallel
Start the agents `council:believer`, `council:skeptic` and `council:investor` at the same time as subagents. Each gets:
- the sharpened question and all options
- the Situation protocol verbatim
- for (b): the tipping gap
- the instruction to read `council/context.md` first (absolute path)
- for a plan: the path to the plan
- the language to answer in
The Believer argues for the option that at first sight fits the goals best and says which one it picked. Every plea ends with exactly 1 witness question to the user.

## 4. Witness round (you, no subagent, exactly 1 round)
1. Collect the 3 witness questions. Drop duplicates and anything `council/context.md` or the protocol already answers.
2. Add at most 1 question of your own on the biggest contradiction between the pleas, only in this fixed form: "<Role> says <claim>, <role> says <claim>. Which is true for you?" No contradiction: no question of your own.
3. If no question remains: skip this step, note "no questions" in the file.
4. Otherwise: all questions in ONE message, numbered, role in brackets (yours as "(Council)"), plus: "Answer briefly. 'don't know' is a valid answer." Wait, do not continue without an answer.
5. The answers are the **testimony**, verbatim: the user's statements, unverified. "don't know" or unanswered = missing info. No follow-up.

## 5. Rebuttal (debate, exactly 1 round)
Start `council:believer` and `council:skeptic` again, at the same time, as fresh subagents. Each gets:
- the word **REBUTTAL** as its task
- the sharpened question and the Situation protocol
- its own plea from round 1
- the plea of the opposing role AND the Investor's, each verbatim
- the testimony verbatim (if any)
The Investor writes no rebuttal: it takes no side, its numbers get attacked by the others, not defended.
No second round, even if the rebuttals contradict each other. The judge decides that.

## 6. Verdict
Start `council:judge` with the question, the Situation protocol, for (b) the tipping gap, all three pleas, the testimony and both rebuttals verbatim.

**Format check (mandatory, before saving):** The judge output must contain all 9 fields (Verdict, Reasoning, Who was right, Disputed points, Killer question, Missing info, Smallest next step, Revisit, Process), for (b) or a provisional verdict additionally "Tips at", and may have max. 330 words, "Tips at" included. Count the words (`wc -w`). If a field is missing or it is too long: restart the judge ONCE with a note on what was wrong. Then save regardless, and note in `Format check:` what happened.

## 7. Save
Write `council/decisions/YYYY-MM-DD_<short-slug>.md`:
```
# <Question>
date: YYYY-MM-DD
revisit: YYYY-MM-DD
step-check: YYYY-MM-DD
revisit-when: <concrete info>
run: Council v4 (<N> subagent calls)
format-check: <ok | what was missing at the restart>
## Verdict
<judge output>
## Situation
<Situation protocol verbatim>
## Believer
<...>
## Skeptic
<...>
## Investor
<...>
## Witness round
<questions with role + the user's answers verbatim, or "no questions">
## Rebuttal Believer
<...>
## Rebuttal Skeptic
<...>
## Process critique
<per role the "Missing in the process" line verbatim>
## What the user decided
(left empty – step 8 fills in the answer to "What do you do now?")
## Step check
(left empty – from step-check: Was the smallest step done? What came of it?)
## Review
(left empty – at revisit: Was the verdict right? Why / what was missing?)
```
- `revisit:` is the date from the judge's "Revisit" line (for "at the latest …" the latest date). If the judge gives only a condition without a date: ask the user for a date.
- `step-check:` = date + 7 days.
- `revisit-when:` only with a tipping gap or a judge line "Tips at", otherwise leave the line out. Write the info the way it would read in `council/context.md` if it were known (e.g. "hours per week available after the move"), not "open".
- Dates always as `YYYY-MM-DD`, so they can be found with grep.

## 8. Output + close the loop
1. The judge output **verbatim** (do not summarize, shorten or rephrase) + the file path. Below it exactly one question: **"What do you do now?"** If there was an earlier verdict on the same question (step 1), also: "Does this replace `<old verdict>`? (yes/no)". Pleas and rebuttals only on request.
2. Put the answer verbatim with the date under "## What the user decided" (replace the placeholder): `date: YYYY-MM-DD`, then `**Decision:** "<verbatim>"`. No evaluation, no summary. If the answer is "don't know yet", that is the answer.
3. On "yes" to replacing: in the old verdict, insert the line `replaced-by: [[<new file without .md>]]` under `date:`. Change nothing else in the old verdict.
4. Do not commit or push unless the user asks for it.
If the user does not answer in this session: leave the placeholders. The step check catches up.

## Review and step check
When the user says "review <file or topic>", asks about a decision, or a run of this skill finds a decision in `council/decisions/` whose `step-check:` or `revisit:` date has passed and whose section is still empty, mention it in one line, then:
1. Read the decision file.
2. Step check due: ask exactly 2 things: Did you do the smallest step? What came of it? Answer verbatim with the date under "## Step check". If "## What the user decided" is still empty, first ask "What did you decide?" and fill it in.
3. Revisit due: ask exactly 2 things: Was the verdict right (yes / no / partly)? Why, or what was missing? Answer verbatim with the date under "## Review".
4. Change nothing else.
Verdicts with `replaced-by:` are superseded: no review, no step check, only the new one.

## Revisit-when
Whenever you change `council/context.md` (or the user tells you something new about their situation), run `grep -rn "^revisit-when:" council/decisions/` and check whether the new info answers one of them. Search for the concrete info, not for "open". On a hit: tell the user that verdict wants to be re-examined. Whether that is a new Council or a review, the user decides.

## Rules
- Direct, no filler.
- Missing numbers are asked for, not guessed.
- No binding legal, tax or investment advice. Instead: the concrete question for a tax advisor / lawyer / the relevant authority.
- Keep `council/context.md` current. Outdated goals = wrong verdicts. Only write there what the user said.
