# Chapter 3: State Management Patterns in Tactical Wargames

*Previous: [Chapter 2: Architectural Philosophy](chapter_02_architectural_philosophy.md)*

---

## 3.1 The State Management Problem

Tactical wargames face a distinct challenge. Units must track dozens of interacting states at once—posture, activity, orders, morale, health, and environmental factors. A single soldier might be prone, crawling, suppressed, reloading, concealed, and moving to a new position simultaneously.

The difficulty lies in how these states interact and constrain each other:

- Reloading prevents firing
- Prone soldiers cannot sprint
- Suppression reduces movement willingness
- Wounds degrade performance
- Orders limit available actions

This chapter explores three core patterns for handling this complexity, all drawn from real implementations in the Close Combat clone ecosystem.

---

## 3.2 Pattern Taxonomy

State management systems divide into three basic types based on their composition:

| Pattern           | Composition Model | Best For                | Complexity |
| ----------------- | ----------------- | ----------------------- | ---------- |
| **Bitfield**      | Orthogonal bits   | Capability tracking     | Low        |
| **Hierarchy**     | Containment tiers | Action sequencing       | Medium     |
| **Dual-State**    | Parallel streams  | Visual/logic separation | Low        |

We'll examine three implementations covering these patterns:

- **OpenCombat-SDL**: Bitfield (64 orthogonal states)
- **OpenCombat**: Hierarchy (Phase → Behavior → Gesture)
- **CloseCombatFree**: Dual-State (Runtime Status + Health State)

---

## 3.3 Pattern 1: Bitfield

### 3.3.1 Core Philosophy: Orthogonal Composition

The bitfield approach treats states as independent capabilities that coexist without conflict. Like filesystem permissions, each bit represents one capability or condition.

**Key Insight**: States don't always exclude each other. A unit can be prone while reloading or suppressed. Encoding states as bits instead of a single value preserves this independence.

```pseudocode
// State as a collection of capability flags
unit_state: 64-bit integer

// Setting a state
SET_BIT(unit_state, PRONE)
SET_BIT(unit_state, RELOADING)
SET_BIT(unit_state, SUPPRESSED)

// Result: unit is prone, reloading, and suppressed at the same time
```

### 3.3.2 Automatic Prerequisite Chaining

The bitfield pattern shines when paired with action requirements. Actions declare their prerequisites, and the system chains necessary state changes automatically.

```pseudocode
// Action definition
define Action {
    name: "RunTo"
    duration: 2.0 seconds
    requires: [STANDING, RELOADED]
    adds: [MOVING, RUNNING]
    removes: [STOPPED, WALKING, CRAWLING]
}

// When a prone soldier tries to run:
// 1. Check prerequisites: Needs STANDING, but unit is PRONE
// 2. Find action that adds STANDING: "StandUp"
// 3. Insert StandUp before RunTo
// 4. StandUp removes PRONE, adds STANDING
// 5. Prerequisites met, execute RunTo
```

This creates natural intelligence—units transition through intermediate states without explicit scripting.

### 3.3.3 Game Design Implications

**Advantages for Design:**

1. **Direct Capability Mapping**: States like "CanFire" or "IsExposed" align with design concepts
2. **Flexible Composition**: New behaviors emerge from bit combinations without extra code
3. **Instant Invalidation**: Removing a bit immediately disables its capabilities
4. **External Action Definitions**: Actions can be stored in JSON or XML files

**Design Constraints:**

1. **Fixed State Limit**: Only 64 bits (or the chosen width) are available
2. **Flat Structure**: No built-in hierarchy—"Moving" can't automatically include "Walking" and "Running"
3. **Manual Exclusivity**: Standing and prone must be enforced externally
4. **Debugging Difficulty**: State 0x0000000000040210 is harder to read than "Standing + Reloaded + Defending"

### 3.3.4 Determinism and Multiplayer

Bitfield patterns are inherently deterministic—the same operations produce identical results across platforms. Still, some considerations apply:

- **Simple Serialization**: A single integer transfers all state data
- **Replay Compatibility**: State changes are compact and precise
- **Desync Detection**: XOR comparison quickly spots state mismatches

For multiplayer, bitfield patterns work best when the server validates state transitions before broadcasting.

---

## 3.4 Pattern 2: Hierarchy

### 3.4.1 Core Philosophy: Temporal Separation

The hierarchy approach sorts states by timescale and authority, creating tiers that handle specific questions:

| Tier         | Question Answered                              | Duration                 | Update Frequency |
| ------------ | ---------------------------------------------- | ------------------------ | ---------------- |
| **Phase**    | "What phase of the game are we in?"            | Minutes to hours         | Event-driven     |
| **Behavior** | "What is the unit trying to accomplish?"       | Seconds to minutes       | As needed        |
| **Gesture**  | "What is the unit physically doing right now?" | Milliseconds to seconds  | Every frame      |

```pseudocode
// Three-tier state representation
unit_state: {
    phase: Phase          // Global game state
    behavior: Behavior    // Tactical goal
    gesture: Gesture      // Immediate action
}

// Example: Soldier moving to cover
unit_state = {
    phase: BATTLE
    behavior: MoveTo(cover_position)
    gesture: Crawling(end_time: 15.3s)
}
```

### 3.4.2 Tier Relationships and Propagation

The hierarchy pattern establishes state propagation rules:

```pseudocode
// Behavior propagation determines squad coordination
enum Propagation {
    ON_CHANGE,    // Notify squad once (Defend, Hide)
    REGULARLY,    // Update every frame (MoveTo, SneakTo)
    NEVER         // No propagation (Dead, Unconscious)
}

// When squad leader changes behavior:
// - Squad members update once for ON_CHANGE
// - Members receive continuous updates for REGULARLY (formation movement)
// - No impact on squad for NEVER (individual death)
```

Soldiers coordinate automatically based on their orders, creating natural squad behavior.

### 3.4.3 Temporal State Completion

The Gesture tier uses temporal constraints—gestures complete at specific simulation times:

```pseudocode
// Gesture with completion time
gesture: Reloading(end_frame: 1847, weapon: RIFLE)

// Each simulation frame:
if current_frame >= gesture.end_frame {
    gesture = Idle
    behavior = determine_next_behavior()
}
```

A reloading soldier finishes at exactly frame 1847, ensuring consistent animation regardless of frame rate or processing speed.

### 3.4.4 Game Design Implications

**Advantages for Design:**

1. Developers can focus on one tier at a time
2. Each state type updates at an appropriate frequency
3. The hierarchy clarifies dependencies
4. Temporal gestures allow frame-accurate gameplay

**Design Constraints:**

1. Three enums replace a single state variable
2. All three tiers must remain consistent
3. New team members need to learn the tier model
4. Some states don't fit neatly into any tier

### 3.4.5 Determinism and Multiplayer

The hierarchy pattern works well for deterministic multiplayer:

```pseudocode
// Message-driven state changes
message SetBehavior {
    unit_id: 47
    behavior: Defend(angle: 45°)
}

// Server broadcasts → All clients apply same message → States match
```

**Key Benefits:**
- Message logs recreate exact state sequences for replays
- The server validates and broadcasts client requests
- State snapshots and message replay help recover from desyncs

This approach matters most in competitive multiplayer, where synchronization must be frame-perfect.

---

## 3.5 Pattern 3: Dual-State

### 3.5.1 Core Philosophy: Separation of Concerns

The dual-state approach divides state into two parallel streams:

1. **Runtime Status**: The unit's current operational state, stored as a string
2. **Health State**: Visual and condition state, managed declaratively

```pseudocode
// Logic side: Runtime status
class Unit {
    string unitStatus;   // "MOVING", "RELOADING", "READY"
    string healthState;  // "healthy", "damaged", "destroyed"
};

// Visual side: Declarative states (QML/declarative syntax)
states: [
    State { name: "healthy"; opacity: 1.0 }
    State { name: "damaged"; opacity: 0.7; smoke_effect: visible }
    State { name: "destroyed"; visible: false; wreck: visible }
]
```

### 3.5.2 Order Queue Integration

The system couples tightly with an order queue, where each order maps to a status:

```pseudocode
// Order processing
void process_queue() {
    for (auto& order : queue) {
        if (order.performed) continue;

        switch (order.type) {
            case "Move":
                unitStatus = "MOVING";
                start_movement(order.target);
                break;
            case "Attack":
                unitStatus = "AIMING";
                start_aiming(order.target);
                break;
        }
        order.performed = true;
        break;  // Process one order at a time
    }
}

// When animation completes:
unitStatus = "READY";
process_queue();  // Move to next order
```

### 3.5.3 Runtime Extensibility

String-based runtime status enables flexibility without code changes:

```pseudocode
// New status added in data files
unitStatus = "OVERWATCH";  // No recompilation needed
unitStatus = "BERSERK";    // Modder-defined state
unitStatus = "CONCEALED";  // Custom campaign state
```

This makes the pattern well-suited for moddable games, where designers—not programmers—define new states.

### 3.5.4 Game Design Implications

**Advantages:**
- New states can be added without recompilation
- Artists control visuals while designers handle logic
- Simple mental model with just two state concepts
- Human-readable statuses like "MOVING" instead of bitflags

