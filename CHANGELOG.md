# Changelog

All notable changes to the Rulekit Unity asset are listed here. This file mirrors the
`CHANGELOG.md` that ships inside the package. This project follows
[Semantic Versioning](https://semver.org/).

## 1.0.0 — 2026-10-03

First public release. Requires **Unity 2021.3 or newer**. No package dependencies, no
third-party DLLs, no render pipeline assumptions. Full C# source included.

### Core (no engine references)

- `Logic<S, A>` — the rule type. A chain takes a state and returns a value, the next state
  and a list of effects. It does not mutate and does not reference `UnityEngine`.
- API surface: `Return`, `Get`, `Gets`, `Put`, `Modify`, `Emit`, `Nothing`, `When`,
  `Sequence`, plus `Bind`, `Map` and `Then`.
- LINQ query syntax support through `Select` / `SelectMany`, so a rule can be written as
  `from s in Logic.Get<EnemyState>() …`.
- `Run(state)` executes a chain and returns a `Step` of value, state and effects.
- `Nothing` and a failing `When` return the same state instance, so shells and the recorder
  can detect "nothing happened" by reference equality.
- `Unit`, `Effect`, `Effects` (`Animate`, `Sound`, `Spawn`, `Despawn`, `Log`).
- `StepLog` — the ring buffer of recorded steps.
- `StateDiff` — reflection-based member-by-member comparison of two states.
- The `Rulekit.Core` and `Rulekit.Samples.Core` assembly
  definitions ship with `noEngineReferences` enabled, so `using UnityEngine;` in a core is a
  compile error rather than a code review note.

### Runtime

- `Shell<TState>` — base `MonoBehaviour` that owns the one mutable state copy, dispatches
  chains, records each step and resolves the resulting effects. `Dispatch` returns the
  chain's value. A dispatch raised from inside an effect handler is queued until the current
  step finishes, so steps never interleave.
- Effect resolution order: in-code `Handle<TEffect>(…)` handlers, then `EffectInterpreter`
  components on the same GameObject in component order, then `OnUnhandled` (warns by
  default, overridable to throw).
- `DefaultEffectInterpreter` — handles the generic effects: `Animate(trigger)` on the
  Animator, `Sound(id)` and `Spawn(id)` through id tables filled in the inspector,
  `Despawn(delay)` and `Log(message)`.
- `EffectInterpreter` — the base class for writing your own interpreter component.
- `IShell` — the non-generic shell surface the editor tooling talks to.
- `StepRecorder` — the recording component. Editor-only by default, configurable
  `Capacity` in steps, skips steps that returned the same state instance with no effects.

### Editor

- **Step Recorder window** (`Window > Rulekit > Step Recorder`):
  - step list with a changed/unchanged marker, step number, frame, dispatch label and a
    before → after state summary with effect count
  - **Follow** keeps the newest step selected
  - **Changed only** filter and a free-text label filter
  - **member diff** in the detail pane (`Health: 30 → 20`) plus the effects in fire order
  - **Restore BEFORE** / **Restore AFTER** set the pure state back to a recorded step.
    Calls `OnStateRestored()` on the shell; does not replay effects or rewind the scene.
  - **Re-run step** restores the Before state and dispatches the same chain again, then
    reports whether the new result matches the recording. On a divergence it lists the
    members that differ — this is the detector for a rule that reads `Time` or `Random`.
  - **Copy log** puts the whole buffer on the clipboard as text, diffs included.
- `StepRecorder` inspector showing the live state field by field during Play Mode, with an
  **Undo last step** button.
- **Demo scene** (`Samples/Demo/Demo.unity`, with `Loot.prefab` and `Enemy.prefab`) — open it
  and press Play.
- **Demo scene builder** (`Window > Rulekit > Create Demo Scene`) — rebuilds those three
  assets in place, so the shipped demo can always be reset to known-good.
- All components are grouped under `Add Component > Rulekit`.

### Samples

Four samples, each with a pure core and a thin shell:

- **Enemy** — perception, chase, attack cool-down, death with loot and despawn. Shows
  movement derived from state rather than emitted as an effect.
- **Door** — early-out by reference equality, composed rules (locking an open door closes
  it first).
- **Switch** — one shell driving another through a domain effect; trigger and click input.
- **Inventory** — chains that return values (`Add` returns the leftover, `Remove` returns
  success), copy-on-write over an array, a domain effect handled in code, and an IMGUI
  panel so it runs without a UI package.

Plus the demo glue (`DemoPatrol`, `DemoLoot`, `DemoRespawner`) used by the demo scene.
Every id a sample uses is a constant in that logic's `Ids` class.

### Tests — one spec suite, two runners, one source

Specs are `public static void` methods on static classes named `*Specs`. No attributes and
no test framework in the spec files themselves; both runners find them by reflection.

- **Unity Test Runner**: `Window > General > Test Runner > EditMode > Run All`.
- **Command line, without Unity installed**: the `Tools/CoreTests` project compiles the same
  `Core`, `Samples/Core` and `Tests/EditMode` source files against the .NET SDK and runs
  them with `dotnet run`. This is the proof that the cores really carry no engine
  dependency. See `Tools/CoreTests/README.md` for the two-step setup.

Coverage: the monad laws for `Logic`, reference-equality early-outs, effect ordering and
concatenation, `StepLog` capacity and eviction, `StateDiff` member comparison, and the
behaviour of all four sample cores.

### Known limitations

- Chains are built per call, so each dispatch allocates a few delegates. Fine for dozens of
  agents; for hundreds ticking every frame, move the per-frame input into the state and
  build the chain once.
- Effects are concatenated into new arrays on every `Bind`. Not zero-allocation.
- With a recorder present, each dispatch allocates one extra closure so the step can be
  re-run.
- Restore and re-run touch the pure state only. The scene is not rewound.
- No serialization, save/load or networking. States are your own classes.
- The Step Recorder window is IMGUI.
