# Chapter 5: Unit Hierarchy and Compositional Architecture

## 5.1 Introduction: The Evolution of Entity Design

How you structure game entities fundamentally shapes every other system in your tactical wargame. The three Close Combat clones demonstrate a nearly 20-year evolution in game architecture:

| Game | Year | Pattern | Core Philosophy |
|------|------|---------|-----------------|
| **OpenCombat-SDL** | 2005 | Deep Inheritance | "Everything IS-A Object" |
| **CloseCombatFree** | 2011 | Component Composition | "Entities are composed of parts" |
| **OpenCombat** | 2024 | Modified ECS | "Entities are data processed by systems" |

This chapter examines these approaches through the lens of tactical wargame requirements: **hierarchical command structures**, **aggregate relationships** (squads containing soldiers), **dynamic capabilities** (crew affecting vehicle function), and **moddability** (adding new unit types without recompilation).

### Key Design Questions

Before diving into implementations, consider these questions for your own project:

1. **Hierarchy vs. Flattening**: Should a squad "be" an object or "contain" objects?
2. **Ownership vs. Referencing**: Who owns a soldier when they're in a vehicle?
3. **Capabilities**: How does a vehicle "know" it can move without a driver?
4. **Extensibility**: Can modders add a "sniper team" without source code changes?
5. **Performance**: Does cache efficiency matter when you have 500 entities, not 50,000?

The three projects answer these differently, reflecting their eras and constraints.

---

## 5.2 The Three Architectural Patterns

### 5.2.1 OpenCombat-SDL: Classical Inheritance (2005)

OpenCombat-SDL represents traditional object-oriented design where everything inherits from a common base:

```
                    ┌─────────────┐
                    │    Object   │  ← Base class with position,
                    │             │    orders, selection, health
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           │               │               │
      ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
      │ Soldier │    │ Vehicle │    │  Squad  │
      └─────────┘    └─────────┘    └─────────┘
```

#### The "Squad as Object" Decision

**Why inheritance seemed right:**
- Squads can be selected, receive orders, and have positions (like Soldiers)
- Code reuse: Squad inherits order queue management from Object
- Polymorphism: Any Object pointer could reference a Soldier, Vehicle, OR Squad

```cpp
// Squad inherits all Object capabilities
class Squad : public Object {
    std::vector<Soldier*> _soldiers;  // Non-owning refs
    std::vector<Vehicle*> _vehicles;  // Non-owning refs
    
    void Simulate(long dt, World* world) override {
        // Delegate to members
        for (auto* soldier : _soldiers) {
            soldier->Simulate(dt, world);
        }
    }
};
```

#### The Command Flow

Inheritance enables a unified command interface:

```
Player gives order → Squad::AddOrder() → Distributes to members
                                    ↓
                          ┌───────────────┐
                          │  Order Queue  │
                          └───────┬───────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ↓             ↓             ↓
              Soldier #1    Soldier #2    Vehicle #1
```

#### Aggregation: What a Squad "Has" vs "Is"

The critical distinction emerges in ownership:

```cpp
// Squad does NOT own its members
class Squad : public Object {
    std::vector<Soldier*> _soldiers;  // Non-owning: World owns soldiers
};

// Squad composition is aggregation (HAS-A), not inheritance (IS-A)
// Soldier IS-A Object, Squad HAS-A Soldiers
```

This creates a hybrid model:
- **Inheritance**: Squad IS-A Object (for game engine uniformity)
- **Aggregation**: Squad HAS-A Soldiers (for tactical organization)

#### The Crew Problem

Vehicles expose the limitation of pure inheritance:

```cpp
class Vehicle : public Object {
    struct CrewSlot {
        Soldier* soldier;  // Non-owning reference
        int weaponSlot;
    };
    
    std::array<CrewSlot, MAX_CREW> _crew;
    
    bool CanMove() {
        // Must check if driver slot is filled
        return _crew[0].soldier != nullptr;
    }
};
```

