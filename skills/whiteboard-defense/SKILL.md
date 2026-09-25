---
name: whiteboard-defense
description: Use when the user asks for a whiteboard defense, to poke holes in, stress-test, challenge or defend a design spec, implementation plan or shipped feature, or wants an adversarial review before a spec is planned or a branch is finished.
---

# Whiteboard Defense

A fresh skeptic subagent attacks the target. You, the main agent, defend each attack with evidence or concede it with a concrete change. Decisions the user made are theirs to reopen, not yours. The user gets one report with a verdict per attack and the argument they can repeat at a whiteboard.

## Arguments

`/whiteboard-defense [target] [--rounds N] [--model M]`

- **target**: a spec or plan file path; or a feature named by branch or words, in which case the target is `git diff main...HEAD` plus the files it touches. No target → the newest file in `docs/superpowers/specs/`.
- **--rounds**: round cap, default 2. A round is one skeptic message: round 1 is the attacks, round 2 its ruling on your verdicts. `--rounds 1` skips the ruling.
- **--model**: the skeptic's model, default `opus`. Opus finds the same defects as the top tier at a fraction of the price; raise it only for a design you distrust.

Cost is driven by files pulled into context, not by rounds. If your session model is Fable, say so in one line before dispatching: the skeptic runs on Opus, the defense side runs on Fable at about 2.5× the cost, and `/model` before the run is the way to lower it. Then continue.

## Round 1: dispatch the skeptic

Spawn one `general-purpose` agent (never a fork: fresh context is the point) with this brief, filling the placeholders:

```
You are the skeptic in a whiteboard defense. Target: <path(s) or diff>. Repo: <root>.
Read-only: never edit, create or delete files, change git state, run the app or tests, or query a database.
Read the target and the code it names. Attack it as a senior engineer pulling the author aside:
- why-not-y: a decision whose stated rationale does not hold, or an alternative it never weighs
- malicious-actor: what a hostile caller, client, or data source can do
- data-structure: a representation, schema, key or cap that is wrong for the access pattern
- where-fails: a failure mode, race, limit or operational gap with no counter
- false-premise: the target says existing code does X; open that code and show it does not
Rules: every attack cites the section or line it targets and, for claims about code, file:line you opened.
Reading: locate a symbol with grep -n, then read a window of about 60 lines around the hit; read a whole file only when it is under 150 lines. Issue independent reads together in one message.
At most 8 attacks, most severe first. No generic checklist items. Fewer than three real holes → say so; never pad.
Output: numbered attacks, each = title · category · severity (blocker/major/minor) · the question · evidence · what would satisfy you. One-paragraph verdict.
```

## Defend

For each attack: open the cited line range yourself with about 30 lines of margin (not a grep result, not the whole file; batch independent reads in one message), then write down what the target itself says about the point (its rationale, its stated purpose, its caps) before choosing a verdict. Verdicts:

- **DEFEND**: the target's rationale still holds against the evidence. Give the argument, with your own file:line.
- **CONCEDE**: a fact or gap the target got wrong: a false premise about existing code, a missing cap, a wrong path, a race, a leak. Give the concrete change to the target (spec edit or code follow-up). The change fixes the recorded mechanism; it does not replace it.
- **OPEN**: the user's call. Use it when the attack would reverse a decision the target records as decided or the user approved in conversation, when it contradicts a purpose the target states, when the fix requires choosing a design (a formula, a weighting, new limits, a different mechanism), when it is a product, priority or cost judgment, or when neither side can verify the claim from the repo. An OPEN carries the skeptic's one-line case, your one-line case and the options. A decision you cannot hold with evidence is OPEN, never CONCEDE.

Send the verdicts to the same skeptic with SendMessage: it returns SATISFIED or NOT SATISFIED with the reason per attack, may raise attacks your answers exposed, and is told that confident tone is not evidence. In a later round you may add evidence to a DEFEND or correct a CONCEDE's change; you never introduce a new design to satisfy the skeptic, that is OPEN. After the last ruling, a NOT SATISFIED you accept becomes CONCEDE with the amended change, one you do not becomes UNRESOLVED, both without another skeptic message.

## Report

The report is these parts in this order, nothing else before the question:

1. Header: target, attack count, rounds used.
2. Table: `# · category · target section · verdict · one line`.
3. **Defended**: per attack, the argument, 1 to 3 sentences, written so the user can repeat it.
4. **Conceded**: per attack, the proposed edit or follow-up.
5. **Open**: per attack, both one-line cases and the question for the user.
6. **Unresolved / unverified**: anything left at the cap or that neither side could check.
7. One AskUserQuestion: apply the conceded edits to the working tree (no commit), or leave the report only. Open items are answered by the user in chat.

## Red flags

- Every attack conceded: you did not test the target's rationale. Re-read its decisions section and rule again.
- A number, formula or mechanism in a concession that the target never contained: that is a design choice, so OPEN with the options.
- "The skeptic's reasoning was wrong but its conclusion is right, so I'll concede": if the concession changes a recorded decision, it is OPEN.
- "I could not find it by grep": open the file:line the skeptic cited.
- "Nothing is left in dispute, so the user has nothing to decide": OPEN items exist to be decided by the user, not settled between agents.
- Editing the target before the user answers.
