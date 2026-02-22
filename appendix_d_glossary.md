# Appendix D: Comprehensive Glossary

## Close Combat Clones Comparative Architecture Book

---

This glossary provides definitions for all significant terms used throughout this book. Terms are organized by category for easy reference, with cross-references to related concepts and games that implement them.

---

## Core Concepts

### Action
A low-level physical operation executed by a unit, representing the actual implementation of a command. Actions have specific requirements (prerequisites), durations, and state transformations.

**Used in**: OpenCombat-SDL (SoldierActions.txt), OpenCombat (Gesture tier)

**Related terms**: Order, Gesture, Prerequisite, Behavior

**See also**: chapter_06_orders_ai.md, chapter_03_state_management.md

---

### Aggregate (Pattern)
A structural pattern where a Squad "has" Soldiers rather than "is" a Soldier. Enables hierarchical command structures while maintaining individual unit autonomy.

**Used in**: All three implementations (Squad→Soldier relationship)

**Related terms**: Entity, Component, Composition, Squad

**See also**: chapter_05_unit_hierarchy.md, chapter_11_universal_patterns.md (Pattern 1)

---

### Ambush
A tactical behavior where units remain concealed and hold fire until enemies enter a defined kill zone, then engage with coordinated fire.

**Used in**: All implementations (ambush states, behaviors)

**Related terms**: Behavior, Stance, Cover, Concealment

**See also**: chapter_06_orders_ai.md, chapter_13_case_studies.md

---

### Behavior
The middle tier in a three-tier state hierarchy, representing tactical decisions made by AI or player commands (MoveTo, Defend, Engage, etc.). Behaviors translate high-level intent into specific physical actions.

**Used in**: OpenCombat (Behavior enum), CloseCombatFree (behavior scripts)

**Related terms**: Order, Gesture, Phase, AI, State Hierarchy

**See also**: chapter_03_state_management.md, chapter_11_universal_patterns.md (Pattern 8)

---

### Bitfield
A data structure where multiple boolean states are packed into a single integer using bitwise operations. Enables 64 orthogonal states in 8 bytes with O(1) queries.

**Used in**: OpenCombat-SDL (State class)

**Related terms**: State, Orthogonality, Composition, Flag

**See also**: chapter_03_state_management.md, appendix_a_state_world_systems.md

---

### Capability
A functional ability of a unit represented as an orthogonal state bit (CanMove, CanFire, IsProne). Capabilities compose through bitwise operations to determine what actions are available.

**Used in**: OpenCombat-SDL (capability bits), hybrid architectures

**Related terms**: Bitfield, Prerequisite, Action, Composition

**See also**: chapter_10_recommendations.md, chapter_11_universal_patterns.md

---

### Component
A modular, reusable data structure that can be attached to entities to provide specific functionality (Health, Weapon, Movement, etc.).

**Used in**: OpenCombat (embedded ECS), CloseCombatFree (QML components)

**Related terms**: Entity, Composition, ECS, Aggregate

**See also**: chapter_05_unit_hierarchy.md, chapter_11_universal_patterns.md (Pattern 3)

---

### Composition
An architectural principle where complex entities are built by combining simple, reusable components rather than through inheritance hierarchies.

**Used in**: CloseCombatFree (primary), OpenCombat (modified ECS)

**Related terms**: Inheritance, Component, Entity, ECS

**See also**: chapter_02_architectural_philosophy.md, chapter_11_universal_patterns.md (Pattern 3)

---

### Cover
Protection from enemy fire provided by terrain features. Cover values vary by stance (prone vs standing) and terrain type (stone wall, grass, trench).

**Used in**: All implementations (terrain cover systems)

**Related terms**: Stance, Terrain, Line of Sight, Suppression

**See also**: chapter_04_world_terrain.md, chapter_11_universal_patterns.md (Pattern 13)

---

### Determinism
The property that given identical initial states and inputs, a simulation produces identical outputs every time. Essential for multiplayer synchronization and replay recording.

**Used in**: OpenCombat (fully deterministic), recommended for multiplayer

**Related terms**: Lockstep, Replay, Synchronization, State

