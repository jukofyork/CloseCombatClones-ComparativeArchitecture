# Chapter 1: Executive Summary

## Universal Design Patterns in Tactical Wargames

*Three decades of Close Combat development show how squad-level tactical simulations endure.*

---

## The Challenge

A compelling real-time tactical wargame must balance authenticity with playability, player agency with emergent behavior, and depth with approachable complexity. The three implementations in this book—spanning nearly twenty years—prove that technology alone can't resolve these tensions. They require deliberate architectural choices that define the player's experience.

This chapter extracts key lessons from three different solutions to the same problem. These patterns help designers build new tactical wargames or evaluate existing systems by clarifying the trade-offs at play.

---

## The Three Architectural Philosophies

The projects here reflect three eras of game development thinking:

| Project         | Era       | Core Philosophy                    | Primary Innovation                                                     |
| --------------- | --------- | ---------------------------------- | ---------------------------------------------------------------------- |
| **OpenCombat-SDL**  | 2005–2008 | Classical Object-Oriented Design   | 64-bit composable state bitfields with automatic prerequisite chaining |
| **CloseCombatFree** | 2011–2012 | Declarative/Component Architecture | QML-based content definition enabling runtime modding                  |
| **OpenCombat**      | 2020–2024 | Systems-Oriented Architecture      | Three-tier state hierarchy with message-driven simulation              |

Each approach makes different assumptions about player interaction, content creation, and where complexity belongs. None is inherently better; each solves specific design problems within its technical limits.

---

## Seven Universal Architectural Decisions

### Decision 1: State Representation Strategy

**The Question:** How should the game track what units are doing at any moment?

Tactical games need to monitor unit states—posture, movement, combat engagement, and physical condition. The three projects take different approaches:

| Approach             | Implementation                         | Strengths                                  | Weaknesses                                        | Best For                                  |
| -------------------- | -------------------------------------- | ------------------------------------------ | ------------------------------------------------- | ----------------------------------------- |
| **Bitfield Compositing** | Single integer with bitwise flags      | Memory efficient; states compose naturally | Limited to 64 states; no inherent state hierarchy | Systems with many orthogonal states       |
| **String Enumeration**   | Human-readable text identifiers        | Self-documenting; flexible for modding     | Runtime errors possible; less efficient           | Content-heavy games with frequent modding |
| **Hierarchical Typing**  | Structured data with nested categories | Type-safe; clear state relationships       | More complex to serialize for network play        | Multiplayer-focused simulations           |

**Recommendation:** Match the approach to your modding strategy. String-based states work best when players create content without programming tools. Hierarchical typing suits competitive multiplayer games, preventing synchronization bugs. For single-player simulations, bitfields offer efficiency and simplicity.

---

### Decision 2: Command Abstraction Layers

**The Question:** How many layers should separate player intent from unit action?

Players think in objectives, but units execute behaviors. The projects handle this gap differently:

**Single-Layer Systems** (CloseCombatFree): Player commands trigger unit actions immediately. A "Move" command starts the movement animation right away. This approach feels responsive but limits intelligent interpretation of player intent.

**Two-Layer Systems** (OpenCombat-SDL): Player orders become actions executed against the state machine. Orders represent intent, while actions handle execution. The system can automatically insert prerequisites—if a prone soldier gets a "run" order, it adds a "stand up" action without player input.

**Three-Layer Systems** (OpenCombat): Orders inform behaviors, which drive gestures. The behavior layer interprets orders contextually—an "Engage Squad" order refines to "Engage Specific Soldier" once a target appears. The gesture layer manages physical actions like aiming and firing. This separation allows sophisticated AI while keeping player intent clear.

**Decision Matrix:**

| Game Type            | Recommended Layers | Rationale                                            |
| -------------------- | ------------------ | ---------------------------------------------------- |
| Fast-paced action    | 1                  | Reduces input-to-response latency                    |
| Traditional tactical | 2                  | Balances control with automation                     |
| Simulation-focused   | 3                  | Supports emergent behavior from complex interactions |

---

### Decision 3: Entity Composition Model

**The Question:** How should soldiers, equipment, and squads relate to each other?

The object model shapes how game entities interact and how data moves through the system.

**Inheritance Hierarchies** (OpenCombat-SDL): Units derive from base classes like Soldier or Vehicle, which define shared behavior. This approach keeps concepts clean—soldiers move, vehicles drive—but struggles when units need to cross categories, like a soldier that behaves like a vehicle.

