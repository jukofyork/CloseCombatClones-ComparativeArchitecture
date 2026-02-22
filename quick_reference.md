# Close Combat Clones: Quick Reference Guide

**For Busy Developers: Essential Patterns, Decisions, and Checklists**

---

## 1. Pattern Quick Reference

### Pattern 1: Soldier/Squad Aggregate Pattern
**One-line:** Hierarchical relationship between individual soldiers and their organizational squad, enabling both autonomous individual behavior and coordinated group action.

**When to use:**
- Group-based unit control required
- Squad-level commands simplify player control
- Formation system needed for tactical coordination
- Morale propagation through squad hierarchy

**Code Skeleton (10 lines):**
```cpp
struct Squad {
    Soldier* leader;
    vector<Soldier*> members;
    void IssueOrder(Order* o) {
        for (auto* m : members) 
            m->ReceiveOrder(AdaptForMember(o, m));
    }
};
```

**See:** Chapter 11.3, Chapter 5.3.1, Chapter 8.3.1

---

### Pattern 2: Weapon Team Pattern
**One-line:** Model weapon systems requiring multiple crew members with distinct roles, where weapon effectiveness depends on crew competence.

**When to use:**
- Crewed vehicles (tanks, artillery)
- Weapon teams (machine guns, mortars)
- Crew casualties affect weapon capability
- Realistic crew modeling desired

**Code Skeleton (10 lines):**
```cpp
struct WeaponSystem {
    map<string, Soldier*> crew;  // role -> soldier
    bool CanOperate() { 
        return crew["Gunner"] && crew["Loader"]; 
    }
    void FireAt(Target t) { if (CanOperate()) /* fire */ }
};
```

**See:** Chapter 11.3, Chapter 5.5.3

---

### Pattern 3: Component Composition Pattern
**One-line:** Build complex entities by composing simple, reusable components rather than using deep inheritance hierarchies.

**When to use:**
- Diverse entity types requiring flexible composition
- Avoiding "diamond problem" with multiple inheritance
- Runtime entity modification needed
- Novel unit combinations expected

**Code Skeleton (10 lines):**
```cpp
struct Entity {
    map<Type, Component*> comps;
    void Add(Component* c) { comps[c->type] = c; }
    template<typename T> T* Get() { 
        return (T*)comps[typeid(T)]; 
    }
};
```

**See:** Chapter 11.3, Chapter 5.3.2, Chapter 8.3.2

---

### Pattern 4: State Hierarchy Pattern
**One-line:** Organize unit behavior into hierarchical states operating on different timescales—strategic (minutes), tactical (seconds), and physical (milliseconds).

**When to use:**
- Complex unit behavior on multiple timescales
- Clear separation of concerns needed
- Frame-perfect timing required (multiplayer)
- Deterministic simulation essential

**Code Skeleton (10 lines):**
```rust
enum Phase { Placement, Battle, End }
enum Behavior { MoveTo(Path), Defend(Angle), EngageSoldier(Index) }
enum Gesture { Idle, Reloading(u64), Aiming(u64) }
struct State { phase: Phase, behavior: Behavior, gesture: Gesture }
```

**See:** Chapter 11.4, Chapter 3.4, Chapter 3.8.2

---

### Pattern 5: Order Queue Pattern
**One-line:** Allow units to accept multiple commands in sequence, executing them one at a time with automatic state transitions.

**When to use:**
- Players need to plan unit actions in advance
- Command chaining desired (move then attack)
- Reducing micromanagement important
- Real-time gameplay with planning elements

**Code Skeleton (10 lines):**
```cpp
class Unit {
    deque<Order*> queue;
    void Enqueue(Order* o) { queue.push_back(o); }
    void Update() {
        if (queue.empty()) return;
        if (queue.front()->IsComplete()) {
            queue.pop_front();
            if (!queue.empty()) queue.front()->Start();
        }
    }
};
```

**See:** Chapter 11.4, Chapter 6.2

---

### Pattern 6: Stance System Pattern
**One-line:** Model how physical posture affects gameplay—movement speed, visibility, cover effectiveness, and weapon accuracy.

**When to use:**
- Tactical positioning matters
- Realistic combat dynamics desired
- Players must trade off speed vs safety
- Cover system integration needed