**Constraints:**
- Units can only have one status at a time
- Typos in strings cause runtime errors
- No explicit transition rules between states
- Cannot combine states like "Moving + Firing"

### 3.5.5 Determinism and Multiplayer

The dual-state pattern creates multiplayer challenges:

- String synchronization uses more bandwidth than integers
- Clients can set any status without server validation
- States may diverge without proper synchronization

**Mitigation approaches:**
- Validate status strings against a whitelist
- Require server approval for status changes
- Use state hashes for desync detection

This pattern works best for single-player or cooperative games where modding flexibility matters more than strict determinism.

---

## 3.6 Decision Framework: Choosing Your Pattern

### 3.6.1 Decision Matrix

| Criteria                       | Bitfield  | Hierarchy | Dual-State  |
| ------------------------------ | --------- | --------- | ----------- |
| **Multiplayer Requirements**   | Good      | Excellent | Challenging |
| **Modding Support**            | Limited   | Moderate  | Excellent   |
| **Complex State Combinations** | Excellent | Good      | Poor        |
| **Team Experience Needed**     | Low       | High      | Low         |
| **Debugging Ease**             | Moderate  | Good      | Excellent   |
| **Determinism**                | Excellent | Excellent | Moderate    |

### 3.6.2 Decision Tree

When choosing a pattern, start by asking:

Do you need deterministic multiplayer?
- If yes, ask: Do you need complex state combinations?
  - Yes? Use a bitfield.
  - No? Use a hierarchy.
- If no, ask: Is modding a priority?
  - Yes? Use a dual-state.
  - No? Consider your state complexity:
    - Simple states? Use dual-state.
    - Complex states? Use bitfield.

### 3.6.3 Pattern Combinations

Most modern tactical games blend multiple patterns. Here's a proven hybrid approach:

```pseudocode
unit_state: {
    // Hierarchy for high-level flow
    phase: Phase           // Enum
    behavior: Behavior     // Component-based, scriptable

    // Bitfield for orthogonal capabilities
    capabilities: Bitfield // CanMove, CanFire, IsProne, etc.

    // Gesture with temporal data
    gesture: Gesture       // Enum + completion_time
}
```

This design delivers:
- Clear hierarchy for game flow
- Orthogonal composition for unit capabilities
- Precise timing for actions
- Modding flexibility through scriptable behaviors

---

## 3.7 Advanced Considerations

### 3.7.1 Determinism in Tactical Games

Determinism ensures identical inputs produce identical outputs. This matters for:

- **Replay systems**: Storing inputs to replay the simulation later
- **Multiplayer synchronization**: Keeping clients in agreement on game state
- **Debugging**: Reproducing bugs reliably

**How Patterns Affect Determinism:**

| Pattern    | Determinism | Considerations                         |
| ---------- | ----------- | -------------------------------------- |
| Bitfield   | High        | Fixed-width operations avoid surprises |
| Hierarchy  | High        | Temporal gestures lock down timing     |
| Dual-State | Moderate    | String comparisons may vary by system  |


**How to Ensure Determinism:**
- Replace floating-point math with fixed-point
- Use frame-based timing instead of wall-clock time
- Seed random number generators
- Sort units before iteration to guarantee order

### 3.7.2 Moddability Trade-offs

State management patterns shape what modders can do:

**Bitfield Limitations:**
- Hard limit of 64 states
- Adding states means changing code
- Enums must match data files exactly

**Hierarchy Moderation:**
- New states still require enum changes
- Behaviors can live in data-driven components
- Scripting languages like Lua allow extensions without recompiling

**Dual-State Strengths:**
- New statuses defined in data files
- No code changes needed for new states
- Modders can create entirely new unit types

**Recommendation:** If modding is important, add a string-based overlay to your primary state system. This lets modders define their own states.

### 3.7.3 Save/Load Architecture

Each pattern suggests a different way to handle saves:

**Bitfield Serialization:**
```pseudocode
save_game() {
    write_64bit(unit.state.bits)
}

load_game() {
    unit.state.bits = read_64bit()
}
```
- **Pros:** Simple, compact, fast
- **Cons:** Opaque—save files are hard to debug

**Hierarchy Serialization:**
```pseudocode
save_game() {
    write_json({
        phase: unit.phase,
        behavior: serialize_behavior(unit.behavior),
        gesture: serialize_gesture(unit.gesture)
    })
}
```
- **Pros:** Human-readable, versionable
- **Cons:** Larger files, more processing

