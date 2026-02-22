# Chapter 1: Executive Summary

## Universal Design Patterns in Tactical Wargames

*Three decades of Close Combat development reveal enduring truths about squad-level tactical simulation.*

---

## The Challenge

Creating a compelling real-time tactical wargame requires balancing competing demands: authenticity versus playability, player agency versus emergent behavior, systemic depth versus approachable complexity. The three implementations examined in this book—developed across nearly two decades—demonstrate that these tensions cannot be resolved by technology alone. Rather, they demand thoughtful architectural decisions that shape the fundamental player experience.

This chapter distills universal lessons from three distinct approaches to the same problem. Whether you are designing a new tactical wargame or analyzing existing systems, these patterns provide a framework for understanding the design space and making informed trade-offs.

---

## The Three Architectural Philosophies

The projects analyzed here represent three distinct eras of game development philosophy:

| Project | Era | Core Philosophy | Primary Innovation |
|---------|-----|-----------------|-------------------|
| **OpenCombat-SDL** | 2005–2008 | Classical Object-Oriented Design | 64-bit composable state bitfields with automatic prerequisite chaining |
| **CloseCombatFree** | 2011–2012 | Declarative/Component Architecture | QML-based content definition enabling runtime modding |
| **OpenCombat** | 2020–2024 | Systems-Oriented Architecture | Three-tier state hierarchy with message-driven simulation |

Each approach embodies different assumptions about how players interact with tactical systems, how content should be authored, and where complexity should reside. None is objectively superior; each represents a valid response to specific design goals and technical constraints.

---

## Seven Universal Architectural Decisions

### Decision 1: State Representation Strategy

**The Question:** How do you represent what units are doing at any given moment?

Every tactical game must track unit states: posture, movement, combat engagement, and physical condition. The three projects reveal three fundamentally different approaches:

| Approach | Implementation | Strengths | Weaknesses | Best For |
|----------|---------------|-----------|------------|----------|
| **Bitfield Compositing** | Single integer with bitwise flags | Memory efficient; states compose naturally | Limited to 64 states; no inherent state hierarchy | Systems with many orthogonal states |
| **String Enumeration** | Human-readable text identifiers | Self-documenting; flexible for modding | Runtime errors possible; less efficient | Content-heavy games with frequent modding |
| **Hierarchical Typing** | Structured data with nested categories | Type-safe; clear state relationships | More complex to serialize for network play | Multiplayer-focused simulations |

**Recommendation:** Choose based on your modding strategy. If you want players to create content without programming tools, string-based states offer the necessary flexibility. If you are building a competitive multiplayer game, hierarchical typing prevents entire classes of synchronization bugs. For single-player focused simulations, bitfields provide elegance and efficiency.

---

### Decision 2: Command Abstraction Layers

**The Question:** How many layers should exist between player intent and unit action?

All three implementations recognize that players think in objectives while units execute behaviors. They differ in how they bridge this gap:

**Single-Layer Systems** (CloseCombatFree): Player commands translate directly to unit actions. A "Move" command immediately begins movement animation. This approach offers immediacy and clarity but limits the system's ability to interpret player intent intelligently.

**Two-Layer Systems** (OpenCombat-SDL): Player orders translate to actions, which execute against the state machine. Orders persist as intent, while actions represent execution. This allows automatic prerequisite handling—if a prone soldier is ordered to run, the system automatically inserts a "stand up" action without player micromanagement.

**Three-Layer Systems** (OpenCombat): Orders inform behaviors, which drive gestures. The behavior layer interprets orders contextually—an "Engage Squad" order becomes "Engage Specific Soldier" when a target is acquired. The gesture layer manages moment-to-moment physical actions like aiming and firing. This separation enables sophisticated AI responses while maintaining clear player intent.

**Decision Matrix:**

| Game Type | Recommended Layers | Rationale |
|-----------|-------------------|-----------|
| Fast-paced action | 1 | Minimizes latency between input and response |
| Traditional tactical | 2 | Balances player control with intelligent automation |
| Simulation-focused | 3 | Enables emergent behavior from complex interactions |

---

### Decision 3: Entity Composition Model

**The Question:** How do you structure the relationship between soldiers, equipment, and squads?

The object model defines how game entities relate to each other and how data flows through the system:

**Inheritance Hierarchies** (OpenCombat-SDL): Units inherit from base classes (Soldier, Vehicle) that define common behavior. This creates clean conceptual models—"all soldiers can move"—but can lead to rigid structures when you need soldiers who behave like vehicles or vice versa.

