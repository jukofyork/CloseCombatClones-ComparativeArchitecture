# Introduction

## What Is a "Close Combat Clone"?

The term "Close Combat clone" means something specific in tactical wargaming circles. It doesn't refer to just any game about small-unit combat. Instead, it describes a particular style of real-time tactical simulation shaped by Atomic Games' *Close Combat* series—one defined by distinct mechanics, design choices, and the way it feels to play.

At its heart, a Close Combat clone is:

> A real-time tactical wargame that models squad-level infantry combat with realistic ballistics, morale, and suppression systems. It uses terrain for meaningful cover and relies on AI that responds to player intent without requiring constant micromanagement.

This definition sets Close Combat clones apart from other, similar genres:

- **Turn-based tactical games** (*XCOM*, *Jagged Alliance*) pause between actions rather than running in real time.
- **Real-time strategy games** (*Command & Conquer*, *Company of Heroes*) focus on larger battles with less attention to individual soldier behavior.
- **Tactical shooters** (*Squad*, *Arma*) put the player in a first-person role instead of an overhead command view.
- **Wargames** (*Combat Mission*, *Graviteam Tactics*) often share realism goals but use turn-based or WeGo systems.

A true Close Combat clone keeps the spirit of the original: commanding a handful of soldiers who act like real people, not robots. Success comes from reading the battlefield, not just mastering controls.

---

## The Close Combat Formula

The Close Combat series created a blueprint that later games have tried to follow. To grasp the architectural challenges this book explores, you first need to understand that blueprint.

### The Seven Pillars of Close Combat

**1. Squad-Based Organization**

Players command squads of 6-12 soldiers, not individual troops. Soldiers within a squad act on their own—finding cover, engaging targets, reacting to threats. This approach frees players to focus on tactics instead of micromanaging every movement.

**2. Realistic Ballistics**

Weapons perform as they do in real combat. Bullets arc under gravity and ricochet off cover. Suppression—firing near enemies to keep them pinned—often matters more than direct hits. Each weapon has distinct traits: submachine guns excel at close range but falter at distance, rifles balance accuracy and reach, and machine guns deliver sustained fire but require setup.

**3. Morale and Suppression**

Soldiers react to combat psychologically. Under fire, they become suppressed—less accurate, slower to respond, sometimes panicking or freezing. Morale drops with casualties, poor leadership, or heavy incoming fire. This system adds depth beyond raw firepower: a few well-placed suppressing shots can cripple a larger force.

**4. Terrain and Cover**

The battlefield shapes every engagement. Buildings block fire completely. Walls shield from specific angles. Higher ground improves visibility. Terrain slows movement in mud or rubble. These elements create the genre's core puzzle: how to cross open ground under fire, or use a building for cover while keeping enemies in sight.

**5. Autonomous AI**

Soldiers think for themselves. They return fire when attacked, take cover when suppressed, and maintain awareness of nearby threats. Players give broad orders—"advance to that house," "pin down that position"—and the AI handles the rest.

**6. Real-Time with Pause (or Continuous)**

Combat happens in real time, building tension and forcing quick decisions. Some games let players pause to issue orders; others, like the original Close Combat, run without interruption. Either way, the real-time pace sets these games apart from turn-based strategy.

**7. Emergent Narrative**

No two battles unfold the same way. Realistic ballistics, morale systems, and autonomous AI generate unscripted stories—a squad holding out against overwhelming odds, a veteran leader lost to a sniper, raw recruits breaking under fire. These moments arise naturally from the game's systems, not pre-written scripts.

---

## Why These Three Projects?

This book examines three implementations that span twenty years of development, from 2005 to 2024. They were selected for their distinct architectural approaches to solving the same fundamental problem.

### Representative Diversity

| Aspect             | OpenCombat-SDL         | CloseCombatFree     | OpenCombat               |
| ------------------ | ---------------------- | ------------------- | ------------------------ |
| **Era**                | 2005–2008              | 2011–2012           | 2020–2024                |
| **Language**           | C++                    | C++/QML             | Rust                     |
| **Paradigm**           | Classical OOP          | Declarative         | Systems-Oriented         |
| **Primary Innovation** | 64-bit state bitfields | QML content modding | Deterministic simulation |
| **Multiplayer**        | No                     | No                  | Yes (designed for)       |
| **Moddability**        | Data files             | Full content        | Configuration            |

