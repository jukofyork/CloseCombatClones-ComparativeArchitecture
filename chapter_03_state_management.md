# Chapter 3: State Management Patterns in Tactical Wargames

## 3.1 The State Management Problem

Tactical wargames present a unique challenge: units must simultaneously track dozens of interacting states—posture, activity, orders, morale, health, and environmental factors. A soldier might be prone, crawling, suppressed, reloading, under concealment, and executing a move order all at once.

The complexity isn't merely the number of states, but their **interactions** and **constraints**:

- A unit cannot fire while reloading
- A prone soldier cannot sprint
- Suppression affects willingness to move
- Wounded units have reduced capabilities
- Orders modify available actions

This chapter examines three fundamental patterns for managing this complexity, derived from real-world implementations in the Close Combat clone ecosystem.

---

## 3.2 Pattern Taxonomy

State management systems fall into four fundamental categories based on their compositional model:

| Pattern | Composition Model | Best For | Complexity |
|---------|------------------|----------|------------|
| **Bitfield System** | Orthogonal bits | Capability tracking | Low |
| **State Hierarchy** | Containment tiers | Action sequencing | Medium |
| **Dual-State** | Parallel streams | Visual/logic separation | Low |
| **Finite State Machine** | Transitions & guards | Strict state control | High |

The three implementations we'll analyze represent the first three patterns:

- **OpenCombat-SDL**: Bitfield System (64 orthogonal states)
- **OpenCombat**: State Hierarchy (Phase → Behavior → Gesture)
- **CloseCombatFree**: Dual-State (Runtime Status + Health State)

---

## 3.3 Pattern 1: The Bitfield System

### 3.3.1 Core Philosophy: Orthogonal Composition

The bitfield approach treats states as **independent capabilities** that can coexist without conflict. Like permissions on a filesystem, each bit represents a single capability or condition.

**Key Insight**: Not all states are mutually exclusive. Being "prone" doesn't prevent being "reloading" or "suppressed." By encoding states as bits rather than a single value, we embrace this orthogonality.

```pseudocode
// State as a collection of capability flags
unit_state: 64-bit integer

// Setting a state
SET_BIT(unit_state, PRONE)
SET_BIT(unit_state, RELOADING)
SET_BIT(unit_state, SUPPRESSED)

// Result: unit is prone, reloading, AND suppressed simultaneously
```

### 3.3.2 Automatic Prerequisite Chaining

The bitfield system's true power emerges when combined with **action requirements**. Instead of hardcoding state transitions, actions declare their prerequisites, and the system automatically chains necessary state changes.

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
// 1. Check prerequisites: Needs STANDING, but has PRONE
// 2. Find action that adds STANDING: "StandUp"
// 3. Insert StandUp before RunTo
// 4. StandUp removes PRONE, adds STANDING
// 5. Now prerequisites met, execute RunTo
```

This creates **emergent intelligence**—soldiers automatically transition through appropriate intermediate states without explicit scripting.

### 3.3.3 Game Design Implications

**Advantages for Design:**

1. **Natural Capability Expression**: States like "CanFire" or "IsExposed" map directly to design concepts
2. **Compositional Flexibility**: New behaviors emerge from bit combinations without explicit coding
3. **Clear Invalidation**: Removing a bit instantly removes associated capabilities
4. **Data-Driven Actions**: Action definitions can live in external files (JSON, XML)

**Design Constraints:**

1. **Hard State Limit**: 64 bits (or whatever width chosen) is the maximum
2. **No Hierarchy**: Cannot express "Moving is a parent of Walking and Running"
3. **Mutual Exclusion Manual**: Standing and Prone being mutually exclusive requires external enforcement
4. **Debugging Challenge**: State 0x0000000000040210 is less readable than "Standing + Reloaded + Defending"

### 3.3.4 Determinism and Multiplayer

Bitfield systems are **naturally deterministic**—the same bit operations produce the same results across platforms. However:

- **Serialization is trivial**: A single integer transfers completely
- **Replay-friendly**: State changes are compact and precise
- **Desync detection**: XOR comparison quickly reveals state mismatches

For multiplayer, bitfield systems work best with **server-authoritative** models where the server validates state transitions before broadcasting.

---

## 3.4 Pattern 2: The State Hierarchy

### 3.4.1 Core Philosophy: Temporal Separation

The hierarchical approach organizes states by **timescale and authority**, creating distinct tiers that answer different questions:

| Tier | Question Answered | Duration | Update Frequency |
|------|------------------|----------|------------------|
| **Phase** | "What phase of the game are we in?" | Minutes to Hours | Event-driven |
| **Behavior** | "What is the unit trying to accomplish?" | Seconds to Minutes | As needed |
| **Gesture** | "What is the unit physically doing right now?" | Milliseconds to Seconds | Every frame |

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

The hierarchy isn't merely organizational—it defines **state propagation rules**:

```pseudocode
// Behavior propagation determines squad coordination
enum Propagation {
    ON_CHANGE,    // Notify squad once (Defend, Hide)
    REGULARLY,    // Update every frame (MoveTo, SneakTo)
    NEVER         // No propagation (Dead, Unconscious)
}