**The awkwardness**: A Vehicle "is" an Object, but it also "has" Soldiers who "are" Objects. This creates circular conceptual dependencies.

### 5.2.2 CloseCombatFree: Component Composition (2011)

CloseCombatFree abandons deep inheritance for declarative composition using Qt's QML:

```
Unit (QML Component)
├── Hull.qml        ← Visual + collision
├── Turret.qml      ← Rotating weapon mount
├── Soldier {}      ← Crew member (reusable component)
├── Soldier {}      ← Another crew member
└── Effects.qml     ← Particles
```

#### Composition in Practice

```qml
// units/tanks/HeavyTank.qml
Unit {
    id: tank
    unitType: "Heavy Tank"
    maxSpeed: 15
    
    // Visual composition
    HeavyTank_hull { id: hull }
    HeavyTank_turret { id: turret }
    
    // Crew composition
    Soldier { role: "Commander" }
    Soldier { role: "Gunner" }
    Soldier { role: "Driver" }
}
```

**Key insight**: The tank doesn't "inherit" crew capability—it "contains" Soldier components.

#### Capabilities Through Composition

```qml
// Tank.qml
function canMove() {
    // Query children for Driver role
    for (var i = 0; i < children.length; i++) {
        if (children[i].role === "Driver" && 
            children[i].status !== "KIA") {
            return true;
        }
    }
    return false;
}
```

**Benefit**: Capabilities emerge from composition. No inheritance hierarchy needed.

#### Runtime Instantiation for Modding

The breakthrough: new unit types without recompilation:

```cpp
void Scenario::loadUnit(const QString& qmlFile) {
    QQmlComponent component(&engine, qmlFile);
    QObject* unit = component.create();  // Runtime instantiation!
    
    // Add to scenario
    units.append(unit);
}
```

Modders create `SniperTeam.qml`:
```qml
Unit {
    unitType: "Sniper Team"
    Soldier { role: "Spotter"; equipment: "Binoculars" }
    Soldier { role: "Sniper"; weapon: "SniperRifle" }
}
```

No C++ code changes. No recompilation. Pure data-driven design.

### 5.2.3 OpenCombat: Modified ECS (2024)

OpenCombat represents modern data-oriented design:

```
┌─────────────────────────────────────────────────────┐
│                  BattleState                         │
├─────────────────────────────────────────────────────┤
│ soldiers: Vec&lt;Soldier&gt;          [contiguous]         │
│ vehicles: Vec&lt;Vehicle&gt;          [contiguous]         │
│ squads: HashMap&lt;Uuid, Squad&gt;   [lookup]             │
└─────────────────────────────────────────────────────┘
        ↓                        ↓
   ┌─────────┐             ┌─────────┐
   │ Systems │             │ Systems │
   │ - AI    │             │ - Move  │
   │ - Combat│             │ - Render│
   └─────────┘             └─────────┘
```

#### Type-Safe Indices Replace Pointers

```rust
#[derive(Debug, Clone, Copy)]
pub struct SoldierIndex(pub usize);

#[derive(Debug, Clone, Copy)]
pub struct VehicleIndex(pub usize);

pub struct Soldier {
    uuid: SoldierIndex,
    squad_uuid: SquadUuid,  // Index reference, not pointer
}
```

**Benefits**:
- **Type safety**: Cannot confuse SoldierIndex with VehicleIndex (compile-time check)
- **Serialization**: Indices are just numbers
- **No lifetimes**: No borrowing complexity
- **Cache efficiency**: Contiguous Vec storage

#### Squad as Lightweight Aggregate

```rust
pub struct SquadComposition(
    pub SoldierIndex,      // Leader
    pub SquadType,
    pub Vec<SoldierIndex>  // Members (indices, not refs)
);

// Stored in BattleState
pub struct BattleState {
    squads: HashMap<SquadUuid, SquadComposition>
}
```

**Critical difference**: Squad is not an entity type—it's a relationship structure.

#### Bidirectional Vehicle Boarding

