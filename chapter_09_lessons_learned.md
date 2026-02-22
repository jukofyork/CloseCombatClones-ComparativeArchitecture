# Chapter 9: Lessons Learned—20 Years of Tactical Wargame Development

## The Wisdom of Three Implementations

*What three iterations spanning nearly two decades teach us about building tactical wargames.*

---

The three Close Combat clones analyzed in this book—OpenCombat-SDL (2005–2008), CloseCombatFree (2011–2012), and OpenCombat (2020–2024)—represent more than parallel implementations of similar ideas. They constitute an evolutionary narrative, each project building upon (or reacting against) the lessons of its predecessors. This chapter distills that accumulated wisdom into actionable insights for the next generation of tactical wargame developers.

The journey from SDL-based C++ to Qt/QML declarative programming to Rust systems engineering reveals enduring truths about game architecture: certain problems persist across technology generations, while others are artifacts of specific technical choices. Understanding which is which separates effective architecture from fashionable complication.

---

## 9.1 The 20-Year Journey

### 2005: The Classical Era—OpenCombat-SDL

In 2005, real-time tactical wargames were predominantly built using object-oriented C++ with direct hardware access. OpenCombat-SDL emerged from this tradition, originally targeting Windows with DirectX before porting to SDL2 for cross-platform compatibility. The project's architecture reflects the design patterns of its era: deep inheritance hierarchies, manual memory management, and XML-based data configuration.

**The Technical Landscape (2005):**
- C++98/03 was the standard; C++11 was years away
- Hardware acceleration was becoming mainstream but software rendering remained viable
- XML was the dominant data interchange format
- Modding support meant externalizing constants, not behaviors
- Multiplayer was an afterthought for most indie projects

OpenCombat-SDL's architecture decisions made perfect sense in this context. The 64-bit bitfield state system provided compact, efficient state tracking. XML configuration enabled balance tweaks without recompilation. The separation of orders from actions created clear abstraction boundaries. These were sophisticated solutions to genuine problems.

What the project could not anticipate was how the modding community would evolve. The assumption that modders would be satisfied with data tweaking—that behaviors could remain hardcoded—proved limiting as player expectations grew.

### 2011: The Declarative Revolution—CloseCombatFree

By 2011, mobile computing was transforming user interface expectations. Qt had matured, QML offered genuine declarative UI development, and the concept of "content as code" was gaining traction. CloseCombatFree embraced this paradigm wholeheartedly, using C++ for performance-critical systems while expressing game content in QML.

**The Technical Landscape (2011):**
- C++11 had arrived with lambdas, auto, and smart pointers
- Mobile-first design was influencing desktop application architecture
- JavaScript was becoming ubiquitous, making declarative approaches familiar
- Modding had evolved from tweaking values to creating total conversions
- Qt 5 promised cross-platform deployment including mobile

CloseCombatFree's QML-based approach was revolutionary for its time. The ability to define units, scenarios, and even behaviors in a declarative language without recompilation represented a genuine paradigm shift. A tank could be defined as:

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

This wasn't merely data externalization; it was behavioral externalization. The Qt runtime interpreted these definitions, instantiated objects, and established relationships—all without the game executable changing.

The cost, however, was performance. QML's flexibility came with overhead unsuited for complex simulation. CloseCombatFree demonstrated the modding ceiling but hit the performance floor.

### 2024: The Systems Renaissance—OpenCombat

By 2020, the game industry had rediscovered systems programming. Rust offered memory safety without garbage collection. Deterministic simulation became essential for competitive multiplayer and replay systems. The ECS (Entity Component System) pattern, long used in high-performance engines, entered mainstream awareness.

**The Technical Landscape (2020–2024):**
- Rust's ecosystem matured with async, Bevy, and game-specific crates
- Deterministic multiplayer became non-negotiable for competitive games
- ECS evolved from niche optimization to architectural pattern
- Event sourcing and CQRS influenced game state management
- Hot reloading became standard for rapid iteration

OpenCombat represents the synthesis of lessons learned. It maintains the modding flexibility CloseCombatFree pioneered but grounds it in systems-oriented architecture. The message-driven state machine enables determinism without sacrificing clarity. The three-tier state hierarchy (Phase → Behavior → Gesture) provides organizational structure without rigidity.

The Rust implementation adds compile-time safety guarantees, but the architecture itself is language-agnostic. The patterns—message passing, type-safe indices, server-authoritative simulation—would work equally well in modern C++.