**Component Composition** (CloseCombatFree): Entities are composed of capabilities attached at runtime. A "tank" might have a "Mobile" component, a "Weapon" component, and a "Crew" component. This flexibility enables unexpected combinations but requires more complex systems to handle interactions between components.

**Embedded Component Systems** (OpenCombat): A middle ground where entities are structs containing all possible components, but systems process them selectively. This provides type safety and cache-friendly memory layouts while avoiding deep inheritance chains.

**Key Insight:** The choice affects not just code organization but gameplay possibilities. Component systems enable unit types that would be awkward in inheritance hierarchies—a drone that acts like both a soldier and a vehicle, or a defensive position that acts like terrain but can be destroyed like a unit.

---

### Decision 4: Simulation Authority Architecture

**The Question:** Where does the "truth" of the game state reside?

**Client-Authoritative** (OpenCombat-SDL, CloseCombatFree): The local game client runs the simulation. This provides responsive controls and works well for single-player or cooperative multiplayer. However, it makes competitive multiplayer vulnerable to cheating and can cause synchronization issues when players have different system performance.

**Server-Authoritative** (OpenCombat): A dedicated server (or embedded server for single-player) maintains the authoritative game state. Clients send commands and receive state updates. This architecture is essential for competitive multiplayer and enables features like deterministic replay recording.

**Trade-off Analysis:**

| Feature | Client-Authoritative | Server-Authoritative |
|---------|---------------------|---------------------|
| Input latency | Minimal | Higher (requires round-trip) |
| Anti-cheat | Client-side only | Server validates all actions |
| Determinism | Platform-dependent | Fully deterministic |
| Replay support | Limited | Complete |
| Single-player complexity | Lower | Higher (server management) |

**Recommendation:** If multiplayer is a core feature, invest in server-authoritative architecture early. Retrofitting deterministic simulation onto a client-authoritative codebase requires fundamental restructuring.

---

### Decision 5: Content Definition Strategy

**The Question:** How do you enable players and designers to create new content?

The three projects demonstrate a spectrum of content authoring approaches:

**Data-Driven** (OpenCombat-SDL): Units, weapons, and terrain are defined in configuration files (XML). This separates content from code but limits content creators to pre-defined parameters—new behaviors require code changes.

**Scenario-Driven** (OpenCombat): Deployment configurations (JSON) define unit placement and initial states, while core mechanics remain hardcoded. This enables scenario variety within fixed systems.

**Declarative Scripting** (CloseCombatFree): Content is defined in QML, a declarative language that can express both data and behavior. This enables true modding—new unit types with unique behaviors can be created without recompiling the game.

**Content Capability Matrix:**

| Modding Goal | XML/JSON | Declarative Scripts |
|-------------|----------|---------------------|
| Add new maps | ✓ | ✓ |
| Create scenarios | ✓ | ✓ |
| Add new weapons | ✓ (if data-only) | ✓ (with new behaviors) |
| Create new unit types | ✗ | ✓ |
| Add new game mechanics | ✗ | ✓ (limited) |

**Recommendation:** If your goal is community-driven content with deep gameplay changes, invest in declarative scripting capabilities. If your focus is on curated scenarios within fixed mechanics, data-driven approaches provide better performance and simpler debugging.

---

### Decision 6: State Synchronization Method

**The Question:** How do you keep all clients consistent in a networked game?

**Snapshot Synchronization**: Periodically send the complete game state. Simple to implement but bandwidth-intensive and can cause hitching.

**Delta Compression**: Send only changed fields. More efficient but requires tracking what each client knows.

**Event Sourcing** (OpenCombat): Send the events (messages) that caused state changes. Clients apply the same events to reach the same state. This enables replay recording and requires less bandwidth than snapshots, but adds complexity to ensure all clients process events deterministically.

**Bandwidth Comparison** (per soldier update):

| Method | Typical Size | Best For |
|--------|-------------|----------|
| Full snapshot | ~200 bytes | Small player counts; simple debugging |
| Delta compression | ~50 bytes | Large player counts; predictable connections |
| Event sourcing | ~20-30 bytes | Deterministic simulation; replay requirements |

---

### Decision 7: AI Behavior Architecture

**The Question:** How do units decide what to do?

**Scripted Behaviors** (OpenCombat-SDL): Actions are explicitly defined with prerequisites and effects. The system chains actions automatically—stand up, then run, then fire. This provides predictable, debuggable behavior but can feel mechanical.

**State-Based AI** (CloseCombatFree): Units transition between states based on orders and events. Simpler to understand but harder to compose complex behaviors from simple rules.

