# Chapter 9: Lessons Learned: 20 Years of Tactical Wargame Development

## The Wisdom of Three Implementations

*What three iterations spanning nearly two decades teach us about building tactical wargames.*

---

The three Close Combat clones in this book—OpenCombat-SDL (2005–2008), CloseCombatFree (2011–2012), and OpenCombat (2020–2024)—form an evolutionary narrative. Each project built on or reacted to its predecessors. This chapter distills their accumulated wisdom into practical guidance for the next generation of tactical wargame developers.

The progression from SDL-based C++ to Qt/QML to Rust systems engineering reveals enduring truths about game architecture. Some problems persist across technology generations; others stem from specific technical choices. Recognizing the difference separates effective architecture from fashionable complication.

---

## 9.1 The 20-Year Journey

### 2005: The Classical Era: OpenCombat-SDL

In 2005, real-time tactical wargames typically used object-oriented C++ with direct hardware access. OpenCombat-SDL followed this tradition, starting with Windows and DirectX before adopting SDL2 for cross-platform support. Its architecture reflected the design patterns of the time: deep inheritance hierarchies, manual memory management, and XML-based configuration.

**The Technical Landscape (2005):**
- C++98/03 was standard; C++11 remained years away
- Hardware acceleration was becoming mainstream, though software rendering stayed viable
- XML dominated data interchange
- Modding support meant externalizing constants, not behaviors
- Multiplayer was an afterthought for most indie projects

OpenCombat-SDL's architecture made sense in this context. The 64-bit bitfield state system offered compact, efficient state tracking. XML configuration allowed balance tweaks without recompilation. Separating orders from actions created clear abstraction boundaries. These were sophisticated solutions to real problems.

The project couldn't predict how modding communities would evolve. The assumption that modders would accept data tweaking—with behaviors remaining hardcoded—proved limiting as player expectations grew.

### 2011: The Declarative Revolution: CloseCombatFree

By 2011, mobile computing was reshaping user interface expectations. Qt had matured, QML enabled declarative UI development, and "content as code" gained traction. CloseCombatFree embraced this approach, using C++ for performance-critical systems while expressing game content in QML.

**The Technical Landscape (2011):**
- C++11 arrived with lambdas, auto, and smart pointers
- Mobile-first design influenced desktop application architecture
- JavaScript's ubiquity made declarative approaches familiar
- Modding evolved from tweaking values to creating total conversions
- Qt 5 promised cross-platform deployment, including mobile

CloseCombatFree's QML-based approach was revolutionary. Defining units, scenarios, and behaviors in a declarative language without recompilation marked a paradigm shift. A tank could be defined as:

```qml
Tank {
    Hull { id: hull }
    Turret {
        id: turret
        rotationSpeed: 15
    }
    Soldier { role: "Commander" }
    Soldier { role: "Gunner" }
    Soldier { role: "Driver" }
}
```

This went beyond data externalization to behavioral externalization. The Qt runtime interpreted these definitions, instantiated objects, and established relationships without modifying the game executable.

The tradeoff was performance. QML's flexibility introduced overhead unsuitable for complex simulation. CloseCombatFree demonstrated the limits of modding but hit performance constraints.

### 2024: The Systems Renaissance: OpenCombat

By 2020, the game industry rediscovered systems programming. Rust provided memory safety without garbage collection. Deterministic simulation became essential for competitive multiplayer and replay systems. The ECS pattern, long used in high-performance engines, entered mainstream awareness.

**The Technical Landscape (2020–2024):**
- Rust's ecosystem matured with async, Bevy, and game-specific crates
- Deterministic multiplayer became non-negotiable for competitive games
- ECS evolved from optimization technique to architectural pattern
- Event sourcing and CQRS influenced game state management
- Hot reloading became standard for rapid iteration

OpenCombat synthesizes these lessons. It maintains the modding flexibility CloseCombatFree pioneered while grounding it in systems-oriented architecture. The message-driven state machine enables determinism without sacrificing clarity. The three-tier state hierarchy (Phase → Behavior → Gesture) provides structure without rigidity.