**See also**: chapter_03_state_management.md, chapter_11_universal_patterns.md (Pattern 16)

---

### Entity
A game object that exists in the world (soldier, vehicle, squad). Can be defined through inheritance (OOP), composition (ECS), or component aggregation.

**Used in**: All implementations

**Related terms**: Component, Object, Unit, Soldier, Vehicle

**See also**: chapter_05_unit_hierarchy.md, chapter_11_universal_patterns.md

---

### Gesture
The lowest tier in a three-tier state hierarchy, representing immediate physical actions with temporal completion tracking (Aiming, Firing, Reloading with end frames).

**Used in**: OpenCombat (Gesture enum)

**Related terms**: Behavior, Phase, Temporal, Frame, Action

**See also**: chapter_03_state_management.md, chapter_11_universal_patterns.md (Pattern 4)

---

### Line of Sight (LOS)
A calculation determining whether one entity can see another, considering terrain elevation, obstacles, and opacity accumulation.

**Used in**: All implementations (Bresenham, accumulated opacity, dual check)

**Related terms**: Visibility, Cover, Opacity, Raycast

**See also**: chapter_04_world_terrain.md, chapter_11_universal_patterns.md (Pattern 11)

---

### Moddability
The architectural capability allowing users to modify game content, behaviors, and mechanics without recompiling the game.

**Used in**: CloseCombatFree (QML - excellent), OpenCombat (JSON - moderate), OpenCombat-SDL (XML - limited)

**Related terms**: Data-Driven, Scripting, Hot-Reload, Content

**See also**: chapter_07_moddability.md, chapter_11_universal_patterns.md (Pattern 17)

---

### Morale
A psychological state attribute (0-100) representing unit confidence and willingness to fight. Affected by casualties, leadership, and suppression.

**Used in**: OpenCombat-SDL (Attributes), OpenCombat (Feeling enum), CloseCombatFree (implicit)

**Related terms**: Suppression, Leadership, Mental State, Cascade

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 7)

---

### Order
A high-level command issued by the player representing strategic intent (MoveTo, Defend, Engage). Orders translate to behaviors then to physical actions.

**Used in**: All implementations

**Related terms**: Command, Behavior, Action, Intent, Queue

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 5, 8)

---

### Phase
The highest tier in a three-tier state hierarchy, representing global game states (Deployment, Battle, End).

**Used in**: OpenCombat (Phase enum)

**Related terms**: Behavior, Gesture, State Hierarchy, Game Flow

**See also**: chapter_03_state_management.md, chapter_11_universal_patterns.md (Pattern 4)

---

### Prerequisite
A required state that must be achieved before an action can execute. The prerequisite chain pattern automatically inserts intermediate actions to satisfy requirements.

**Used in**: OpenCombat-SDL (automatic chaining), OpenCombat (implicit via gestures)

**Related terms**: Action, Chain, Requirement, State

**See also**: chapter_03_state_management.md, chapter_11_universal_patterns.md (Pattern 10)

---

### Queue
A data structure managing sequential orders or actions. Enables players to issue multiple commands without waiting for completion.

**Used in**: All implementations (Order queues, action queues)

**Related terms**: Order, Action, Sequence, FIFO

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 5)

---

### Squad
A military unit consisting of multiple soldiers organized under a leader. Squads receive orders that distribute to members, often with formation positioning.

**Used in**: All implementations

**Related terms**: Soldier, Aggregate, Formation, Order, Hierarchy

**See also**: chapter_05_unit_hierarchy.md, chapter_11_universal_patterns.md (Pattern 1)

---

### State
A condition or mode of being for a game entity. Can be represented as bits (bitfield), enums (hierarchical), or strings (flexible).

**Used in**: All implementations (three different approaches)

**Related terms**: Bitfield, Enum, Behavior, Status

**See also**: chapter_03_state_management.md, appendix_a_state_world_systems.md

---

### Stance
Physical posture of a unit (Standing, Crouching, Prone) affecting movement speed, cover effectiveness, visibility, and weapon accuracy.

**Used in**: All implementations

**Related terms**: Cover, Visibility, Posture, Movement

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 6)

---

