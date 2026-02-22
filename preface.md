# Preface

## The Enduring Legacy of Close Combat

In 1996, Atomic Games released *Close Combat*, a game that would fundamentally reshape how we think about tactical wargaming. Unlike its contemporaries—hex-based turn-based affairs where units were abstractions on a map—Close Combat brought the brutal reality of squad-level World War II combat to life with an immediacy that was unprecedented. Individual soldiers displayed fear and fatigue. Suppressing fire actually suppressed. Cover wasn't just a statistic—it was the difference between life and death.

The series spawned sequels that explored the Eastern Front, the Battle of the Bulge, the Normandy campaign, and the brutal street-to-street fighting in Aachen. Each iteration refined the formula: real-time tactical decision-making, realistic ballistics, morale systems that modeled combat psychology, and an emphasis on terrain and positioning over raw firepower. These weren't games about clicking to attack; they were games about understanding how infantry actually fights—about fire and movement, about suppressing an enemy so your squad could advance, about knowing when to hold and when to break contact.

Nearly three decades later, the influence of Close Combat remains undeniable. You can see its DNA in modern tactical shooters, in the resurgence of realistic military simulations, and in the continuing popularity of games that prioritize thoughtful positioning over reflexes.

---

## Why This Book Exists

This book exists because for twenty years, developers have been trying to recapture that Close Combat magic—and because those attempts, whether successful or not, have left behind a treasure trove of architectural wisdom.

Between 2005 and 2024, three significant open-source projects attempted to build Close Combat-inspired tactical wargames. Each approached the same fundamental problem—how do you model squad-level real-time tactical combat on a computer?—and each arrived at dramatically different architectural answers. These weren't amateur experiments; they were serious attempts by skilled developers to solve genuinely difficult problems:

- **How do you represent unit state** when a soldier can simultaneously be prone, under fire, reloading, and moving?
- **How do you structure the relationship** between a squad and its individual members?
- **How do you translate player intent** ("move to that building") into the thousands of individual actions that constitute tactical movement?
- **How do you model morale** and the psychological pressures of combat?
- **How do you make it moddable** so the community can create new content?

These questions don't have obvious answers. The three projects analyzed in this book—**OpenCombat-SDL** (2005–2008), **CloseCombatFree** (2011–2012), and **OpenCombat** (2020–2024)—represent three distinct philosophical approaches to answering them. Each project made different trade-offs, each discovered different patterns, and each left behind working code that demonstrates their solutions in action.

---

## The Three Projects

### OpenCombat-SDL: The Classical Approach (2005–2008)

Built in C++ using traditional object-oriented design, OpenCombat-SDL represents the game development paradigm of the mid-2000s. It features deep inheritance hierarchies, explicit state machines using 64-bit bitfields, and a sophisticated prerequisite system that automatically chains actions together. If you're interested in how classical game architecture handles tactical simulation—or why that architecture can become limiting—this project's codebase offers invaluable insights.

### CloseCombatFree: The Declarative Experiment (2011–2012)

CloseCombatFree took the radical approach of building a game using Qt's QML (Qt Modeling Language) not just for the UI, but as the primary content definition language. By treating game content as declarative data rather than procedural code, CCF achieved remarkable moddability—scenarios, units, and even behaviors could be modified without recompiling. This project demonstrates both the power and the limitations of declarative approaches in game development.

### OpenCombat: The Modern Systems Approach (2020–2024)

Written in Rust and designed from the ground up for multiplayer, OpenCombat represents modern systems-oriented game architecture. It uses a three-tier state hierarchy (Phase → Behavior → Gesture), message-driven state updates for deterministic simulation, and a modified ECS pattern that provides type safety without the complexity of pure entity-component systems. If you're building a tactical game today, OpenCombat's architecture offers the most immediately applicable patterns.

---

## The Purpose of This Book

This book has one primary purpose: to distill the architectural wisdom accumulated across twenty years of tactical wargame development into a set of universal patterns that can guide future implementations.

This is not a programming tutorial. We won't teach you C++, Rust, or QML. Instead, we focus on the **design patterns**—the recurring solutions to common problems that appear across all three implementations, regardless of language or framework. These patterns include:

- **State representation strategies** for modeling complex unit behavior
- **Command abstraction layers** that bridge player intent and unit execution
- **Entity composition models** for structuring soldiers, squads, and vehicles
- **Perception systems** for line-of-sight, cover evaluation, and threat assessment
- **Deterministic simulation architectures** for multiplayer and replay support

Each pattern is presented with:
- **The problem** it solves
- **The solution** approach
- **How each of the three projects implemented it**
- **The trade-offs** of each approach
- **Recommendations** for modern implementations

