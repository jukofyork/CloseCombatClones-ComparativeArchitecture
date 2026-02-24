# Preface

## The Enduring Legacy of Close Combat

Atomic Games released *Close Combat* in 1996, and it changed tactical wargaming forever. While other games of the time relied on hex grids and turn-based play—treating units as little more than numbers on a map—*Close Combat* delivered the raw, unfiltered chaos of World War II combat at the squad level. Soldiers flinched under fire, exhaustion slowed their movements, and suppressing fire actually pinned enemies down. Cover wasn't just a stat; it decided who lived and who died.

The series expanded with sequels covering the Eastern Front, the Battle of the Bulge, Normandy, and the brutal urban combat of Aachen. Each entry sharpened the formula: real-time tactics, accurate ballistics, morale systems that mirrored real combat psychology, and a focus on terrain and positioning over sheer firepower. These weren't games about mindlessly clicking to attack. They demanded an understanding of how infantry actually fights—when to lay down suppressing fire, when to push forward, when to hold ground, and when to retreat.

Thirty years later, *Close Combat*'s influence is everywhere. Its DNA runs through modern tactical shooters, the revival of realistic military simulations, and the enduring appeal of games that reward careful positioning over twitch reflexes.

---

## Why This Book Exists

This book exists because for twenty years, developers have pursued the essence of Close Combat, leaving behind a wealth of architectural insights.

From 2005 to 2024, three major open-source projects tackled the challenge of building Close Combat-inspired tactical wargames. Each addressed the same core question—how to model squad-level real-time tactical combat on a computer—and each found unique architectural solutions. These were serious efforts by skilled developers working through genuinely difficult problems:

- **How do you represent unit state** when a soldier can be prone, under fire, reloading, and moving at once?
- **How do you structure the relationship** between a squad and its individual members?
- **How do you translate player intent** ("move to that building") into the thousands of individual actions that make up tactical movement?
- **How do you model morale** and the psychological pressures of combat?
- **How do you make it moddable** so the community can create new content?

The answers weren't obvious. The three projects examined here—**OpenCombat-SDL** (2005–2008), **CloseCombatFree** (2011–2012), and **OpenCombat** (2020–2024)—each took a different philosophical approach. They made distinct trade-offs, uncovered different patterns, and produced working code that put their solutions into practice.

---

## The Three Projects

### OpenCombat-SDL: The Classical Approach (2005–2008)

Built in C++ with traditional object-oriented design, OpenCombat-SDL reflects the game development paradigm of the mid-2000s. It relies on deep inheritance hierarchies, explicit state machines using 64-bit bitfields, and a sophisticated prerequisite system that chains actions automatically. The codebase reveals how classical game architecture handles tactical simulation—and why it can become limiting.

### CloseCombatFree: The Declarative Experiment (2011–2012)

CloseCombatFree adopted Qt's QML (Qt Modeling Language) as its primary content definition language. By treating game content as declarative data instead of procedural code, CCF enabled remarkable moddability. Scenarios, units, and behaviors could be modified without recompiling. The project illustrates both the strengths and weaknesses of declarative approaches in game development.

### OpenCombat: The Modern Systems Approach (2020–2024)

Written in Rust and designed for multiplayer from the start, OpenCombat embodies modern systems-oriented game architecture. It employs a three-tier state hierarchy (Phase → Behavior → Gesture), message-driven state updates for deterministic simulation, and a modified ECS pattern that balances type safety with simplicity. For developers building tactical games today, OpenCombat's architecture provides the most practical patterns.

---

## The Purpose of This Book

This book distills the architectural wisdom gathered over twenty years of tactical wargame development into universal patterns for future implementations.

It isn't a programming tutorial. You won't find lessons in C++, Rust, or QML here. The focus lies on **design patterns**—recurring solutions to common problems that emerge in all three implementations, regardless of language or framework. These patterns cover:

- **State representation strategies** for modeling complex unit behavior
- **Command abstraction layers** that bridge player intent and unit execution
- **Entity composition models** for structuring soldiers, squads, and vehicles
- **Perception systems** for line-of-sight, cover evaluation, and threat assessment
- **Deterministic simulation architectures** for multiplayer and replay support