**Component Composition** (CloseCombatFree): Entities assemble from capabilities added at runtime. A tank might combine a Mobile component, a Weapon component, and a Crew component. This flexibility allows unusual combinations but demands more complex systems to manage interactions.

**Embedded Component Systems** (OpenCombat): Entities are structs holding all possible components, while systems process only the relevant ones. This balances type safety and cache efficiency without deep inheritance.

**Key Insight:** The choice shapes gameplay. Component systems make unit types possible that inheritance can't easily support—a drone acting as both soldier and vehicle, or a defensive position that functions as terrain but can be destroyed like a unit.

---

### Decision 4: Simulation Authority Architecture

**The Question:** Where does the game state live?

**Client-Authoritative** (OpenCombat-SDL, CloseCombatFree): The local client runs the simulation. Controls feel responsive, and it works well for single-player or cooperative games. Competitive multiplayer becomes vulnerable to cheating, though, and performance differences between players can cause sync issues.

**Server-Authoritative** (OpenCombat): A dedicated server (or an embedded one for single-player) holds the true game state. Clients send commands and receive updates. This setup is necessary for competitive multiplayer and enables features like deterministic replays.

**Trade-offs:**

| Feature             | Client-Authoritative | Server-Authoritative        |
| ------------------- | -------------------- | --------------------------- |
| Input latency       | Minimal              | Higher (round-trip delay)   |
| Anti-cheat          | Client-side only     | Server validates everything |
| Determinism         | Depends on platform  | Fully deterministic         |
| Replay support      | Limited              | Complete                    |
| Single-player setup | Simpler              | More complex                |

**Recommendation:** If multiplayer matters, build server-authoritative from the start. Retrofitting it later means rewriting core simulation logic.

---

### Decision 5: Content Definition Strategy

**The Question:** How should players and designers create new content?

The three projects take different approaches to content creation:

**Data-Driven** (OpenCombat-SDL): Units, weapons, and terrain live in XML configuration files. This keeps content separate from code but restricts creators to existing parameters—new behaviors mean modifying the source.

**Scenario-Driven** (OpenCombat): JSON files define unit placement and starting conditions, while core mechanics stay hardcoded. The system allows varied scenarios within fixed rules.

**Declarative Scripting** (CloseCombatFree): QML handles both data and behavior. Modders can create new unit types with unique behaviors without recompiling.

**Content Capability Matrix:**

| Modding Goal           | XML/JSON         | Declarative Scripts    |
| ---------------------- | ---------------- | ---------------------- |
| Add new maps           | ✓                | ✓                      |
| Create scenarios       | ✓                | ✓                      |
| Add new weapons        | ✓ (if data-only) | ✓ (with new behaviors) |
| Create new unit types  | ✗                | ✓                      |
| Add new game mechanics | ✗                | ✓ (limited)            |

**Recommendation:** For deep gameplay changes and community-driven content, declarative scripting works best. For curated scenarios with fixed mechanics, data-driven approaches offer better performance and easier debugging.

---

### Decision 6: State Synchronization Method

**The Question:** How do you maintain consistency across networked clients?

**Snapshot Synchronization**: The game sends the complete state at intervals. The method is simple but consumes more bandwidth and may cause stuttering.

**Delta Compression**: Only changed fields are transmitted. This saves bandwidth but requires tracking each client's knowledge.

**Event Sourcing** (OpenCombat): The system sends events that alter state. Clients apply the same events to reach identical states. This approach supports replays and uses less bandwidth than snapshots, though it demands deterministic event processing.

**Bandwidth Comparison** (per soldier update):

| Method            | Typical Size | Best For                                |
| ----------------- | ------------ | --------------------------------------- |
| Full snapshot     | ~200 bytes   | Small player counts; simple debugging   |
| Delta compression | ~50 bytes    | Large player counts; stable connections |
| Event sourcing    | ~20-30 bytes | Deterministic simulations; replay needs |

---

### Decision 7: AI Behavior Architecture

**The Question:** How do units make decisions?

**Scripted Behaviors** (OpenCombat-SDL): Actions follow explicit rules with prerequisites and effects. The system chains them automatically—stand, run, fire. The result is predictable and easy to debug but can feel rigid.

**State-Based AI** (CloseCombatFree): Units shift between states based on orders and events. The model is simple to grasp but makes complex behaviors harder to build.