**Behavior Trees with Perception** (OpenCombat): Units maintain awareness of their environment (visible enemies, under-fire status) and choose behaviors accordingly. A unit moving to a location might spontaneously switch to "engage" if an enemy appears, then resume movement afterward. This creates emergent behavior that feels more organic but requires careful tuning to remain predictable.

**Design Implications:**

| Approach | Player Expectation | Implementation Complexity |
|----------|-------------------|--------------------------|
| Scripted | High (predictable) | Low |
| State-based | Medium | Low-Medium |
| Perception-driven | Variable | High |

---

## Synthesis: Choosing Your Architecture

No single approach is correct for all tactical wargames. Your choices should flow from your core design goals:

**For Competitive Multiplayer**: Prioritize server-authoritative architecture, deterministic simulation, and hierarchical typing. Invest in event sourcing for efficient networking. Accept the complexity cost—these systems provide fairness and anti-cheat that competitive play demands.

**For Single-Player Simulation**: Consider client-authoritative design with three-layer command abstraction. Focus on perception-driven AI to create emergent storytelling. Bitfield states offer efficiency and elegance for solo play.

**For Modding Communities**: Invest in declarative content definition from day one. String-based states and component composition enable the flexibility that modders need. Document your content APIs as thoroughly as your code APIs.

**For Rapid Prototyping**: Start with two-layer command abstraction and data-driven content. These provide clear structure while remaining flexible enough to iterate on core mechanics.

---

## Key Insights for Game Designers

### 1. State Management Is the Foundation

All three projects invest heavily in state management because it underpins every other system. A well-designed state architecture makes features easier to add; a poorly designed one makes them nearly impossible. Choose your state representation before writing your first unit behavior.

### 2. The Order/Behavior/Gesture Separation Is Universal

Despite different terminology and implementation details, all three projects separate player intent from execution. This separation is not optional—it is the fundamental abstraction that makes tactical games playable. The question is not whether to use this separation, but how many layers to include.

### 3. Moddability Requires Architectural Commitment

True moddability—enabling players to add new mechanics, not just new content—must be designed in from the beginning. CloseCombatFree's QML approach offers the most flexibility because it was designed for it. OpenCombat-SDL's XML system supports content but not mechanics because that was the design goal. Know which you want before you start.

### 4. Message-Driven Architecture Scales

OpenCombat's message-driven state updates, while initially appearing complex, provide the cleanest foundation for multiplayer, replay recording, and debugging. The ability to log and replay every state change is invaluable for both development and player experience.

### 5. Technical Debt Accumulates Around State Transitions

The most fragile code in all three projects involves state transitions—what happens when a unit dies mid-movement, or when a new order arrives during a prerequisite action. Invest in robust transition handling early; it will save countless debugging hours later.

---

## Looking Forward

The following chapters examine each of these architectural decisions in detail, comparing how the three implementations handle specific challenges:

- **Chapter 2: Entity Systems** – How units, equipment, and squads relate
- **Chapter 3: State Machines** – Managing unit behavior across contexts
- **Chapter 4: Command Systems** – From player intent to unit action
- **Chapter 5: World Simulation** – Terrain, line-of-sight, and environment
- **Chapter 6: Moddability** – Content creation and community extension

Each chapter maintains the focus on universal patterns rather than implementation details, providing a reference that remains relevant regardless of programming language or engine choice.

---

*The architecture of tactical wargames has evolved, but the core challenge remains: creating systems that are simultaneously deep enough to reward mastery and clear enough to be enjoyable. The patterns in this book provide a foundation for meeting that challenge.*

---

**Document Information**  
**Version:** 2.0  
**Date:** February 2026  
**Status:** Publishable Draft  
**Target Audience:** Game designers, technical architects, and researchers studying tactical wargame systems

---

## Decision Matrix Summary

| Decision | OpenCombat-SDL | CloseCombatFree | OpenCombat | Universal Recommendation |
|----------|---------------|-----------------|------------|------------------------|
| **State Representation** | 64-bit bitfield | String-based | Hierarchical enum | Choose based on modding needs |
| **Command Layers** | 2 (Order → Action) | 1 (direct) | 3 (Order → Behavior → Gesture) | 2 for balance, 3 for simulation |
| **Entity Model** | Deep inheritance | Component composition | Modified ECS | Embedded components for type safety |
| **Authority** | Client | Client | Server | Server for competitive multiplayer |
| **Content Definition** | XML | QML (declarative) | JSON | Declarative for modding, data for fixed mechanics |
| **Synchronization** | N/A (single-player) | N/A (single-player) | Event sourcing | Event sourcing for determinism |
| **AI Architecture** | Scripted actions | State-based | Perception-driven | Perception-driven for emergent behavior |

---

*End of Chapter 1*