// When squad leader changes behavior:
// - If ON_CHANGE: Squad members update once
// - If REGULARLY: Members receive continuous updates (formation movement)
// - If NEVER: No impact on squad (individual death)
```

This creates **emergent squad behavior**—soldiers automatically coordinate based on the nature of their orders.

### 3.4.3 Temporal State Completion

The Gesture tier introduces **temporal constraints**—gestures complete at specific simulation times:

```pseudocode
// Gesture with completion time
gesture: Reloading(end_frame: 1847, weapon: RIFLE)

// Each simulation frame:
if current_frame >= gesture.end_frame {
    gesture = Idle
    behavior = determine_next_behavior()
}
```

This enables **deterministic animation**—a reloading soldier will finish at exactly frame 1847, regardless of frame rate or processing speed.

### 3.4.4 Game Design Implications

**Advantages for Design:**

1. **Clear Mental Model**: Developers reason about one tier at a time
2. **Natural Timescales**: Appropriate update frequencies per state type
3. **Explicit Relationships**: Hierarchy makes dependencies obvious
4. **Precision Timing**: Temporal gestures enable frame-accurate gameplay

**Design Constraints:**

1. **Verbosity**: Three enums instead of one state variable
2. **Synchronization Complexity**: All three tiers must stay consistent
3. **Learning Curve**: New team members must understand the tier model
4. **State Overflow**: Unclear which tier gets new states

### 3.4.5 Determinism and Multiplayer

The hierarchical pattern excels at **deterministic multiplayer**:

```pseudocode
// Message-driven state changes
message SetBehavior {
    unit_id: 47
    behavior: Defend(angle: 45°)
}

// Server broadcasts → All clients apply same message → Identical states
```

**Key Benefits:**
- **Replay Capability**: Message log recreates exact state sequence
- **Server Authority**: Clients request, server validates and broadcasts
- **Desync Recovery**: State snapshots at intervals, message replay to catch up

This pattern is essential for competitive multiplayer where **frame-perfect synchronization** matters.

---

## 3.5 Pattern 3: The Dual-State System

### 3.5.1 Core Philosophy: Separation of Concerns

The dual-state approach splits state into two parallel streams:

1. **Runtime Status**: The unit's current operational state (QString/string-based)
2. **Health State**: Visual/condition state managed declaratively

```pseudocode
// C++ side: Runtime status
define class Unit {
    property unitStatus: String  // "MOVING", "RELOADING", "READY"
    property healthState: String // "healthy", "damaged", "destroyed"
}

