# Rulekit

Documentation, samples and issue tracker for Rulekit, a paid Unity Asset Store package. The
package source is not published here.

Rulekit is a Unity asset for writing game rules as plain C#. This repository is its support
channel. If you bought Rulekit, the full C# source is in the package you downloaded from the
Unity Asset Store.

- **Report a bug or ask a question:** [open an issue](../../issues/new/choose)
- **Email:** philip.meeske@outlook.com — for anything you cannot put in public
- **Changelog:** [CHANGELOG.md](CHANGELOG.md)

## What Rulekit is

Game rules do not need `MonoBehaviour`. Health, aggro, inventories, doors, turns: all of that is
"state in, new state and effects out".

Rulekit gives you three things:

1. **A core** (`Logic<S, A>`) for writing rules as plain C# with no engine references. The rules
   compile without Unity, so you can unit-test them from a terminal.
2. **A shell** (`Shell<TState>`) — a base `MonoBehaviour` that holds the state, runs the rules, and
   carries out the effects they return.
3. **A Step Recorder** that records every step a shell takes. In the editor you can see what was
   dispatched, which fields changed, and which effects fired. You can put the state back to any
   step, or run a step again.

If you re-run a step and the result does not match the recording, a rule reads time or randomness.
The window names the fields that differ.

What Rulekit is **not**: an ECS, a movement or physics framework, or visual scripting.

## Supported Unity versions

| | |
|---|---|
| Minimum Unity version | **2021.3 LTS** |
| Tested on | 2021.3 LTS, 2022.3 LTS, Unity 6 |
| Render pipelines | Built-in, URP, HDRP — Rulekit draws nothing, so the pipeline does not matter |
| Package dependencies | none |
| Third-party DLLs | none |
| Platforms | any; the runtime code is plain C# and `MonoBehaviour` |

The command-line test runner needs the **.NET SDK** (8.0 or newer). It is optional.

## Install

1. Buy Rulekit on the Unity Asset Store.
2. In Unity: **Window > Package Manager > My Assets**, find Rulekit, **Download**, then **Import**.
3. Import everything the first time. Everything lands in one folder, `Assets/Rulekit/`.

You can delete `Samples/`, `Tests/` and `Tools/` later. Nothing in `Core/`, `Runtime/` or
`Editor/` depends on them.

## Quickstart

### See it work first

**Window > Rulekit > Create Demo Scene**, then press Play.

A capsule walks a loop. An enemy cube chases it, dies after a few hits, drops loot, and respawns.
A switch opens a door. Nothing in the scene reads input, so it runs whatever your input and render
settings are.

The Step Recorder window opens with the scene. Click the door, the switch, the inventory, or an
enemy, and watch the steps arrive. Pick a step and press **Re-run step**. Pick an `Add` step on the
inventory and press **Restore BEFORE**: the potion is gone again.

### Write a rule

```csharp
public sealed class DoorState
{
    public readonly bool IsOpen, IsLocked;
    public DoorState(bool isOpen, bool isLocked) { IsOpen = isOpen; IsLocked = isLocked; }
    public DoorState With(bool? isOpen = null, bool? isLocked = null) =>
        new DoorState(isOpen ?? IsOpen, isLocked ?? IsLocked);
}

public static class DoorLogic
{
    public static Logic<DoorState, Unit> Open() =>
        Logic.Get<DoorState>().Bind(d =>
            d.IsLocked ? Logic.Emit<DoorState>(new Effects.Sound("door_locked"))
            : d.IsOpen  ? Logic.Nothing<DoorState>()
            : Logic.Put(d.With(isOpen: true)).Then(Logic.Emit<DoorState>(
                new Effects.Animate("Open"), new Effects.Sound("door_open"))));
}
```

Query syntax works too:

```csharp
static Logic<EnemyState, Unit> Damage(int amount) =>
    from s  in Logic.Get<EnemyState>()
    from _  in Logic.Put(s.With(health: Math.Max(0, s.Health - amount)))
    from __ in Logic.Emit<EnemyState>(new Effects.Sound("enemy_hit"))
    select Unit.Value;
```

### Run it

```csharp
[RequireComponent(typeof(DefaultEffectInterpreter))]
public sealed class DoorShell : Shell<DoorState>
{
    protected override DoorState InitialState() => DoorState.Closed;

    public void Open() => Dispatch(DoorLogic.Open(), "Open");   // the label shows in the recorder
}
```

Add a `StepRecorder` component next to the shell and open **Window > Rulekit > Step Recorder**.

### Two rules to remember

- **Early-out with `Nothing`, never with `Put(state)`.** `Nothing` returns the same state instance.
  Shells and the recorder use reference equality to know that nothing happened, which is why an
  idle enemy ticks for free.
- **No `Time.time`, no `Random`, no statics in a rule.** Anything a rule depends on comes in as a
  parameter or lives in the state. If something slips in, **Re-run step** finds it.

The full manual ships with the package as `Assets/Rulekit/README.md`.

## Run the tests without Unity

One-time setup: in `Assets/Rulekit/Tools/CoreTests/`, drop the trailing `.txt` from
`CoreTests.csproj.txt` and `Program.cs.txt`. Then:

```
cd Assets/Rulekit/Tools/CoreTests
dotnet run
```

The last line reads `60 specs passed, no engine in sight.` Exit code is 0 on success and 1 on
failure, so it drops into CI as it is. This run compiles the same source files Unity compiles, so
it is the proof that the rules carry no engine dependency.

In Unity: **Window > General > Test Runner > EditMode > Run All**.

## How to report a bug

Open an issue with the **Bug report** template. Please include:

1. **A Step Recorder log.** In the Step Recorder window press **Copy log** and paste it into the
   issue. The log carries the dispatch labels, the field diffs, and the effects, which is usually
   the whole story. This is the single most useful thing you can attach.
2. Your **Unity version** and the **Rulekit version**.
3. What you expected, and what happened instead.
4. The smallest rule or shell that shows the problem.

Please do not paste Rulekit's package source into a public issue. Name the file and the member instead
(`Shell<TState>.Dispatch`), and the version. Your own code is fine to paste.

For anything you do not want in public — a licence or invoice question, or code you cannot share —
email **philip.meeske@outlook.com** instead. Licence, invoice and refund questions for the paid
asset are handled by Unity, not here.

## Repository contents

| | |
|---|---|
| `README.md` | this file |
| `CHANGELOG.md` | mirrors the `CHANGELOG.md` in the package |
| `.github/ISSUE_TEMPLATE/` | bug, question, and feature request forms |
| `docs/` | source of the project page published with GitHub Pages |
| `LICENSE` | covers **this repository only** — not the paid asset |

The Rulekit asset is licensed under the [Unity Asset Store End User License
Agreement](https://unity.com/legal/as-terms). Buying it does not give you any rights over the
source beyond that agreement, and nothing in this repository changes it.