**Code Skeleton (10 lines):**
```cpp
enum Stance { Standing, Crouching, Prone };
struct StanceMods {
    float speed, cover, visibility, accuracy;
};
map<Stance, StanceMods> mods = {
    {Standing, {1.0, 0.1, 1.0, 0.8}},
    {Crouching, {0.6, 0.5, 0.7, 1.0}},
    {Prone, {0.3, 0.9, 0.1, 1.2}}
};
```

**See:** Chapter 11.4, Chapter 6.4

---

### Pattern 7: Morale Cascade Pattern
**One-line:** Model psychological state and its propagation through unit hierarchies, where casualties and threats affect individual and group morale.

**When to use:**
- Realistic combat psychology desired
- Elite vs green troop differentiation
- Tactical suppression as viable tactic
- Emergent panic behaviors wanted

**Code Skeleton (10 lines):**
```cpp
struct Morale {
    float value = 100;  // 0-100
    void Update(Soldier* s) {
        if (s->underFire) value -= 10;
        if (s->squad->Casualties() > 0) value -= 20;
        if (s->leaderAlive) value += 5;
        value = clamp(value, 0, 100);
    }
};
```

**See:** Chapter 11.4

---

### Pattern 8: Command Abstraction Pattern
**One-line:** Separate high-level player intent (orders) from low-level unit execution (actions/gestures) through translation layers.

**When to use:**
- Complex command hierarchies
- AI operates at appropriate abstraction level
- Player intent must be preserved through chain
- Multiple abstraction layers needed

**Code Skeleton (10 lines):**
```cpp
// Orders → Behaviors → Actions → Gestures
Order MoveTo(pos) → Behavior MoveTo(Path) 
    → Action WalkTo(tile) → Gesture Walking(until: frame_245)
```

**See:** Chapter 11.5, Chapter 6.3

---

### Pattern 9: Formation Control Pattern
**One-line:** Enable coordinated movement of squad members in geometric formations, reducing pathfinding overhead.

**When to use:**
- Squad-based tactical games
- Reduced pathfinding cost desired
- Tactical positioning important
- Realistic military movement wanted

**Code Skeleton (10 lines):**
```cpp
enum Formation { Column, Line, Wedge, File };
vec2 GetFormationPos(Formation f, int memberIdx, 
                     vec2 leaderPos, float heading) {
    switch(f) {
        case Column: return leaderPos - heading * memberIdx * SPACING;
        case Line: return leaderPos + Perp(heading) * (memberIdx - center) * SPACING;
    }
}
```

**See:** Chapter 11.5, Chapter 6.3.2

---

### Pattern 10: Prerequisite Chain Pattern
**One-line:** Automatically insert required intermediate actions when a unit attempts an action it cannot currently perform.

**When to use:**
- Complex state machines
- Reducing player micromanagement
- Emergent behavior from simple rules
- Natural state transitions desired

**Code Skeleton (10 lines):**
```cpp
void QueueAction(Action* a) {
    auto missing = a->requires & ~currentState;
    while (missing) {
        auto prereq = FindActionAdding(missing.FirstBit());
        actionQueue.push_front(prereq);
        missing &= ~prereq->adds;
    }
    actionQueue.push_back(a);
}
```

**See:** Chapter 11.5, Chapter 3.3.2

---

### Pattern 11: Line of Sight Pattern
**One-line:** Determine entity visibility considering terrain elevation, obstacles, and stance-based visibility modifiers.

**When to use:**
- Tactical games with concealment
- Elevation matters tactically
- Fog of war implementation
- Cover system integration

**Code Skeleton (10 lines):**
```cpp
bool IsVisible(Point from, Point to, World* w) {
    float opacity = 0;
    for (Point p : BresenhamLine(from, to)) {
        if (Distance(from, p) > 6)  // Near exemption
            opacity += w->GetTerrain(p).opacity;
        if (opacity >= 0.5) return false;
    }
    return true;
}
```

**See:** Chapter 11.6, Chapter 14.3.2

---

### Pattern 12: Threat Assessment Pattern
**One-line:** Enable units to autonomously detect and respond to threats, making intelligent decisions about engagement and cover.

**When to use:**
- AI autonomy desired
- Reduced micromanagement needed
- Self-preservation instincts wanted
- Emergent tactical behaviors

