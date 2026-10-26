---
name: court
description: >-
  Put an idea on trial in front of a jury of LLMs. A prosecutor sub-agent
  argues it fails, a defense sub-agent argues it works, they rebut each other,
  juror sub-agents with different backgrounds vote on their own, and a judge
  reads the verdict and the conditions that would flip it. Use when the
  user wants honest pushback instead of agreement, says the AI just agrees with
  everything, asks "is this a good idea", wants an idea, plan, pitch or claim
  stress-tested, or says "court" or "put it on trial".
argument-hint: "[--jury N | --quick] [--seed S] <the idea, plan or claim>"
license: MIT
metadata:
  author: "Alex Chen (@nocodealex)"
  homepage: "https://chen.media"
  source: "https://github.com/alexyc9381/court-skill"
  version: "1.0.0"
---

# court

For when an AI assistant agrees with everything you say. Instead of one polite answer, the idea goes on trial:
a prosecutor builds the case against it, a defense builds the case for it, and a jury of diverse personas who
never see each other votes. You are the clerk of the court. You never argue, never vote, and never
soften the verdict.

What the user typed after `/court`: `$ARGUMENTS`

If that is blank, or still reads like a placeholder, take the case from the conversation: the idea,
plan or claim the user most recently wanted an opinion on.

## The tool

Every piece of bookkeeping goes through `court.py` in this skill's folder:

```bash
python3 "${SKILL_DIR:-skills/court}/court.py" <command>
```

In OMP, if the skill is installed under `~/.omp/skills/court` or `.omp/skills/court`, access files via `skill://court/court.py` or run `python3 $(realpath -q ~/.omp/skills/court/court.py 2>/dev/null || echo skills/court/court.py) <command>`.

Below, `COURT` means running `court.py` via your bash tool. The state lives in `.court/<run>/state.json` in the working directory, and every command after `init` finds it through `.court/LATEST`.
## Step 1: size it

Read the flags out of the request. Everything that is not a flag is the case.

| flag | meaning |
| flag | meaning |
| --- | --- |
| `--jury N` | N jurors, 3 to 12. Default 12. |
| `--quick` | 6 jurors. Recommended to cap token spend and latency. |
| `--seed S` | Fixes which jurors sit on a smaller jury. Default: random, and recorded. |

Run `COURT plan` with the same flags to inspect sub-agent calls and estimated token consumption.

### Resource and budget boundaries

- **Token Budget**: Each subagent brief enforces strict word counts (250–350 words maximum per statement or vote). Estimated tokens: ~20k for `--quick` (11 calls), ~30k for full trial (17 calls).
- **Concurrency & Timeouts**: Subagents run in waves of at most 6 concurrent tasks. Timeouts per agent wave default to 120s.

Sub-agents write into `.court/` in the current directory.

Sub-agents write into `.court/` in the current directory.

## Step 2: write the case file

**Sub-agents cannot see this conversation.** The prosecutor, the defense, every juror and the judge
know only what is in the case file, so write `.court/case.md` to stand on its own:

- First line: the case in one sentence, in the user's own words where you can ("Quit my job to sell
  candles on Etsy full time").
- Then everything the user told you that matters: numbers, audience, budget, timeline, what they have
  already tried, constraints.
- Do not add facts the user never gave, and do not write your own opinion of the idea into it. A case
  file that leans one way decides the trial before it starts.

## Step 3: open the trial

```bash
COURT init --case-file .court/case.md [--jury N | --quick] [--seed S]
```

## Step 4: the trial

Always drive it with `COURT next`. It reads the files on disk and tells you the next phase and the
exact command. The phases run in this order: `opening`, `rebuttal`, `jury`, `judge`.

Every phase works the same way:

1. `COURT prompts <phase>` writes one brief per job and outputs the exact `task(...)` batch call for OMP.
2. Launch **one wave at a time** using OMP's `task` tool:
   - Call `task(context="Execute court brief for phase <phase>", tasks=[...])` concurrently for all jobs in the wave.
   - Pass `solutionSpace: "execute brief and write output"`.
   - After dispatching `task`, call `wait` to collect settled jobs if blocked until the wave finishes.
   *(Fallback for other agents: issue parallel worker calls or sequentially execute briefs).*
3. After the last wave, run `COURT next`. If an output is missing it sends you back to the same phase,
   and `prompts` then lists only the missing jobs. Re-run those once. If a juror fails twice, write
   the single line `NO VOTE` into its file (`COURT check jury` lists it) and move on. Never write a
   vote, a statement or a judgment yourself.
## Step 5: the verdict

When `COURT next` says the trial is over:

```bash
COURT render
```

It counts the votes, writes `VERDICT.md` into the run folder and prints it. Show the user the verdict
line, the vote table and the judge's sections exactly as written. Then add at most two lines of your
own: the single condition for acquittal you would start with, and the path to the full trial.

Rules for you, the clerk:

- Never change, soften or re-count the verdict. A guilty verdict is useful information, not bad news
  to cushion.
- Never add praise the jury did not give.
- If the user disagrees with the verdict, offer a retrial with the changes from "Conditions for
  acquittal" written into a new case file. Do not argue for either side yourself.