### Suppression
Psychological effect of being under fire that reduces combat effectiveness, accuracy, and willingness to move. Accumulates from nearby explosions and bullet impacts.

**Used in**: OpenCombat (Feeling::UnderFire), OpenCombat-SDL (suppression attributes)

**Related terms**: Morale, Combat, Fire, Danger

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 7)

---

## AI Terms

### Behavior Tree
A hierarchical tree structure for AI decision-making using Selector and Sequence nodes to organize behaviors by priority.

**Used in**: Recommended pattern, OpenCombat (influenced by)

**Related terms**: Selector, Sequence, Node, AI, GOAP

**See also**: chapter_06_orders_ai.md (6.6.1), chapter_12_implementation_guide.md

---

### Finite State Machine (FSM)
A computation model with finite states and transitions between them. Simple but can explode in complexity with many state combinations.

**Used in**: CloseCombatFree (implicit), mentioned in Chapter 3

**Related terms**: State, Transition, Automaton

**See also**: chapter_03_state_management.md (3.2)

---

### GOAP (Goal-Oriented Action Planning)
An AI architecture where agents plan backwards from goals using A* search through action space. Enables emergent behavior but computationally expensive.

**Used in**: Recommended for squad-level planning

**Related terms**: Planning, A*, Action, Goal, AI

**See also**: chapter_06_orders_ai.md (6.6.2), chapter_12_implementation_guide.md

---

### Perception
The AI subsystem processing sensory input (visible enemies, sounds, explosions) to inform decision-making.

**Used in**: OpenCombat (visible enemy detection), recommended pattern

**Related terms**: AI, LOS, Threat, Detection, Awareness

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 12)

---

### Reactive AI
An AI system that responds to immediate stimuli (visible enemies, incoming fire) with predetermined behaviors rather than long-term planning.

**Used in**: OpenCombat (primary AI), recommended approach

**Related terms**: Proactive AI, Threat Assessment, Self-Preservation

**See also**: chapter_06_orders_ai.md (6.5.3), chapter_09_lessons.md

---

### Selector (Behavior Tree)
A behavior tree node that tries children in order until one succeeds. Used for priority-ordered decision making.

**Used in**: Behavior tree pattern

**Related terms**: Behavior Tree, Sequence, Node, AI

**See also**: chapter_06_orders_ai.md (6.6.1)

---

### Sequence (Behavior Tree)
A behavior tree node that executes children in order until one fails. Used for action chains.

**Used in**: Behavior tree pattern

**Related terms**: Behavior Tree, Selector, Node

**See also**: chapter_06_orders_ai.md (6.6.1)

---

### Utility AI
An AI architecture scoring all possible actions and selecting the highest-scoring option. Sophisticated but requires extensive tuning.

**Used in**: Mentioned as alternative

**Related terms**: AI, Scoring, Decision

**See also**: chapter_12_implementation_guide.md (ADR-015)

---

## Architecture Patterns

### Command Pattern
A design pattern encapsulating requests as objects, allowing queuing, logging, and undoing of operations.

**Used in**: All order systems

**Related terms**: Order, Queue, Action, Encapsulation

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 8)

---

### Composition Over Inheritance
The principle of building complex objects by composing simpler ones rather than inheriting from base classes.

**Used in**: CloseCombatFree (QML), OpenCombat (ECS), recommended approach

**Related terms**: Inheritance, Component, Entity, ECS

**See also**: chapter_02_architectural_philosophy.md, chapter_11_universal_patterns.md (Pattern 3)

---

### Data-Driven Design
An approach separating game logic from data, enabling content modification without code changes.

**Used in**: OpenCombat-SDL (XML), OpenCombat (JSON), CloseCombatFree (QML)

**Related terms**: Moddability, Scripting, Configuration

**See also**: chapter_07_moddability.md, chapter_10_recommendations.md

---

### ECS (Entity-Component-System)
An architectural pattern where entities are lightweight IDs, components are pure data, and systems process homogeneous component sets.

**Used in**: OpenCombat (modified ECS)

**Related terms**: Entity, Component, System, Composition

**See also**: chapter_02_architectural_philosophy.md, chapter_12_implementation_guide.md (12.2.3)

---

