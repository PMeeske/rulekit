---
title: Rulekit
---

# Rulekit

### Write your game rules in plain C#. Test them with no engine running. Watch every step they take.

A Unity package for programmers building turn-based, tactics, card, roguelike, puzzle, management
and simulation games — and the systems layer of everything else.

Unity 2021.3 or newer. Published by MeeskeSoft.

## The problem

You have a bug in your enemy AI and you cannot reproduce it.

It happened once, in Play Mode, to a tester. The enemy chased when it should have been idle. You
have a screenshot and a hunch. You cannot get back to the state it was in, so you add log lines,
press Play forty times, and hope.

That is not a Unity problem. It is a problem with where your rules live. A `MonoBehaviour` that
holds mutable state, reads `Time.time`, rolls a `Random` and moves a transform cannot be put back
the way it was — so it cannot be tested, and it cannot be re-run.

## What Rulekit does

Rulekit moves the decisions out. Health, aggro, inventories, doors, locks, turns, cooldowns: all of
it becomes "state in, new state and effects out". Those rules live in an inner core of plain C#
with no engine references. Your `MonoBehaviour` becomes the outer shell that does what the core
decided.

**Every step is recorded.** The Step Recorder window lists every decision every object made, in
order, with the fields that changed and the effects that fired.

**You can put the state back and run the step again.** Restore the state to the instant before a
decision, put a breakpoint in your rule, re-run it — then let the window compare the re-run against
the recording and tell you whether your logic is still deterministic.

**The tests run with no Unity at all.** The spec suite ships with the package and runs two ways:
Unity's Test Runner, and `dotnet run` from the command line with no editor open. Your CI can run
your game's rules.

## What it isn't

Not an ECS. Not a movement, physics or animation framework. Not visual scripting — there is no node
graph, and the Step Recorder is a debugger, not an editor. It handles the rules of your game, not
the feel.

It does not give you serialization, save/load or networking. Restore and re-run touch the pure
state only — they do not rewind the scene.

**Rulekit is not open source.** This site and the GitHub repository hold documentation, the issue
tracker and samples. The package source ships with the asset and is yours to read and modify once
you buy it.

## Where to buy

Rulekit is a paid package for the Unity Editor. The store link appears here once the listing is
live. Until then the contact details below reach the publisher directly.

## Contact

**Support:** <https://github.com/OWNER/rulekit/issues> — a public tracker, so you can read what has
already been asked and answered before you buy. Bug reports ask you to paste a Step Recorder log;
the window has a **Copy log** button, and that paste is usually the whole diagnosis.

**Email:** philip.meeske@outlook.com

**Publisher:** MeeskeSoft

---

© 2026 MeeskeSoft. Unity is a trademark of Unity Technologies. Rulekit is not affiliated with or
endorsed by Unity Technologies.