---

## 9.2 What OpenCombat-SDL Got Right

Despite its age and limitations, OpenCombat-SDL implemented several patterns that remain relevant today. These successes demonstrate that good architecture transcends technology trends.

### 9.2.1 Bitfield State System

The 64-bit bitfield approach to state management is elegant in its simplicity:

```cpp
// 64 orthogonal states in 8 bytes
uint64_t state_bits;

// Setting states
state_bits |= PRONE | RELOADING | SUPPRESSED;

// Querying capabilities
bool canFire = (state_bits & RELOADED) && !(state_bits & SUPPRESSED);
```

**Why This Still Works:**

1. **Memory Efficiency**: 64 states consume only 8 bytes—critical for cache performance when iterating thousands of units
2. **Orthogonal Composition**: States naturally combine without explicit handling of every permutation
3. **Fast Operations**: Bitwise AND/OR are single CPU instructions
4. **Deterministic Behavior**: Bit operations produce identical results across platforms

The bitfield system excels for capability tracking—what a unit *can* do rather than what it *is* doing. This distinction matters: "CanFire" and "IsProne" can coexist without conflict, whereas "Moving" and "Idle" cannot.

**Modern Relevance**: Even in 2024, when 64GB RAM is common, cache efficiency matters. A simulation with 500 units updating at 60Hz performs 30,000 updates per second. Memory layout directly impacts frame times.

### 9.2.2 Separation of Simulation from Rendering

OpenCombat-SDL maintained strict separation between game logic and visual presentation:

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

This separation enables:
- **Variable Frame Rates**: Simulation runs at fixed timestep while rendering adapts to display refresh
- **Headless Servers**: Game logic executes without graphics subsystem
- **Deterministic Replay**: Same simulation inputs produce identical results regardless of rendering
- **Testing**: Simulation logic testable without display initialization

**Lesson Validated**: OpenCombat retained this separation, using the same pattern with Rust's stricter ownership. CloseCombatFree blurred this boundary, with QML handling both logic and presentation, creating testing and debugging challenges.

### 9.2.3 XML Data Files for Configuration

Externalizing game data to XML enabled balance iteration without recompilation:

```xml
<SoldierTemplate type="Rifleman">
    <Attribute name="Health" value="100"/>
    <Attribute name="Speed" value="5"/>
    <Weapon type="M1_Garand"/>
</SoldierTemplate>
```

While XML has fallen out of fashion (replaced by JSON, YAML, and TOML), the principle remains sound: data that changes frequently should not require code changes.

**Modern Adaptation**: OpenCombat uses JSON for the same purpose:

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

The syntax changed; the architectural principle endured.

### 9.2.4 Two-Tier Command System

The separation of Orders (player intent) from Actions (physical execution) created a natural abstraction boundary:

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

This two-tier system enabled automatic prerequisite resolution—a prone soldier ordered to run would automatically insert StandUp before RunTo without player micromanagement.

**Evolution**: OpenCombat expanded this to three tiers (Orders → Behaviors → Gestures), but the core insight—separating intent from execution—originated in OpenCombat-SDL.

---

## 9.3 What OpenCombat-SDL Got Wrong

Examining failures is as instructive as examining successes. OpenCombat-SDL's limitations shaped the design reactions of subsequent projects.

### 9.3.1 Deep Inheritance Hierarchies

The object model relied heavily on inheritance:

```cpp
class Object { /* base */ };
class Soldier : public Object { /* ... */ };
class Vehicle : public Object { /* ... */ };
class Tank : public Vehicle { /* ... */ };
```

**The Problem**: Inheritance creates rigid taxonomies. What happens when you need a drone that behaves like both a soldier and a vehicle? Or a defensive emplacement that acts like terrain but can be destroyed like a unit? Deep hierarchies force premature classification.

**The Lesson**: CloseCombatFree and OpenCombat both moved toward composition. CCF used QML component composition; OpenCombat used ECS-style embedded components. Modern C++ with std::variant and type erasure offers additional flexibility that wasn't available in 2005.

### 9.3.2 Hardcoded Behavior Logic

Actions were implemented as C++ classes with hardcoded logic:

```cpp
class FireAction : public Action {
    void Execute(Soldier* soldier) override {
        // Hardcoded firing logic
        soldier->weapon->Fire(target);
        soldier->ammo--;
    }
}
```