### Event Sourcing
Storing state changes as a sequence of events rather than current state, enabling replay and temporal queries.

**Used in**: OpenCombat (message log)

**Related terms**: Message, Determinism, Replay, Log

**See also**: chapter_03_state_management.md, chapter_11_universal_patterns.md (Pattern 16)

---

### Factory Pattern
A creational pattern centralizing object instantiation logic, often used with data-driven content loading.

**Used in**: OpenCombat-SDL (Manager classes)

**Related terms**: Manager, Creation, Instantiation

**See also**: chapter_05_unit_hierarchy.md, chapter_07_moddability.md

---

### Inheritance
An OOP pattern where classes derive from base classes, inheriting behavior and attributes. Can create rigid hierarchies.

**Used in**: OpenCombat-SDL (primary), legacy approach

**Related terms**: OOP, Polymorphism, Base Class, Composition

**See also**: chapter_02_architectural_philosophy.md (2.2.1), chapter_05_unit_hierarchy.md

---

### Message Bus
A communication pattern where systems publish messages to a central bus that routes them to subscribers, decoupling sender and receiver.

**Used in**: OpenCombat (BattleStateMessage)

**Related terms**: Event, Communication, Decoupling, Observer

**See also**: chapter_03_state_management.md, chapter_11_universal_patterns.md (Pattern 15)

---

### Modified ECS
A hybrid ECS where entities are structs with embedded components rather than pure IDs, providing type safety and simpler serialization.

**Used in**: OpenCombat (BattleState with contiguous arrays)

**Related terms**: ECS, Entity, Component, Type Safety

**See also**: chapter_02_architectural_philosophy.md (2.2.2)

---

### Observer Pattern
A pattern where objects subscribe to events and receive notifications when state changes.

**Used in**: CloseCombatFree (Qt signals/slots)

**Related terms**: Signal, Slot, Notification, Reactive

**See also**: chapter_02_architectural_philosophy.md (2.3.2), chapter_11_universal_patterns.md

---

### OOP (Object-Oriented Programming)
A programming paradigm organizing code into objects with data (attributes) and behavior (methods).

**Used in**: OpenCombat-SDL (classical), CloseCombatFree (C++ backend)

**Related terms**: Inheritance, Polymorphism, Encapsulation, Class

**See also**: chapter_02_architectural_philosophy.md (2.2.1)

---

### Service Locator
A pattern providing global access to services (terrain system, renderer) without tight coupling.

**Used in**: Recommended pattern

**Related terms**: Dependency Injection, Singleton, Decoupling

**See also**: chapter_12_implementation_guide.md (12.3.2)

---

### Spatial Partitioning
Dividing world space into cells or regions to accelerate spatial queries ("entities within radius").

**Used in**: All implementations (tiles, grids, linked lists)

**Related terms**: Grid, Hash, Quadtree, Index, Query

**See also**: chapter_04_world_terrain.md, chapter_11_universal_patterns.md (Pattern 14)

---

## Networking Terms

### Client-Authoritative
An architecture where clients run the simulation and report results to the server. Fast but vulnerable to cheating.

**Used in**: OpenCombat-SDL, CloseCombatFree (single-player focus)

**Related terms**: Server-Authoritative, Anti-Cheat, Synchronization

**See also**: chapter_01_executive_summary.md, chapter_10_recommendations.md

---

### Delta Compression
Network synchronization sending only changed fields rather than complete state snapshots.

**Used in**: Mentioned as optimization

**Related terms**: Snapshot, Compression, Network, Sync

**See also**: chapter_01_executive_summary.md (Decision 6)

---

### Deterministic Lockstep
A multiplayer technique where all clients simulate the same frames with synchronized inputs, requiring deterministic code.

**Used in**: OpenCombat, recommended for multiplayer

**Related terms**: Determinism, Lockstep, Synchronization, Replay

**See also**: chapter_10_recommendations.md (10.1.1), chapter_11_universal_patterns.md (Pattern 16)

---

### Latency
The delay between player action and visible response in networked games. Must be minimized or concealed.

**Used in**: Networking discussions

**Related terms**: Lag, Ping, RTT, Compensation