**Code Skeleton (10 lines):**
```cpp
Behavior EvaluateThreats(Soldier* s, World* w) {
    if (auto* enemy = FindVisibleEnemy(s, w))
        return Behavior::EngageSoldier(enemy);
    if (s->underFire == DANGER)
        if (auto* cover = FindCover(s, w))
            return Behavior::Hide(cover->direction);
    return Behavior::None;
}
```

**See:** Chapter 11.6, Chapter 14.3

---

### Pattern 13: Cover Evaluation Pattern
**One-line:** Enable units to identify, evaluate, and utilize protective terrain features that reduce incoming damage.

**When to use:**
- All tactical games
- Terrain importance in gameplay
- Realistic combat flow
- AI competent play required

**Code Skeleton (10 lines):**
```cpp
struct CoverEval {
    float protection, distance, exposure;
    float Score() { return protection / (distance * exposure); }
};
Cover* FindBestCover(Soldier* s, World* w) {
    auto covers = QueryCoverInRadius(s->pos, 50);
    return MaxBy(covers, [](Cover* c) { return c->Score(); });
}
```

**See:** Chapter 11.6, Chapter 14.5.2

---

### Pattern 14: Spatial Partitioning Pattern
**One-line:** Efficiently organize entities in game world space for fast queries without checking every entity.

**When to use:**
- Entity queries needed (nearby, LOS, etc.)
- Scalable to large worlds
- AI and combat systems
- Performance optimization required

**Code Skeleton (10 lines):**
```cpp
class SpatialHash {
    map<uint64, vector<Entity*>> cells;
    uint64 Hash(vec2 p) { 
        return ((uint64)(p.x/CELL_SIZE) << 32) | (uint32)(p.y/CELL_SIZE); 
    }
    void Insert(Entity* e) { cells[Hash(e->pos)].push_back(e); }
    vector<Entity*> QueryRadius(vec2 center, float r) { /* check neighbor cells */ }
};
```

**See:** Chapter 11.7

---

### Pattern 15: Message Bus Pattern
**One-line:** Enable decoupled communication between game systems through a centralized message queue.

**When to use:**
- Complex games with many interacting systems
- Replay recording needed
- Network synchronization required
- Decoupled systems desired

**Code Skeleton (10 lines):**
```cpp
enum MsgType { WeaponFired, UnitDamaged, UnitDestroyed };
class MessageBus {
    map<MsgType, vector<Handler*>> listeners;
    void Subscribe(MsgType t, Handler* h) { listeners[t].push_back(h); }
    void Publish(Message m) { for (auto* h : listeners[m.type]) h->OnMessage(m); }
};
```

**See:** Chapter 11.7

---

### Pattern 16: Deterministic Simulation Pattern
**One-line:** Ensure identical initial state and inputs produce identical results—essential for multiplayer and replays.

**When to use:**
- Multiplayer synchronization required
- Replay recording needed
- Verification/testing important
- Competitive gameplay

**Code Skeleton (10 lines):**
```rust
const DT: f32 = 1.0 / 60.0;  // Fixed timestep
struct Sim { 
    rng: SeededRng,  // Seeded RNG
    frame: u64,
    fn Tick(&mut self, inputs: &[Input]) {
        for i in inputs { self.Apply(i); }
        self.frame += 1;
    }
}
```

**See:** Chapter 11.7, Chapter 15.2

---

### Pattern 17: Hot Reload Pattern
**One-line:** Enable modification of game data and scripts at runtime without restarting, improving iteration speed.

**When to use:**
- Rapid iteration needed
- Modding support desired
- Immediate feedback wanted
- Development velocity important

**Code Skeleton (10 lines):**
```cpp
class HotReload {
    map<string, time_t> lastModified;
    void CheckFiles() {
        for (auto& [file, last] : lastModified) {
            if (GetModTime(file) > last) {
                Reload(file);
                lastModified[file] = GetModTime(file);
            }
        }
    }
};
```

**See:** Chapter 11.7

---

## 2. Decision Matrix Summary

### State Management Decision Matrix

| Criteria | Bitfield | Hierarchy | Dual-State |
|----------|----------|-----------|------------|
| **Multiplayer** | Good | Excellent | Challenging |
| **Modding** | Limited | Moderate | Excellent |
| **Complex Combinations** | Excellent | Good | Poor |
| **Team Experience** | Low | High | Low |
| **Debugging** | Moderate | Good | Excellent |
| **Determinism** | Excellent | Excellent | Moderate |