```rust
pub struct BattleState {
    // Soldier → Vehicle lookup
    soldiers_on_board: HashMap<SoldierIndex, VehicleBoardPlace>,
    
    // Vehicle → Soldiers lookup  
    vehicle_board: HashMap<VehicleIndex, HashMap<OnBoardPlace, SoldierIndex>>,
}
```

Both lookups are O(1). No pointer chasing. No ownership confusion.

---

## 5.3 Universal Composition Patterns

Extracting patterns that work across all three approaches:

### 5.3.1 The Aggregate Pattern

**Problem**: How do you represent "a squad contains soldiers"?

**Three Approaches**:

| Aspect | Inheritance (SDL) | Composition (CCF) | ECS (OpenCombat) |
|--------|-------------------|-------------------|------------------|
| **Relationship** | Squad inherits Object | Unit contains Soldiers | Squad struct holds indices |
| **Storage** | Pointers in Squad | QML child objects | Vec in BattleState |
| **Lifecycle** | World owns all | Qt parent-child | BattleState owns |
| **Access** | O(1) pointer | O(n) traversal | O(1) index |
| **Type Safety** | Runtime casts | Dynamic QML | Compile-time indices |

**Universal Pattern**:

```pseudocode
// Aggregation: HAS-A relationship
Aggregate UnitGroup {
    // Members are referenced, not owned
    members: List<EntityReference>
    
    // Operations delegate to members
    function issueOrder(order) {
        for member in members {
            if member.isActive() {
                member.addOrder(order)
            }
        }
    }
}
```

### 5.3.2 The Capability Pattern

**Problem**: How does a vehicle "know" it can move?

**Three Approaches**:

**Inheritance approach** (problematic):
```cpp
class Vehicle : public Object {
    // Hardcoded: Vehicle is mobile
    bool IsMobile() override { return true; }  // Wrong if no driver!
};
```

**Composition approach** (dynamic):
```qml
// Capabilities depend on current state
function canMove() {
    return hasFunctionalCrew("Driver") && 
           fuel > 0 && 
           !isDestroyed;
}
```

**ECS approach** (data-driven):
```rust
pub fn can_vehicle_move(
    vehicle_idx: VehicleIndex,
    state: &BattleState
) -> bool {
    let vehicle = &state.vehicles[vehicle_idx.0];
    let crew = state.vehicle_board.get(&vehicle_idx);
    
    // Check for driver
    crew.map(|c| c.contains_key(&OnBoardPlace::Driver))
        .unwrap_or(false)
}
```

**Universal Pattern**:

```pseudocode
// Capabilities are calculated, not inherited
function getCapabilities(entity) {
    capabilities = {}
    
    // Check components/parts
    for component in entity.components {
        capabilities.add(component.provides)
    }
    
    // Check state
    if entity.health <= 0 {
        capabilities.remove(Move)
        capabilities.remove(Fire)
    }
    
    return capabilities
}
```

### 5.3.3 The Behavior Pattern

**Problem**: How do different unit types act differently?

**Inheritance approach** (rigid):
```cpp
// Behavior hardcoded in class hierarchy
class Sniper : public Soldier {
    void SelectTarget() override {
        // Sniper-specific target selection
    }
};
```

**Composition approach** (flexible):
```qml
// Behavior as interchangeable component
Soldier {
    behavior: SniperBehavior {}  // Swappable
}

Soldier {
    behavior: MachineGunnerBehavior {}
}
```

**ECS approach** (system-driven):
```rust
// Behavior determined by component flags
if soldier.has_trait(Trait::Stealthy) {
    ai.choose_hidden_position();
} else {
    ai.choose_defensive_position();
}
```

**Universal Pattern**:

```pseudocode
// Behavior is separate from identity
Entity Soldier {
    identity: Role  // "Sniper", "Rifleman"
    behavior: BehaviorComponent
    equipment: EquipmentComponent
}

// Behavior system processes by role
BehaviorSystem {
    function update(soldier) {
        match soldier.role {
            Sniper => sniper_ai.update(soldier),
            Leader => leader_ai.update(soldier),
            _ => standard_ai.update(soldier)
        }
    }
}
```

