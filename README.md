# The court skill

When your AI assistant agrees with everything you say, put your idea on trial in front of a jury of 12 LLMs.
Ported and optimized specifically for **OMP (Oh My Pi)** with native multi-agent batch execution via the `task` tool. Free, MIT licensed, no signup, no API key, nothing to connect.

Originally created by **Alex Chen** ([@nocodealex](https://instagram.com/nocodealex)) for Claude Code. This repository is an independent MIT fork maintained by [Vonarian](https://github.com/Vonarian) providing native OMP compatibility.

Ask an AI "is this a good idea?" and it usually says yes. `/court` takes the same idea and runs a
trial instead:
1. A **prosecutor** sub-agent builds the strongest honest case that your idea fails as described.
2. A **defense** sub-agent builds the strongest honest case that it works.
3. Each side answers the other's points.
4. **12 jurors** read the record and vote GUILTY or NOT GUILTY on their own. Each one sees the case
   through different eyes: an accountant, a first-time customer, a competitor, a retired judge, a
   teenager, an engineer, a small-business owner, a statistician, a parent, an investor, a designer
   and a lawyer. No juror sees another's vote, so nobody can just go along with the room.
5. A **judge** sub-agent reads the verdict, the charges that stuck, what the defense got right, and the
   conditions for acquittal: the specific changes that would flip the most votes.

What comes back is `VERDICT.md`: the vote (for example GUILTY, 9 to 3), every juror's one-line reason,
and what to change before you try again.

**It never touches your files.** Everything is written inside `.court/` in the folder you run it in.

## Install

### For OMP (Oh My Pi)

Install into your OMP skills folder:

```bash
# User-level skill
git clone https://github.com/Vonarian/court-skill
cp -r court-skill/skills/court ~/.omp/skills/

# Or project-level skill
cp -r court-skill/skills/court .omp/skills/
```

OMP discovers `court` automatically. You can invoke it via `/court` or refer to `skill://court`.

### For other harnesses

This fork is developed and tested for **OMP**. If you use Claude Code, install the original upstream skill:

```bash
# Upstream Claude Code skill by @nocodealex:
git clone https://github.com/alexyc9381/court-skill ~/.claude/skills/court
```
## Use

```
/court Quit my job to sell candles on Etsy full time. I have $8,000 saved and 40 sales so far.
```

| flag | meaning |
| --- | --- |
| `--jury N` | 3 to 12 jurors. Default 12. |
| `--quick` | 6 jurors. |
| `--seed S` | Fixes which jurors sit on a smaller jury. |

A full trial is 17 sub-agent calls (2 openings, 2 rebuttals, 12 jurors, 1 judge); `--quick` is 11.
Every sub-agent counts toward your model plan's usage, so start with `--quick` if you are near your
limit.

See `examples/candles/VERDICT.md` for a real (3-juror) trial: "Quit my job as a nurse to sell
handmade candles on Etsy full time" came back GUILTY, 3 to 0, with the conditions that would flip it.

## How it works

`skills/court/court.py` does the bookkeeping: it opens the trial, writes one brief per sub-agent, checks
that every output exists, counts the votes and renders the verdict. The orchestrating agent only coordinates: it never
argues a side, never votes and never edits the verdict. The jurors are in `skills/court/jurors.json`.

```bash
python3 -m unittest discover -s tests
```

## Making a video or post about this?

Go ahead. Credit it like this, in your caption or description:

```text
/court skill by Alex Chen (@nocodealex): github.com/alexyc9381/court-skill
```

Tag [@nocodealex](https://instagram.com/nocodealex) so I can see it. Every verdict the skill writes already ends with the same credit, so leave it in the shot.

Writing about it or citing it in a paper? Use **Cite this repository** in the sidebar on GitHub.
## Uninstall

Remove the installed skill directory from your OMP installation:

```bash
rm -rf ~/.omp/skills/court      # user-level
rm -rf .omp/skills/court       # project-level
```

## License and Attribution

Released under the MIT License. Based on the original [court-skill](https://github.com/alexyc9381/court-skill) by Alex Chen ([@nocodealex](https://instagram.com/nocodealex)). This fork adds native OMP subagent batching and test coverage.

The jury is made of language models, so treat the verdict as a structured
second opinion, not legal, financial or professional advice.