**Quick Decision Tree:**
```
Need deterministic multiplayer?
├── YES → Need complex state combinations?
│         ├── YES → Bitfield
│         └── NO  → State Hierarchy
└── NO  → Is modding priority?
          ├── YES → Dual-State
          └── NO  → How complex are states?
                    ├── Simple → Dual-State
                    └── Complex → Bitfield
```

**Recommended Hybrid (2026+):**
- **Hierarchy** for timescale separation (Phase → Behavior → Gesture)
- **Bitfield** for orthogonal capabilities (CanMove, CanFire, IsProne)

---

### Unit Model Decision Matrix

| Aspect | OOP Inheritance | Component Composition | Modified ECS |
|--------|-----------------|----------------------|--------------|
| **Performance** | Poor | Good | Excellent |
| **Flexibility** | Poor | Excellent | Good |
| **Type Safety** | Runtime | Runtime | Compile-time |
| **Cache Efficiency** | Poor | Good | Excellent |
| **Modding** | Difficult | Excellent | Moderate |
| **Learning Curve** | Low | Medium | High |

**Recommendation by Project Type:**
- **First game/Rapid prototype:** OOP Inheritance (familiar)
- **Commercial multiplayer:** Modified ECS (deterministic, type-safe)
- **Modding platform:** Component Composition (runtime flexibility)

---

### AI Architecture Decision Matrix

| Aspect | Behavior Trees | GOAP | Utility AI | FSM |
|--------|---------------|------|-----------|-----|
| **Complexity** | Medium | High | Medium | Low |
| **Planning** | Reactive | Proactive | Reactive | Reactive |
| **Emergence** | Limited | High | Medium | Low |
| **Debugging** | Easy | Hard | Moderate | Easy |
| **Performance** | O(depth) | O(actions^depth) | O(actions) | O(1) |
| **Best For** | Individual AI | Squad tactics | Target selection | Simple states |

**Recommended Hybrid:**
- **Behavior Trees** for individual soldier reactive behaviors
- **GOAP** for squad-level tactical planning
- **Utility AI** for target selection and position evaluation

---

### Multiplayer Architecture Decision Matrix

| Aspect | Deterministic Lockstep | State Sync | Event Replication |
|--------|------------------------|-----------|-------------------|
| **Bandwidth** | Very Low | High | Medium |
| **Latency Tolerance** | Poor | Good | Good |
| **Replay** | Perfect | None | Partial |
| **Cheat Resistance** | Excellent | Good | Moderate |
| **Implementation** | Complex | Simple | Medium |
| **Use Case** | Competitive | Casual PvP | Cooperative |

**Recommendation:**
- **Competitive tactical games:** Deterministic simulation + message-based sync
- **Cooperative PvE:** Event replication acceptable
- **Rapid prototyping:** State sync for quick iteration

---

## 3. Common Implementation Patterns

### State Machine Skeleton (20 lines)

```cpp
enum class State { Idle, Moving, Attacking, Dead };

class Unit {
    State currentState = State::Idle;
    map<State, map<State, function<void()>>> transitions;
    
public:
    bool CanTransition(State newState) {
        return transitions[currentState].count(newState) > 0;
    }
    
    void ChangeState(State newState) {
        if (!CanTransition(newState)) return;
        OnExit(currentState);
        currentState = newState;
        OnEnter(currentState);
    }
    
    void Update(float dt) {
        switch (currentState) {
            case State::Idle: UpdateIdle(dt); break;
            case State::Moving: UpdateMoving(dt); break;
            case State::Attacking: UpdateAttacking(dt); break;
        }
    }
};
```

---

### Component Composition Example (20 lines)

```cpp
struct Entity { map<type_index, shared_ptr<Component>> components; };
struct Transform : Component { vec2 pos; float rot; };
struct Health : Component { int current, max; };
struct Weapon : Component { string type; int ammo; };

template<typename T>
T* GetComponent(Entity* e) {
    auto it = e->components.find(typeid(T));
    return (it != e->components.end()) ? static_cast<T*>(it->second.get()) : nullptr;
}

// Usage
Entity* soldier = new Entity();
soldier->components[typeid(Transform)] = make_shared<Transform>();
soldier->components[typeid(Health)] = make_shared<Health>();

auto* health = GetComponent<Health>(soldier);
if (health) health->current -= 10;
```