**Behavior Trees with Perception** (OpenCombat): Units track their surroundings—visible enemies, incoming fire—and adjust actions dynamically. A unit moving to a position might engage an enemy, then resume its path. This creates more natural behavior but requires careful tuning to stay predictable.

**Design Implications:**

| Approach          | Player Expectation | Implementation Complexity |
| ----------------- | ------------------ | ------------------------- |
| Scripted          | High (predictable) | Low                       |
| State-based       | Medium             | Low-Medium                |
| Perception-driven | Variable           | High                      |

---

## Synthesis: Choosing Your Architecture

No single approach works for every tactical wargame. Your architecture should align with your core design goals:

**Competitive Multiplayer** needs server-authoritative architecture, deterministic simulation, and hierarchical typing. Event sourcing keeps networking efficient. The complexity is justified—these systems ensure fairness and prevent cheating.

**Single-Player Simulation** benefits from client-authoritative design with three-layer command abstraction. Perception-driven AI creates emergent storytelling, while bitfield states offer efficiency and elegance for solo play.

**Modding Communities** require declarative content definition from the start. String-based states and component composition provide the flexibility modders need. Document your content APIs as thoroughly as your code.

**Rapid Prototyping** works best with two-layer command abstraction and data-driven content. This approach balances structure and flexibility for iterating on core mechanics.

---

## Key Insights for Game Designers

### 1. State Management Is the Foundation

All three projects prioritize state management because it supports every other system. A well-designed state architecture simplifies feature development; a poor one makes it nearly impossible. Decide on your state representation before writing unit behaviors.

### 2. The Order/Behavior/Gesture Separation Is Essential

Though terminology and implementation differ, all three projects separate player intent from execution. This separation isn't optional—it's the core abstraction that makes tactical games playable. The real question is how many layers to include.

### 3. Moddability Requires Early Commitment

True moddability—letting players add new mechanics, not just content—must be part of the initial design. CloseCombatFree's QML approach offers the most flexibility because it was built for it. OpenCombat-SDL's XML system supports content but not mechanics because that wasn't the goal. Decide what you need before starting.

### 4. Message-Driven Architecture Scales Well

OpenCombat's message-driven state updates may seem complex at first, but they provide the cleanest foundation for multiplayer, replay recording, and debugging. Logging and replaying every state change proves invaluable for development and player experience.

### 5. State Transitions Create Technical Debt

The most fragile code in all three projects involves state transitions—like a unit dying mid-movement or a new order arriving during an action. Build robust transition handling early to avoid countless debugging hours later.

---

## Looking Forward

The next chapters break down each architectural decision, showing how the three implementations tackle specific challenges:

- **Chapter 2: Entity Systems** – How units, equipment, and squads connect
- **Chapter 3: State Machines** – Controlling unit behavior in different situations
- **Chapter 4: Command Systems** – Turning player input into unit actions
- **Chapter 5: World Simulation** – Terrain, visibility, and environmental effects
- **Chapter 6: Moddability** – Building tools for content creation and community expansion

Each chapter keeps the focus on patterns that work across languages and engines, offering a durable reference.

---

Tactical wargames have changed, but the central problem hasn't: systems must be deep enough to reward skill yet simple enough to stay fun. The patterns here help solve that problem.

---

## Decision Matrix Summary

| Decision             | OpenCombat-SDL      | CloseCombatFree       | OpenCombat                     | Universal Recommendation                              |
| -------------------- | ------------------- | --------------------- | ------------------------------ | ----------------------------------------------------- |
| **State Representation** | 64-bit bitfield     | String-based          | Hierarchical enum              | Pick based on modding needs                           |
| **Command Layers**       | 2 (Order → Action)  | 1 (direct)            | 3 (Order → Behavior → Gesture) | Two layers for balance, three for depth               |
| **Entity Model**         | Deep inheritance    | Component composition | Modified ECS                   | Embedded components keep type safety                  |
| **Authority**            | Client              | Client                | Server                         | Use servers for competitive multiplayer               |
| **Content Definition**   | XML                 | QML (declarative)     | JSON                           | Declarative works best for mods, data for fixed rules |
| **Synchronization**      | N/A (single-player) | N/A (single-player)   | Event sourcing                 | Event sourcing ensures determinism                    |
| **AI Architecture**      | Scripted actions    | State-based           | Perception-driven              | Perception-driven AI creates emergent behavior        |

---

*Next: [Chapter 2: Architectural Philosophy](chapter_02_architectural_philosophy.md)*