The Rust implementation adds compile-time safety, but the architecture itself is language-agnostic. The patterns—message passing, type-safe indices, server-authoritative simulation—would work equally well in modern C++.

---

## 9.2 What OpenCombat-SDL Got Right

Despite its age, OpenCombat-SDL implemented several enduring patterns. These successes show that good architecture outlasts technology trends.

### 9.2.1 Bitfield State System

The 64-bit bitfield approach to state management remains elegant:

```cpp
// 64 orthogonal states in 8 bytes
uint64_t state_bits;

// Setting states
state_bits |= PRONE | RELOADING | SUPPRESSED;

// Querying capabilities
bool canFire = (state_bits & RELOADED) && !(state_bits & SUPPRESSED);
```

**Why This Still Works:**

1. **Memory Efficiency**: 64 states consume only 8 bytes, critical for cache performance when iterating thousands of units
2. **Orthogonal Composition**: States combine naturally without handling every permutation
3. **Fast Operations**: Bitwise AND/OR execute as single CPU instructions
4. **Deterministic Behavior**: Bit operations produce identical results across platforms

The bitfield system works well for tracking capabilities—what a unit can do, not its current state. This distinction is important: "CanFire" and "IsProne" coexist without conflict, while "Moving" and "Idle" cannot.

**Modern Relevance**: Cache efficiency still matters in 2024, even with 64GB RAM. A simulation with 500 units updating at 60Hz handles 30,000 updates per second. Memory layout affects frame times directly.

### 9.2.2 Separation of Simulation from Rendering

OpenCombat-SDL kept game logic and visual presentation strictly separate:

```cpp
// Simulation updates in World::Simulate()
void World::Simulate(float dt) {
    for (auto& soldier : _soldiers) {
        soldier->Update(dt);
    }
}

// Rendering in separate phase
void World::Render(Screen* screen) {
    for (auto& soldier : _soldiers) {
        soldier->Draw(screen);
    }
}
```

This approach provides several advantages:
- **Variable Frame Rates**: The simulation runs at a fixed timestep while rendering adapts to display refresh
- **Headless Servers**: Game logic executes without graphics
- **Deterministic Replay**: The same simulation inputs produce identical results regardless of rendering
- **Testing**: Simulation logic can be tested without display initialization

**Lesson Validated**: OpenCombat maintained this separation using Rust's stricter ownership model. CloseCombatFree merged these boundaries by having QML handle both logic and presentation, which created testing and debugging difficulties.

### 9.2.3 XML Data Files for Configuration

Storing game data in XML allowed balance changes without recompilation:

```xml
<SoldierTemplate type="Rifleman">
    <Attribute name="Health" value="100"/>
    <Attribute name="Speed" value="5"/>
    <Weapon type="M1_Garand"/>
</SoldierTemplate>
```

Though XML has become less common (replaced by JSON, YAML, and TOML), the principle holds: data that changes often shouldn't require code changes.

**Modern Adaptation**: OpenCombat now uses JSON for the same purpose:

```json
{
  "soldiers": [
    {
      "type": "rifleman",
      "health": 100,
      "speed": 5.0,
      "weapon": "m1_garand"
    }
  ]
}
```

The syntax evolved, but the architectural principle remained.

### 9.2.4 Two-Tier Command System

Separating Orders (player intent) from Actions (physical execution) created a clean abstraction:

```cpp
// Order: High-level player intent
class MoveOrder : public Order {
    Vector2D destination;
    MoveSpeed speed;
};

// Action: Low-level physical execution
class RunToAction : public Action {
    Path path;
    float duration;
};
```

This system allowed automatic prerequisite resolution. For example, a prone soldier ordered to run would automatically stand up before moving, without player micromanagement.

**Evolution**: OpenCombat expanded this to three tiers (Orders → Behaviors → Gestures), but the core idea—separating intent from execution—came from OpenCombat-SDL.

---

## 9.3 What OpenCombat-SDL Got Wrong

Failures teach as much as successes. OpenCombat-SDL's limitations influenced how later projects approached design.

### 9.3.1 Deep Inheritance Hierarchies

The object model depended on inheritance:

```cpp
class Object { /* base */ };
class Soldier : public Object { /* ... */ };
class Vehicle : public Object { /* ... */ };
class Tank : public Vehicle { /* ... */ };
```

**The Problem:** Inheritance creates rigid taxonomies. When you need a drone that behaves like both a soldier and a vehicle, or a defensive emplacement that functions as terrain but can be destroyed like a unit, deep hierarchies force premature classification.

**The Lesson:** CloseCombatFree and OpenCombat both shifted to composition. CCF used QML component composition, while OpenCombat adopted ECS-style embedded components. Modern C++ with `std::variant` and type erasure provides even more flexibility—options unavailable in 2005.

### 9.3.2 Hardcoded Behavior Logic

Actions were implemented as C++ classes with fixed logic:

```cpp
class FireAction : public Action {
    void Execute(Soldier* soldier) override {
        // Hardcoded firing logic
        soldier->weapon->Fire(target);
        soldier->ammo--;
    }
}
```

**The Problem:** Adding new behaviors meant writing new C++ classes and recompiling. Modders could adjust weapon damage in XML but couldn't create new weapon types with unique behaviors.

**The Lesson:** CloseCombatFree proved behaviors could be defined declaratively. OpenCombat showed that behavior trees and scripting offered flexibility without sacrificing performance for critical paths.

### 9.3.3 Modding Limitations

OpenCombat-SDL allowed "data modding" but not "behavior modding":

| Mod Type              | Supported | Implementation           |
| --------------------- | --------- | ------------------------ |
| New soldier types     | ✓         | XML configuration        |
| Weapon balance tweaks | ✓         | XML configuration        |
| New terrain types     | ✓         | XML configuration        |
| New behaviors         | ✗         | Required code changes    |
| New order types       | ✗         | Required code changes    |
| New AI logic          | ✗         | No AI system implemented |

**The Lesson:** Moddability must be designed in from the start. CloseCombatFree's QML approach enabled true behavioral modding because it was built for it. Retrofitting modding onto a hardcoded system is nearly impossible.

### 9.3.4 Single-Threaded Architecture

The simulation ran entirely on the main thread:

```cpp
void GameLoop() {
    while (running) {
        ProcessInput();
        World::Simulate(dt);
        Render();
    }
}
```

**The Problem:** As unit counts increased, simulation time directly cut into frame rates. Pathfinding for 100 units could cause visible stuttering.

**The Lesson:** OpenCombat addressed this with job systems and parallel pathfinding. Modern CPUs have multiple cores—architecture should use them. Even without full parallelism, decoupling simulation timestep from rendering framerate (as OpenCombat-SDL did) provides a foundation for optimization.

---

## 9.4 What CloseCombatFree Got Right

CloseCombatFree's declarative approach was ahead of its time. Several of its innovations became industry standards.

### 9.4.1 QML for Declarative Content

Using QML for game content was groundbreaking:

```qml
Scenario {
    id: demo_scenario
    name: "Village Defense"

    Map { source: "maps/village.tmx" }

    Squad {
        side: "Allies"
        soldiers: [
            Soldier { type: "Rifleman"; x: 100; y: 200 },
            Soldier { type: "MachineGunner"; x: 110; y: 200 }
        ]
    }
}
```

**Why This Matters:**

1. **Runtime Loading:** Scenarios load without recompilation
2. **Visual-Logic Binding:** QML's property system syncs UI and simulation automatically
3. **Modder Accessibility:** JavaScript-like syntax familiar to web developers
4. **Tool Integration:** Qt Creator provides IDE support for content creation

**Modern Parallel:** This foreshadowed approaches like Unity's prefabs, Godot's scenes, and Bevy's entity bundles. Declarative composition is now standard; CCF was early to adopt it.

### 9.4.2 Component Composition Over Inheritance

CCF's component system avoided inheritance pitfalls:

```qml
// Tank composed of components, not inheriting from Vehicle
Entity {
    id: tank

    Mobile { maxSpeed: 15 }
    Armored { armor: 50 }
    Turret { rotationSpeed: 20 }
    CrewCapacity { maxCrew: 5 }
}
```

Components could be combined freely without inheritance. A "motorcycle with sidecar" might use Mobile and CrewCapacity without fitting any class hierarchy.