---

### Order Queue Implementation (20 lines)

```cpp
struct Order { 
    enum Type { Move, Attack, Defend } type;
    vec2 targetPos;
    bool completed = false;
};

class Unit {
    deque<Order> orders;
    Order* current = nullptr;
    
public:
    void IssueOrder(Order o) { orders.push_back(o); }
    void CancelOrders() { orders.clear(); current = nullptr; }
    
    void Update(float dt) {
        if (!current && !orders.empty()) {
            current = &orders.front();
            StartOrder(current);
        }
        if (current && UpdateOrder(current, dt)) {
            current->completed = true;
            orders.pop_front();
            current = orders.empty() ? nullptr : &orders.front();
        }
    }
};
```

---

### Behavior Tree Node (20 lines)

```cpp
enum class Status { Success, Failure, Running };

class BTNode {
public:
    virtual Status Tick(Entity* e, World* w) = 0;
    virtual void Reset() {}
};

class Selector : public BTNode {
    vector<unique_ptr<BTNode>> children;
    size_t current = 0;
public:
    Status Tick(Entity* e, World* w) override {
        while (current < children.size()) {
            auto status = children[current]->Tick(e, w);
            if (status == Status::Success) { current = 0; return Status::Success; }
            if (status == Status::Running) return Status::Running;
            current++;
        }
        current = 0; return Status::Failure;
    }
};

class Action : public BTNode {
    function<Status(Entity*, World*)> action;
public:
    Status Tick(Entity* e, World* w) override { return action(e, w); }
};
```

---

### Spatial Hash Grid (20 lines)

```cpp
class SpatialHash {
    static constexpr float CELL_SIZE = 100.0f;
    unordered_map<uint64_t, vector<Entity*>> cells;
    
    uint64_t Hash(vec2 pos) {
        int x = (int)(pos.x / CELL_SIZE);
        int y = (int)(pos.y / CELL_SIZE);
        return ((uint64_t)(x) << 32) | (uint32_t)(y);
    }
    
public:
    void Insert(Entity* e) { cells[Hash(e->pos)].push_back(e); }
    void Remove(Entity* e) { /* find and erase */ }
    
    vector<Entity*> QueryRadius(vec2 center, float radius) {
        vector<Entity*> results;
        int r = (int)(radius / CELL_SIZE) + 1;
        for (int dy = -r; dy <= r; dy++)
            for (int dx = -r; dx <= r; dx++)
                for (Entity* e : cells[Hash(center + vec2(dx, dy) * CELL_SIZE)])
                    if (Length(e->pos - center) <= radius)
                        results.push_back(e);
        return results;
    }
};
```

---

## 4. Checklists

### Pre-Development Architecture Checklist

**Game Design Questions:**
- [ ] Single-player, multiplayer, or both?
- [ ] Competitive multiplayer (requires determinism)?
- [ ] Modding support needed?
- [ ] Target audience (hardcore vs casual)?
- [ ] Scale (max units on screen)?

**Technical Questions:**
- [ ] Target platforms (PC, mobile, console)?
- [ ] Team size and expertise?
- [ ] Timeline (rapid prototype vs long-term)?
- [ ] Performance constraints?
- [ ] Budget (engine licensing)?

