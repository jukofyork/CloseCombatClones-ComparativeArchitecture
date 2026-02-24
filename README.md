# Close Combat Clones: A Comparative Architecture Study

## *Universal Patterns for Tactical Wargame Development*

---

## Book Overview

This book offers the most thorough analysis yet of open-source Close Combat-inspired tactical wargames developed over two decades. By examining three distinct projects—each built with a different architectural approach—it extracts patterns that work for any tactical wargame, regardless of technology, team size, or platform.

The insights apply to any programming language or game engine. While the source projects use C++, Rust, and QML, the book presents patterns through universal pseudocode and architectural diagrams.

---

## Table of Contents

### Front Matter

**[Preface](preface.md)**

- The legacy and influence of Close Combat
- Twenty years of open-source attempts
- The three projects under analysis
- Why these patterns matter for tactical wargame development
- Acknowledgments and who should read this book

**[Introduction](introduction.md)**

- Defining a "Close Combat Clone"
- The seven pillars of the Close Combat formula
- Why these three projects were selected
- How to navigate this book
- What the book doesn't cover
- Conventions: pseudocode, diagrams, and terminology

**[How to Use This Book](how_to_use_this_book.md)**

- Four reading paths: Designer, Programmer, Modder, Researcher
- Chapter summaries with time estimates
- Where to find the pattern index
- Code repository references
- Decision frameworks and quick-reference guides

---

### Main Chapters

**[Chapter 1: Executive Summary & Project Overview](chapter_01_executive_summary.md)**

- The three Close Combat clones at a glance
- Project timelines from 2005 to 2024
- Architectural comparisons
- How to use this document

**[Chapter 2: Architectural Philosophy Comparison](chapter_02_architectural_philosophy.md)**

- Three approaches: Classical OOP, Systems-Oriented, Declarative
- How architectures handle communication
- State synchronization strategies
- Choosing the right philosophy for your project

**[Chapter 3: State Management Deep Dive](chapter_03_state_management.md)**

- OpenCombat-SDL: 64-bit bitfield state machine
- OpenCombat: Three-tier state hierarchy (Phase→Behavior→Gesture)
- CloseCombatFree: Dual-state system (Runtime + Visual)
- Comparisons and recommendations

**[Chapter 4: World & Terrain Design Patterns](chapter_04_world_terrain.md)**

- The three pillars: Cover, Hindrance, Elevation
- OpenCombat-SDL: Tile-based spatial partitioning
- OpenCombat: Opacity-based visibility system
- CloseCombatFree: Height maps and raycasting
- Best practices for terrain design

**[Chapter 5: Unit Hierarchy & Object Models](chapter_05_unit_hierarchy.md)**

- OpenCombat-SDL: Deep inheritance hierarchy
- OpenCombat: Modified ECS pattern
- CloseCombatFree: Component composition with QML
- Comparisons and modern recommendations

**[Chapter 6: Order & AI System Design](chapter_06_orders_ai.md)**

- The spectrum of command abstraction
- OpenCombat-SDL: Two-tier command system
- OpenCombat: Behavior-driven AI
- CloseCombatFree: Queue-based orders
- The AI autonomy spectrum and modern approaches

**[Chapter 7: Moddability & Data-Driven Design](chapter_07_moddability.md)**

- The modding spectrum
- OpenCombat-SDL: XML content modding
- OpenCombat: JSON configuration modding
- CloseCombatFree: Full QML content modding
- Designing for moddability

**[Chapter 8: Modern GameDev Pattern Adoption](chapter_08_architectural_patterns.md)**

- Classic patterns: Command, Observer, State, Factory
- Game-specific patterns: ECS, Object Pool, Service Locator
- Modern patterns: Event Sourcing, Data-Oriented Design
- Anti-patterns to avoid

**[Chapter 9: Strengths, Weaknesses & Lessons Learned](chapter_09_lessons_learned.md)**

- Strengths and weaknesses of each project
- Comparative feature matrix
- Universal lessons from all three projects
- Common pitfalls and success patterns

**[Chapter 10: Recommendations for New Implementations](chapter_10_recommendations.md)**

- Recommended technology stack
- Entity system architecture
- State management recommendations
- Order and command system design
- World and terrain system
- AI and behavior trees
- Modding system design
- Multiplayer architecture
- Implementation roadmap

**[Chapter 11: The Close Combat Pattern Language](chapter_11_universal_patterns.md)**

