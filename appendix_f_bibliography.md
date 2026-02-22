# Appendix F: Bibliography and References

## F.1 Primary Sources - The Three Projects

### F.1.1 OpenCombat (Rust/GGEZ Implementation)

**Project Information:**
- **Repository**: https://github.com/buxx/OpenCombat
- **Author**: Bastien Sevajol (contact@bux.fr)
- **License**: GNU Affero General Public License v3 (AGPL-3.0)
- **Language**: Rust (Edition 2021)
- **Lines of Code**: ~24,000
- **Files**: 229
- **Current Version**: 0.4.0
- **Status**: Active development

**Primary Documentation Files:**
- `/home/juk/temp/CloseCombatClones/OpenCombat/README.md` - Project overview and quick start
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/ARCHITECTURE.md` - Technical architecture index
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/01-overview.md` - High-level architecture (2,233 lines)
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/02-object-system.md` - Entity architecture
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/03-state-machine.md` - State management
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/04-orders-pathfinding.md` - AI and pathfinding
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/05-graphics.md` - Rendering system
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/06-world-terrain.md` - Map and terrain
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/07-configuration.md` - Data schemas
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/08-assets.md` - Asset pipeline
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/09-ui-combat.md` - UI system
- `/home/juk/temp/CloseCombatClones/OpenCombat/doc/chapters/10-build-system.md` - Build infrastructure
- `/home/juk/temp/CloseCombatClones/OpenCombat/LICENSE` - AGPL-3.0 license text

**Key Source Code Files:**
- `battle_core/src/` - Core game logic (~8,650 lines)
  - `state/` - State management system
  - `physics/` - Pathfinding and visibility
  - `game/` - Weapons, combat, mechanics
  - `behavior/` - AI behaviors
  - `graphics/` - Rendering components
  - `map/` - Terrain and world
  - `order/` - Command system
  - `entity/` - Entity management
  - `message/` - Network messages
  - `deployment/` - Scenario setup
- `battle_server/src/bin.rs` - Server executable (~2,899 lines)
- `battle_gui/src/` - Client GUI (~10,650 lines)
- `oc_core/src/` - Shared types (~323 lines)
- `oc_launcher/src/` - Launcher application
- `examples/src/` - Example scenarios

**Key Dependencies:**
- ggez 0.9.3 - 2D game framework
- ggegui 0.3.7 - egui integration for UI
- zmq 0.9 - ZeroMQ networking
- serde + bincode - Serialization
- glam 0.22.0 - GPU-compatible math
- tiled 0.10.2 - Tiled map format
- pathfinding 4.2.1 - A* algorithm
- puffin 0.19.0 - Profiling

---

### F.1.2 OpenCombat-SDL (C++/SDL2 Port)

**Project Information:**
- **Original Project**: https://sourceforge.net/projects/adv-warfare/
- **Original Author**: Atomic Games/Matrix Games heritage (reverse-engineered)
- **SDL2 Port Repository**: `/home/juk/temp/CloseCombatClones/OpenCombat-SDL`
- **License**: GNU General Public License v2 (GPL-2.0)
- **Language**: C++17
- **Lines of Code**: ~21,500
- **Files**: 139 source files
- **Original Date**: 2005 (Windows/DirectX)
- **Port Date**: 2024-2026
- **Status**: SDL2 port complete, testing phase