**Dual-State Serialization:**
```pseudocode
save_game() {
    write_qml({
        unitStatus: unit.unitStatus,
        state: unit.healthState,
        x: unit.x, y: unit.y
    })
}
```
- **Pros:** Human-readable, editable by modders
- **Cons:** Requires parsing, no built-in validation

### 3.7.4 State Validation and Cheating Prevention

**The Problem:** In multiplayer, clients can't be trusted. A hacked client might claim a unit has reloaded without completing the animation.

**Bitfield Validation:**
```pseudocode
server_validate(action, unit_state) {
    if action.requires & ~unit_state.bits {
        return INVALID  // Client claims state they don't have
    }
    return VALID
}
```

**Hierarchy Validation:**
```pseudocode
server_validate(behavior_request, current_state) {
    // Only allow certain transitions
    if current_state.gesture != Idle {
        return INVALID  // Can't change behavior mid-gesture
    }
    return VALID
}
```

**Dual-State Validation:**
```pseudocode
server_validate(status_request, current_status) {
    // Whitelist valid transitions
    valid_transitions = {
        "READY": ["MOVING", "AIMING", "RELOADING"],
        "MOVING": ["READY", "STOPPED"]
    }

    if status_request not in valid_transitions[current_status] {
        return INVALID
    }
    return VALID
}
```

**Recommendation:** No matter which pattern you choose, make the server the authority. Clients request state changes, but the server validates and broadcasts them.

---

## 3.8 Implementation Patterns in Pseudocode

### 3.8.1 Bitfield Pattern

```pseudocode
define State {
    bits: 64-bit unsigned integer

    function is_set(state_id: integer): boolean {
        flag = 1 << state_id
        return (bits & flag) != 0
    }

    function set(state_id: integer) {
        flag = 1 << state_id
        bits = bits | flag
    }

    function clear(state_id: integer) {
        flag = ~(1 << state_id)
        bits = bits & flag
    }
}

define Action {
    name: string
    duration: float
    requires: bitmask
    adds: bitmask
    removes: bitmask
}

define StateMachine {
    current_state: State
    action_queue: List<Action>

    function queue_action(action: Action) {
        missing = action.requires & ~current_state.bits

        while missing != 0 {
            prerequisite = find_action_adding(missing.first_bit())
            action_queue.prepend(prerequisite)
            missing = missing & ~prerequisite.adds
        }

        action_queue.append(action)
    }

    function update(delta_time: float) {
        if action_queue.empty() then return

        current = action_queue.first()

        if current.meets_requirements(current_state) {
            current.execute(current_state)
            action_queue.remove_first()
        }
    }
}
```

### 3.8.2 Hierarchy Pattern

```pseudocode
define Phase {
    Placement,
    Battle,
    End
}

define Behavior {
    MoveTo(path: Path),
    MoveFastTo(path: Path),
    SneakTo(path: Path),
    Idle(body: Body),
    Defend(angle: Angle),
    Hide(angle: Angle),
    EngageSoldier(target: Unit),
    Dead,
    Unconscious
}

define Gesture {
    Idle,
    Reloading(end_frame: integer, weapon: Weapon),
    Aiming(end_frame: integer, weapon: Weapon),
    Firing(end_frame: integer, weapon: Weapon)
}

define StatePropagation {
    OnChange,
    Regularly,
    Never
}

define UnitState {
    phase: Phase
    behavior: Behavior
    gesture: Gesture

    function update(current_frame: integer) {
        if gesture.has_end_frame() {
            if current_frame >= gesture.end_frame {
                gesture = Gesture.Idle
                on_gesture_complete()
            }
        }

        if behavior.changed() {
            propagation = behavior.propagation()
            if propagation == OnChange {
                notify_squad()
            }
        }
    }
}
```

### 3.8.3 Dual-State Pattern

```pseudocode
define Unit {
    unit_status: string = "READY"
    health_state: string = "healthy"
    orders: List<Order>
    current_order: integer = -1

    function continue_queue() {
        for i in 0..orders.length() {
            order = orders[i]
            if order.performed then continue

            current_order = i

            match order.type {
                "Move" => {
                    unit_status = "MOVING"
                    perform_movement(order.target, speed=normal)
                }
                "MoveFast" => {
                    unit_status = "MOVING FAST"
                    perform_movement(order.target, speed=fast)
                }
                "Attack" => {
                    unit_status = "AIMING"
                    perform_attack(order.target)
                }
            }

            order.performed = true
            break
        }
    }

    function on_hit(attacker: Unit, position: Point) {
        cancel_orders()
        health_state = "damaged"
    }
}

define VisualUnit {
    // Note: QML/declarative syntax for visual state management
    states: [
        State {
            name: "healthy"
            PropertyChanges { opacity: 1.0 }
        },
        State {
            name: "damaged"
            PropertyChanges { opacity: 0.7 }
            PropertyChanges { smoke_effect.visible: true }
        },
        State {
            name: "destroyed"
            PropertyChanges { visible: false }
            PropertyChanges { wreck.visible: true }
        }
    ]

    transitions: [
        Transition {
            from: "healthy"; to: "damaged"
            PropertyAnimation { property: "opacity"; duration: 500 }
        }
    ]
}
```