---

## 5.4 Military Hierarchy: Soldier → Squad → Team → Platoon

### 5.4.1 The OOB Challenge

Order of Battle (OOB) structures in tactical games:

```
Platoon (15-40 soldiers)
├── Squad 1 (8-12 soldiers)
│   ├── Team Alpha (4 soldiers)
│   └── Team Bravo (4 soldiers)
├── Squad 2
│   └── ...
└── Vehicle Section
    ├── Vehicle 1 (Tank)
    └── Vehicle 2 (Tank)
```

**Key insight**: This is a tree structure, not an inheritance hierarchy.

### 5.4.2 Implementing the Hierarchy

**Inheritance approach** (problematic):
```cpp
// Deep inheritance becomes brittle
class Platoon : public UnitGroup {
    std::vector<Squad*> squads;
};

class Squad : public UnitGroup {
    std::vector<Team*> teams;
};

// Problem: Platoon IS-A UnitGroup, Squad IS-A UnitGroup
// But Platoon contains Squads, creating parallel hierarchies
```

**ECS approach** (flat with parent references):
```rust
pub struct Unit {
    uuid: UnitIndex,
    unit_type: UnitType,        // Soldier, Squad, Platoon
    parent: Option<UnitIndex>,  // Parent in hierarchy
    children: Vec<UnitIndex>,   // Children in hierarchy
}

// Hierarchy is a property, not a type
```

**Universal Pattern**:

```pseudocode
// Tree structure with explicit parent/child
Entity Unit {
    unitType: enum { Soldier, Squad, Platoon }
    parent: Option<EntityReference>
    children: List<EntityReference>
    
    // Navigation
    function getRoot() -> Unit {
        if parent == null {
            return this
        }
        return parent.getRoot()
    }
    
    // Operations propagate down
    function issueOrder(order) {
        // Issue to self
        execute(order)
        
        // Issue to children
        for child in children {
            child.issueOrder(order)
        }
    }
}
```

### 5.4.3 Formation Following

Formations work by offsetting from a leader:

```pseudocode
Formation Column {
    // Member 0 is leader at position P
    // Member 1 is at P - (spacing behind leader)
    // Member 2 is at P - (2 * spacing behind leader)
    
    function getPosition(memberIndex, leaderPosition, leaderHeading) {
        offset = memberIndex * spacing
        return leaderPosition - (leaderHeading * offset)
    }
}

Formation Line {
    // Members form line perpendicular to heading
    function getPosition(memberIndex, leaderPosition, leaderHeading) {
        perpendicular = leaderHeading.rotate(90°)
        offset = (memberIndex - centerIndex) * spacing
        return leaderPosition + (perpendicular * offset)
    }
}
```

**Implementation in each pattern**:

| Pattern | Formation Storage | Application |
|---------|-------------------|-------------|
| **Inheritance** | Squad field | Each soldier calculates position relative to leader |
| **Composition** | Formation component | QML calculates target positions |
| **ECS** | Formation system | System updates soldier positions each frame |

---

## 5.5 Command and Control Relationships

### 5.5.1 Order Propagation

**Flow**: Player → Selected Unit → Subordinates

```
Player clicks "Move to X"
    ↓
Squad receives MoveOrder
    ↓
Squad::HandleMoveOrder():
  - Calculate path for leader
  - Leader gets FollowPath order
  - Others get Follow order (targeting leader)
    ↓
Soldier AI converts orders to actions
    ↓
Physics system executes movement
```

### 5.5.2 Leadership Dynamics

When the squad leader dies, the squad must reorganize:

**Inheritance approach**:
```cpp
void Squad::Update() {
    if (!leader->IsAlive()) {
        // Find new leader
        for (auto* soldier : _soldiers) {
            if (soldier->IsAlive() && soldier->HasLeadership()) {
                leader = soldier;
                break;
            }
        }
    }
}
```

