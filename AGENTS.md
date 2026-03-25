# AGENTS.md

## Project
Unity prototype: **The Hole**  
Genre/mechanic: player controls a hole that consumes fruits and fruit pieces.

## Goal
Build a clean, expandable prototype with data-driven gameplay systems.

Current gameplay priorities:
1. Shelled fruits: `Closed -> Cracked -> Opened`
2. Spawn edible pieces after opening
3. Overripe timer / spoilage
4. Score reduction for overripe fruits
5. Sticky puddle
6. Hole slowdown inside sticky zones
7. Visual feedback layer
8. Final config/data cleanup

---

## Core development rules

### 1. Keep gameplay data-driven
Do not hardcode behavior for specific fruit names like `Coconut`, `Walnut`, etc.

Use configurable data objects for fruit behavior:
- base value
- shell enabled
- touches to open
- opened piece count
- spoilage enabled
- time to overripe
- overripe value multiplier
- auto-decay into pieces
- sticky puddle settings
- visual references

Preferred approach:
- `ScriptableObject` for static fruit configuration
- runtime state stored separately from config

---

### 2. Separate config from runtime state
Static data and live state must not be mixed.

Examples:
- config/data: `touchesToOpen`, `pieceCount`, `canSpoil`
- runtime state: current shell state, current freshness state, current touch count, already spawned pieces, already consumed

---

### 3. Separate logic from visuals
Gameplay code must not directly depend on specific sprites, materials, or VFX.

Use a dedicated visual component/layer that reacts to state changes:
- shell state changed
- fruit became overripe
- sticky puddle spawned

If visual assets are missing, the game must continue to work.

---

### 4. Preserve existing fruit behavior
Normal fruits without shell/spoilage mechanics must continue to behave exactly like before.

Any new system must be opt-in through data.

---

### 5. Avoid score duplication
There must be a single clear scoring flow.

Rules:
- no double reward from parent fruit + spawned pieces unless explicitly intended by data
- prevent repeated reward from the same object
- spawned pieces should use controlled value distribution

---

### 6. Prevent duplicate transitions
State transitions must be idempotent and safe.

Examples:
- opening should not trigger twice
- overripe transition should happen once
- piece spawning should happen once
- puddle spawn should happen once per intended event

---

### 7. Prefer small, local changes
Do not do broad refactors unless necessary for the requested feature.

When changing code:
- modify the smallest number of files possible
- keep inspector compatibility if possible
- do not rename unrelated classes/files
- do not introduce new frameworks

---

## Suggested code structure

Use or move toward structure like this:

- `Assets/Scripts/Core`
- `Assets/Scripts/Fruits`
- `Assets/Scripts/Fruits/Data`
- `Assets/Scripts/Fruits/Runtime`
- `Assets/Scripts/Fruits/Visuals`
- `Assets/Scripts/Hole`
- `Assets/Scripts/Zones`
- `Assets/Scripts/Score`

Possible responsibilities:
- `FruitDefinition` / `FruitData`: static config
- `FruitRuntimeState`: live state
- `FruitBehaviour`: root gameplay component
- `ShellFruitController`: shell logic
- `OverripeFruitController`: spoilage logic
- `FruitPieceSpawner`: spawn pieces
- `FruitVisualController`: visual updates only
- `StickyZone`: negative area behavior
- `HoleMovementController`: final movement calculation

---

## Coding conventions

### C#
- Prefer `private [SerializeField]` over public fields
- Prefer composition over deep inheritance
- Keep methods short and focused
- Use enums for explicit state machines
- Null-check optional references
- Avoid magic numbers
- Add concise comments only where behavior is non-obvious

### Unity
- Prefer clear MonoBehaviour responsibilities
- Avoid hidden side effects in `Update`
- Use events/callbacks when state changes need visual refresh
- Use prefabs/config assets instead of scene-only hardcoding
- Keep physics interactions deterministic and simple

---

## State model expectations

### Shell state
For shelled fruits:
- `Closed`
- `Cracked`
- `Opened`

Behavior:
- touching the hole advances shell progress
- fruit is not fully edible until `Opened`

### Freshness state
For spoilable fruits:
- `Fresh`
- `Overripe`

Architecture should allow adding later:
- `Rotten`

---

## When implementing a task
When returning work, always include:

1. **Changed files/classes**
2. **What was added**
3. **Where the feature is configured in Inspector/data**
4. **Any required prefab or scene setup**
5. **Manual test checklist**

---

## Manual test expectations
For gameplay tasks, verify at least:

- normal fruits still work
- shelled fruits cannot be eaten before opened
- shell opens after configured number of touches
- pieces spawn once
- score is not duplicated
- overripe transition happens once
- sticky zone applies slowdown while inside
- speed restores after exit
- missing visual assets do not break gameplay

---

## Non-goals for now
Do not spend time on:
- advanced VFX polish
- final art pipeline
- audio systems
- save/load
- optimization unless a change obviously causes a problem

Focus on playable prototype logic first.