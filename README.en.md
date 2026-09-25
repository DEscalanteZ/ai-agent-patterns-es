# Six patterns for working with AI agents (and the loop that holds them up)

> 🇪🇸 [Versión original en español](README.md)

A practical guide for people who direct AI agents, whether or not they write code themselves. The six-pattern taxonomy comes from the infographics in **Boris Cherny**'s interview (Boris created Claude Code). The explanations, examples and traps are ours. They come from months of running production automations across three companies.

---

## Minimal glossary

| Term | What it is |
|---|---|
| **Agent** | An AI working on its own on a task, with its own tools |
| **Subagent** | An agent launched by another agent for part of a job, which hands the result back |
| **Fan-out** | Splitting a job across many agents at once |
| **Rubric** | The list of criteria something is scored against |

---

## First: the loop

```
   Gather context  →  Act  →  Verify the work
          ↑__________________________________|
```

1. **Gather context**: look at the files, read the logs, see what is actually there.
2. **Act**: write the code, send the alert, move the file.
3. **Verify the work**: check that what came out is what should have come out. If not, go round again.

### Why this matters more than everything else

**Almost every serious outage we have had was step 3 skipped.** The robots did not fail. They ran, finished green, and nobody looked at what they produced.

- A backup that ran every day and had not copied anything for a week.
- An accounting job that finished "fine" with an empty output file.
- Duplicated scheduled jobs hitting the same system twice, each one "working".
- A watchdog that silently dropped off the guardian's list while overall health stayed green. The guardian counted what it saw, not what was missing.

The rule that comes out of it: **check what a job produced, never just that it ran.** An "OK", exit code 0, or a file dated today proves nothing, because any of them can be empty.

---

## The six patterns

### 1 · Classify-and-Act

**What it is:** something comes in, a classifier decides what kind of thing it is, and routes it to the right specialist. It works like a switchboard.

**Example:** emails arrive in several inboxes belonging to several companies. A classifier decides which company each one belongs to and tags it with a traffic light. Nobody reads three hundred emails: the classifier routes them.

**The trap:** if the classifier is wrong, everything downstream is wrong too, silently and confidently. Measure the classifier **on its own**, against a handful of cases you already know how to classify by hand.

**Cost:** low. The classifier can be the cheapest model that holds up.

### 2 · Fan-out-and-Synthesize

**What it is:** a big job is split into pieces, each piece goes to a different agent **at the same time**, and at the end one agent merges everything into a single answer.

**Example:** a weekly report covering a dozen watchdogs. Instead of walking through them one by one, each block is checked in parallel and the last agent writes the report.

**The trap:** this is a swarm. Eight agents cost eight times as much. It only works when the pieces **are truly independent**: if piece 2 needs what piece 1 found, it doesn't work.

**Cost:** the most expensive of the six. It deserves an explicit rule: no large swarm without sign-off from whoever pays.

### 3 · Adversarial Verification

**What it is:** one agent does the work. Others, who don't know who did it or what was expected, try to **prove it wrong**. If it survives, it counts as right.

**Example, and a house rule for us:** the *cold reviewer*. No significant finding (a bug, a figure in euros, a business conclusion) counts as right until another agent, with a clean context and a brief to refute rather than confirm, has tried to knock it down. When in doubt, it rules the finding false.

**Why it exists:** ask an AI to find a bug and it will find one **even if there isn't one**, because it tends to please. The cheap antidote is to have someone else try to tear it apart.

**The nuance:** when something can fail in several ways, **reviewers with different angles** (is it correct? does it reproduce? does it break something else?) beat several identical ones. Identical reviewers make the same mistake.

**The prompts and what we measured:** the two refuter prompts we use, plus an 8-job test of whether a single agent holding both briefs was enough. It wasn't. See [refutadores-en-frio.md](refutadores-en-frio.md#english-summary).

### 4 · Generate-and-Filter

**What it is:** several agents produce lots of candidates, then a filter with written criteria keeps the good ones and drops the rest.

**Example:** screening for a systematic review. Thousands of papers come in, and a filter with PRISMA criteria decides which ones pass.

**The trap:** **a filter is only as good as its criteria.** If the rubric is made up rather than taken from the source, the screening goes blind without anyone noticing: results still come out, just the wrong ones. Write the criteria **before** looking at the candidates, and test them on cases you already know how to classify.

### 5 · Tournament

**What it is:** several complete attempts at the same job are compared **two at a time**, and the winner of each pair moves up. At the end, one is left.

**Example:** for a design piece that matters, prepare several variants in opposite styles and choose between them in pairs.

**Why two at a time:** comparing two things is far more reliable than giving one a 7.5. You know which of two covers you prefer before you know why.

**The trap:** the judge inherits the tastes of whoever wrote the rubric. If the criterion says "modern", the modern one wins, not the one that sells. For commercial decisions, the judge should be sales data, not an opinion.

### 6 · Loop Until Done

**What it is:** the agent searches, and if it finds something new, it runs another round. When two rounds in a row find nothing, it stops. **It stops because there is nothing left to find, not after a fixed number of attempts.**

**Why it matters:** ask for "10 bugs" and you get 10, then silence, whether there were 30 or only 4 real ones. A fixed number misleads in both directions.

**Example:** an audit that accepted its count after checking one data store, when there were two. A second round asking "what haven't I looked at?" would have caught it.

**The trap:** keep track of everything **found**, not just what was approved. If you only remember what passed the filter, the rejected items come back every round and the loop never ends. And set a **ceiling on rounds** before you start.

---

## What to take away

1. **You probably already use several of these** under other names. No need to start from scratch: just name them and use them on purpose.
2. **The one that saves the most is the full loop**, especially step 3. Almost every outage is the same hole: nobody checked **what came out**.
3. **The one to use with care is fan-out**, because it multiplies cost.
4. **A caveat about the patterns themselves:** they are ways of organizing work, not intelligence. A tournament between four bad answers picks the least bad. None of them fixes bad input data. Only looking at the source does that.

---

⭐ **If this guide helps you, star the repo**: it's the simplest way to help it reach more people working with agents.

*Author: David Escalante ([@DEscalanteZ](https://github.com/DEscalanteZ)). The six-pattern taxonomy comes from Boris Cherny's interview (Anthropic). License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You may copy and adapt it with attribution.*