Their differences let us:
- Compare object-oriented, declarative, and systems-oriented paradigms
- Trace how best practices have evolved over two decades
- Identify patterns that emerge regardless of approach
- Understand the trade-offs each method involves

### Serious Work on Hard Problems

These are not toy projects or weekend experiments. Each one tackles the challenges of building a functional tactical wargame:

- OpenCombat-SDL implements a complete state machine system, action prerequisites, terrain analysis, and working combat mechanics
- CloseCombatFree adopts a declarative approach with runtime content loading
- OpenCombat delivers deterministic simulation for multiplayer replay

Each project made unique architectural choices, faced distinct challenges, and arrived at different solutions. Analyzing all three gives us a clear view of the solution space.

### Open Source Availability

Since all three projects are open source, we can examine their actual code instead of speculating about implementation. The book includes code excerpts, architectural diagrams derived from the source, and detailed comparisons of how specific systems work.

---

## How to Read This Book

This book adapts to your needs, whether you want a quick overview or deep technical detail.

### For the Executive Overview Reader

**Read**: Chapter 1 (Executive Summary), Chapter 11 (Universal Patterns), Chapter 12 (Implementation Guide)
**Time**: 2-3 hours
**Result**: You'll grasp the core architectural approaches, key patterns, and practical recommendations without wading through technical specifics.

This path works well for producers, technical directors evaluating options, or developers deciding if they need to explore further.

### For the Game Architect

**Read**: Chapters 1-3 (Overview, Philosophy, State Management), Chapter 11 (Universal Patterns), Chapter 12 (Implementation Guide)
**Optional**: Chapters 4-10 for systems that interest you
**Time**: 8-12 hours
**Result**: You'll understand the trade-offs behind each design decision and how to apply those lessons to your own projects.

This path focuses on the high-level choices that shape a game's architecture.

### For the Systems Programmer

**Read**: Chapters 3-8 (State Management, World Systems, Unit Hierarchy, Orders/AI, Moddability, Patterns), Chapter 11 (Universal Patterns), Chapter 12 (Implementation Guide), Technical Appendices A-C
**Time**: 15-20 hours
**Result**: You'll get hands-on details—algorithms, data structures, and implementation patterns—with code examples in multiple languages.

This path is for developers writing the actual code.

### For the Modder or Content Creator

**Read**: Chapter 7 (Moddability), Chapter 9 (Lessons Learned—modding sections), Chapter 10 (Recommendations—modding), Appendix C (File Formats)
**Time**: 4-6 hours
**Result**: You'll learn how content systems work and how to extend them.

This path covers data-driven design, file formats, and extensibility patterns, showing how different architectures help or hinder community content.

### For the Academic or Researcher

**Read**: The full book, with special attention to Chapter 2 (Architectural Philosophy), Chapter 9 (Lessons Learned), and Chapter 11 (Universal Patterns)
**Time**: 20+ hours
**Result**: You'll see the book as a case study in software architecture evolution, useful for research into game development or design patterns.

---

## What's Not Covered

This book examines the architectural and systems design of tactical wargames. Some related topics fall outside its scope:

- **Graphics and Rendering**: We study how the simulation state is represented, not how it appears on screen. The three projects use different rendering methods (SDL, Qt/QML, custom), but these are implementation details.

- **Audio Systems**: Sound design, spatial audio, and music enhance immersion but don't shape the core tactical simulation.

- **Input Handling**: We explore how player commands enter the system, not how input devices, key bindings, or UI widgets work.

- **Networking Infrastructure**: We cover deterministic simulation architecture (essential for multiplayer) but skip matchmaking, lobby systems, and network protocols.

- **Marketing and Business**: This is a technical book about architecture. Monetization, community building, and publishing strategies aren't discussed.

- **Art Pipeline**: Creating sprites, 3D models, animations, and other assets is vital for a finished game but not part of our focus.

- **Platform-Specific Optimization**: Console certification, mobile optimization, and platform SDKs aren't included.

### What We Cover

