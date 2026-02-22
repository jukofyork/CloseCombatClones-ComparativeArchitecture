# Chapter 2: Architectural Philosophy

## The Foundations of Tactical Simulation Design

*Three implementations, two decades, one eternal question: how do we model the complexity of combat in software?*

---

## 2.1 The Nature of Tactical Wargame Simulation

### 2.1.1 What Makes Close Combat Special

The Close Combat series occupies a unique position in the landscape of real-time strategy games. Unlike traditional RTS games where units are expendable resources, Close Combat treats each soldier as an individual—with fears, skills, and a will to fight that can break under pressure. This philosophy of "combat at the human scale" creates architectural challenges that distinguish tactical wargames from other genres.

**The Core Distinctions:**

| Feature | Traditional RTS | Tactical Wargame |
|---------|-----------------|------------------|
| **Unit Count** | Hundreds to thousands | Dozens to hundreds |
| **Unit Identity** | Generic ("Rifleman #47") | Individual ("Pvt. John Miller") |
| **Morale System** | Binary (alive/dead) | Continuous (suppressed → panicked → routed) |
| **Time Scale** | Abstract (minutes per second) | Real-time (1:1 or compressed) |
| **Combat Resolution** | Hit points, deterministic | Ballistics, probabilistic |
| **Terrain Impact** | Binary (passable/blocked) | Complex (cover, concealment, line-of-sight) |

These distinctions create fundamental tensions in architecture design. A system optimized for managing 10,000 identical units fails when asked to simulate 50 individuals with unique states, complex morale interactions, and realistic ballistics.

### 2.1.2 The Simulation vs. Game Balance

Every tactical wargame faces an eternal tension between **simulation authenticity** and **gameplay clarity**. This tension manifests in architectural decisions:

**Authenticity Demands:**
- Realistic ballistics with projectile physics
- Detailed morale modeling based on psychology research
- Complex line-of-sight calculations
- Authentic weapon characteristics and limitations
- Realistic command and communication delays

**Playability Requires:**
- Clear feedback for player actions
- Predictable unit responses
- Manageable information display
- Reasonable learning curves
- Engaging pacing

The three implementations examined in this book represent different points on this spectrum:

```
Authenticity ◀────────────────────────────────────▶ Playability
     │                                               │
     │  OpenCombat-SDL                               │
     │  (Deep simulation)                            │
     │       │                                       │
     │       │     OpenCombat                        │
     │       │     (Balanced)                        │
     │       │          │                            │
     │       │          │     CloseCombatFree        │
     │       │          │     (Accessible)           │
     └───────┴──────────┴────────────────────────────┘
```

### 2.1.3 The Authenticity Paradox

Counterintuitively, greater authenticity sometimes reduces the sense of realism. When soldiers behave too erratically—refusing orders due to morale checks, taking cover unpredictably, or panicking at unexpected moments—players perceive the system as "broken" rather than "realistic." The architecture must therefore model reality while maintaining player agency and predictability.

**The Three-Tier Approach to Authenticity:**

1. **Physical Layer**: Ballistics, line-of-sight, and movement physics must be accurate. Players accept that bullets follow realistic trajectories.

2. **Tactical Layer**: Unit behaviors should reflect training and doctrine. A squad ordered to defend should establish fields of fire and use cover.

3. **Psychological Layer**: Morale and suppression effects should create tension without frustration. Soldiers should become less effective when suppressed, not randomly disobedient.

---

## 2.2 The Three Philosophical Approaches

Over two decades, three distinct architectural philosophies emerged from attempts to implement Close Combat-style gameplay. Each philosophy reflects the era of its creation, the constraints of its technology, and the vision of its creators.

### 2.2.1 OpenCombat-SDL (2005–2008): The Simulation-First Philosophy

**Core Philosophy**: *"Model reality as accurately as possible within computational constraints. Let the simulation emerge from authentic individual behaviors."*

OpenCombat-SDL represents the classical approach to simulation software—deeply influenced by military modeling and simulation traditions of the late 1990s and early 2000s.

#### Key Tenets

**1. Object-Oriented Reality Mapping**

The world is composed of objects with state and behavior. Model reality through object relationships.