---

## 3.9 Synthesis: Designing Your State Architecture

### 3.9.1 Key Insights from Comparative Analysis

1. Different patterns serve different game design goals.
2. Orthogonal states work best with bitfields.
3. Hierarchical organization simplifies complex state management by separating timescales.
4. Modding requires runtime flexibility, which strings and scripts provide.
5. Determinism must be part of the initial design.

### 3.9.2 Recommended Architecture for New Projects

Most tactical wargames benefit from a hybrid approach:

**Core Principles:**

1. **Separate timescales with hierarchy**
   - Global game flow (Phase)
   - Tactical goals (Behavior, using data-driven components)
   - Immediate actions with timing data (Gesture)

2. **Track orthogonal capabilities with bitfields**
   - CanMove, CanFire, IsSuppressed, IsProne
   - Query with: `(capabilities & REQUIRED) == REQUIRED`

3. **Support modding through scriptable behaviors**
   - Behaviors reference scripts: `behavior: { type: "Defend", script: "defensive.lua" }`
   - Lua scripts handle custom logic without recompiling

4. **Maintain server authority in multiplayer**
   - Clients request state changes.
   - The server validates against game rules.
   - The server broadcasts authoritative updates.

### 3.9.3 Validation Checklist

Before finalizing your state architecture, verify:

- [ ] Can a unit reload while prone? (Orthogonality)
- [ ] Does the game save and load state correctly? (Serialization)
- [ ] Do clients stay synchronized in multiplayer? (Network consistency)
- [ ] Can designers add new states without programmer help? (Modding)
- [ ] Can bugs be reproduced from save files? (Debugging)
- [ ] Can you easily query a unit's current action? (Readability)

### 3.9.4 Final Recommendations

**For Competitive Multiplayer Games:**
Use hierarchical states for deterministic, frame-perfect gameplay. Add bitfields for capability checks. Implement strict server validation.

**For Single-Player with Mod Support:**
Use dual-state for flexibility. Layer bitfields for capability tracking. Expose the state system to scripting.

**For Retro/Minimalist Games:**
Use pure bitfields for simplicity. Limit to 32 or 64 states. Define actions in external data files.

---

## 3.10 Conclusion

State management forms the foundation of tactical gameplay. Whether you choose bitfield, hierarchy, or dual-state patterns determines how every other system in your game behaves.

The Close Combat clones show that priorities shape the best approach:

- **OpenCombat-SDL** used bitfields for performance and emergent behavior.
- **OpenCombat** relied on hierarchical states for deterministic multiplayer.
- **CloseCombatFree** favored dual-state to maximize modding.

Your choice depends on:
1. Your target platform (single or multiplayer).
2. Your community goals (closed or moddable).
3. Your team's expertise (simple or sophisticated).
4. Your performance constraints (cache efficiency or flexibility).

Commit fully to your chosen pattern. A poorly implemented hybrid causes more problems than a well-executed single approach.

---

## Appendix A: State Pattern Quick Reference

### Bitfield
```
Best for: Orthogonal capabilities where memory matters
Avoid when: You need more than 64 states or complex transitions
Key metric: State combinations equal 2ⁿ (n = bit count)
```

### Hierarchy
```
Best for: Deterministic multiplayer with clear relationships
Avoid when: You need rapid iteration without recompiling
Key metric: Three tiers multiplied by update frequency determine organization
```

### Dual-State
```
Best for: Modding, visual separation, and rapid prototyping
Avoid when: Type safety is critical or you need complex state machines
Key metric: States can be unlimited (string-based)
```

---

## Appendix B: Common State Design Pitfalls

1. **The God State** – A single massive enum handling everything
2. **Implicit Transitions** – State changes scattered across the codebase
3. **Mutual Exclusion Blindness** – Failing to enforce Standing vs. Prone rules
4. **Temporal Ambiguity** – Not tracking when states complete
5. **Network Naivety** – Assuming clients won't cheat
6. **Debug Hostility** – States that resist inspection
7. **Save Incompatibility** – State changes breaking old save files

---

*Next: [Chapter 4: World and Terrain Architecture](chapter_04_world_terrain.md)*