**See also**: chapter_10_recommendations.md (10.1.1)

---

### Server-Authoritative
An architecture where the server validates all state changes, preventing cheating but adding latency.

**Used in**: OpenCombat (embedded server), recommended for multiplayer

**Related terms**: Client-Authoritative, Validation, Anti-Cheat

**See also**: chapter_01_executive_summary.md (Decision 4), chapter_10_recommendations.md

---

### Snapshot Synchronization
Sending complete game states periodically for network synchronization. Simple but bandwidth-intensive.

**Used in**: Mentioned as alternative

**Related terms**: Delta Compression, State, Network, Bandwidth

**See also**: chapter_01_executive_summary.md (Decision 6)

---

## Modding Terms

### Data-Driven
Designing systems to read parameters from external files (JSON, XML, YAML) rather than hardcoding values.

**Used in**: All implementations

**Related terms**: Configuration, Moddability, External Data

**See also**: chapter_07_moddability.md, chapter_10_recommendations.md

---

### Declarative
A programming style describing "what" should happen rather than "how" to do it. QML is declarative.

**Used in**: CloseCombatFree (QML primary), recommended for UI

**Related terms**: QML, Functional, Imperative, Specification

**See also**: chapter_02_architectural_philosophy.md (2.2.3), chapter_07_moddability.md

---

### Hot Reload
The ability to modify game content at runtime and see changes immediately without restarting.

**Used in**: CloseCombatFree (QML hot reload), recommended feature

**Related terms**: Live Editing, Iteration, Development, Modding

**See also**: chapter_07_moddability.md (7.5.4), chapter_11_universal_patterns.md (Pattern 17)

---

### Lua
A lightweight, embeddable scripting language commonly used for game modding and AI behaviors.

**Used in**: Recommended for scripting (OpenCombat-style)

**Related terms**: Scripting, Modding, Behavior, AI

**See also**: chapter_12_implementation_guide.md (12.2.7), chapter_13_case_studies.md

---

### QML (Qt Meta Language)
A declarative language for UI and content definition used by CloseCombatFree for maximum moddability.

**Used in**: CloseCombatFree (primary content definition)

**Related terms**: Declarative, Qt, Component, Property Binding

**See also**: chapter_02_architectural_philosophy.md (2.2.3), chapter_07_moddability.md (7.5)

---

### Scripting
Using interpreted languages (Lua, QML, Python) for game logic, enabling runtime modification without recompilation.

**Used in**: Recommended approach, CloseCombatFree (QML behavior scripts)

**Related terms**: Lua, QML, Modding, Hot Reload

**See also**: chapter_07_moddability.md, chapter_12_implementation_guide.md

---

## Game Design Terms

### Concealment
Being hidden from enemy view through terrain or posture. Different from cover (which protects from fire).

**Used in**: All implementations (stealth mechanics)

**Related terms**: Cover, Visibility, LOS, Stealth

**See also**: chapter_04_world_terrain.md, chapter_11_universal_patterns.md

---

### Elevation
Height above ground level, affecting line of sight (high ground advantage) and movement speed.

**Used in**: All implementations (height maps, per-tile elevation)

**Related terms**: Height, Terrain, LOS, Cover

**See also**: chapter_04_world_terrain.md (4.3.3), appendix_a_state_world_systems.md

---

### Formation
A geometric arrangement of squad members during movement (Column, Line, Wedge) affecting coordination and vulnerability.

**Used in**: OpenCombat-SDL, OpenCombat (formation calculations)

**Related terms**: Squad, Movement, Coordination, Position

**See also**: chapter_06_orders_ai.md, chapter_11_universal_patterns.md (Pattern 9)

---

### Hindrance
Terrain penalty to movement speed (percentage of normal speed or cost multiplier).

**Used in**: All implementations

**Related terms**: Terrain, Movement, Cost, Pathfinding

**See also**: chapter_04_world_terrain.md (4.3.2)

---

### Kill Zone
A defined area where ambushing units will engage enemies. Defined by center, radius, and arc.

**Used in**: Ambush behavior implementations

**Related terms**: Ambush, Trigger, Engagement, Zone

**See also**: chapter_13_case_studies.md (13.3)

---