```
                    ┌─────────────┐
                    │    Object   │  ← Base class with position,
                    │             │    orders, selection, health
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           │               │               │
      ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
      │ Soldier │    │ Vehicle │    │  Squad  │
      └─────────┘    └─────────┘    └─────────┘
```

This inheritance hierarchy reflects how humans conceptualize military organizations:
- A soldier IS-A combatant with individual attributes
- A vehicle IS-A mobile platform with crew requirements
- A squad IS-A group that coordinates multiple combatants

**2. Deep State Modeling with Bitfields**

OpenCombat-SDL treats unit states as orthogonal capabilities encoded in a 64-bit bitfield:

```cpp
// State as capability composition
class State {
    uint64_t _bits;
    
public:
    void Set(unsigned int state) { _bits |= (1ULL << state); }
    void Clear(unsigned int state) { _bits &= ~(1ULL << state); }
    bool IsSet(unsigned int state) const { return (_bits >> state) & 1; }
};

// A soldier can be prone, reloading, AND suppressed simultaneously
State soldierState;
soldierState.Set(StateProne);
soldierState.Set(StateReloading);
soldierState.Set(StateSuppressed);
```

This approach enables emergent behavior—states combine in ways not explicitly programmed.

**3. Automatic Prerequisite Chaining**

Actions declare their requirements, and the system automatically inserts prerequisite actions:

```cpp
// Action definition in SoldierActions.txt
Action RunTo {
    Duration: 2000ms
    Requires: [Standing, Reloaded]
    Adds: [Moving, Running]
    Removes: [Stopped, Walking, Crawling]
}

// When a prone soldier tries to run:
// 1. System detects missing prerequisite: Standing
// 2. Finds action that adds Standing: StandUp
// 3. Inserts StandUp before RunTo
// 4. Soldier stands, then runs—automatically
```

**4. XML for Data, C++ for Behavior**

A clean separation between configurable parameters (XML) and hardcoded logic (C++):

```xml
<!-- config/Soldiers.xml -->
<Soldier>
    <Name>Garand</Name>
    <WalkingSpeed>2.5</WalkingSpeed>
    <RunningSpeed>5.36</RunningSpeed>
    <PrimaryWeapon>M1 Garand</PrimaryWeapon>
</Soldier>
```

```cpp
// src/objects/Soldier.cpp - Behavior hardcoded
void Soldier::Simulate(long dt, World* world) {
    // Update position based on current action
    // Check line-of-sight to enemies
    // Update morale based on nearby threats
    // All behavior logic in C++
}
```

#### Why This Philosophy?

**Historical Context (2005–2008):**
- Object-oriented design dominated software engineering
- C++ was the lingua franca of game development
- Military simulation software heavily influenced game design
- Modding was an afterthought, not a design goal
- Single-player focus with no multiplayer requirements

**Technical Advantages:**
- Intuitive mapping from game concepts to code
- Strong encapsulation prevents unintended interactions
- Natural fit for simulation-style games
- Excellent performance on hardware of the era

**Philosophical Trade-offs:**

| Advantage | Disadvantage |
|-----------|--------------|
| Intuitive modeling | Rigid inheritance hierarchies |
| Encapsulation | Behavior requires recompilation |
| Natural for simulation | Difficult to add new unit types |
| Clear organization | Object reference complexity |

### 2.2.2 CloseCombatFree (2011–2012): The Modding-First Philosophy

**Core Philosophy**: *"Content should be defined declaratively, not procedurally. Modders should create without recompiling. The UI is the game."*

CloseCombatFree emerged during the rise of mobile gaming and declarative UI frameworks. Its author recognized that the barrier between developer and modder was artificial—why require compilation for content creation?

#### Key Tenets

**1. Maximum Extensibility Through QML**

QML (Qt Meta Language) provides a declarative JavaScript-like syntax for defining both UI and game content:

```qml
// units/tanks/HeavyTank.qml
Tank {
    id: root
    unitType: "Heavy Tank"
    maxSpeed: 18
    
    // Visual composition
    HeavyTankHull { id: hull }
    HeavyTankTurret { id: turret }
    
    // Crew composition
    Soldier { role: "Commander" }
    Soldier { role: "Gunner" }
    Soldier { role: "Driver" }
    
    // Behavior defined in QML
    function onEnemySpotted(enemy) {
        if (distanceTo(enemy) < 200) {
            turret.rotateToward(enemy)
            fire()
        }
    }
}
```