**Primary Documentation Files:**
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/README.md` - Project overview
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/AGENTS.md` - Coding guidelines
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/ARCHITECTURE.md` - Architecture index
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/01-overview.md` - Overview (1,139 lines)
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/02-object-system.md` - Object system
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/03-state-machine.md` - State management
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/04-orders-pathfinding.md` - Pathfinding
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/05-graphics.md` - Rendering
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/06-world-terrain.md` - World system
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/07-configuration.md` - Configuration
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/08-assets.md` - Assets
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/09-ui-combat.md` - UI and combat
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/chapters/10-build-system.md` - Build system (809 lines)
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/CRITICAL_DESIGN_PATTERNS.md` - Design patterns
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/MODDING_ARCHITECTURE_IDEAS.md` - Modding exploration
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/FORMATION_IMPLEMENTATION_LESSONS.md` - Formations
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/VEHICLE_COMBAT_IMPLEMENTATION_PLANS.md` - Vehicles
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/DECOUPLING_ATTEMPT_POSTMORTEM.md` - Architecture lessons
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/ASSET_REORGANIZATION_PLAN.md` - Asset plans
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/PROBLEMS.md` - Known issues
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/SOLDIER_ANIMATION_MIGRATION_PLAN.md` - Animation
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/docs/other/ZOOM_IMPLEMENTATION_NOTES.md` - Zoom feature
- `/home/juk/temp/CloseCombatClones/OpenCombat-SDL/LICENSE` - GPL-2.0 license text

**Key Source Code Files:**
- `src/main.cpp` - Application entry point
- `src/application/GameApplication.cpp/h` - Game state management
- `src/application/CombatModule.cpp/h` - Main combat gameplay
- `src/graphics/Screen.cpp/h` - SDL2 rendering and blitting
- `src/graphics/FontManager.cpp/h` - Text rendering
- `src/graphics/Animation.cpp/h` - Animation system
- `src/world/World.cpp/h` - Game world simulation
- `src/world/Map.cpp/h` - Terrain system
- `src/world/LineOfSight.cpp/h` - Visibility calculations
- `src/world/Building.cpp/h` - Building system
- `src/objects/Object.cpp/h` - Base object class
- `src/objects/Soldier.cpp/h` - Soldier implementation
- `src/objects/Vehicle.cpp/h` - Vehicle implementation
- `src/objects/Squad.cpp/h` - Squad system
- `src/states/State.cpp/h` - State machine (64-bit bitfield)
- `src/states/Action.h/cpp` - Action system
- `src/orders/Order.cpp/h` - Order base class
- `src/ai/AStar.cpp/h` - A* pathfinding
- `src/ai/Path.cpp/h` - Path structures
- `src/misc/tinyxml2.cpp/h` - XML parsing (bundled)
- `config/*.xml` - Configuration files

**Key Dependencies:**
- SDL2 - Core windowing, events, 2D rendering
- SDL2_mixer - Audio playback
- SDL2_ttf - TrueType font rendering
- tinyxml2 - XML parsing (bundled in src/misc/)

---

### F.1.3 CloseCombatFree (CCF) - Qt/QML Implementation

**Project Information:**
- **Author**: Tomasz 'sierdzio' Siekierda
- **License**: GNU General Public License v3 (GPL-3.0)
- **Language**: C++17 with QML (QtQuick 2.1)
- **Version**: 0.0.5 (Early Development)
- **Lines of Code**: ~258 total files, ~122 C++ files, ~50 QML files
- **Documentation**: ~12,500 lines across 10 chapters
- **Original Date**: 2011-2012
- **Last Updated**: 2024