### Opacity
A visibility property of terrain representing how much it blocks line of sight (0.0 = transparent, 1.0 = opaque).

**Used in**: OpenCombat (accumulated opacity system)

**Related terms**: LOS, Visibility, Terrain, Accumulation

**See also**: chapter_04_world_terrain.md (4.4.2)

---

### Orthogonality
The property of states being independent and combinable (a unit can be both Prone AND Reloading).

**Used in**: OpenCombat-SDL (bitfield composition)

**Related terms**: Bitfield, Composition, State, Independence

**See also**: chapter_03_state_management.md (3.3), chapter_11_universal_patterns.md

---

### Point Man
The lead soldier in a formation who performs pathfinding, with other members following at formation offsets.

**Used in**: OpenCombat-SDL (formation system)

**Related terms**: Formation, Squad, Leader, Pathfinding

**See also**: chapter_05_unit_hierarchy.md, chapter_06_orders_ai.md

---

### Suppression Fire
Deliberate firing at an area to pin down enemies rather than kill them. Reduces target effectiveness.

**Used in**: All implementations

**Related terms**: Suppression, Fire, Pinning, Combat

**See also**: chapter_06_orders_ai.md

---

### Temporal
Time-related properties, especially gesture completion times enabling frame-accurate gameplay.

**Used in**: OpenCombat (Gesture tier with end frames)

**Related terms**: Time, Frame, Gesture, Completion

**See also**: chapter_03_state_management.md (3.4.3)

---

### Visibility Modifier
Adjustments to how easily a unit is spotted based on behavior (moving = more visible, hiding = less visible).

**Used in**: OpenCombat (behavior-based visibility)

**Related terms**: Visibility, LOS, Detection, Behavior

**See also**: chapter_04_world_terrain.md (4.4.2)

---

## Technical Terms

### A* (A-Star)
A pathfinding algorithm finding the shortest path between nodes using heuristics. Standard for grid-based pathfinding.

**Used in**: All implementations, industry standard

**Related terms**: Pathfinding, Heuristic, Algorithm, Dijkstra

**See also**: chapter_04_world_terrain.md, chapter_12_implementation_guide.md

---

### Bresenham
An algorithm for rasterizing lines on a grid, used for line of sight calculations.

**Used in**: OpenCombat-SDL (3D Bresenham), OpenCombat (LOS)

**Related terms**: LOS, Algorithm, Grid, Raycast

**See also**: chapter_04_world_terrain.md (4.4.1)

---

### Cache Efficiency
Optimizing memory layout so frequently accessed data stays in CPU cache, improving performance.

**Used in**: OpenCombat (contiguous arrays)

**Related terms**: Performance, ECS, Data-Oriented, Memory

**See also**: chapter_05_unit_hierarchy.md, chapter_12_implementation_guide.md

---

### Contiguous Arrays
Storing data in consecutive memory locations for cache efficiency (Structure of Arrays pattern).

**Used in**: OpenCombat (BattleState vectors)

**Related terms**: Cache, ECS, Performance, Memory

**See also**: chapter_05_unit_hierarchy.md, chapter_12_implementation_guide.md (12.2.3)

---

### Desync
When multiplayer clients have divergent game states, breaking synchronization.

**Used in**: Networking discussions

**Related terms**: Synchronization, Determinism, Multiplayer, Bug

**See also**: chapter_11_universal_patterns.md (Pattern 16)

---

### Fixed Timestep
Running simulation updates at a constant rate (e.g., 60 FPS) regardless of rendering frame rate, enabling determinism.

**Used in**: OpenCombat (deterministic), recommended

**Related terms**: Delta Time, Determinism, Frame, Tick

**See also**: chapter_11_universal_patterns.md (Pattern 16)

---

### Floating-Point
Decimal number representation using IEEE 754 standard. Non-deterministic across platforms; use fixed-point for determinism.

**Used in**: Position calculations (with caveats)

**Related terms**: Fixed-Point, Determinism, Precision, Math

**See also**: chapter_11_universal_patterns.md (Pattern 16)

---

### Hash Map
A data structure providing O(1) average-case lookup. Used for spatial indexing and entity lookups.