**Lesson Validated**: OpenCombat's ECS approach delivers similar flexibility with better performance. Composition over inheritance has become the standard architectural choice.

### 9.4.3 Maximum Moddability

CCF's declarative system enabled unprecedented modding capabilities:

| Mod Type                          | CCF Support               | OpenCombat-SDL      |
| --------------------------------- | ------------------------- | ------------------- |
| New scenarios                     | ✓ (QML files)             | ✗                   |
| New unit types                    | ✓ (QML entities)          | Partial (XML)       |
| New behaviors                     | ✓ (QML state machines)    | ✗                   |
| UI customization                  | ✓ (QML UI files)          | ✗                   |
| New weapons with unique mechanics | ✓ (Component composition) | Partial (data only) |

The C++ backend provided functionality; QML assembled it. Modders created entirely new game modes without touching C++ code.

### 9.4.4 Clean Separation of Concerns

CCF maintained distinct boundaries:

- **C++ Backend**: Simulation logic, physics, pathfinding
- **QML Frontend**: UI, scenario definition, visual effects
- **Communication**: Qt signals/slots for loose coupling

This separation allowed:
- Artists to modify visuals independently
- Designers to iterate scenarios without recompiling
- Programmers to refactor simulation without breaking UI

---

## 9.5 What CloseCombatFree Got Wrong

CCF's innovations came with tradeoffs that later projects had to address.

### 9.5.1 QML Performance Overhead

QML's flexibility carried runtime costs:

```qml
// Each QML property adds meta-object system overhead
property int health: 100  // Behind the scenes: QMetaProperty, signals, bindings
```

**The Problem**: Hundreds of QML entities updating every frame strained the Qt runtime. The garbage collector caused frame hitches. Property bindings created invisible dependencies that were difficult to optimize.

**Measured Impact**: CCF documentation acknowledges "hundreds of units may lag." For squad-level combat (50-200 units), this was acceptable. For larger scales, it became a problem.

**The Lesson**: OpenCombat's Rust ECS offers similar flexibility with compile-time optimization. Data-oriented design—using arrays of structs instead of structs of arrays—improves cache performance.

### 9.5.2 Limited Determinism for Multiplayer

CCF's architecture made deterministic multiplayer difficult:

- **QML State Changes**: String-based status transitions ("MOVING" → "AIMING") lacked strict validation
- **Floating-Point Variance**: Physics calculations differed across platforms
- **No Event Log**: State mutations weren't centrally recorded
- **Client Authority**: Each client ran its own simulation

**The Problem**: Determinism—identical inputs producing identical outputs—is crucial for:
- Lockstep multiplayer (all clients must agree on state)
- Replay recording (inputs must reconstruct the game)
- Debugging (bugs must be reproducible)

CCF prioritized flexibility over determinism, making these features hard to implement.

### 9.5.3 Qt Dependency

Building on Qt enabled rapid development but created limitations:

**Deployment Complexity:**
- Qt libraries had to ship with the executable
- Platform-specific builds were required
- Static linking increased binary size

**Learning Curve:**
- Contributors needed Qt expertise
- C++/QML integration had subtle complexities
- Qt's meta-object compiler (moc) added build complexity

**Platform Limitations:**
- Qt 5's mobile support was incomplete
- Web deployment (Qt WebAssembly) wasn't production-ready
- Console platforms were unsupported

**The Lesson**: OpenCombat minimized dependencies, using ggez for graphics, zmq for networking, and serde for serialization. Each dependency was lightweight and replaceable. Rust's ecosystem enables this modular approach more effectively than C++.

---

## 9.6 What OpenCombat Got Right

OpenCombat synthesized lessons from earlier projects. Its architecture solves previous limitations while introducing new solutions.

### 9.6.1 ECS for Simulation

OpenCombat's modified Entity Component System provides flexibility without performance tradeoffs:

```rust
// Contiguous arrays for cache efficiency
pub struct BattleState {
    pub soldiers: Vec<Soldier>,
    pub vehicles: Vec<Vehicle>,
    pub squads: Vec<Squad>,
}

// Type-safe indices prevent mixing entity types
pub struct SoldierIndex(pub usize);
pub struct VehicleIndex(pub usize);
```