**ECS approach**:
```rust
fn tick_update_squad_leaders(state: &mut BattleState) {
    for (squad_uuid, composition) in &mut state.squads {
        let leader = &state.soldiers[composition.0.0];
        
        if !leader.alive || leader.unconscious {
            // Promote next eligible
            for member_idx in &composition.2 {
                let member = &state.soldiers[member_idx.0];
                if member.alive && !member.unconscious {
                    composition.0 = *member_idx;
                    break;
                }
            }
        }
    }
}
```

### 5.5.3 Vehicle Command Structure

Vehicles add complexity: crew are soldiers, but the vehicle acts as a unit.

```
Player orders Tank Squad to move
    ↓
Squad orders each tank to move
    ↓
Tank checks: Do I have a driver?
    - Yes: Calculate path and move
    - No: Report "immobilized"
    ↓
Driver (soldier) executes steering
```

**Key insight**: Command flows through the organizational hierarchy, but capability depends on the composition state.

---

## 5.6 Moddability: Adding New Unit Types

### 5.6.1 The Recompilation Barrier

**Scenario**: Add a "Sniper Team" unit (2 soldiers, special behaviors).

**OpenCombat-SDL (requires recompile)**:
```cpp
// 1. Modify Soldier enum
enum SoldierType { Rifleman, MachineGunner, Sniper };  // Recompile!

// 2. Add to SoldierManager
void SoldierManager::CreateSoldier(Type type) {
    switch(type) {
        case Sniper: /* ... */ break;  // Recompile!
    }
}
```

**OpenCombat (requires recompile)**:
```rust
// 1. Modify SoldierType enum
pub enum SoldierType { Type1, Bren, Mg34, Sniper };  // Recompile!

// 2. Add behavior in AI system
match soldier.type_ {
    Sniper => sniper_behavior(soldier),  // Recompile!
    _ => standard_behavior(soldier),
}
```

**CloseCombatFree (no recompile)**:
```qml
// Create units/SniperTeam.qml
Unit {
    unitType: "Sniper Team"
    
    Soldier { role: "Spotter"; behavior: "Recon" }
    Soldier { role: "Sniper"; behavior: "SniperAI" }
}
```

**Runtime**: `Scenario::loadUnit("units/SniperTeam.qml")`

### 5.6.2 Designing for Modding

**Principle**: Separate **identity** from **implementation**.

**Identity** (data):
- Unit type name: "Sniper Team"
- Composition: 2 soldiers (Spotter, Sniper)
- Equipment: Sniper rifle, binoculars

**Implementation** (code):
- How snipers select targets
- How spotters mark enemies
- Accuracy calculations

**Modding-friendly architecture**:

```pseudocode
// Data-driven unit definition
UnitTemplate {
    name: "Sniper Team"
    
    // Composition
    members: [
        { role: "Spotter", behavior: "recon.lua" },
        { role: "Sniper", behavior: "sniper.lua" }
    ]
    
    // Capabilities
    capabilities: [CanMove, CanHide, CanFire]
    
    // Behavior scripts
    onEnemySpotted: "sniper_team_enemy_spotted.lua"
    onUnderFire: "sniper_team_under_fire.lua"
}
```

### 5.6.3 Hot-Reloading Behaviors

For rapid iteration, behaviors should reload without restarting:

```pseudocode
BehaviorSystem {
    scriptCache: Map<String, Script>
    
    function getBehavior(name) -> Script {
        // Check if file modified
        if fileChanged(name) || !scriptCache.contains(name) {
            scriptCache[name] = loadScript(name)
        }
        return scriptCache[name]
    }
}
```

**Benefits**:
- Modders test changes instantly
- Balance tweaks without patches
- AI improvements mid-development

---

## 5.7 Recommendations for New Projects

### 5.7.1 The Hybrid Approach

Synthesize the best of all three:

```
┌──────────────────────────────────────────────────────────────┐
│                    ENTITY STORAGE                            │
│         (Contiguous arrays - cache efficient)                │
├──────────────────────────────────────────────────────────────┤
│ soldiers: Vec&lt;Soldier&gt;                                       │
│ vehicles: Vec&lt;Vehicle&gt;                                       │
└──────────────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────────────┐
│                   COMPONENT SYSTEM                           │
│         (Composition over inheritance)                       │
├──────────────────────────────────────────────────────────────┤
│ Entity = ID + ComponentMask                                  │
│ Component = Plain data struct                                │
└──────────────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────────────┐
│                 SCRIPTABLE BEHAVIORS                         │
│         (Modding without recompilation)                      │
├──────────────────────────────────────────────────────────────┤
│ role: "Sniper" → behavior: "sniper_ai.lua"                   │
│ onEvent: callback from script                                │
└──────────────────────────────────────────────────────────────┘
```

### 5.7.2 Type-Safe Indices

Always use wrapper types for entity references:

```cpp
// C++
struct SoldierIndex { size_t value; };
struct VehicleIndex { size_t value; };

// Cannot accidentally mix up
void moveVehicle(VehicleIndex idx);  // Compiler error if passed SoldierIndex
```

```rust
// Rust
#[derive(Debug, Clone, Copy)]
pub struct SoldierIndex(pub usize);

// Type safety at compile time
let soldier = state.soldiers[soldier_idx.0];  // OK
let soldier = state.soldiers[vehicle_idx.0];  // Compile error!
```

### 5.7.3 Universal Entity API

Clean interface for all entity operations:

```pseudocode
EntityManager {
    // Creation
    createEntity(type: String) -> EntityId
    destroyEntity(id: EntityId)
    
    // Component access
    getComponent&lt;T&gt;(id: EntityId) -> Option&lt;T&gt;
    addComponent&lt;T&gt;(id: EntityId, component: T)
    removeComponent&lt;T&gt;(id: EntityId)
    
    // Relationships
    setParent(child: EntityId, parent: EntityId)
    getChildren(parent: EntityId) -> List&lt;EntityId&gt;
    
    // Queries
    query(mask: ComponentMask) -> List&lt;EntityId&gt;
    queryInRadius(center: Point, radius: float) -> List&lt;EntityId&gt;
}
```

### 5.7.4 Template/Prefab System

Define base templates, override for instances:

```json
{
    "template": "GermanRifleSquad",
    "overrides": {
        "experience": "Veteran",
        "morale": 85,
        "equipment": {
            "primary": "MP40",
            "secondary": "Panzerfaust"
        }
    }
}
```

---

## 5.8 Conclusion

The evolution from OpenCombat-SDL (2005) to OpenCombat (2024) reveals a clear trajectory:

1. **Inheritance to Composition**: Modern games favor composition for flexibility
2. **Pointers to Indices**: Type-safe references prevent bugs
3. **Code to Data**: Data-driven design enables modding
4. **Scattered to Contiguous**: Cache efficiency matters even for moderate entity counts

### Decision Matrix

| If you need... | Consider... |
|----------------|-------------|
| Maximum moddability | Component composition (QML/JSON) |
| Maximum performance | Modified ECS with contiguous storage |
| Rapid development | Deep inheritance (if team knows OOP well) |
| Console deployment | Data-oriented ECS (cache-friendly) |
| Community content | Scriptable behaviors + templates |

### Final Recommendation

For a new Close Combat clone in 2024:

1. **Storage**: Contiguous arrays (ECS-style) for cache efficiency
2. **Relationships**: Type-safe indices, not pointers
3. **Composition**: Component-based (entities are bags of components)
4. **Hierarchy**: Explicit parent/child relationships, not inheritance
5. **Behavior**: Scriptable (Lua/Wren) for moddability
6. **Templates**: JSON-based prefabs for rapid content creation

The goal is a system where:
- Adding a sniper team requires only data files
- Cache misses don't kill performance with 500 entities
- Type errors are caught at compile time
- Command hierarchies are explicit and debuggable

This is the architecture that will serve the next generation of tactical wargames.

---

**Next**: [Chapter 6: Order & AI System Design](#chapter-6-order--ai-system-design)