**2. Content as Code**

In CloseCombatFree, the distinction between "game code" and "game data" dissolves. Scenarios, units, and maps are all QML files:

```qml
// scenarios/BridgeDefense.qml
Item {
    // Complete scenario definition
    HeavyTank {
        x: 150; y: 500
        unitSide: "allies"
        Component.onCompleted: {
            queueOrder("Defend", 200, 450)
        }
    }
    
    // Victory conditions
    function checkVictory() {
        var playerAlive = !isDestroyed("player_commander")
        var timeRemaining = timer.elapsed < timeLimit
        return playerAlive && timeRemaining ? "victory" : "defeat"
    }
}
```

**3. Declarative Over Imperative**

State changes happen through property bindings rather than explicit method calls:

```qml
// Visual state declaratively defined
states: [
    State {
        name: "healthy"
        PropertyChanges { target: unit; opacity: 1.0 }
    },
    State {
        name: "damaged"
        PropertyChanges { target: unit; opacity: 0.7 }
        PropertyChanges { target: smoke_effect; visible: true }
    }
]

// Transitions automatically animate state changes
transitions: [
    Transition {
        from: "healthy"; to: "damaged"
        PropertyAnimation { property: "opacity"; duration: 500 }
    }
]
```

**4. Community-Driven Development**

The architecture assumes that players will become creators:

```cpp
// Runtime content loading without recompilation
void Scenario::loadUnit(const QString& qmlFile) {
    QQmlComponent component(&engine, qmlFile);
    QObject* unit = component.create();  // Runtime instantiation!
    units.append(unit);
}
```

**5. Hot-Reload for Rapid Iteration**

```cpp
void CcfGameManager::enableHotReload() {
    _fileWatcher = new QFileSystemWatcher(this);
    _fileWatcher->addPath("units/");
    _fileWatcher->addPath("scenarios/");
    
    connect(_fileWatcher, &QFileSystemWatcher::fileChanged,
            this, &CcfGameManager::reloadContent);
}
```

Changes to QML files are reflected immediately—no restart required.

#### Why This Philosophy?

**Historical Context (2011–2012):**
- Qt5/QML emerged as powerful declarative framework
- Mobile development trended toward declarative approaches
- Modding communities demanded greater access
- Web technologies influenced game architecture thinking
- Rapid iteration valued over raw performance

**Technical Advantages:**
- Extremely moddable—everything is a QML file
- Rapid iteration without compile step
- Non-programmers can create content
- UI and game naturally integrated
- Declarative syntax readable by designers

**Philosophical Trade-offs:**

| Advantage | Disadvantage |
|-----------|--------------|
| Maximum moddability | QML performance limitations |
| No recompilation | Tight visual-logic coupling |
| Designer-friendly | Qt ecosystem lock-in |
| Hot-reload | Debugging across C++/QML boundary |
| Declarative clarity | Less control over execution order |

### 2.2.3 OpenCombat (2020–2024): The Systems-First Philosophy

**Core Philosophy**: *"The game is a deterministic simulation. State changes flow through messages. Architecture should support verification, replay, and network synchronization."*

OpenCombat represents the convergence of several trends in systems programming: Rust's ownership model, functional programming influence, and the requirements of modern multiplayer games.

#### Key Tenets

**1. Correctness Through Type Safety**

Rust's type system prevents entire classes of bugs at compile time:

```rust
// Type-safe entity indices prevent mixing up references
#[derive(Debug, Clone, Copy)]
pub struct SoldierIndex(pub usize);

#[derive(Debug, Clone, Copy)]
pub struct VehicleIndex(pub usize);

// Compiler prevents accidental misuse
let soldier = state.soldiers[soldier_idx.0];  // OK
let soldier = state.soldiers[vehicle_idx.0];  // Compile error!
```

**2. Determinism for Multiplayer**

Identical inputs must produce identical outputs across all clients:

```rust
// Message-driven state updates ensure determinism
pub enum BattleStateMessage {
    Soldier(SoldierIndex, SoldierMessage),
    Vehicle(VehicleIndex, VehicleMessage),
    SetPhase(Phase),
    PushBulletFire(BulletFire),
}

// All state changes flow through messages
fn tick(&mut self) {
    let messages = collect_messages();
    for msg in messages {
        self.battle_state.apply(msg);  // Deterministic application
    }
}
```

**3. Data Flow Over Control Flow**

Systems query state, compute changes, and send messages—no direct mutation:

```
┌─────────────────────────────────────────────────────┐
│                  BattleState                         │
├─────────────────────────────────────────────────────┤
│ soldiers: Vec<Soldier>          [contiguous]        │
│ vehicles: Vec<Vehicle>          [contiguous]        │
│ squads: HashMap<Uuid, Squad>   [lookup]             │
└─────────────────────────────────────────────────────┘
         ↓                        ↓
    ┌─────────┐             ┌─────────┐
    │ Systems │             │ Systems │
    │ - AI    │             │ - Move  │
    │ - Combat│             │ - Render│
    └─────────┘             └─────────┘
```

**4. Three-Tier State Hierarchy**

States organized by timescale and authority:

```rust
// Three-tier state representation
pub struct UnitState {
    phase: Phase,       // Global game state (Placement, Battle, End)
    behavior: Behavior, // Tactical decisions (MoveTo, Engage, Hide)
    gesture: Gesture,   // Moment-to-moment actions (Aiming, Firing, Reloading)
}

// Example: Soldier moving to cover
let state = UnitState {
    phase: Phase::Battle,
    behavior: Behavior::MoveTo(cover_position),
    gesture: Gesture::Crawling { end_frame: 1847 },
};
```

Each tier operates on different timescales with different propagation rules.

**5. Separation of Concerns**

Clear boundaries between simulation, networking, and rendering:

```rust
// battle_core: Pure simulation, no I/O
pub fn tick(battle_state: &mut BattleState, messages: Vec<Message>) {
    // Update game state deterministically
}

// battle_server: Networking layer
pub async fn handle_client(stream: TcpStream) {
    // Serialize/deserialize messages
}

// battle_client: Rendering and input
pub fn render(battle_state: &BattleState) {
    // Draw the world
}
```

#### Why This Philosophy?

**Historical Context (2020–2024):**
- Rust gained traction in game development
- Multiplayer became essential, not optional
- Replay/recording features expected by players
- Functional programming concepts mainstreamed
- Determinism recognized as architectural requirement

**Technical Advantages:**
- Deterministic simulation enables multiplayer and replays
- Type safety prevents synchronization bugs
- Clear data flow aids debugging
- Excellent testability (pure functions on state)
- Memory safety without garbage collection

**Philosophical Trade-offs:**

| Advantage | Disadvantage |
|-----------|--------------|
| Determinism guaranteed | More verbose than direct mutation |
| Type safety | Learning curve for message-passing |
| Clear data flow | Indirection through indices |
| Testability | Centralized state structure |
| Memory safety | Rust complexity barrier |

---

## 2.3 Comparing the Philosophies

### 2.3.1 Values Comparison Table

| Value | OpenCombat-SDL | CloseCombatFree | OpenCombat |
|-------|---------------|-----------------|------------|
| **Primary Goal** | Simulation authenticity | Modding accessibility | Multiplayer correctness |
| **Core Abstraction** | Object-oriented | Declarative/Component | Systems-oriented |
| **State Management** | Bitfield composition | Dual-state (runtime + visual) | Three-tier hierarchy |
| **Behavior Location** | C++ methods | QML functions | System functions |
| **Content Creation** | XML + recompile | QML (no recompile) | JSON + recompile |
| **Execution Model** | Immediate updates | Property bindings | Message queue |
| **Determinism** | Implicit | Implicit | Explicit (design goal) |
| **Modding Level** | Parameters only | Full content + behavior | Scenarios only |
| **Type Safety** | Runtime | Runtime | Compile-time |
| **Team Size** | Small (1-3) | Any (designer-friendly) | Medium (3-8) |

### 2.3.2 When Each Philosophy Shines

**Choose OpenCombat-SDL Philosophy When:**
- Building a single-player focused simulation
- Team is most comfortable with traditional OOP
- Authenticity is more important than moddability
- Rapid prototyping is more important than architectural purity
- Target hardware is constrained (embedded, retro platforms)