**The Problem**: New behaviors required new C++ classes and recompilation. Modders could tweak weapon damage in XML but could not create new weapon types with unique behaviors.

**The Lesson**: CloseCombatFree demonstrated that behaviors could be expressed declaratively. OpenCombat showed that behavior trees and scripting could provide flexibility while maintaining performance for critical paths.

### 9.3.3 Modding Limitations

OpenCombat-SDL supported "data modding" but not "behavior modding":

| Mod Type | Supported | Implementation |
|----------|-----------|----------------|
| New soldier types | ✓ | XML configuration |
| Weapon balance tweaks | ✓ | XML configuration |
| New terrain types | ✓ | XML configuration |
| New behaviors | ✗ | Required code changes |
| New order types | ✗ | Required code changes |
| New AI logic | ✗ | No AI system implemented |

**The Lesson**: Moddability must be architected from day one. CloseCombatFree's QML approach enabled true behavioral modding because it was designed for it. Retrofitting modding onto a hardcoded architecture is nearly impossible.

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

**The Problem**: As unit counts grew, simulation time directly reduced frame rate. Pathfinding for 100 units could cause visible hitching.

**The Lesson**: OpenCombat addressed this with job systems and parallel pathfinding. Modern CPUs have multiple cores; architecture should utilize them. Even without full parallelism, separating simulation timestep from rendering framerate (as OpenCombat-SDL actually did) provides a foundation for future optimization.

---

## 9.4 What CloseCombatFree Got Right

CloseCombatFree's declarative approach represented a paradigm shift. Several of its innovations proved prescient.

### 9.4.1 QML for Declarative Content

Using QML for game content was genuinely innovative:

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

1. **Runtime Loading**: Scenarios load without recompilation
2. **Visual-Logic Binding**: QML's property system automatically syncs UI and simulation
3. **Modder Accessibility**: JavaScript-like syntax familiar to web developers
4. **Tool Integration**: Qt Creator provides IDE support for content creation

**Modern Parallel**: This anticipates modern approaches like Unity's prefabs, Godot's scenes, and Bevy's entity bundles. Declarative composition is now standard; CCF was early.

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

Components could be mixed and matched without inheritance constraints. A "motorcycle with sidecar" might combine Mobile + CrewCapacity without fitting into any inheritance hierarchy.

**Lesson Validated**: OpenCombat's ECS (Entity Component System) approach achieves similar flexibility with better performance. Composition over inheritance is now architectural consensus.

### 9.4.3 Maximum Moddability

CCF's declarative approach enabled unprecedented modding:

| Mod Type | CCF Support | OpenCombat-SDL |
|----------|-------------|----------------|
| New scenarios | ✓ (QML files) | ✗ |
| New unit types | ✓ (QML entities) | Partial (XML) |
| New behaviors | ✓ (QML state machines) | ✗ |
| UI customization | ✓ (QML UI files) | ✗ |
| New weapons with unique mechanics | ✓ (Component composition) | Partial (data only) |

The C++ backend exposed functionality; QML composed it. Modders could create entirely new game modes without touching C++.

### 9.4.4 Clean Separation of Concerns

CCF maintained clear boundaries:

- **C++ Backend**: Simulation logic, physics, pathfinding
- **QML Frontend**: UI, scenario definition, visual effects
- **Communication**: Qt signals/slots for loose coupling

This separation meant:
- Artists could modify visuals without programmer support
- Designers could iterate scenarios without recompilation
- Programmers could refactor simulation without breaking UI

---

## 9.5 What CloseCombatFree Got Wrong

Innovation brings risks. CCF's declarative approach introduced challenges that subsequent projects had to solve.

### 9.5.1 QML Performance Overhead

QML's flexibility came with runtime cost:

```qml
// Each QML property requires meta-object system overhead
property int health: 100  // Behind the scenes: QMetaProperty, signals, bindings
```

**The Problem**: Hundreds of QML entities updating every frame strained the Qt runtime. The QML garbage collector could cause frame hitches. Property bindings, while convenient, created invisible dependencies hard to optimize.

**Measured Impact**: CCF documentation notes "hundreds of units may lag." For a tactical wargame targeting squad-level combat (typically 50-200 units), this was acceptable. For larger scales, it became prohibitive.