**Used in**: OpenCombat (spatial grids, BattleState maps)

**Related terms**: Map, Dictionary, Index, Lookup

**See also**: chapter_05_unit_hierarchy.md

---

### Index (Type-Safe)
A wrapper around integer indices preventing accidental mixing of entity types (SoldierIndex vs VehicleIndex).

**Used in**: OpenCombat (Rust newtypes), recommended

**Related terms**: Type Safety, Entity, Reference, Pointer

**See also**: chapter_05_unit_hierarchy.md, chapter_12_implementation_guide.md

---

### JSON (JavaScript Object Notation)
A lightweight data interchange format used for configuration and save files.

**Used in**: OpenCombat (deployments, configuration)

**Related terms**: XML, YAML, Data, Serialization

**See also**: chapter_07_moddability.md (7.4)

---

### Linked List
A data structure where each element points to the next. Used by OpenCombat-SDL for tile-based object storage.

**Used in**: OpenCombat-SDL (object lists per tile)

**Related terms**: List, Data Structure, Pointer, Tile

**See also**: chapter_04_world_terrain.md (4.6.1)

---

### Mermaid
A markdown-based diagramming tool used throughout this book for architecture diagrams.

**Used in**: All chapters for diagrams

**Related terms**: Diagram, Visualization, Documentation

**See also**: All chapters with diagrams

---

### Pseudocode
A high-level description of algorithms using code-like syntax without language-specific details.

**Used in**: All chapters for algorithm descriptions

**Related terms**: Algorithm, Implementation, Documentation

**See also**: chapter_03_state_management.md, chapter_12_implementation_guide.md

---

### Quadtree
A spatial partitioning data structure dividing space into four quadrants recursively. Efficient for uneven entity distribution.

**Used in**: Mentioned as spatial indexing option

**Related terms**: Spatial Partitioning, Tree, Grid, Index

**See also**: chapter_12_implementation_guide.md (12.2.5)

---

### Rust
A systems programming language emphasizing safety and performance. Used by OpenCombat.

**Used in**: OpenCombat (2020-2024)

**Related terms**: C++, Language, Safety, Performance

**See also**: chapter_02_architectural_philosophy.md (2.2.2)

---

### SDL2 (Simple DirectMedia Layer)
A cross-platform development library for graphics, audio, and input. Used by OpenCombat-SDL.

**Used in**: OpenCombat-SDL (2005-2008)

**Related terms**: Graphics, Input, Library, C++

**See also**: chapter_02_architectural_philosophy.md (2.2.1)

---

### Seeded RNG
Random number generator initialized with a specific seed, producing deterministic sequences for multiplayer synchronization.

**Used in**: OpenCombat (deterministic simulation)

**Related terms**: Random, Determinism, Seed, Synchronization

**See also**: chapter_11_universal_patterns.md (Pattern 16)

---

### Serialization
Converting game state to a format that can be stored (save games) or transmitted (networking).

**Used in**: All implementations (different approaches)

**Related terms**: Save, Load, Network, State, Binary

**See also**: chapter_12_implementation_guide.md (12.2.6)

---

### Shadow Casting
An efficient visibility algorithm simulating light propagation from a source through a grid.

**Used in**: Mentioned for fog of war

**Related terms**: LOS, Visibility, Algorithm, Grid

**See also**: chapter_13_case_studies.md (13.5)

---

### TMX
The Tiled map format (XML-based) for 2D tile maps. Industry standard.

**Used in**: OpenCombat (map format), recommended

**Related terms**: Tiled, Map, Terrain, Grid

**See also**: chapter_07_moddability.md (7.4.4)

---

### Type Safety
Compile-time guarantees preventing invalid operations (e.g., treating VehicleIndex as SoldierIndex).

**Used in**: OpenCombat (Rust), recommended C++ approaches

**Related terms**: Safety, Compile-Time, Error Prevention

**See also**: chapter_05_unit_hierarchy.md, chapter_12_implementation_guide.md

---

### XML (eXtensible Markup Language)
A markup language for structured data. Used by OpenCombat-SDL and TMX maps.

**Used in**: OpenCombat-SDL (configuration), TMX format