**Why ECS Matters**

1. **Cache Efficiency**: Contiguous memory in `soldiers` speeds iteration
2. **Type Safety**: SoldierIndex cannot be mistaken for VehicleIndex
3. **Serialization**: Indices work naturally for network and replay systems
4. **Parallelism**: Systems process components in parallel

**Evolution from Predecessors**
OpenCombat-SDL moved from deep inheritance to a flat ECS. CloseCombatFree replaced QML composition with data-oriented design.

### 9.6.2 Message-Based Deterministic Architecture

All state changes flow through messages:

```rust
pub enum BattleStateMessage {
    Soldier(SoldierIndex, SoldierMessage),
    SetPhase(Phase),
    PushBulletFire(BulletFire),
}

pub fn tick(state: &mut BattleState, messages: &[BattleStateMessage]) {
    for msg in messages {
        state.apply(msg);  // Deterministic application
    }
}
```

**Benefits**

1. **Determinism**: Same messages and initial state always produce the same result
2. **Replay**: Message logs recreate any game
3. **Multiplayer**: Clients apply server-broadcast messages
4. **Debugging**: Message logs show exactly what changed and when

This pattern enables replay recording, multiplayer synchronization, and debug logging through a single architectural choice.

### 9.6.3 Hot-Reload Development

Runtime configuration enables rapid iteration:

```rust
// 60+ tweakable parameters
pub struct ServerConfig {
    pub visibility_modifier: f32,
    pub suppression_recovery_rate: f32,
    pub morale_decay_factor: f32,
    // ...
}

// Changed without restart
config.visibility_modifier = new_value;
```

**Impact on Development**
Hot reload allows balance testing without recompilation. Developers get immediate feedback on parameter changes and can debug by modifying values live.

CloseCombatFree achieved similar benefits through QML hot reload. OpenCombat's Rust implementation adds type safety and compile-time verification to runtime flexibility.

### 9.6.4 Data-Driven Design

JSON deployments define scenarios:

```json
{
  "map": "village",
  "soldiers": [
    {
      "type": "rifleman",
      "side": "Allies",
      "position": {"x": 100, "y": 200}
    }
  ],
  "victory_conditions": {
    "hold_objective": "church",
    "duration": 600
  }
}
```

Combined with hot reload, this approach enables:
- Fast scenario iteration
- Modding without code changes
- Version-controlled content
- A/B testing of different configurations

---

## 9.7 What OpenCombat Got Wrong

Even well-designed systems have trade-offs. OpenCombat's complexity creates barriers developers should consider.

### 9.7.1 Complexity Overhead

The three-tier state hierarchy adds conceptual weight:

```rust
pub struct SoldierState {
    pub phase: Phase,       // Game level: Deployment, Battle, End
    pub behavior: Behavior, // Tactical: MoveTo, Defend, Engage
    pub gesture: Gesture,   // Immediate: Aiming, Firing, Reloading
}
```

**The Cost**
New team members must learn the tier model. State transitions require checking all three levels. Debugging means examining three enums instead of one. Serialization must handle all three tiers.

**When It's Worth It**
The complexity pays off when you need:
- Multiplayer determinism
- Complex state interactions
- Replay or recording features

**When It's Not**
The overhead isn't justified for:
- Single-player games
- Simple state machines
- Rapid prototyping

### 9.7.2 Rust Learning Curve

Rust's ownership and borrow checker create challenges:

```rust
// Fighting the borrow checker
fn update_soldier(world: &mut World, idx: SoldierIndex) {
    let soldier = &mut world.soldiers[idx.0];
    // ERROR: cannot borrow `world.soldiers` as immutable
    // because it's borrowed as mutable
    let cover = find_cover(&world.soldiers, soldier.position);
}
```

**The Challenge**
Rust has a steep learning curve. Teams need Rust expertise. Some patterns become awkward (graph structures, cyclic references). Compile times can slow development.

**The Trade-off**
Memory safety eliminates entire classes of bugs—null pointers, use-after-free, data races. For long-term projects with community contributions, this may justify the initial investment. For rapid prototyping, it may not.