- Catalog of 17+ design patterns
- Pattern taxonomy: Entity, State, Command, Perception, System
- Problem-solution format with variations
- Decision framework for pattern selection

**[Chapter 12: Implementation Guide](chapter_12_implementation_guide.md)**

- Architecture Decision Record (ADR) templates
- State system implementation options
- Entity model implementation options
- AI system implementation options
- Technology stack selection flowchart
- Complete milestone checklist

**[Chapter 13: Case Studies](chapter_13_case_studies.md)**

- Case Study 1: Implementing a Sniper Team
- Case Study 2: Line of Sight Optimization
- Case Study 3: Adding Smoke Grenades
- Case Study 4: Multiplayer Synchronization
- Case Study 5: Total Conversion Mod

**[Chapter 14: AI Systems](chapter_14_ai_systems.md)**

- Four AI types: System, Unit, Tactical, Strategic
- Four constraints: Believability, Fairness, Performance, Determinism
- Behavior Trees in depth
- GOAP (Goal-Oriented Action Planning)
- Utility AI
- Perception systems
- Tactical planning

**[Chapter 15: Multiplayer Architecture](chapter_15_multiplayer.md)**

- Why multiplayer complicates everything
- Deterministic simulation approach
- Lockstep synchronization
- Bandwidth optimization
- Latency compensation
- Security and anti-cheat
- Replay systems

---

### Reference Guides

**[Decision Flowcharts](decision_flowcharts.md)**

- Flowchart 1: Choosing Your State Management System
- Flowchart 2: Choosing Your Entity Model
- Flowchart 3: Choosing Your AI Architecture
- Flowchart 4: Multiplayer Decision Tree
- Flowchart 5: Modding Strategy
- Flowchart 6: Overall Architecture Selection
- Implementation checklists for each major system

**[Common Pitfalls Guide](common_pitfalls.md)**

- State Management Pitfalls: God objects, spaghetti transitions, prerequisite handling
- Architecture Pitfalls: Deep inheritance, premature ECS, simulation/rendering coupling
- Multiplayer Pitfalls: Late determinism, RNG issues, floating-point inconsistencies
- AI Pitfalls: Omniscient AI, frame-rate dependency, complexity, autonomy
- Modding Pitfalls: Hardcoded content, binary formats, no hot-reload
- Performance Pitfalls: O(n²) coordination, spatial query issues

---

### Technical Appendices

**[Appendix A: State and World Systems](appendix_a_state_world_systems.md)**

- Technical reference for state management in all three games
- Bitfield architecture (OpenCombat-SDL)
- Three-tier state hierarchy (OpenCombat)
- Dual-state system (CloseCombatFree)
- Design pattern comparisons and recommendations

**[Appendix B: Unit, Vehicle, and Squad Attribute Systems](appendix_b_unit_attribute_systems.md)**

- Entity system architectures in detail
- Deep OOP hierarchy (OpenCombat-SDL)
- Modified ECS pattern (OpenCombat)
- Component composition (CloseCombatFree)
- Attribute system comparisons and design recommendations

**[Appendix C: File Formats and Data Hierarchy](appendix_c_file_formats.md)**

- Complete file format specifications
- XML data files (OpenCombat-SDL)
- JSON and TMX formats (OpenCombat)
- QML declarative data (CloseCombatFree)
- Format comparisons and architecture recommendations

---

## The Three Projects Analyzed

### [OpenCombat-SDL](https://github.com/jukofyork/OpenCombat-SDL) (2005-2008)

- **Language:** C++
- **Paradigm:** Classical Object-Oriented
- **Innovation:** 64-bit composable state bitfields with automatic prerequisite chaining
- **Best For:** Deep simulation, single-player experiences
- **Key Insight:** Automatic state transitions reduce micromanagement while keeping player control intact

### [CloseCombatFree](https://github.com/sierdzio/closecombatfree) (2011-2012)

- **Language:** C++/QML
- **Paradigm:** Declarative Component Architecture
- **Innovation:** QML for runtime content definition and hot reload
- **Best For:** Maximum moddability, community content
- **Key Insight:** Modding communities flourish when creators can modify behavior without recompiling

### [OpenCombat](https://github.com/buxx/OpenCombat) (2020-2024)

- **Language:** Rust
- **Paradigm:** Systems-Oriented Architecture
- **Innovation:** Three-tier state hierarchy with message-driven deterministic simulation
- **Best For:** Multiplayer, competitive play, replay systems
- **Key Insight:** Server-authoritative design enables competitive multiplayer, complete replay recording, and emergent AI behavior