**Related terms**: JSON, YAML, Data, Markup

**See also**: chapter_07_moddability.md (7.3)

---

### YAML (YAML Ain't Markup Language)
A human-readable data serialization standard often used for configuration.

**Used in**: Recommended for modding

**Related terms**: JSON, XML, Data, Configuration

**See also**: chapter_10_recommendations.md (10.2.6)

---

## Acronyms and Abbreviations

| Acronym | Full Form | Chapter Reference |
|---------|-----------|-------------------|
| ADR | Architecture Decision Record | chapter_12_implementation_guide.md (12.1) |
| AI | Artificial Intelligence | chapter_06_orders_ai.md |
| API | Application Programming Interface | chapter_07_moddability.md |
| BT | Behavior Tree | chapter_06_orders_ai.md (6.6.1) |
| CCF | CloseCombatFree | Throughout |
| CPU | Central Processing Unit | chapter_12_implementation_guide.md |
| ECS | Entity-Component-System | chapter_02_architectural_philosophy.md |
| FSM | Finite State Machine | chapter_03_state_management.md |
| FPS | Frames Per Second | chapter_12_implementation_guide.md |
| GOAP | Goal-Oriented Action Planning | chapter_06_orders_ai.md (6.6.2) |
| GUI | Graphical User Interface | chapter_07_moddability.md |
| KIA | Killed In Action | appendix_a_state_world_systems.md |
| LOS | Line of Sight | chapter_04_world_terrain.md |
| OOP | Object-Oriented Programming | chapter_02_architectural_philosophy.md |
| QML | Qt Meta Language | chapter_02_architectural_philosophy.md |
| RNG | Random Number Generator | chapter_11_universal_patterns.md |
| SDL | Simple DirectMedia Layer | chapter_02_architectural_philosophy.md |
| SoA | Structure of Arrays | chapter_12_implementation_guide.md |
| TMX | Tiled Map XML format | chapter_07_moddability.md |
| UI | User Interface | chapter_07_moddability.md |
| VM | Virtual Machine | chapter_12_implementation_guide.md |
| XML | eXtensible Markup Language | chapter_07_moddability.md |

---

## Cross-Reference Index

### By Game Implementation

**OpenCombat-SDL (2005-2008)**
- Bitfield state system
- Deep inheritance
- Automatic prerequisite chaining
- XML data files
- Tile-based world with linked lists
- 3D Bresenham LOS
- Manager pattern
- No AI / pure obedience

**OpenCombat (2020-2024)**
- Three-tier state hierarchy
- Modified ECS
- Type-safe indices
- Message-driven updates
- Deterministic simulation
- Reactive AI
- JSON/TMX data
- Runtime configuration

**CloseCombatFree (2011-2012)**
- QML declarative content
- Component composition
- Dual-state system
- Hot reload
- Runtime modding
- Height map LOS
- Visual-first design
- No AI / state triggers

### By Pattern Category

**Entity Patterns** (chapter_11_universal_patterns.md)
- Soldier/Squad Aggregate (Pattern 1)
- Weapon Team (Pattern 2)
- Component Composition (Pattern 3)

**State Patterns** (chapter_11_universal_patterns.md)
- State Hierarchy (Pattern 4)
- Order Queue (Pattern 5)
- Stance System (Pattern 6)
- Morale Cascade (Pattern 7)

**Command Patterns** (chapter_11_universal_patterns.md)
- Command Abstraction (Pattern 8)
- Formation Control (Pattern 9)
- Prerequisite Chain (Pattern 10)

**Perception Patterns** (chapter_11_universal_patterns.md)
- Line of Sight (Pattern 11)
- Threat Assessment (Pattern 12)
- Cover Evaluation (Pattern 13)

**System Patterns** (chapter_11_universal_patterns.md)
- Spatial Partitioning (Pattern 14)
- Message Bus (Pattern 15)
- Deterministic Simulation (Pattern 16)
- Hot Reload (Pattern 17)

---

*This glossary serves as a comprehensive reference for terminology used throughout the Close Combat Clones Comparative Architecture Book. Terms are organized alphabetically within categories and include cross-references to chapters, patterns, and related concepts.*