### 9.7.3 Limited Accessibility for Modders

While OpenCombat supports data modding, behavioral modding requires Rust:

| Mod Type         | OpenCombat        | CloseCombatFree      |
| ---------------- | ----------------- | -------------------- |
| New scenarios    | ✓ (JSON)          | ✓ (QML)              |
| New unit types   | Partial (JSON)    | ✓ (QML)              |
| New behaviors    | ✗ (requires Rust) | ✓ (QML)              |
| AI customization | ✗ (requires Rust) | Partial (QML states) |

**The Gap**
QML allowed non-programmers to create game content. Rust requires programming expertise, which limits the modding audience for community-driven games.

**Potential Solutions**
- Lua scripting for behaviors (adds dependency)
- Visual behavior tree editors (requires significant tooling)
- WebAssembly modding (complex but sandboxed)

---

## 9.8 Universal Lessons

Beyond specific implementations, these three projects reveal enduring truths about tactical wargame architecture.

### 9.8.1 Moddability Must Be Designed Early

True moddability cannot be added later. The projects show different approaches:

**OpenCombat-SDL**: Hardcoded behaviors with XML data allowed limited modding
**CloseCombatFree**: Declarative QML enabled extensive modding
**OpenCombat**: Deterministic Rust with JSON offered moderate modding

Set modding goals before coding:
- Parameter tweaking needs XML or JSON
- New content requires data-driven definitions
- New mechanics demand scripting or declarative systems
- Total conversions need a full mod API with documentation

### 9.8.2 Determinism Works Best When Built In

OpenCombat-SDL lacked multiplayer. CloseCombatFree lacked determinism. OpenCombat included determinism from the start.

Retrofitting fails because:
- Code scatters direct state mutations
- Floating-point operations vary across platforms
- Random number generators use inconsistent seeds
- Logic depends on frame rate

Determinism requires:
- Fixed timestep simulation
- Seeded RNG with replay capability
- Message-based state changes, not direct mutation
- No floating-point in critical calculations (or consistent rounding)
- Deterministic iteration order through sorted collections

### 9.8.3 State Management Drives Everything

All three projects focused heavily on state management because it supports every other system:

| Project         | Approach     | Strength            | Weakness       |
| --------------- | ------------ | ------------------- | -------------- |
| OpenCombat-SDL  | Bitfield     | Composable, fast    | 64-state limit |
| CloseCombatFree | String-based | Flexible, moddable  | No type safety |
| OpenCombat      | Hierarchical | Clear relationships | Complex        |

State architecture determines what your game can do. Choose based on needs:
- Orthogonal states work well with bitfields (OpenCombat-SDL pattern)
- Modding flexibility benefits from string-based systems (CloseCombatFree pattern)
- Multiplayer determinism needs hierarchical temporal data (OpenCombat pattern)

### 9.8.4 Keep AI and Simulation Separate

OpenCombat-SDL included simulation but no AI. CloseCombatFree had minimal AI. OpenCombat integrated both.

The principle divides responsibilities:
- Simulation handles physics, ballistics, and line-of-sight (deterministic)
- AI manages decision-making, target selection, and pathfinding (heuristic)

Separation matters because:
1. Simulation becomes testable without AI
2. Difficulty scales without changing physics
3. Modders can replace AI without affecting core systems
4. Debugging reveals whether bugs are in decision-making or execution

### 9.8.5 Content Belongs in Data, Not Code

Each project moved toward data-driven design with different capabilities:

```
OpenCombat-SDL: Data defines parameters
CloseCombatFree: Data defines structure and behavior
OpenCombat: Data defines structure, code defines behavior
```

The trend is clear—separate frequently changing content from stable engine code.

---

## 9.9 Advice for New Projects

### The 10 Commandments of Tactical Wargame Development

**1. Separate intent from execution**

Player orders differ from unit actions. Maintain two tiers:
- Orders: "Defend this position"
- Actions: "Move to position, face direction, set stance"

**2. Avoid hardcoded behaviors**

If modders might change it, externalize it:
- Weapon mechanics go in scripts or data
- AI decisions belong in behavior trees
- Movement rules live in configuration