- **State representation**: How unit state is modeled and managed
- **Entity composition**: How soldiers, squads, and vehicles relate
- **Command systems**: How player intent translates to unit actions
- **AI architecture**: How units make autonomous decisions
- **World simulation**: How terrain, cover, and visibility function
- **Moddability**: How content systems allow community extensions
- **Pattern language**: Universal solutions to common problems

---

## Conventions Used in This Book

### Pseudocode

We use pseudocode to explain concepts without tying them to a specific language. Here's how we structure it:

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

Mermaid diagrams show architecture, data flow, and relationships. They appear as flowcharts, class diagrams, and sequence diagrams. ASCII alternatives are included where helpful.

### Code Examples

When showing real code, we label the language:

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

We use these terms consistently:

- **Soldier**: An individual infantryman (sometimes called "unit")
- **Squad**: A group of 6–12 soldiers who move and fight together
- **Team**: A larger military formation—platoon, company, or battalion
- **Order**: A high-level command from the player, like "move to location"
- **Action**: A single executable task, such as "walk to tile"
- **Behavior**: An AI-driven tactical choice, like "engage enemy"
- **Gesture**: A momentary physical state, such as "aiming frame 245"
- **State**: The condition of a soldier or squad—posture, morale, or activity
- **Phase**: A global game state—deployment, combat, or end

### Pattern Format

Chapter 11 (Universal Patterns) presents each pattern in the same structure:

1. **Name and Intent**: What the pattern does
2. **Problem**: Why the pattern matters
3. **Solution**: How to implement it
4. **Mermaid Diagram**: A visual overview
5. **Variations**: How each project handles it
6. **Consequences**: The trade-offs involved
7. **Related Patterns**: Links to other patterns

### Recommendation Boxes

Key advice stands out like this:

> **Recommendation**: Build multiplayer games with deterministic simulation from the start. Adding it later means rewriting the codebase.

---

## The Pattern Language

This book introduces a **pattern language** for tactical wargame architecture—a shared vocabulary and a catalog of proven solutions.

Architect Christopher Alexander designed pattern languages as interconnected webs, not isolated fixes. Each pattern references and reinforces others. Here's how the patterns break down:

- **Entity Patterns** (Soldier/Squad Aggregate, Weapon Team, Component Composition) define unit structure.
- **State Patterns** (State Hierarchy, Order Queue, Stance System, Morale Cascade) model behavior.
- **Command Patterns** (Command Abstraction, Formation Control, Prerequisite Chain) handle player intent.
- **Perception Patterns** (Line of Sight, Threat Assessment, Cover Evaluation) shape how units observe their environment.
- **System Patterns** (Spatial Partitioning, Message Bus, Deterministic Simulation) build the underlying infrastructure.

Patterns don't stand alone. **State Hierarchy** powers **Command Abstraction**, which depends on **Order Queue**, which in turn uses **Prerequisite Chain**. The connections matter as much as the patterns themselves.

The full catalog appears in Chapter 11. We'll reference these patterns whenever we examine specific implementations.

---

## A Note on Completeness

The three projects in this book aren't finished commercial games. Each reached a different stage of development:

- **OpenCombat-SDL**: Core simulation systems work, but content is limited
- **CloseCombatFree**: Content systems are complete, though gameplay lacks depth
- **OpenCombat**: Multiplayer infrastructure is ready, while content remains in progress

This unfinished state works to our advantage. These projects explore architecture—they show how to structure systems, not how to ship a product. Their incomplete features teach us as much as their finished ones, maybe more.

When we say a system works or doesn't, we're describing the code as it exists. This isn't about judging the developers. All three projects are impressive and worth studying.

---

## Moving Forward

Now we can begin the detailed analysis. Chapter 1 offers an executive summary for readers who want the highlights first. Chapter 2 starts the deep dive with a comparison of the three architectural approaches these projects represent.

Whether you're building a tactical wargame, studying game architecture, or just curious about how these systems work, this book should help. The patterns here come from years of development; they've served their creators well and may serve you too.

---

**The architecture of tactical wargames has changed over the past twenty years, but the central challenge hasn't: building systems deep enough to reward mastery yet clear enough to be fun. The patterns in this book offer a way forward.**

---

*Next: [How to Use This Book](how_to_use_this_book.md)*