// QML/Visual side: Declarative states
states: [
    State { name: "healthy"; opacity: 1.0 }
    State { name: "damaged"; opacity: 0.7; smoke_effect: visible }
    State { name: "destroyed"; visible: false; wreck: visible }
]
```

### 3.5.2 Order Queue Integration

The dual-state system tightly couples to an **order queue**—each order maps to a status:

```pseudocode
// Order processing
define function process_queue() {
    for order in queue {
        if order.performed then continue
        
        match order.type {
            "Move" => {
                unitStatus = "MOVING"
                start_movement(order.target)
            }
            "Attack" => {
                unitStatus = "AIMING"
                start_aiming(order.target)
            }
        }
        order.performed = true
        break  // One order at a time
    }
}

// When animation completes:
unitStatus = "READY"
process_queue()  // Next order
```

### 3.5.3 Runtime Extensibility

The string-based runtime status provides **maximum flexibility**:

```pseudocode
// New status added in data, no code changes
unitStatus = "OVERWATCH"  // No recompilation needed
unitStatus = "BERSERK"    // Modder-defined state
unitStatus = "CONCEALED"  // Custom campaign state
```

This makes the dual-state pattern ideal for **moddable games** where designers (not programmers) define new states.

### 3.5.4 Game Design Implications

**Advantages for Design:**

1. **Maximum Moddability**: New states without recompilation
2. **Visual-Logic Separation**: Artists control visuals, designers control logic
3. **Simple Mental Model**: Just two state concepts to understand
4. **Human-Readable**: "MOVING" is clearer than 0x00000008

**Design Constraints:**

1. **No Orthogonal Composition**: A unit can only have one status at a time
2. **No Type Safety**: Typos create runtime errors ("MOVING" vs "MOvING")
3. **Implicit State Machine**: No explicit transition rules
4. **Limited Combinations**: Cannot express "Moving + Firing" as a single status

### 3.5.5 Determinism and Multiplayer

The dual-state pattern presents **challenges for multiplayer**:

- **String Synchronization**: More bandwidth than integers
- **No Validation**: Clients can set any status without server authority
- **Desync Risk**: Without server validation, states can diverge

**Mitigation Strategies:**
- Status enumeration (still strings, but validated against whitelist)
- Server authority with status requests rather than direct sets
- State hashes for periodic desync detection

This pattern is best suited for **single-player** or **cooperative multiplayer** where strict determinism is less critical than modding flexibility.

---

## 3.6 Decision Framework: Choosing Your Pattern

### 3.6.1 Decision Matrix

| Criteria | Bitfield | Hierarchy | Dual-State |
|----------|----------|-----------|------------|
| **Multiplayer Requirements** | Good | Excellent | Challenging |
| **Modding Support** | Limited | Moderate | Excellent |
| **Complex State Combinations** | Excellent | Good | Poor |
| **Team Experience Needed** | Low | High | Low |
| **Debugging Ease** | Moderate | Good | Excellent |
| **Determinism** | Excellent | Excellent | Moderate |

### 3.6.2 Decision Tree

```
Do you need deterministic multiplayer?
├── YES → Do you need complex state combinations?
│         ├── YES → Bitfield System
│         └── NO  → State Hierarchy
└── NO  → Is modding a priority?
          ├── YES → Dual-State System
          └── NO  → How complex are your states?
                    ├── Simple → Dual-State
                    └── Complex → Bitfield