**3. Optimize memory layout**

Memory layout affects performance more than algorithmic complexity:
- Use contiguous arrays instead of linked lists
- Prefer structs of arrays for SIMD
- Avoid pointer chasing

**4. Keep rendering and simulation separate**

```cpp
// WRONG
void Soldier::Update() {
    position += velocity * dt;
    Draw(sprite, position);  // Don't render during simulation!
}

// RIGHT
void Simulation::Update() {
    for (auto& soldier : soldiers) {
        soldier.position += soldier.velocity * dt;
    }
}

void Renderer::Draw() {
    for (auto& soldier : soldiers) {
        DrawSprite(soldier.sprite, soldier.position);
    }
}
```

**5. Make state explicit**

```cpp
// WRONG: Implicit state scattered through code
if (soldier.health > 0 && soldier.ammo > 0 && !soldier.suppressed) {
    soldier.Fire();
}

// RIGHT: Explicit capability tracking
if (soldier.HasCapability(Capability::CanFire)) {
    soldier.Fire();
}
```

**6. Plan for multiplayer from the start**

Even single-player games should:
- Use message-driven state updates
- Avoid client-authoritative simulation
- Design for determinism

Adding multiplayer later requires major architectural changes.

**7. Don't optimize prematurely**

OpenCombat-SDL's bitfield system was elegant but unnecessary for fewer than 100 units. CloseCombatFree's QML overhead didn't matter for a proof-of-concept.

Optimize when:
- Profiling identifies bottlenecks
- Target scale exceeds current performance
- User experience suffers

**8. Document why, not what**

```cpp
// BAD: What the code does
state_bits |= PRONE;  // Set prone bit

// GOOD: Why it matters
state_bits |= PRONE;  // Prone reduces silhouette, increasing cover effectiveness
```

**9. Test determinism**

```rust
#[test]
fn test_determinism() {
    let state1 = run_simulation(SEED, INPUTS);
    let state2 = run_simulation(SEED, INPUTS);
    assert_eq!(hash(state1), hash(state2));
}
```

**10. Choose the right complexity**

| Game Type                  | Recommended Approach                                      |
| -------------------------- | --------------------------------------------------------- |
| First tactical game        | OpenCombat-SDL patterns (simple, intuitive)               |
| Commercial multiplayer     | OpenCombat patterns (deterministic, server-authoritative) |
| Community modding platform | CloseCombatFree patterns (declarative, flexible)          |
| AAA production             | Synthesis of all three                                    |

---

## 9.10 Conclusion: The Wisdom of Iteration

Three projects, three eras, three philosophies. Each made trade-offs, succeeding in some areas and struggling in others. The accumulated wisdom maps the design space rather than prescribing solutions.

**What Endures:**
1. Separation of concerns (orders/behaviors, simulation/rendering)
2. State management discipline (explicit, debuggable, serializable)
3. Data-driven design (separate changing content from stable engine)
4. Type safety (prevent bugs through architecture)
5. Modular architecture (replace components without rewriting)

**What Depends on Context:**
1. Language choice (architecture matters more than Rust vs. C++ vs. C#)
2. State representation (bitfield vs. enum vs. string based on needs)
3. Inheritance depth (composition generally preferred, but inheritance isn't evil)
4. Scripting vs. compiled (performance vs. flexibility trade-off)
5. Determinism overhead (necessary for multiplayer, overhead for single-player)

The final lesson: architecture manages trade-offs. The Close Combat clones prove the same game concept can succeed with different approaches. Your task is to understand your constraints—team size, timeline, target audience, platform—and choose accordingly.

The 20-year journey from OpenCombat-SDL through CloseCombatFree to OpenCombat shows that good ideas persist (bitfield states, component composition) while technological constraints shift (XML to JSON, QML to ECS, manual memory management to Rust's ownership). The patterns in this book provide vocabulary; the lessons in this chapter provide judgment.

Build thoughtfully. Iterate fearlessly. Learn continuously.

---

*Next: [Chapter 10: Architectural Recommendations for Tactical Wargames](chapter_10_recommendations.md)*