Each pattern includes:
- **The problem** it solves
- **The solution** approach
- **How each of the three projects implemented it**
- **The trade-offs** of each approach
- **Recommendations** for modern implementations

The aim is to offer a **pattern language** for tactical wargame architecture—a vocabulary for discussing these systems and a catalog of proven solutions to adapt to your own projects.

---

## Acknowledgments

This book owes its existence to the generosity and hard work of developers who created these three open-source projects. By sharing their work, they allowed others to learn from both what worked and what didn't.

We recognize the developers of **OpenCombat-SDL**, who built a tactical simulation engine that still compiles and runs nearly two decades later. Their C++ engineering remains impressive, particularly their work on state machines and action prerequisites.

We recognize the developers of **CloseCombatFree**, who explored declarative game development before it gained popularity. Their experimental approach revealed possibilities that purely imperative programming often hides.

We recognize the developers of **OpenCombat**, who applied modern systems programming to tactical wargame development. They proved that determinism and replay support could coexist with clean architecture.

We also recognize the original **Close Combat** series by Atomic Games and later Matrix Games. Without their pioneering work, none of these projects would exist. They defined the core elements—morale, suppression, realistic ballistics, terrain-based cover—that give tactical wargaming its depth.

Finally, we recognize the broader game development community. Their shared knowledge about entity systems, state machines, and architectural patterns has shaped modern game programming. This book builds on foundational work, from the Gang of Four's *Design Patterns* to the data-oriented design movement.

---

## Who Should Read This Book

This book serves several key audiences:

### Game Architects and Technical Directors

Designing the architecture for a tactical wargame or any real-time simulation game? Chapters 2–4 offer deep architectural comparisons, while Chapter 11 presents a complete pattern language drawn from all three projects. You'll find detailed analysis of state management strategies, entity composition models, and command abstraction layers.

### Gameplay Programmers

Implementing the systems that power tactical games—state machines, AI, combat mechanics? Chapters 5–7 break down specific systems, and Chapter 12 provides practical implementation advice. You'll get concrete code examples and guidance to streamline your work.

### Modders and Content Creators

Want to understand how tactical games structure their data or modify them? Chapter 7 focuses on moddability, covering data-driven design, scripting integration, and how architectural choices shape community content. Learn what enables—or limits—your creativity.

### Game Designers

Designing tactical gameplay systems and need to grasp the technical side? This book explains how morale systems, cover mechanics, and AI decision-making work. You'll gain the context to collaborate effectively with programmers.

### Researchers and Students

Studying game architecture, software design patterns, or the evolution of game development? This book offers a twenty-year case study. The comparative analysis reveals how architectural paradigms have shifted and why.

### Indie Developers

Building a tactical game solo or with a small team? Skip months of trial and error. Learn from past mistakes and discoveries, and choose your technology stack with confidence from the start.

---

## Why Tactical Wargames Matter

Before we dive into the technical analysis, a personal note on why these games—and this book—matter.

Tactical wargames occupy a unique space in gaming. They demand thought over reflex, planning over improvisation, and understanding over twitch skill. They teach you to think like a small-unit leader: to assess terrain, coordinate fire and movement, and recognize the human limitations of your soldiers.

These games offer a profound education. They present war's brutal realities. When a squad is pinned down by machine gun fire, when soldiers break and run under heavy bombardment, when a veteran squad leader falls to a random mortar shell, you grasp the human cost of combat in ways no documentary can match.

They also pose a fascinating technical challenge. Modeling human psychology in code, representing combat's chaos while keeping gameplay clear, creating AI that behaves realistically without becoming predictable—these questions push the boundaries of game development.

Whether you're building the next great tactical wargame or studying how games work at their core, this book aims to guide and inspire. The patterns here come from years of development, testing, and refinement. Use them wisely, and help carry forward the legacy of thoughtful tactical gaming that Close Combat began nearly thirty years ago.

---

*Next: [Introduction](introduction.md)*