**The Lesson**: OpenCombat's Rust ECS provides similar flexibility with compile-time optimization. Data-oriented design (arrays of structs vs. structs of arrays) matters for cache performance.

### 9.5.2 Limited Determinism for Multiplayer

CCF's architecture made deterministic multiplayer challenging:

- **QML State Changes**: String-based status transitions ("MOVING" → "AIMING") lacked strict validation
- **Floating-Point Variance**: Physics calculations varied across platforms
- **No Event Log**: State mutations weren't centrally logged
- **Client Authority**: Each client ran its own simulation

**The Problem**: Determinism—the property that identical inputs produce identical outputs—is essential for:
- Lockstep multiplayer (all clients agree on state)
- Replay recording (inputs reconstruct game)
- Debugging (reproduce bugs reliably)

CCF's design prioritized flexibility over determinism, making these features difficult to add.

### 9.5.3 Qt Dependency

Building on Qt provided rapid development but created constraints:

**Deployment Complexity:**
- Qt libraries must accompany the executable
- Platform-specific builds required
- Static linking increased binary size significantly

**Learning Curve:**
- Contributors needed Qt knowledge
- C++/QML integration had subtle complexities
- Qt's meta-object compiler (moc) added build complexity

**Platform Limitations:**
- Qt 5's mobile support was promising but incomplete
- Web deployment (Qt WebAssembly) wasn't production-ready
- Console platforms unsupported

**The Lesson**: OpenCombat minimized dependencies: ggez (graphics), zmq (networking), serde (serialization). Each dependency was lightweight and replaceable. Modern Rust's ecosystem enables this modular approach more than C++ ever could.

---

## 9.6 What OpenCombat Got Right

OpenCombat represents the synthesis of lessons learned. Its architecture addresses limitations of both predecessors while adding innovations of its own.

### 9.6.1 ECS for Simulation

OpenCombat's modified Entity Component System provides flexibility without sacrificing performance:

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

**Why ECS Matters:**

1. **Cache Efficiency**: Iterating `soldiers` accesses contiguous memory
2. **Type Safety**: Cannot accidentally use a SoldierIndex as VehicleIndex
3. **Serialization**: Indices serialize naturally for network/replay
4. **Parallelism**: Systems can process components in parallel

**Evolution from Predecessors**: 
- OpenCombat-SDL's deep inheritance → Flat ECS
- CloseCombatFree's QML composition → Data-oriented composition

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

**Benefits:**

1. **Determinism**: Same messages + same initial state = same result
2. **Replay**: Log messages to recreate any game
3. **Multiplayer**: Clients apply server-broadcast messages
4. **Debugging**: Message log reveals exactly what changed and when

**The Insight**: This pattern enables features that seem unrelated. Replay recording, multiplayer synchronization, and debug logging all emerge from the same architectural choice.

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

**Impact on Development:**
- Balance testing without recompilation
- Immediate feedback on parameter changes
- Debugging via live value modification

CloseCombatFree achieved similar benefits through QML hot reload, but OpenCombat's Rust implementation provides type safety and compile-time verification alongside runtime flexibility.

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

Combined with hot reload, this enables:
- Rapid scenario iteration
- Modding without code changes
- Version-controlled content
- A/B testing different configurations

---

## 9.7 What OpenCombat Got Wrong

Even well-designed systems have limitations. OpenCombat's complexity creates barriers that subsequent developers should consider.

### 9.7.1 Complexity Overhead

The three-tier state hierarchy adds conceptual overhead:

```rust
pub struct SoldierState {
    pub phase: Phase,       // Game level: Deployment, Battle, End
    pub behavior: Behavior, // Tactical: MoveTo, Defend, Engage
    pub gesture: Gesture,   // Immediate: Aiming, Firing, Reloading
}
```

**The Cost:**
- New team members must understand the tier model
- State transitions must consider all three levels
- Debugging requires checking three enums, not one
- Serialization must handle all three tiers

**When It's Worth It:**
- Multiplayer determinism required
- Complex state interactions common
- Replay/recording features essential

**When It's Not:**
- Single-player only
- Simple state machines suffice
- Rapid prototyping phase

### 9.7.2 Rust Learning Curve

Rust's ownership and borrow checker create friction:

```rust
// Fighting the borrow checker
fn update_soldier(world: &mut World, idx: SoldierIndex) {
    let soldier = &mut world.soldiers[idx.0];
    // ERROR: cannot borrow `world.soldiers` as immutable 
    // because it's borrowed as mutable
    let cover = find_cover(&world.soldiers, soldier.position);
}
```