```

### 3.6.3 Pattern Combinations

Modern tactical games often combine patterns:

**Recommended Hybrid:**
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

This provides:
- Clear hierarchy for game flow (from State Hierarchy)
- Orthogonal composition for capabilities (from Bitfield)
- Temporal precision for actions (from State Hierarchy)
- Room for modding via scriptable behaviors (from Dual-State)

---

## 3.7 Advanced Considerations

### 3.7.1 Determinism in Tactical Games

Determinism—the property that identical inputs produce identical outputs—is crucial for:

- **Replay systems**: Store inputs, replay simulation
- **Multiplayer synchronization**: Clients agree on state
- **Debugging**: Reproduce bugs reliably

**Pattern Impact on Determinism:**

| Pattern | Determinism | Considerations |
|---------|-------------|----------------|
| Bitfield | High | Fixed-width operations are deterministic |
| Hierarchy | High | Temporal gestures guarantee timing |
| Dual-State | Moderate | String comparison order may vary |

**Ensuring Determinism:**
- Use fixed-point math, not floating-point
- Frame-based timing, not wall-clock time
- Seeded random number generators
- Deterministic ordering (e.g., sort units before iteration)

### 3.7.2 Moddability Trade-offs

State management patterns significantly impact modding potential:

**Bitfield Limitations:**
- Fixed state count (64 bits = 64 states)
- Adding states requires code changes
- Enums must stay synchronized with data files

**Hierarchy Moderation:**
- New states require enum additions
- But behaviors can be data-driven components
- Lua/ scripting can extend without recompilation

**Dual-State Strengths:**
- New statuses defined in data
- No code changes for new states
- Modders can create entirely new unit types

**Recommendation**: If modding is a core feature, provide a **string-based overlay** on top of your primary state system for modder-defined states.

### 3.7.3 Save/Load Architecture

Different patterns suggest different serialization strategies:

**Bitfield Serialization:**
```pseudocode
save_game() {
    write_64bit(unit.state.bits)
}

load_game() {
    unit.state.bits = read_64bit()
}
```
- **Pros**: Trivial, compact, fast
- **Cons**: Opaque (hard to debug save files)

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
- **Pros**: Human-readable, versionable
- **Cons**: Larger files, more processing

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
- **Pros**: Human-readable, editable by modders
- **Cons**: Requires parsing, no validation

### 3.7.4 State Validation and Cheating Prevention

**The Problem**: In multiplayer, clients cannot be trusted. A hacked client might set "Reloaded" without completing the reload animation.

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
    // Only certain transitions allowed
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

**Recommendation**: Regardless of pattern, implement **server authority**—clients request state changes, the server validates and broadcasts.

---

## 3.8 Implementation Patterns in Pseudocode

### 3.8.1 Bitfield System Pattern

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
        // Automatic prerequisite resolution
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

### 3.8.2 State Hierarchy Pattern

```pseudocode
// Enumeration definitions
define Phase {
    Placement,
    Battle,
    End(Victorious, Reason)
}

define Behavior {
    // Movement
    MoveTo(path: Path),
    MoveFastTo(path: Path),
    SneakTo(path: Path),
    
    // Combat
    Idle(body: Body),
    Defend(angle: Angle),
    Hide(angle: Angle),
    EngageSoldier(target: Unit),
    
    // Terminal
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
    OnChange,    // Update once when behavior changes
    Regularly,   // Update every frame
    Never        // No propagation
}

define UnitState {
    phase: Phase
    behavior: Behavior
    gesture: Gesture
    
    function update(current_frame: integer) {
        // Gesture temporal completion
        if gesture.has_end_frame() {
            if current_frame >= gesture.end_frame {
                gesture = Gesture.Idle
                on_gesture_complete()
            }
        }
        
        // Behavior propagation to squad
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
    // Runtime status (C++ side)
    unit_status: string = "READY"
    
    // Health state (declarative/visual side)
    health_state: string = "healthy"
    
    // Order queue
    orders: List<Order>
    current_order: integer = -1
    
    // C++: Process next order in queue
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
            break  // One order at a time
        }
    }
    
    // C++: Handle hit
    function on_hit(attacker: Unit, position: Point) {
        cancel_orders()
        health_state = "damaged"
        // QML automatically handles visual changes
    }
}

