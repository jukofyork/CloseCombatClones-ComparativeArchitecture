# Introduction

## What Is a "Close Combat Clone"?

The term "Close Combat clone" carries specific meaning within the tactical wargaming community. It does not simply mean "any game about tactical combat." Instead, it refers to a particular style of real-time tactical simulation that emerged from Atomic Games' seminal Close Combat series—a style characterized by specific mechanics, design philosophies, and player experiences.

At its core, a Close Combat clone is:

> A real-time tactical wargame modeling squad-level infantry combat with an emphasis on realistic ballistics, morale and suppression mechanics, terrain-based cover systems, and autonomous AI that interprets player intent rather than requiring micromanagement of individual actions.

This definition distinguishes Close Combat clones from several related but distinct genres:

- **Turn-based tactical games** (XCOM, Jagged Alliance) use discrete turns rather than continuous real-time simulation
- **Real-time strategy games** (Command & Conquer, Company of Heroes) operate at larger scales with less focus on individual soldier psychology
- **Tactical shooters** (Squad, Arma) place the player in a first-person perspective rather than an overhead command view
- **Wargames** (Combat Mission, Graviteam Tactics) may share realism goals but often use turn-based or WeGo systems

A true Close Combat clone captures the essence of the original: the feeling of commanding a small group of soldiers who behave like human beings rather than automatons, where success depends on understanding and exploiting the tactical environment rather than mastering interface mechanics.

---

## The Close Combat Formula

The Close Combat series established a formula that subsequent clones attempt to replicate. Understanding this formula is essential to understanding the architectural challenges analyzed in this book.

### The Seven Pillars of Close Combat

**1. Squad-Based Organization**

Players don't control individual soldiers directly in most circumstances. Instead, they issue orders to squads—groups of 6-12 soldiers who move and fight together. Individual soldiers within a squad act autonomously, finding their own positions, engaging targets of opportunity, and reacting to threats. This abstraction level is crucial: it allows the player to focus on tactical decisions rather than micromanagement.

**2. Realistic Ballistics**

Weapons behave realistically. Bullets follow trajectories affected by gravity and cover. Suppression—firing near an enemy to pin them down—is often more important than actually hitting them. Different weapons have different characteristics: submachine guns are deadly at close range but ineffective at distance, rifles offer accuracy and range, machine guns provide sustained suppressive fire but require setup time.

**3. Morale and Suppression**

Soldiers have psychological states. Under fire, they become suppressed—less accurate, less responsive to orders, eventually panicking and fleeing or freezing. Morale is affected by casualties, leadership quality, and the intensity of incoming fire. This system creates tactical depth beyond pure firepower: a small volume of well-placed suppressing fire can neutralize a superior force.

**4. Terrain and Cover**

The environment matters profoundly. Buildings provide hard cover. Walls offer protection from specific angles. Elevation provides visibility advantages. Different terrain types affect movement speed. The interplay between terrain and combat creates the tactical puzzle that defines the genre: how do I advance my squad across this open field under fire? How can I use this building for cover while maintaining fields of fire?

**5. Autonomous AI**

Soldiers act intelligently without constant player input. They return fire when engaged. They seek cover when suppressed. They maintain situational awareness, tracking visible enemies and sharing information within their squad. The player issues high-level orders—"move to that building," "suppress that position"—and the AI handles the execution details.

**6. Real-Time with Pause (or Continuous)**

Combat unfolds in real-time, creating tension and preventing perfect optimization. Some implementations allow pausing to issue orders; others (like the original Close Combat) run continuously, demanding quick thinking under pressure. Either way, the real-time element distinguishes these games from turn-based alternatives.

**7. Emergent Narrative**

No two battles play out identically. The combination of realistic ballistics, morale systems, and AI autonomy creates emergent stories: the heroism of a pinned-down squad holding against impossible odds, the tragedy of a veteran leader cut down by a sniper, the panic of green troops breaking under their first bombardment. These narratives emerge organically from the systems rather than being scripted by designers.

---

## Why These Three Projects?

This book analyzes three specific implementations spanning twenty years of development (2005–2024). We chose these projects because they represent distinct architectural approaches while attempting to solve essentially the same problem.

### Representative Diversity

| Aspect | OpenCombat-SDL | CloseCombatFree | OpenCombat |
|--------|---------------|-----------------|------------|
| **Era** | 2005–2008 | 2011–2012 | 2020–2024 |
| **Language** | C++ | C++/QML | Rust |
| **Paradigm** | Classical OOP | Declarative | Systems-Oriented |
| **Primary Innovation** | 64-bit state bitfields | QML content modding | Deterministic simulation |
| **Multiplayer** | No | No | Yes (designed for) |
| **Moddability** | Data files | Full content | Configuration |

This diversity allows us to:
- **Compare paradigms**: Object-oriented vs. declarative vs. systems-oriented
- **Trace evolution**: How have best practices changed over two decades?
- **Identify universals**: What patterns appear regardless of paradigm?
- **Understand trade-offs**: What do you gain and lose with each approach?

### Serious Attempts at Hard Problems