Our goal is to provide you with a **pattern language** for tactical wargame architecture—a vocabulary for discussing these systems and a catalog of proven solutions you can adapt to your own projects.

---

## Acknowledgments

This book would not exist without the generosity and hard work of the developers who created these three open-source projects. They chose to share their work with the world, allowing others to learn from their successes and their failures alike.

We acknowledge the developers of **OpenCombat-SDL**, who built a functioning tactical simulation engine that continues to compile and run nearly two decades later—a testament to solid C++ engineering. Their work on state machines and action prerequisites remains relevant today.

We acknowledge the developers of **CloseCombatFree**, who had the vision to explore declarative game development long before it became fashionable. Their experimental approach opened our eyes to possibilities that purely imperative programming obscures.

We acknowledge the developers of **OpenCombat**, who brought modern systems programming practices to tactical wargame development and proved that determinism and replay support need not come at the cost of clean architecture.

We also acknowledge the original **Close Combat** series by Atomic Games and later Matrix Games. Without their pioneering work, none of these projects would exist. They established the formula—morale, suppression, realistic ballistics, terrain-based cover—that makes tactical wargaming meaningful.

Finally, we acknowledge the broader game development community, whose shared knowledge about entity systems, state machines, and architectural patterns has elevated the craft of game programming. This book stands on the shoulders of giants, from the Gang of Four's *Design Patterns* to the modern data-oriented design movement.

---

## Who Should Read This Book

This book is written for several distinct audiences:

### Game Architects and Technical Directors

If you're designing the architecture for a tactical wargame—or any real-time simulation game—you'll find detailed analysis of state management strategies, entity composition models, and command abstraction layers. Chapters 2–4 provide deep architectural comparisons, while Chapter 11 presents a complete pattern language synthesized from all three projects.

### Gameplay Programmers

If you're implementing the systems that make tactical games work—the state machines, the AI, the combat mechanics—you'll find concrete code examples and implementation guidance. Chapters 5–7 examine specific systems in detail, and Chapter 12 provides practical implementation recommendations.

### Modders and Content Creators

If you want to understand how tactical games structure their data and how you can modify or extend them, Chapter 7 focuses specifically on moddability. You'll learn about data-driven design, scripting integration, and how different architectural choices enable or constrain community content.

### Game Designers

If you're designing tactical gameplay systems and want to understand the technical implications of design choices—how morale systems work, how cover mechanics are implemented, how AI makes decisions—you'll find the technical context necessary to collaborate effectively with programmers.

### Researchers and Students

If you're studying game architecture, software design patterns, or the evolution of game development practices, this book provides a longitudinal case study spanning twenty years of development. The comparative analysis reveals how architectural paradigms have shifted and why.

### Indie Developers

If you're a solo developer or small team building a tactical game, this book will save you months of architectural exploration. Learn from the mistakes and discoveries of those who came before, and make informed decisions about your technology stack from day one.

---

## Why Tactical Wargames Matter

Before we dive into the technical analysis, a personal note on why these games—and by extension, this book—matter.

Tactical wargames occupy a unique space in gaming. They demand thought over reflex, planning over improvisation, and understanding over twitch skill. They teach you to think like a small-unit leader: to assess terrain, to coordinate fire and movement, to understand that your soldiers are human beings with limitations rather than game pieces with statistics.

There's something profoundly educational about these games. They don't glorify war—they model its brutal realities. When you watch a squad pinned down by machine gun fire, when you see soldiers break and run under heavy bombardment, when you lose a veteran squad leader to a random mortar shell, you understand something about the human cost of combat that no documentary can convey.

These games also represent a fascinating technical challenge. How do you model human psychology in code? How do you represent the chaos of combat while maintaining gameplay clarity? How do you create AI that behaves realistically without becoming predictable? The answers to these questions push the boundaries of what's possible in game development.

Whether you're building the next great tactical wargame or simply studying how games work at a fundamental level, I hope this book serves as both a practical guide and an inspiration. The patterns within these pages have been hard-won through years of development, testing, and refinement. Use them wisely, and carry forward the legacy of thoughtful tactical gaming that Close Combat began nearly three decades ago.

---

**The patterns in this book are not rules—they are tools. Use them thoughtfully, adapt them to your needs, and may your squads always find good cover.**

---

*The comparative analysis in this book represents the synthesis of nearly two decades of open-source tactical wargame development. May it serve future developers as well as the original projects served their creators.*

---

**Book Version**: 1.0  
**Publication Date**: February 2026  
**Status**: Complete

---

*End of Preface*