**Architecture Decisions:**
- [ ] Language chosen (Rust/C++/C#)?
- [ ] State management pattern selected?
- [ ] Entity model decided (OOP/ECS/Composition)?
- [ ] Modding approach (data/scripting)?
- [ ] Networking model (if applicable)?

---

### Implementation Phase Checklist

**Phase 1: Foundation (Months 1-3)**
- [ ] Basic rendering working
- [ ] Entity system with type-safe indices
- [ ] State management implemented
- [ ] Simple terrain loaded
- [ ] One unit moves on screen

**Phase 2: Core Gameplay (Months 4-6)**
- [ ] Order system with prerequisites
- [ ] Combat (shooting, damage)
- [ ] Terrain effects (cover, LOS)
- [ ] Basic AI (reactive behaviors)
- [ ] Squad command hierarchy

**Phase 3: Content Systems (Months 7-9)**
- [ ] Data-driven unit definitions
- [ ] JSON scenario system
- [ ] Lua scripting support
- [ ] Mod loading system
- [ ] Hot reload implementation

**Phase 4: Polish (Months 10-12)**
- [ ] UI/UX complete
- [ ] Audio integrated
- [ ] Visual effects
- [ ] Performance optimized
- [ ] Target unit count achieved

---

### Testing Checklist

**Unit Tests (Required):**
- [ ] State orthogonal combination works
- [ ] Prerequisite chain automatic
- [ ] Determinism verified (same seed = same result)
- [ ] Entity reference type safety
- [ ] Spatial query accuracy

**Integration Tests:**
- [ ] Full scenario playthrough
- [ ] AI behavior validation
- [ ] Mod loading/hot reload
- [ ] Save/load roundtrip
- [ ] Network sync (if multiplayer)

**Performance Tests:**
- [ ] 100 units at 60 FPS
- [ ] 500 units at 30 FPS
- [ ] Spatial query < 1ms
- [ ] Pathfinding < 10ms
- [ ] Memory usage acceptable

---

### Release/Modding Checklist

**Documentation:**
- [ ] API documentation complete
- [ ] Modding guide written
- [ ] Example mods provided
- [ ] Troubleshooting guide
- [ ] Video tutorials (optional)

**Modding Support:**
- [ ] Mod manager implemented
- [ ] Dependency resolution
- [ ] Conflict detection
- [ ] Load order configuration
- [ ] Hot reload stable

**Distribution:**
- [ ] Installer/package created
- [ ] Update mechanism
- [ ] Crash reporting
- [ ] Analytics (optional)
- [ ] Community forums/Discord

---

## 5. Anti-Patterns Quick Reference

### 1. The God Object
**The Mistake:** Creating a massive class with 50+ fields.
```cpp
// WRONG
class Unit {
    // 50+ fields: position, health, ammo, morale, 
    // experience, weapon1, weapon2, squad, orders...
};
```
**Why it's bad:** Violates single responsibility, hard to test, inflexible.

**How to fix:** Use composition (Pattern 3).
```cpp
// RIGHT
struct Transform { vec2 pos; };
struct Health { int hp; };
struct Weapon { string type; };
struct Unit {
    Transform* transform;
    Health* health;
    Weapon* weapon;
};
```
**See:** Chapter 9.3.1, Chapter 12.3.2

---

### 2. Implicit State Scattered Through Code
**The Mistake:** State checks spread across code.
```cpp
// WRONG
if (soldier->health > 0 && soldier->ammo > 0 && !soldier->suppressed) {
    soldier->Fire();
}
```
**Why it's bad:** Can't query "what can this unit do?", fragile, inconsistent.

**How to fix:** Use explicit capability tracking.
```cpp
// RIGHT
if (soldier->HasCapability(Capability::CanFire)) {
    soldier->Fire();
}
```
**See:** Chapter 9.8.5, Chapter 12.2.1

---

### 3. Mixing Simulation and Rendering
**The Mistake:** Rendering during simulation update.
```cpp
// WRONG
void Soldier::Update() {
    position += velocity * dt;
    Draw(sprite, position);  // NO!
}
```
**Why it's bad:** Ties simulation to frame rate, impossible to test, can't run headless.

**How to fix:** Strict separation.
```cpp
// RIGHT
void Simulation::Update() { /* update positions */ }
void Renderer::Draw() { /* render all */ }
```
**See:** Chapter 9.2.2, Chapter 9.8.4

---

### 4. Hardcoded Behaviors
**The Mistake:** Behavior logic in C++ classes.
```cpp
// WRONG
class Sniper : public Soldier {
    void SelectTarget() override { /* hardcoded logic */ }
};
```
**Why it's bad:** Can't mod, requires recompile for changes, rigid.

**How to fix:** Scripting or behavior trees.
```cpp
// RIGHT
soldier->behavior = LoadScript("ai/sniper.lua");
// OR
soldier->behaviorTree = LoadFromYAML("behaviors/sniper.yaml");
```
**See:** Chapter 9.3.2, Chapter 10.2.4

---

### 5. Pointer Chasing for Entity References
**The Mistake:** Using raw pointers for entity relationships.
```cpp
// WRONG
class Squad {
    vector<Soldier*> members;  // Dangling pointer risk
    Soldier* leader;           // Ownership unclear
};
```
**Why it's bad:** Use-after-free, null pointer bugs, serialization nightmares.

**How to fix:** Type-safe indices.
```cpp
// RIGHT
struct SoldierIndex { size_t value; };
struct Squad {
    vector<SoldierIndex> members;
    SoldierIndex leader;
};
```
**See:** Chapter 5.2.3, Chapter 10.3.1

---

### 6. Direct State Mutation
**The Mistake:** Modifying state directly.
```cpp
// WRONG
soldier->health -= damage;  // Who changed this? When?
```
**Why it's bad:** Can't debug, no replay, no networking, no determinism.

**How to fix:** Message-driven updates.
```cpp
// RIGHT
bus.Publish(SoldierDamaged{soldierIdx, 10, attacker});
// Later: battleState.Apply(message);
```
**See:** Chapter 10.3.2, Chapter 15.2.1

---

### 7. Deep Inheritance Hierarchies
**The Mistake:** Multiple inheritance levels.
```cpp
// WRONG
class Object {};
class Soldier : public Object {};
class Vehicle : public Object {};
class Tank : public Vehicle {};
class HeavyTank : public Tank {};
```
**Why it's bad:** Fragile, diamond problem, hard to modify.

**How to fix:** Composition over inheritance.
```cpp
// RIGHT
struct Entity { vector<Component*> components; };
struct Armor { int value; };
struct Turret { float rotationSpeed; };
// Tank = Entity + Armor + Turret + Tracks
```
**See:** Chapter 5.3, Chapter 8.3.2

---

### 8. No Determinism in Multiplayer
**The Mistake:** Using floating-point math, unseeded RNG, frame-rate dependent logic.
```cpp
// WRONG
float damage = baseDamage * random();  // Non-deterministic!
soldier->pos += velocity * dt;         // FP precision varies
```
**Why it's bad:** Desyncs, can't replay, unfair multiplayer.

**How to fix:** Deterministic patterns.
```cpp
// RIGHT
float damage = baseDamage * rng.NextFloat();  // Seeded RNG
// Fixed timestep, no real-time dependencies
```
**See:** Chapter 15.1.1, Chapter 15.2

---

### 9. Naive O(N²) Queries
**The Mistake:** Checking every entity against every other.
```cpp
// WRONG
for (auto* a : allUnits)
    for (auto* b : allUnits)
        if (CanSee(a, b)) /* ... */;
// 500 units = 250,000 checks per frame!
```
**Why it's bad:** Performance death at scale.

**How to fix:** Spatial partitioning.
```cpp
// RIGHT
auto nearby = spatialHash.QueryRadius(unit->pos, viewDistance);
for (auto* other : nearby)
    if (CanSee(unit, other)) /* ... */;
```
**See:** Chapter 11.7, Chapter 10.3.3

---

### 10. Retrofitting Multiplayer
**The Mistake:** Adding multiplayer late without deterministic architecture.
**Why it's bad:** Requires complete architectural rewrite, often impossible.

**How to fix:** Plan for multiplayer from day one.
- Message-driven state updates
- Server-authoritative design
- Deterministic simulation
- No direct state mutation

**See:** Chapter 15.1, Chapter 9.8.2

---

## 6. File Format Quick Reference

### XML (OpenCombat-SDL) Structure

**Root Pattern:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<RootElement>
    <Child>
        <Field>Value</Field>
        <NumericField>123</NumericField>
    </Child>
</RootElement>
```

**Common Elements:**
| Element | Description | Example |
|---------|-------------|---------|
| `<Name>` | Unique identifier | `<Name>M1 Garand</Name>` |
| `<Attributes>` | Capability flags | `<CanMove/><CanFire/>` |
| `<States>` | State definitions | `<State><Name>Standing</Name></State>` |
| `<Graphic>` | Sprite reference | `path.width.height.tga` |

**Soldier Template:**
```xml
<Soldier>
    <Name>Rifleman</Name>
    <PrimaryWeapon>M1 Garand</PrimaryWeapon>
    <States>
        <State><Name>Standing</Name><Animation>Standing Rest</Animation></State>
    </States>
    <Attributes>
        <WalkingSpeed>2.5</WalkingSpeed>
        <CanMove/><CanFire/>
    </Attributes>
</Soldier>
```

---

### JSON (OpenCombat) Structure

**Root Pattern:**
```json
{
    "key": "value",
    "number": 123,
    "array": [1, 2, 3],
    "object": { "nested": "value" }
}
```

**Deployment File:**
```json
{
    "soldiers": [
        {
            "uuid": 1,
            "side": "A",
            "world_point": {"x": 100.5, "y": 200.3},
            "main_weapon": {"MosinNagant": 5},
            "behavior": {"Idle": "StandUp"}
        }
    ]
}
```

**Weapon Configuration:**
```json
{
    "MosinNagantM1924": [
        false,
        {"MosinNagant": 5}
    ]
}
```

---

### QML (CloseCombatFree) Structure

**Root Pattern:**
```qml
import QtQuick 2.1
import "../path"

Item {
    property type name: value
    
    ChildItem {
        property: value
    }
}
```

**Scenario File:**
```qml
import QtQuick 2.1
import "../units"

Item {
    property string mapFile: "maps/Map.qml"
    
    Tank {
        objectName: "tank1"
        x: 150
        y: 400
        rotation: 90
    }
}
```

**Unit Definition:**
```qml
Tank {
    id: root
    unitType: "Heavy Tank"
    maxSpeed: 15
    
    Hull { id: hull }
    Turret { id: turret }
    
    Soldier { role: "Commander" }
    Soldier { role: "Gunner" }
}
```

---

### TMX Map Format

**Structure:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<map version="1.10" orientation="orthogonal" 
     width="112" height="64" tilewidth="5" tileheight="5">
    
    <tileset firstgid="1" source="terrain.tsx"/>
    
    <imagelayer name="background_image">
        <image source="map.png" width="560" height="320"/>
    </imagelayer>
    
    <layer name="terrain" width="112" height="64">
        <data encoding="csv">1,1,1,4,4,4,...</data>
    </layer>
    
    <objectgroup name="spawn_zones">
        <object name="NW" x="0" y="0" width="148" height="109"/>
    </objectgroup>
</map>
```

**Key Attributes:**
| Attribute | Description |
|-----------|-------------|
| `firstgid` | First global tile ID |
| `orientation` | orthogonal/isometric |
| `tilewidth/height` | Tile dimensions in pixels |
| `encoding="csv"` | Comma-separated tile IDs |

---

### Format Comparison Summary

| Feature | XML | JSON | QML |
|---------|-----|------|-----|
| **Readability** | Moderate | High | High |
| **Verbosity** | High | Low | Moderate |
| **Tooling** | Generic editors | Excellent | Qt Creator |
| **Schema** | XSD/DTD | JSON Schema | QML types |
| **Scripting** | None | None | JavaScript |
| **Hot Reload** | No | No | Yes |
| **Parsing** | Complex | Simple | Qt engine |

---

### Recommended Format by Use Case

| Use Case | Format | Rationale |
|----------|--------|-----------|
| Unit Definitions | JSON/YAML | Schema-validatable, tool-friendly |
| Maps | TMX | Industry standard, Tiled editor |
| Scenarios | JSON | Consistent, easy generation |
| Configuration | TOML | Clean syntax, user-friendly |
| Save Games | Binary (MessagePack) | Performance, compact |
| Mods | JSON + Lua | Data + scripting |

---

## Quick Reference Index

**Patterns by Category:**
- **Entity:** Soldier/Squad Aggregate (1), Weapon Team (2), Component Composition (3)
- **State:** State Hierarchy (4), Order Queue (5), Stance System (6), Morale Cascade (7)
- **Command:** Command Abstraction (8), Formation Control (9), Prerequisite Chain (10)
- **Perception:** Line of Sight (11), Threat Assessment (12), Cover Evaluation (13)
- **System:** Spatial Partitioning (14), Message Bus (15), Deterministic Simulation (16), Hot Reload (17)

**Decision Matrix Quick Pick:**
- **Multiplayer →** State Hierarchy + Modified ECS + Deterministic Simulation
- **Single-player →** Bitfield + OOP + Automatic Prerequisite Chain
- **Modding →** Dual-State + Component Composition + Hot Reload

---

*This quick reference was synthesized from Chapters 3, 5, 6, 8, 9, 10, 11, 12, 14, 15 and Appendix C of the Close Combat Clones Comparative Architecture Book.*

**Version:** 1.0  
**Date:** February 2026  
**Pages:** ~25 (when printed)