**Primary Documentation Files:**
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/ARCHITECTURE.md` - Architecture index (499 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/01-overview.md` - Overview (1,189 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/02-object-system.md` - Object system (1,387 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/03-state-machine.md` - State machine (1,484 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/04-orders-pathfinding.md` - Orders/pathfinding (1,326 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/05-graphics.md` - Graphics (1,546 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/06-world-terrain.md` - World/terrain (1,206 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/07-configuration.md` - Configuration (1,352 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/08-assets.md` - Assets (1,227 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/09-ui-combat.md` - UI/combat (1,567 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/chapters/10-build-system.md` - Build system (1,184 lines)
- `/home/juk/temp/CloseCombatClones/closecombatfree/doc/LICENSE.txt` - GPL-3.0 license text

**Key Source Code Files:**
- `src/main.cpp` - Entry point
- `src/ccfmain.cpp/h` - Main window and QML view manager
- `src/ccfglobal.cpp/h` - Global utilities
- `src/ccferror.cpp/h` - Error handling
- `src/ccfgamemanager.cpp/h` - Scenario/campaign management
- `src/config/ccfconfig.cpp/h` - Configuration management
- `src/config/ccflogger.cpp/h` - Logging
- `src/logic/ccfobjectbase.cpp/h` - Base object class
- `src/logic/ccfscenariostate.cpp/h` - Scenario state
- `src/logic/ccfenginehelpers.cpp/h` - Engine utilities
- `src/qmlBase/ccfqmlbasescenario.cpp/h` - Scenario controller
- `src/qmlBase/ccfqmlbasemap.cpp/h` - Map data
- `src/qmlBase/ccfqmlbaseunit.cpp/h` - Unit base class
- `src/qmlBase/ccfqmlbasesoldier.cpp/h` - Soldier unit
- `src/qmlBase/ccfqmlbasevehicle.cpp/h` - Vehicle unit
- `qml/` - QML UI components and scenarios

**Key Dependencies:**
- Qt5 Core - Event loop, object model, file I/O
- Qt5 GUI - Window management, OpenGL
- Qt5 QML - Declarative UI engine
- Qt5 Quick - Scene graph, animations

---

## F.2 Secondary Sources - Original Games and Related Works

### F.2.1 Close Combat Series (Original Games)

**Primary Reference Games:**
- **Close Combat** (1996) - Atomic Games, Microsoft
- **Close Combat: A Bridge Too Far** (1997) - Atomic Games
- **Close Combat III: The Russian Front** (1999) - Atomic Games
- **Close Combat IV: Battle of the Bulge** (1999) - Atomic Games
- **Close Combat V: Invasion Normandy** (2000) - Atomic Games
- **Close Combat: Marines** (2004) - Atomic Games
- **Close Combat: The Longest Day** (2009) - Matrix Games
- **Close Combat: Panthers in the Fog** (2012) - Matrix Games/Slitherine
- **Close Combat: Gateway to Caen** (2014) - Matrix Games/Slitherine
- **Close Combat: Bloody First** (2019) - Matrix Games/Slitherine

**Publisher/Developers:**
- Atomic Games (original developer, 1996-2004)
- Matrix Games (current rights holder, 2005-present)
- Slitherine Software (publisher, 2012-present)

**Reference Websites:**
- Close Combat Series Community: http://www.closecombatseries.net/
- Matrix Games Close Combat Forum

---

### F.2.2 Related Tactical Wargames

**Squad-Level Tactical Games:**
- **Combat Mission** series - Battlefront.com (tactical simulation)
- **Theatre of War** series - 1C Company (real-time tactical)
- **Graviteam Tactics** series - Graviteam (operational-tactical)
- **Men of War** series - Best Way/1C (real-time tactics)
- **Company of Heroes** series - Relic Entertainment (RTS with tactical elements)
- **Wargame** series - Eugen Systems (modern warfare RTS)

**Turn-Based Tactical:**
- **X-COM** series - MicroProse/Firaxis (squad tactics)
- **Jagged Alliance** series - Sir-Tech (squad management)
- **Silent Storm** series - Nival Interactive (WWII tactics)

**References for Game Mechanics:**
- Line-of-sight systems
- Suppression mechanics
- Morale and psychology modeling
- Ballistics simulation
- Cover systems

---

### F.2.3 Academic Papers and Research

**Entity Component System (ECS):**
- Bilas, Scott. "A Data-Driven Game Object System." GDC 2002.
- West, Matthew. "Evolve Your Hierarchy." Cowboy Programming, 2007.
- Various papers on Data-Oriented Design (DOD)

**State Machines and AI:**
- Rabin, Steve. "AI Game Programming Wisdom" series (Charles River Media)
- Buckland, Mat. "Programming Game AI by Example" (Wordware Publishing, 2005)
- Champandard, Alex J. "AI Game Development" (New Riders, 2004)

**Pathfinding:**
- Hart, P.E.; Nilsson, N.J.; Raphael, B. "A Formal Basis for the Heuristic Determination of Minimum Cost Paths." IEEE Transactions on Systems Science and Cybernetics, 1968.
- Dijkstra, E.W. "A Note on Two Problems in Connexion with Graphs." Numerische Mathematik, 1959.
- Various A* optimization papers

**Game Architecture:**
- Gregory, Jason. "Game Engine Architecture" (see F.4.1)
- Nystrom, Robert. "Game Programming Patterns" (see F.4.2)

---

## F.3 Software and Tools

### F.3.1 Graphics and Multimedia Libraries

**SDL (Simple DirectMedia Layer)**
- **Website**: https://www.libsdl.org/
- **License**: zlib License
- **Used by**: OpenCombat-SDL
- **Components**:
  - SDL2 2.0.14+ - Core windowing, events, 2D rendering
  - SDL2_mixer - Audio playback and mixing
  - SDL2_ttf - TrueType font rendering
- **Purpose**: Cross-platform multimedia library for OpenCombat-SDL

**GGEZ (Rust Game Framework)**
- **Website**: https://ggez.rs/
- **Repository**: https://github.com/ggez/ggez
- **License**: MIT
- **Version**: 0.9.3
- **Used by**: OpenCombat (Rust)
- **Purpose**: 2D game framework with hardware acceleration
- **Features**: Rendering, audio, input, window management

**Qt Framework**
- **Website**: https://www.qt.io/
- **License**: LGPL/GPL/Commercial
- **Version**: Qt5 (5.x or later)
- **Used by**: CloseCombatFree
- **Components**:
  - Qt5 Core - Event loop, object model
  - Qt5 GUI - Window management, OpenGL
  - Qt5 QML - Declarative UI
  - Qt5 Quick - Scene graph, animations
- **Purpose**: Cross-platform application framework with QML for UI

**egui**
- **Repository**: https://github.com/emilk/egui
- **License**: MIT/Apache-2.0
- **Used by**: OpenCombat (via ggegui)
- **Purpose**: Immediate mode GUI for Rust

---

### F.3.2 ECS Frameworks

**Specs (Rust ECS)**
- **Repository**: https://github.com/amethyst/specs
- **License**: MIT/Apache-2.0
- **Note**: Referenced in OpenCombat documentation as ECS pattern example
- **Purpose**: Parallel entity component system for Rust

**Bevy ECS**
- **Repository**: https://github.com/bevyengine/bevy
- **License**: MIT/Apache-2.0
- **Note**: Modern Rust ECS architecture reference

**EntityX (C++)**
- **Repository**: https://github.com/alecthomas/entityx
- **License**: MIT
- **Note**: C++ ECS framework reference

---

### F.3.3 Map and Level Design Tools

**Tiled Map Editor**
- **Website**: https://www.mapeditor.org/
- **Repository**: https://github.com/mapeditor/tiled
- **License**: GPL-2.0
- **Used by**: OpenCombat
- **Format**: TMX/TSX files
- **Purpose**: 2D tile map editor
- **Rust Crate**: tiled 0.10.2

**QML Scene Editor**
- **Part of**: Qt Creator
- **Used by**: CloseCombatFree
- **Purpose**: Visual QML editing

---

### F.3.4 Networking Libraries

**ZeroMQ**
- **Website**: https://zeromq.org/
- **Repository**: https://github.com/zeromq/libzmq
- **License**: MPL-2.0
- **Version**: 4.3.4+
- **Used by**: OpenCombat
- **Rust Crate**: zmq 0.9
- **Purpose**: High-performance asynchronous messaging
- **Patterns**: REQ/REP, PUB/SUB

---

### F.3.5 Serialization and Data

**Serde (Rust)**
- **Repository**: https://github.com/serde-rs/serde
- **License**: MIT/Apache-2.0
- **Used by**: OpenCombat
- **Purpose**: Serialization framework
- **Companion**: bincode 1.3.3 (binary serialization)

**tinyxml2**
- **Repository**: https://github.com/leethomason/tinyxml2
- **License**: zlib
- **Used by**: OpenCombat-SDL (bundled)
- **Purpose**: XML parsing

**JSON**
- Used by all three projects for configuration
- OpenCombat: serde_json
- CloseCombatFree: Qt JSON support

---

### F.3.6 Build Tools

**Cargo (Rust)**
- **Website**: https://doc.rust-lang.org/cargo/
- **Used by**: OpenCombat
- **Purpose**: Rust package manager and build system
- **Features**: Dependency management, compilation, testing

**GNU Make**
- **Used by**: OpenCombat-SDL
- **Purpose**: Build automation
- **Requirements**: Make 3.81+

**qmake**
- **Part of**: Qt framework
- **Used by**: CloseCombatFree
- **Purpose**: Qt project build system

**CMake**
- **Website**: https://cmake.org/
- **Note**: Used by many game projects for cross-platform builds

---

### F.3.7 Profiling and Debugging

**Puffin**
- **Repository**: https://github.com/EmbarkStudios/puffin
- **License**: MIT/Apache-2.0
- **Used by**: OpenCombat
- **Purpose**: Rust profiling
- **Tool**: puffin_viewer

**RenderDoc**
- **Website**: https://renderdoc.org/
- **Purpose**: Graphics debugging
- **Note**: Useful for OpenCombat graphics debugging

---

### F.3.8 Documentation Tools

**Doxygen**
- **Website**: https://www.doxygen.nl/
- **License**: GPL-2.0
- **Used by**: CloseCombatFree
- **Purpose**: API documentation generation

**doxyqml**
- **Website**: http://agateau.com/projects/doxyqml/
- **Used by**: CloseCombatFree
- **Purpose**: QML support for Doxygen

**Mermaid**
- **Website**: https://mermaid.js.org/
- **License**: MIT
- **Used in**: All three projects' documentation
- **Purpose**: Diagram generation from markdown

---

## F.4 Architecture Patterns References

### F.4.1 Game Engine Architecture Books

**Gregory, Jason. "Game Engine Architecture" (3rd Edition)**
- **Publisher**: CRC Press
- **ISBN**: 978-1138035454
- **Topics**:
  - Engine architecture patterns
  - Subsystem organization
  - Game loop design
  - Memory management
  - Multi-threading
- **Relevance**: Foundation for understanding all three implementations

**Eberly, David H. "3D Game Engine Design" (2nd Edition)**
- **Publisher**: Morgan Kaufmann
- **ISBN**: 978-0122290633
- **Topics**: Engine fundamentals, mathematical foundations

---

### F.4.2 Game Programming Patterns

**Nystrom, Robert. "Game Programming Patterns"**
- **Publisher**: Genever Benning
- **ISBN**: 978-0990582908
- **Available**: https://gameprogrammingpatterns.com/
- **Topics**:
  - Design patterns in games
  - Command pattern (orders system)
  - State machines
  - Object pools
  - Event queues
  - Component pattern
- **Relevance**: Directly applicable to all three projects

**Gamma, Erich; Helm, Richard; Johnson, Ralph; Vlissides, John. "Design Patterns: Elements of Reusable Object-Oriented Software" (GoF)**
- **Publisher**: Addison-Wesley
- **ISBN**: 978-0201633610
- **Patterns Used**:
  - State (state machines in all projects)
  - Observer (event systems)
  - Command (order systems)
  - Factory (entity creation)
  - Singleton (global managers)
  - Strategy (behavior variation)

---

### F.4.3 Language-Specific References

**Rust Programming:**
- **Klabnik, Steve; Nichols, Carol. "The Rust Programming Language"**
  - Available: https://doc.rust-lang.org/book/
- **Rust Design Patterns**: https://rust-unofficial.github.io/patterns/
- **Rust API Guidelines**: https://rust-lang.github.io/api-guidelines/

**C++ Programming:**
- **Meyers, Scott. "Effective C++" series**
- **Stroustrup, Bjarne. "The C++ Programming Language" (4th Edition)**
- **C++ Core Guidelines**: https://isocpp.github.io/CppCoreGuidelines/

**Qt/QML:**
- **Blanchette, Jasmin; Summerfield, Mark. "C++ GUI Programming with Qt 4"**
- **Qt Documentation**: https://doc.qt.io/
- **QML Reference**: https://doc.qt.io/qt-6/qmlreference.html

---

## F.5 Online Resources

### F.5.1 Game Development Communities

**Reddit Communities:**
- r/gamedev - General game development
- r/rust_gamedev - Rust game development
- r/roguelikedev - Roguelike development (relevant for tactical games)
- r/wargames - Wargaming community

**Forums:**
- Close Combat Series: http://www.closecombatseries.net/
- Matrix Games Forums (Close Combat section)
- Gamedev.net - Game development community
- TIGSource Forums - Indie game development

### F.5.2 Architecture Documentation Repositories

**Game Architecture Examples:**
- Handmade Hero: https://handmadehero.org/ (educational game programming)
- Quake III Arena source code (id Software, GPL)
- Doom 3 source code (id Software, GPL)

**Open Source Game Engines:**
- Godot Engine: https://godotengine.org/
- Bevy Engine: https://bevyengine.org/
- Amethyst Engine: https://amethyst.rs/ (discontinued but educational)

### F.5.3 Modding Communities

**Close Combat Modding:**
- Close Combat Series modding forums
- Matrix Games modding community

**General Modding Resources:**
- Mod DB: https://www.moddb.com/
- Nexus Mods: https://www.nexusmods.com/

### F.5.4 Technical Blogs and Articles

**ECS Architecture:**
- "ECS FAQ" by Sander Mertens: https://github.com/SanderMertens/ecs-faq
- "Data-Oriented Design" by Noel Llopis
- "Unity ECS" documentation and blog posts

**Game AI:**
- Game AI Pro: http://www.gameaipro.com/
- AI and Games: https://www.aiandgames.com/

**Pathfinding:**
- Red Blob Games: https://www.redblobgames.com/
- Amit Patel's pathfinding visualizations

---

## F.6 Historical Context

### F.6.1 Evolution of Game Development (2005-2024)

**2005-2010: DirectX Era**
- Original OpenCombat (adv-warfare) released for Windows/DirectX
- CCF development begins (2011)
- Hardware: Single-core dominance, GPU acceleration emerging
- Paradigm: Object-oriented design, inheritance-heavy hierarchies

**2011-2015: Cross-Platform Transition**
- CCF adopts Qt5/QML for cross-platform support
- Rise of mobile gaming influences UI design
- Multi-core processors become standard
- ECS pattern emerges as alternative to OOP

**2016-2020: Rust and Modern C++**
- Rust 1.0 released (2015)
- OpenCombat (Rust) development begins
- C++17 standardization
- Data-oriented design gains popularity
- OpenCombat-SDL port begins

**2021-2024: Current State**
- Rust Edition 2021
- Modern game frameworks (Bevy, GGEZ)
- Cross-platform as default expectation
- ECS is mainstream
- OpenCombat-SDL port completed

### F.6.2 Changes in Programming Paradigms

**Object-Oriented Programming (OOP):**
- **Peak**: 2000s-2010s
- **Characteristics**: Deep inheritance hierarchies
- **Seen in**: OpenCombat-SDL (legacy), CloseCombatFree

**Entity Component System (ECS):**
- **Emergence**: 2010s
- **Characteristics**: Composition over inheritance
- **Seen in**: OpenCombat (Rust) - modified ECS pattern
- **Benefits**: Cache-friendly, parallelizable

**Data-Oriented Design (DOD):**
- **Emergence**: 2010s
- **Characteristics**: Structure of Arrays, cache optimization
- **Reference**: All projects document DOD considerations

### F.6.3 Hardware Evolution Impact

**CPU:**
- 2005: Single-core dominant
- 2010: Multi-core becomes standard
- 2024: Many-core, heterogeneous computing
- **Impact**: ECS and parallel architecture patterns

**GPU:**
- 2005: Fixed-function pipelines
- 2010: Programmable shaders
- 2024: General-purpose GPU computing
- **Impact**: Hardware acceleration in ggez (OpenCombat)

**Memory:**
- 2005: 512MB-2GB typical
- 2024: 16GB+ typical
- **Impact**: Less emphasis on extreme optimization, more on architecture

---

## F.7 Version History

### F.7.1 OpenCombat (Rust) Version History
- **v0.4.0** - Current (battle_core, battle_server, battle_gui)
- **v0.2.0** - oc_core established
- **v0.1.0** - Initial development

### F.7.2 CloseCombatFree Version History
- **v0.0.5** - Qt5 port, expanded docs, particle redesign, LOS detection, side markers
- **v0.0.4** (2012.02.10) - Save/load, LOS detection, QML 2.0, Doxygen docs
- **v0.0.3** (2011.12.20) - Scripting, waypoints, preferences, zooming

### F.7.3 OpenCombat-SDL Port Timeline
- **2005** - Original adv-warfare release (snapshot 0.1a)
- **2024-2026** - SDL2 port phases:
  - Phase 0: Preparation
  - Phase 1: Graphics (D3D→SDL2)
  - Phase 2: Input (Win32→SDL2)
  - Phase 3: Audio (DSound→SDL2_mixer)
  - Phase 4: XML (MSXML→tinyxml2)
  - Phase 5: Testing & Polish (current)

---

## F.8 Comparative Analysis References

### F.8.1 Architecture Comparison Sources

This book compares three distinct architectural approaches:

1. **Rust/GGEZ (OpenCombat)**
   - Modern systems language
   - Client-server architecture
   - Modified ECS pattern
   - Message-driven state machines

2. **C++/SDL2 (OpenCombat-SDL)**
   - Traditional game engine port
   - Single-player focused
   - Inheritance-based OOP
   - Bitfield state machines

3. **C++/Qt-QML (CloseCombatFree)**
   - Hybrid C++/declarative approach
   - QML for UI and scenarios
   - Traditional OOP with Qt patterns
   - Property-based state management

### F.8.2 Design Pattern Cross-References

| Pattern | OpenCombat (Rust) | OpenCombat-SDL | CloseCombatFree |
|---------|-------------------|----------------|-----------------|
| State Machine | Message-driven enum-based | 64-bit bitfield | State enum + QML states |
| Entity Management | Modified ECS | Inheritance hierarchy | QObject hierarchy |
| Rendering | GGEZ (hardware) | SDL2 (software) | Qt Quick Scene Graph |
| UI | ggegui + custom | SDL2 immediate mode | QML declarative |
| Networking | ZeroMQ | N/A | N/A |
| Pathfinding | pathfinding crate | Custom A* | Custom A* |
| Configuration | JSON | XML | JSON/XML/QML |

---

## F.9 Additional References

### F.9.1 World War II Historical References

For historical accuracy in unit representations and scenarios:
- **Weapons and Equipment**: Various WWII small arms references
- **Tactics**: Squad-level infantry tactics manuals
- **Battlefields**: European theater geography and terrain

### F.9.2 Asset Sources

**Public Domain Imagery:**
- NASA imagery (used in CCF development)
- Creative Commons licensed aerial photographs

**Audio:**
- OGG format (OpenCombat)
- WAV format (OpenCombat-SDL)

**Graphics:**
- PNG sprite sheets (OpenCombat)
- TGA images (OpenCombat-SDL)
- Qt Resource System (CloseCombatFree)

---

*Document compiled for the Close Combat Clones Comparative Architecture Book*
*Last Updated: 2026*