**The Challenge:**
- Rust has a steep learning curve
- Team members need Rust expertise
- Some patterns are awkward in Rust (graph structures, cyclic references)
- Compile times can be slow

**The Trade-off**: Memory safety eliminates entire classes of bugs (null pointers, use-after-free, data races). For a project intended to run for years with community contributions, this may be worth the initial investment. For rapid prototyping, it may not be.

### 9.7.3 Limited Accessibility for Modders

While OpenCombat supports data modding, behavioral modding requires Rust:

| Mod Type | OpenCombat | CloseCombatFree |
|----------|------------|-----------------|
| New scenarios | ✓ (JSON) | ✓ (QML) |
| New unit types | Partial (JSON) | ✓ (QML) |
| New behaviors | ✗ (requires Rust) | ✓ (QML) |
| AI customization | ✗ (requires Rust) | Partial (QML states) |

**The Gap**: QML enabled non-programmers to create game content. Rust requires programming expertise. For community-driven games, this limits the modding audience.

**Potential Solutions:**
- Lua scripting for behaviors (adds dependency)
- Visual behavior tree editors (significant tooling investment)
- WebAssembly modding (complex but sandboxed)

---

## 9.8 Universal Lessons

Beyond specific implementations, three projects reveal enduring truths about tactical wargame architecture.

### 9.8.1 Moddability Must Be Architected from Day One

You cannot retrofit true moddability onto a hardcoded system. The three projects demonstrate a spectrum:

**OpenCombat-SDL**: Hardcoded behaviors, XML data → Limited modding
**CloseCombatFree**: Declarative QML → Extensive modding
**OpenCombat**: Deterministic Rust with JSON → Moderate modding

**The Rule**: Decide your modding goals before writing code:
- Parameter tweaking? → XML/JSON sufficient
- New content? → Data-driven definitions required
- New mechanics? → Scripting or declarative system required
- Total conversions? → Full mod API with documentation

### 9.8.2 Determinism Is Easier to Add Early Than Later

OpenCombat-SDL had no multiplayer. CloseCombatFree had no determinism. OpenCombat built determinism from day one.

**Why Retrofitting Fails:**
- Direct state mutation scattered through code
- Floating-point operations vary by platform
- Random number generators not seeded consistently
- Frame-rate dependent logic

**Determinism Checklist:**
- [ ] Fixed timestep simulation (not framerate-dependent)
- [ ] Seeded RNG with replay capability
- [ ] No direct state mutation (messages only)
- [ ] No floating-point in critical calculations (or consistent rounding)
- [ ] Deterministic iteration order (sorted collections)

### 9.8.3 State Management Is the Core Challenge

All three projects invested heavily in state management because it underpins every other system:

| Project | Approach | Strength | Weakness |
|---------|----------|----------|----------|
| OpenCombat-SDL | Bitfield | Composable, fast | 64-state limit |
| CloseCombatFree | String-based | Flexible, moddable | No type safety |
| OpenCombat | Hierarchical | Clear relationships | Complex |

**The Insight**: State architecture decisions shape what's possible in your game. Choose based on:
- **Orthogonal states**: Bitfield (OpenCombat-SDL pattern)
- **Modding flexibility**: String-based (CloseCombatFree pattern)
- **Multiplayer determinism**: Hierarchical with temporal data (OpenCombat pattern)

### 9.8.4 AI and Simulation Must Be Separate

OpenCombat-SDL had simulation but no AI. CloseCombatFree had minimal AI. OpenCombat integrated both.

**The Principle:**
- **Simulation**: Physics, ballistics, line-of-sight (deterministic)
- **AI**: Decision-making, target selection, pathfinding (heuristic)

**Why Separation Matters:**
1. **Testing**: Simulation testable without AI
2. **Difficulty Scaling**: AI can be tuned without changing physics
3. **Modding**: AI behavior replaceable without affecting core systems
4. **Debugging**: Clear whether bugs are in decision-making or execution

### 9.8.5 Content Should Be Data, Not Code

All three projects moved toward data-driven design, but with different capabilities:

```
OpenCombat-SDL: Data defines parameters
CloseCombatFree: Data defines structure AND behavior
OpenCombat: Data defines structure, code defines behavior
```

