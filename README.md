# Whiteboard Defense (Claude Code plugin)

> "I should be able to pull you aside at any moment and ask you to explain any customer-facing
> system you've shipped. You should be able to clearly explain how it works and defend the
> decisions you made." — Mitchell Hashimoto's benchmark for responsible AI usage

This plugin turns that benchmark into a step you can run on demand. A fresh skeptic subagent,
which has none of your conversation's context, reads a design spec, an implementation plan or a
shipped feature and attacks it the way Hashimoto would: *why X instead of Y? what happens if this
actor is malicious? what data structure did you use and why? where does this fail?* The main agent,
which made the decisions, has to defend each attack with evidence or concede it with a concrete
change. The skeptic pushes back on weak defenses over several rounds. You get one report: every
attack, its verdict, and the argument you can repeat at a whiteboard yourself.

## What it does

- **Fresh eyes.** The skeptic is a new subagent with only the target and read access to the repo,
  so it is not primed by the reasoning that produced the design.
- **Specific attacks, no checklists.** Every attack cites the section or line it targets. Generic
  "have you considered logging?" items are not allowed, and padding to fill a quota is forbidden:
  fewer than three real holes means the skeptic says so.
- **Evidence, not confidence.** The defender answers each attack as DEFEND (checked against spec,
  code or constraints), CONCEDE (with the change) or OPEN (a call only you can make). The skeptic
  is not satisfied by tone, only by evidence, and can raise new attacks that an answer exposed.
- **Bounded.** The exchange stops when the skeptic has nothing left or after the round cap
  (default 2, `--rounds N`; a round is one skeptic message: the attacks, then its ruling on the verdicts).
- **You decide.** Conceded points become proposed spec edits or, for code, proposed follow-ups.
  Nothing is edited until you approve, and nothing is committed.

## Install

```
/plugin marketplace add GergKllai1/whiteboard-defense
/plugin install whiteboard-defense@whiteboard-defense
```

## Use

```
/whiteboard-defense docs/design/rate-limiting.md
/whiteboard-defense docs/plans/rate-limiting.md --rounds 3
/whiteboard-defense the rate limiting feature on this branch --model fable
```

The target is any spec or plan file, in whatever format you write them, or a feature named by branch or
words. With no target it asks which one to defend. `--rounds` sets the round cap
(default 2). `--model` sets the skeptic's model, default `opus`: it finds the same defects as the top tier at a fraction of the price. The cost of a run is dominated by the files the two agents read, not by rounds; when the session runs on Fable the skill says so up front, since the defending side inherits that model.

It fits at two points of any workflow: after a spec is approved and before the implementation plan is
written, and after implementation before the branch is merged. It works alongside
[superpowers](https://github.com/obra/superpowers) but does not depend on it. It is never automatic.

## License

[MIT](LICENSE)