// QML: Declarative visual states
define VisualUnit {
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

1. **No pattern is universal**: Each serves different game design goals
2. **Orthogonality matters**: Bitfields excel where states naturally combine
3. **Timescale separation reduces complexity**: Hierarchy pattern organizes thinking
4. **Moddability requires runtime flexibility**: Strings enable data-driven design
5. **Determinism is architectural**: Must be designed in, not added later

### 3.9.2 Recommended Architecture for New Projects

Based on the comparative analysis, a **hybrid approach** serves most tactical wargames:

**Core Principles:**

1. **Use Hierarchy for Timescale Separation**
   - Phase: Global game flow
   - Behavior: Tactical goals (data-driven components)
   - Gesture: Immediate actions with temporal data

2. **Use Bitfield for Orthogonal Capabilities**
   - CanMove, CanFire, IsSuppressed, IsProne
   - Query with: `(capabilities & REQUIRED) == REQUIRED`

3. **Enable Modding via Scriptable Behaviors**
   - Behaviors reference scripts: `behavior: { type: "Defend", script: "defensive.lua" }`
   - Lua scripts handle custom logic without recompilation

4. **Enforce Server Authority for Multiplayer**
   - Clients request state changes
   - Server validates against rules
   - Server broadcasts authoritative updates

### 3.9.3 Validation Checklist

Before finalizing your state architecture, verify:

- [ ] Can a unit be both prone AND reloading? (Orthogonality check)
- [ ] Can you serialize and deserialize game state? (Save/load check)
- [ ] Can two clients stay synchronized? (Multiplayer check)
- [ ] Can designers add new states without programmers? (Modding check)
- [ ] Can you reproduce bugs from save files? (Debugging check)
- [ ] Can you query "what is this unit doing?" easily? (Readability check)

### 3.9.4 Final Recommendations

**For Competitive Multiplayer Games:**
- Use State Hierarchy for deterministic, frame-perfect gameplay
- Add bitfield for capability queries
- Implement comprehensive server validation

**For Single-Player with Mod Support:**
- Use Dual-State for maximum flexibility
- Add bitfield overlay for capability tracking
- Expose state system to scripting

**For Retro/Minimalist Games:**
- Use pure Bitfield for simplicity
- Limit to 32 or 64 states maximum
- Externalize action definitions to data files

---

## 3.10 Conclusion

State management is the foundation upon which tactical gameplay is built. The choice between bitfield orthogonality, hierarchical organization, or dual-state separation shapes every other system in your game.

The Close Combat clones demonstrate that **there is no single correct answer**—each pattern emerged from different priorities:

- **OpenCombat-SDL** optimized for performance and emergent behavior through bitfields
- **OpenCombat** prioritized deterministic multiplayer through hierarchical states
- **CloseCombatFree** maximized modding potential through dual-state separation

Your choice should be guided by:
1. **Your primary platform** (single vs. multiplayer)
2. **Your community goals** (closed vs. moddable)
3. **Your team expertise** (simple vs. sophisticated)
4. **Your performance constraints** (cache efficiency vs. flexibility)

Whatever pattern you choose, commit to it fully—a hybrid implemented inconsistently is worse than any single pattern executed well.

**Next**: Chapter 4 examines how these state systems interact with world representation and terrain data structures.

---

## Appendix A: State Pattern Quick Reference

### Bitfield System
```
Best for: Orthogonal capabilities, memory efficiency
Avoid when: >64 states needed, complex transitions required
Key metric: State combinations = 2^n (n = bit count)
```

### State Hierarchy
```
Best for: Deterministic multiplayer, clear relationships
Avoid when: Rapid iteration without recompilation needed
Key metric: 3 tiers × update frequency = organization
```

### Dual-State
```
Best for: Moddability, visual separation, rapid prototyping
Avoid when: Type safety critical, complex state machines needed
Key metric: States = unlimited (string-based)
```

### Finite State Machine (Not Covered)
```
Best for: Strict state control, complex transition rules
Avoid when: Many orthogonal states, simple behavior
Key metric: States × Transitions = complexity
```

---

## Appendix B: Common State Design Pitfalls

1. **The God State**: Putting everything in one massive enum
2. **Implicit Transitions**: State changes scattered through code
3. **Mutual Exclusion Blindness**: Not enforcing Standing vs Prone
4. **Temporal Ambiguity**: Not tracking when states complete
5. **Network Naivety**: Assuming clients won't cheat
6. **Debug Hostility**: States that can't be easily inspected
7. **Save Incompatibility**: State changes breaking old saves

Avoid these by choosing the right pattern and implementing it consistently.