**The Evolution**: Each project pushed further toward data. The trend is clear—separate what changes frequently (content) from what changes rarely (engine).

---

## 9.9 Advice for New Projects

### The 10 Commandments of Tactical Wargame Development

**1. Thou Shalt Separate Intent from Execution**

Player orders are not unit actions. Maintain at least two tiers:
- Orders: "Defend this position"
- Actions: "Move to position → Face direction → Set stance"

**2. Thou Shalt Not Hardcode Behaviors**

If modders might want to change it, externalize it:
- Weapon mechanics → Script or data
- AI decisions → Behavior trees
- Movement rules → Configuration

**3. Thou Shalt Honor the Cache**

Memory layout affects performance more than algorithmic complexity:
- Contiguous arrays over linked lists
- Structs of arrays (SoA) for SIMD
- Avoid pointer chasing

**4. Thou Shalt Not Mix Rendering with Simulation**

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

**5. Thou Shalt Make State Explicit**

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

**6. Thou Shalt Plan for Multiplayer from Day One**

Even if single-player only now:
- Use message-driven state updates
- Avoid client-authoritative simulation
- Design for determinism

Retrofitting multiplayer requires architectural overhaul.

**7. Thou Shalt Not Optimize Prematurely**

OpenCombat-SDL's bitfield system was elegant but unnecessary for <100 units. CloseCombatFree's QML overhead didn't matter for proof-of-concept.

Optimize when:
- Profiling identifies bottlenecks
- Target scale exceeds current performance
- User experience degraded

**8. Thou Shalt Document Why, Not What**

```cpp
// BAD: What the code does
state_bits |= PRONE;  // Set prone bit

// GOOD: Why it matters
state_bits |= PRONE;  // Prone reduces silhouette, increasing cover effectiveness
```

**9. Thou Shalt Test Determinism**

```rust
#[test]
fn test_determinism() {
    let state1 = run_simulation(SEED, INPUTS);
    let state2 = run_simulation(SEED, INPUTS);
    assert_eq!(hash(state1), hash(state2));
}
```

**10. Thou Shalt Choose the Right Complexity**

| Game Type | Recommended Approach |
|-----------|---------------------|
| First tactical game | OpenCombat-SDL patterns (simple, intuitive) |
| Commercial multiplayer | OpenCombat patterns (deterministic, server-authoritative) |
| Community modding platform | CloseCombatFree patterns (declarative, flexible) |
| AAA production | Synthesis of all three |

---

## 9.10 Conclusion: The Wisdom of Iteration

Three projects. Three eras. Three philosophies. Each made different trade-offs; each succeeded in some ways and failed in others. The accumulated wisdom isn't a prescription but a map of the design space.

**What Endures:**

1. **Separation of concerns** (orders/behaviors, simulation/rendering)
2. **State management discipline** (explicit, debuggable, serializable)
3. **Data-driven design** (separate changing content from stable engine)
4. **Type safety** (prevent bugs through architecture)
5. **Modular architecture** (replace components without rewriting)

**What Depends on Context:**

1. **Language choice** (Rust vs. C++ vs. C# matters less than architecture)
2. **State representation** (bitfield vs. enum vs. string based on needs)
3. **Inheritance depth** (composition generally preferred, but inheritance isn't evil)
4. **Scripting vs. compiled** (performance vs. flexibility trade-off)
5. **Determinism overhead** (necessary for multiplayer, overhead for single-player)

**The Final Lesson:**

Architecture is the art of managing trade-offs. The Close Combat clones demonstrate that the same game concept can be implemented successfully with radically different approaches. Your job is not to find the "correct" architecture but to understand your constraints—team size, timeline, target audience, platform—and choose appropriately.

The 20-year journey from OpenCombat-SDL through CloseCombatFree to OpenCombat shows that good ideas persist (bitfield states, component composition) while technological constraints shift (XML to JSON, QML to ECS, manual memory management to Rust's ownership). The patterns in this book provide vocabulary; the lessons in this chapter provide judgment.

Build thoughtfully. Iterate fearlessly. Learn continuously.

---

**End of Chapter 9**

---

*This chapter synthesizes development experience from three Close Combat clone implementations spanning 2005–2024. The lessons represent accumulated wisdom from approximately 20 years of tactical wargame development across multiple technology generations.*

**Version:** 2.0  
**Date:** February 2026  
**Status:** Publishable Reference