These aren't toy projects or weekend experiments. Each represents a serious attempt to build a functioning tactical wargame:

- **OpenCombat-SDL** has a complete state machine system, action prerequisites, terrain analysis, and working combat mechanics
- **CloseCombatFree** successfully implements a declarative approach with runtime content loading
- **OpenCombat** achieves deterministic simulation suitable for multiplayer replay

Each project made different architectural choices, encountered different challenges, and discovered different solutions. By analyzing all three, we gain a comprehensive view of the solution space.

### Open Source Availability

All three projects are open source, meaning we can analyze their actual code rather than speculating about implementation. This book includes code excerpts, architectural diagrams derived from the source, and detailed comparisons of how specific systems work.

---

## How to Read This Book

This book is organized to serve different reading paths depending on your goals and background.

### For the Executive Overview Reader

**Read**: Chapter 1 (Executive Summary), Chapter 11 (Universal Patterns), Chapter 12 (Implementation Guide)  
**Time Investment**: 2-3 hours  
**Outcome**: Understanding of architectural approaches, key patterns, and practical recommendations

This path provides the "greatest hits"—the essential insights without the deep technical detail. Suitable for producers, technical directors evaluating approaches, or developers deciding whether to dive deeper.

### For the Game Architect

**Read**: Chapters 1-3 (Overview, Philosophy, State Management), Chapter 11 (Universal Patterns), Chapter 12 (Implementation Guide)  
**Optional**: Chapters 4-10 for specific systems of interest  
**Time Investment**: 8-12 hours  
**Outcome**: Comprehensive understanding of architectural trade-offs and pattern applications

This path focuses on the high-level design decisions that shape a project. You'll understand why each project made the choices they did and how to apply those lessons to your own architecture.

### For the Systems Programmer

**Read**: Chapters 3-8 (State Management, World Systems, Unit Hierarchy, Orders/AI, Moddability, Patterns), Chapter 11 (Universal Patterns), Chapter 12 (Implementation Guide), Technical Appendices A-C  
**Time Investment**: 15-20 hours  
**Outcome**: Detailed understanding of implementation approaches for specific systems

This path is for developers who will be writing the actual code. You'll find specific algorithms, data structures, and implementation patterns with code examples in multiple languages.

### For the Modder or Content Creator

**Read**: Chapter 7 (Moddability), Chapter 9 (Lessons Learned - modding sections), Chapter 10 (Recommendations - modding), Appendix C (File Formats)  
**Time Investment**: 4-6 hours  
**Outcome**: Understanding of how content systems work and how to extend them

This path focuses on data-driven design, file formats, and extensibility patterns. You'll learn how different architectures enable or constrain community content.

### For the Academic or Researcher

**Read**: Full book with emphasis on Chapter 2 (Architectural Philosophy), Chapter 9 (Lessons Learned), Chapter 11 (Universal Patterns)  
**Time Investment**: 20+ hours  
**Outcome**: Comprehensive understanding of architectural evolution and pattern theory

This path treats the book as a longitudinal case study in software architecture evolution, suitable for research into game development practices or software design patterns.

---

## What's NOT Covered

This book focuses specifically on the architectural and systems design aspects of tactical wargame development. Several related topics are explicitly outside our scope:

### Not Covered

- **Graphics and Rendering**: We analyze how the simulation state is represented, not how it's rendered. The three projects use different rendering approaches (SDL, Qt/QML, custom), but these are implementation details rather than architectural patterns.

- **Audio Systems**: Sound design, spatial audio, and music systems are important for immersion but don't affect the core tactical simulation architecture.

- **Input Handling**: While we discuss how player commands flow into the system (the command abstraction), we don't cover input device handling, key binding systems, or UI widget implementation.

- **Networking Infrastructure**: We discuss deterministic simulation architecture (necessary for multiplayer) but don't cover matchmaking, lobby systems, or network transport protocols.

- **Marketing and Business Considerations**: This is a technical book focused on architecture. We don't discuss monetization, community building, or publishing strategies.

- **Art Pipeline**: Creating sprites, 3D models, animations, and other art assets is crucial for a finished game but outside our architectural focus.

- **Platform-Specific Optimization**: Console certification, mobile optimization, and platform-specific SDKs are not discussed.

### What We DO Cover

- **State representation**: How unit state is modeled and managed
- **Entity composition**: How soldiers, squads, and vehicles relate
- **Command systems**: How player intent flows to unit actions
- **AI architecture**: How units make autonomous decisions
- **World simulation**: How terrain, cover, and visibility work
- **Moddability**: How content systems enable community extension
- **Pattern language**: Universal solutions to common problems

---

## Conventions Used in This Book

### Pseudocode

Throughout this book, we use pseudocode to illustrate concepts in a language-agnostic way. Our pseudocode conventions:

```
// Comments explain intent

ClassName {
    +publicField: Type
    -privateField: Type
    
    methodName(parameter: Type): ReturnType {
        // Implementation
        if (condition) {
            doSomething()
        }
        return value
    }
}

// Type declarations
variableName: Type = initialValue
constantName: ConstantType = value

// Generic types
Vec<T>           // Vector of type T
HashMap<K, V>    // Hash map from K to V
Option<T>        // Optional value (Some(T) or None)
Result<T, E>     // Result type (Ok(T) or Err(E))

// Control flow
if (condition) { } else { }
for (item in collection) { }
while (condition) { }
match (value) { pattern => result }

// Function declarations
fn functionName(param: Type) -> ReturnType { }
```

### Diagrams

We use Mermaid diagrams to illustrate architecture, data flow, and relationships. These diagrams render as flowcharts, class diagrams, and sequence diagrams. Where relevant, we include ASCII alternatives for accessibility.

### Code Examples

When showing actual implementation code, we indicate the source language:

```cpp
// C++ example from OpenCombat-SDL
class Soldier : public Object {
    State _currentState;
};
```

```rust
// Rust example from OpenCombat
struct Soldier {
    behavior: Behavior,
    gesture: Gesture,
}
```

```qml
// QML example from CloseCombatFree
Unit {
    id: soldier
    status: "IDLE"
}
```

### Terminology

We use specific terminology consistently throughout the book:

- **Soldier**: An individual infantryman (may be called "unit" in some contexts)
- **Squad**: A group of soldiers that move and fight together (6-12 soldiers)
- **Team**: A military formation larger than a squad (platoon, company, etc.)
- **Order**: A high-level command issued by the player ("move to location")
- **Action**: A low-level executable task ("walk to tile")
- **Behavior**: An AI-driven tactical decision ("engage enemy")
- **Gesture**: A moment-to-moment physical state ("aiming frame 245")
- **State**: The condition of a soldier or squad (posture, morale, activity)
- **Phase**: A global game state (deployment, combat, end)

### Pattern Format

In Chapter 11 (Universal Patterns), we present patterns using a standardized format:

1. **Name and Intent**: What the pattern accomplishes
2. **Problem**: The forces that make this pattern necessary
3. **Solution**: The structural and behavioral approach
4. **Mermaid Diagram**: Visual representation
5. **Variations**: How each project implements it
6. **Consequences**: Trade-offs and considerations
7. **Related Patterns**: Connections to other patterns

### Recommendation Boxes

Throughout the book, we highlight recommendations with special formatting:

> **Recommendation**: For multiplayer games, implement deterministic simulation from day one. Retrofitting determinism onto a non-deterministic codebase requires fundamental restructuring.

These recommendations synthesize lessons from all three projects and represent our best guidance for new implementations.

---

## The Pattern Language

The core contribution of this book is a **pattern language** for tactical wargame architecture—a vocabulary for discussing these systems and a catalog of proven solutions.

A pattern language, as conceived by architect Christopher Alexander, is not a collection of isolated solutions but an interconnected web where patterns reference and support each other. In our pattern language:

- **Entity Patterns** (Soldier/Squad Aggregate, Weapon Team, Component Composition) describe how units are structured
- **State Patterns** (State Hierarchy, Order Queue, Stance System, Morale Cascade) describe how behavior is modeled
- **Command Patterns** (Command Abstraction, Formation Control, Prerequisite Chain) describe how player intent flows
- **Perception Patterns** (Line of Sight, Threat Assessment, Cover Evaluation) describe how units perceive the world
- **System Patterns** (Spatial Partitioning, Message Bus, Deterministic Simulation) describe infrastructure

These patterns work together. The **State Hierarchy** pattern enables **Command Abstraction**, which relies on **Order Queue**, which uses **Prerequisite Chain**. Understanding the relationships between patterns is as important as understanding individual patterns.

Chapter 11 presents the complete pattern catalog. Throughout the book, we reference these patterns when discussing specific implementations.

---

## A Note on Completeness

None of the three projects analyzed in this book are complete, commercial-quality games. Each reached different levels of functionality:

- **OpenCombat-SDL**: Core simulation systems complete, limited content
- **CloseCombatFree**: Content systems complete, limited gameplay depth  
- **OpenCombat**: Multiplayer infrastructure complete, ongoing content development

This incompleteness is actually valuable for our purposes. These projects represent **architectural exploration**—they answer questions about how to structure systems, not necessarily about how to ship a finished product. We learn as much from their unfinished features as from their completed ones, perhaps more.

When we discuss a system working or not working, we're describing the code as it exists, not making judgments about the developers' skill or effort. All three projects represent significant achievements worthy of study.

---

## Moving Forward

With this foundation in place, we can now dive into the detailed analysis. Chapter 1 provides an executive summary of key findings for readers who want the "greatest hits" before committing to the full book. Chapter 2 begins our deep dive with a comparison of the three architectural philosophies represented by these projects.

Whether you're building a tactical wargame, studying game architecture, or simply curious about how these systems work, we hope this book serves as a valuable guide. The patterns within have been discovered through years of development effort; may they serve you as well as they served their original creators.

---

**The architecture of tactical wargames has evolved significantly over the past two decades, but the core challenge remains: creating systems that are simultaneously deep enough to reward mastery and clear enough to be enjoyable. The patterns in this book provide a foundation for meeting that challenge.**

---

*End of Introduction*