**Choose CloseCombatFree Philosophy When:**
- Moddability is a primary design goal
- Non-programmers will create content
- UI and game are tightly integrated
- Rapid iteration is essential
- Community-driven development is desired

**Choose OpenCombat Philosophy When:**
- Multiplayer is a core requirement
- Replay/recording features are important
- Team values determinism and testability
- Using a language with strong type systems (Rust, etc.)
- Correctness and anti-cheat are paramount

### 2.3.3 Trade-offs Matrix

```
                    MODDABILITY
                          ▲
                         ╱│╲
                        ╱ │ ╲
                       ╱  │  ╲
                      ╱   │   ╲
                     ╱    │    ╲
                    ╱ CCF │     ╲
                   ╱      │      ╲
        OpenCombat-SDL    │   OpenCombat
                 ╲        │      ╱
                  ╲       │     ╱
                   ╲      │    ╱
                    ╲     │   ╱
                     ╲    │  ╱
                      ╲   │ ╱
                       ╲  │╱
                        ╲ │
                         ╲│
                          ▼
                    DETERMINISM
        
        High moddability + High determinism = Difficult
        (requires sophisticated architecture)
```

---

## 2.4 The Evolution of Thought

### 2.4.1 OOP → Component → ECS: The Industry Trajectory

Game architecture has evolved through distinct phases, each solving problems left by the previous:

**Phase 1: Deep Inheritance (1990s–2000s)**
```cpp
// Problem: Rigid hierarchies
class Vehicle : public Object { };
class Tank : public Vehicle { };
class Sherman : public Tank { };

// What if you need a boat that acts like a tank?
// Multiple inheritance leads to diamond problem
```

**Phase 2: Component Composition (2000s–2010s)**
```cpp
// Solution: Composition over inheritance
class Entity {
    std::vector<Component*> components;
};

// Tank is entity with Mobile + Armored + Weapon components
// Boat can also have Mobile + Weapon (different implementation)
```

**Phase 3: Entity-Component-System (2010s–present)**
```rust
// Solution: Data-oriented design
struct BattleState {
    positions: Vec<Point>,      // All positions contiguous
    velocities: Vec<Vector>,    // All velocities contiguous
    healths: Vec<u8>,          // All healths contiguous
}

// Systems process data, not objects
fn movement_system(state: &mut BattleState) {
    for (pos, vel) in state.positions.iter_mut().zip(&state.velocities) {
        *pos += *vel;
    }
}
```

### 2.4.2 Why Each Shift Happened

| From | To | Reason |
|------|-----|--------|
| **Inheritance** | **Composition** | Flexibility—avoid rigid hierarchies; enable emergent combinations |
| **Composition** | **ECS** | Performance—cache-friendly data layouts; parallel processing |
| **OOP** | **Functional** | Correctness—immutable data; deterministic behavior |
| **C++** | **Rust/Go** | Safety—memory safety without GC; concurrency safety |
| **Hardcoded** | **Data-driven** | Moddability—separate content from code |

### 2.4.3 What Stayed Constant

Despite architectural evolution, certain principles remain constant:

**1. State Management Importance**

All three implementations invest heavily in state management:
- OpenCombat-SDL: 64-bit bitfields for orthogonal states
- CloseCombatFree: Dual-state separation (runtime + visual)
- OpenCombat: Three-tier hierarchy (phase/behavior/gesture)

**Quote from OpenCombat-SDL documentation:**
> "State management is the foundation upon which all other systems are built. A poorly designed state architecture makes features nearly impossible to add."

**2. The Order/Action Separation**

All three separate player intent from execution:
- OpenCombat-SDL: Order → Action → State
- CloseCombatFree: Order Queue → Status Changes
- OpenCombat: Order → Behavior → Gesture

This separation is **not optional**—it is the fundamental abstraction that makes tactical games playable.

**3. Importance of Determinism**

Even in single-player focused OpenCombat-SDL, determinism matters for:
- Replay recording
- Bug reproduction
- Save/load reliability

OpenCombat simply makes determinism an explicit architectural requirement rather than an emergent property.

---

## 2.5 Philosophical Questions for Your Project

Before choosing an architectural philosophy, answer these fundamental questions:

### 2.5.1 Simulation Depth vs. Accessibility

**The Question:** How much complexity should players manage?

**Deep Simulation (OpenCombat-SDL style):**
- Individual soldier morale tracking
- Detailed ammunition management
- Realistic ballistics calculations
- Complex command hierarchies

**Accessible Simulation (CloseCombatFree style):**
- Squad-level morale
- Abstracted ammunition
- Simplified combat resolution
- Streamlined controls

**Decision Matrix:**

| Target Audience | Recommended Depth |
|-----------------|-------------------|
| Military enthusiasts | Deep simulation |
| General strategy players | Moderate simulation |
| Casual/mobile players | Accessible simulation |
| Educational | Configurable depth |

### 2.5.2 Modding vs. Control

**The Question:** How much control do you retain over the player experience?

**Maximum Modding (CloseCombatFree):**
- Players can change core mechanics
- New unit types without developer involvement
- Community creates total conversions
- Risk: Fragmented player base; incompatible mods

**Controlled Experience (OpenCombat-SDL/OpenCombat):**
- Developers curate available content
- Consistent player experience
- Easier balance and tuning
- Risk: Community alienation; limited longevity

### 2.5.3 Multiplayer vs. Single-Player Focus

**The Question:** Is multiplayer a core feature or an add-on?

**If Multiplayer is Core:**
- Choose OpenCombat-style determinism
- Invest in server-authoritative architecture
- Accept complexity cost for synchronization benefits

**If Single-Player is Core:**
- OpenCombat-SDL or CloseCombatFree approaches work
- Focus on AI and emergent storytelling
- Client-authoritative acceptable

**The Multiplier Effect:**

| Feature | Single-Player Cost | Multiplayer Cost |
|---------|-------------------|------------------|
| State management | 1x | 3x (sync, validation, reconciliation) |
| Content creation | 1x | 2x (server + client assets) |
| Testing | 1x | 5x (network conditions, latency) |
| Architecture complexity | Baseline | +200% |

### 2.5.4 Procedural vs. Authored Content

**The Question:** How much content do you generate vs. hand-craft?

**Procedural Content:**
- Random map generation
- Dynamic campaign structure
- Emergent scenarios
- Requires robust simulation systems

**Authored Content:**
- Hand-crafted maps
- Scripted campaigns
- Set-piece battles
- Requires strong modding tools

**Hybrid Approaches:**
- Procedural terrain with authored elements
- Authored scenarios with procedural reinforcements
- Dynamic campaigns with authored story beats

---

## 2.6 The Hybrid Philosophy

### 2.6.1 Synthesizing the Best of All Three

Modern game development increasingly synthesizes elements from all three philosophies:

```
┌──────────────────────────────────────────────────────────────┐
│                    RECOMMENDED HYBRID                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              CORE SIMULATION                          │   │
│  │  Systems-oriented (OpenCombat-style)                 │   │
│  │  - Deterministic                                     │   │
│  │  - Type-safe                                         │   │
│  │  - Server-authoritative                              │   │
│  └──────────────────────────────────────────────────────┘   │
│                         ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           ENTITY DEFINITION                           │   │
│  │  Component composition (CloseCombatFree-style)       │   │
│  │  - JSON/YAML/QML for content                         │   │
│  │  - Hot-reload capable                                │   │
│  │  - Modder-accessible                                 │   │
│  └──────────────────────────────────────────────────────┘   │
│                         ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            BEHAVIOR SYSTEM                            │   │
│  │  Scriptable components (OpenCombat-SDL + scripting)  │   │
│  │  - Lua/Wren for AI behaviors                         │   │
│  │  - Data-driven action definitions                    │   │
│  │  - Automatic prerequisite chaining                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 2.6.2 Context-Dependent Choices

Different subsystems may benefit from different approaches:

| Subsystem | Recommended Approach | Reason |
|-----------|---------------------|--------|
| **Core simulation** | Systems-oriented | Determinism, testability |
| **Entity definitions** | Component composition | Moddability, flexibility |
| **State management** | Three-tier hierarchy | Clear mental model |
| **Content authoring** | Declarative (QML/YAML) | Designer-friendly |
| **AI behaviors** | Scriptable (Lua) | Rapid iteration |
| **Rendering** | ECS-style | Cache efficiency |

### 2.6.3 Pragmatic Architecture Principles

**1. Start with the Simulation Core**

Regardless of other choices, the core simulation should be:
- Deterministic
- Testable
- Server-authoritative (even for single-player)

**2. Separate Content from Mechanics**

- Mechanics: How the game works (code)
- Content: What exists in the game (data)
- Behavior: How entities act (scripts)

**3. Design for Modding from Day One**

Even if you don't plan for mods initially, design as if you will:
- Externalize entity definitions
- Scriptable behaviors
- Clear APIs for extension

**4. Embrace the Right Tool**

Don't force one paradigm everywhere:
- Use OOP where it makes sense (UI, tool code)
- Use functional patterns where they help (simulation)
- Use data-oriented design where performance matters (rendering)

### 2.6.4 The Synthesis Decision Tree

```
Start: New Close Combat Clone Project
│
├─ Multiplayer required?
│  ├─ YES → Systems-oriented core (OpenCombat-style)
│  │         - Message-driven state updates
│  │         - Server-authoritative
│  │         - Type-safe indices
│  │
│  └─ NO  → Flexible choice
│            - Can use OOP or component-based
│
├─ Modding community desired?
│  ├─ YES → Declarative content (CloseCombatFree-style)
│  │         - JSON/YAML entity definitions
│  │         - Lua scripting for behaviors
│  │         - Hot-reload support
│  │
│  └─ NO  → Data-driven sufficient
│            - XML/JSON configuration
│            - Hardcoded behaviors acceptable
│
├─ Deep simulation required?
│  ├─ YES → Bitfield + automatic prerequisite chaining
│  │         (OpenCombat-SDL style state management)
│  │
│  └─ NO  → Simple state hierarchy sufficient
│
└─ Result: Hybrid architecture optimized for YOUR project
```

---

## 2.7 Conclusion

### 2.7.1 The Philosophical Foundation

The three Close Combat clones represent not just technical choices but **philosophical stances** on how software should model reality:

- **OpenCombat-SDL** (2005–2008): The world is composed of objects. Model it faithfully.
- **CloseCombatFree** (2011–2012): Content creators are users too. Remove barriers.
- **OpenCombat** (2020–2024): Correctness and determinism are paramount. Enforce them.

None of these philosophies is objectively superior. Each represents a valid response to specific design goals, technical constraints, and historical contexts.

### 2.7.2 The Constant Through Change

Despite evolving from OOP to components to systems, certain truths remain:

1. **State management is foundational**—invest in it early
2. **Player intent must be separated from execution**—the order/action distinction is universal
3. **Determinism matters**—even single-player games benefit from reproducible behavior
4. **Modding requires architectural commitment**—can't be bolted on later
5. **There are no silver bullets**—each approach has trade-offs

### 2.7.3 Looking Forward

As you design your own tactical wargame, remember:

- **Choose based on your goals**, not technical fashion
- **Synthesize rather than copy**—take the best from each approach
- **Commit to your philosophy**—inconsistent hybrids are worse than pure approaches
- **Design for the team you have**—not the team you wish you had

The architecture choices you make today will shape your game for years to come. Choose thoughtfully, implement consistently, and remain open to evolution.

---

## Key Takeaways

1. **Three philosophies represent different priorities**: authenticity (SDL), moddability (CCF), correctness (OpenCombat)

2. **State management is the foundation**: Every other system depends on it

3. **Determinism is architectural**: Must be designed in, not added later

4. **Moddability requires commitment**: Cannot be retrofitted successfully

5. **Hybrid approaches work best**: Match patterns to specific subsystem needs

6. **Team expertise matters**: Choose paradigms your team can execute well

7. **Evolution is constant**: Industry moves from OOP → Components → ECS

8. **What stays constant**: State importance, order/action separation, need for determinism

---

**Next**: [Chapter 3: State Management Patterns](#chapter-3-state-management-patterns) examines how these philosophical choices manifest in concrete state management implementations.

---

**Document Information**  
**Version:** 2.0  
**Date:** February 2026  
**Status:** Publishable Draft  
**Target Audience:** Game architects, technical leads, and researchers studying tactical game systems

---

*The philosophy you choose will echo through every line of code you write. Choose with intention.*
