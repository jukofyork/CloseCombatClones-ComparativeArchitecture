# Chapter 14: AI Systems in Tactical Wargames

## 14.1 The AI Landscape in Tactical Wargames

### 14.1.1 Introduction: Why AI Matters

Artificial Intelligence is the invisible hand that shapes player experience in tactical wargames. Unlike first-person shooters where AI opponents simply need to appear intelligent for brief encounters, tactical wargames demand AI that maintains believable behavior over extended engagements spanning minutes or hours of real-time gameplay. The challenge is not merely creating challenging opponents, but opponents that feel authentic—units that respond to threats with appropriate caution, seize opportunities with tactical cunning, and occasionally make mistakes that feel human rather than systematic.

The three Close Combat clones examined in this book represent different points along the AI sophistication spectrum:

| Project | AI Approach | Autonomy Level | Key Innovation |
|---------|-------------|----------------|----------------|
| **OpenCombat-SDL** | No autonomous AI | Pure obedience | Automatic prerequisite handling |
| **OpenCombat** | Reactive self-preservation | Tactical initiative | "Feeling" system for emotional states |
| **CloseCombatFree** | State-based triggers | Minimal autonomy | Visual feedback integration |

### 14.1.2 The Four AI Types in Tactical Games

Tactical wargames require multiple distinct AI systems operating at different scales and timescales:

#### **System AI (Milliseconds)**
Manages the lowest-level behaviors:
- Animation state machines
- Physics interactions
- Collision responses
- Weapon cycling and reloading

#### **Unit AI (Seconds)**
Controls individual soldiers or vehicles:
- Target selection and engagement
- Cover seeking and position evaluation
- Threat response (return fire, suppression)
- Self-preservation behaviors

#### **Tactical AI (Minutes)**
Coordinates small groups (squads, fire teams):
- Formation maintenance
- Fire and maneuver tactics
- Suppression and flanking
- Casualty response and reorganization

#### **Strategic AI (Hours)**
Manages high-level battle planning:
- Objective prioritization
- Resource allocation
- Reinforcement decisions
- Retreat and regroup triggers

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    AI HIERARCHY IN TACTICAL WARGAMES                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  STRATEGIC AI (Hours)                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Objective prioritization                                      │   │
│  │ • Resource allocation                                           │   │
│  │ • Reinforcement decisions                                       │   │
│  │ • Retreat triggers                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  TACTICAL AI (Minutes)                                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Squad coordination                                            │   │
│  │ • Formation selection                                           │   │
│  │ • Fire and maneuver                                             │   │
│  │ • Flanking decisions                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  UNIT AI (Seconds)                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Target selection                                                │   │
│  │ • Cover evaluation                                                │   │
│  │ • Threat response                                                 │   │
│  │ • Self-preservation                                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  SYSTEM AI (Milliseconds)                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Animation states                                                │   │
│  │ • Weapon cycling                                                  │   │
│  │ • Collision response                                              │   │
│  │ • Physics integration                                             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 14.1.3 The Four Challenges of Tactical AI

Creating effective tactical AI requires balancing four competing requirements:

#### **Believability**
The AI must behave in ways that align with player expectations of military tactics:
- Units should use cover appropriately
- Suppression should affect behavior
- Soldiers should show self-preservation instincts
- Tactical mistakes should feel human, not robotic

#### **Fairness**
The AI must not cheat in ways that frustrate players:
- Information access should be limited (fog of war)
- Accuracy should follow same rules as player units
- Reaction times should be realistic
- Resources should be constrained

#### **Performance**
AI calculations must not bog down real-time simulation:
- Pathfinding must complete within frame budgets
- Target evaluation should use efficient algorithms
- Decision trees should minimize branching
- Spatial queries need acceleration structures

#### **Determinism**
For multiplayer and replay systems, AI must be predictable:
- Same inputs must produce same outputs
- Random decisions must use seeded generators
- Timing must be frame-rate independent
- Floating-point operations must be consistent

```
┌─────────────────────────────────────────────────────────────────────────┐
│              THE FOUR CONSTRAINTS OF TACTICAL AI                       │
│                                                                         │
│                    ┌──────────────┐                                     │
│                    │  BELIEVABLE  │                                     │
│                    │    AI        │                                     │
│                    └──────┬───────┘                                     │
│                           │                                             │
│        ┌──────────────────┼──────────────────┐                         │
│        │                  │                  │                         │
│   ┌────┴────┐        ┌────┴────┐        ┌────┴────┐                    │
│   │  FAIR   │←──────→│PERFORM │←──────→│DETERMIN│                    │
│   │         │        │   ANT   │        │  ISTIC   │                    │
│   └─────────┘        └─────────┘        └─────────┘                    │
│                                                                         │
│  Believable: Uses cover, shows fear, makes human-like mistakes         │
│  Fair: Same information access, same accuracy rules                    │
│  Performant: Completes decisions within frame budget                   │
│  Deterministic: Same seed → same behavior (for multiplayer)            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 14.2 The Three AI Architectures Compared

### 14.2.1 OpenCombat-SDL: Rule-Based with State Machines

OpenCombat-SDL adopts the simplest AI philosophy: **no autonomous AI at all**. Every action requires explicit player orders.

```cpp
void Soldier::Simulate(long dt, World* world) {
    if (!_orders.empty()) {
        ProcessOrder();
    } else {
        // Idle—no autonomous behavior
        // Soldier simply waits for next command
    }
}
```

**Design Philosophy**: The player controls everything. Soldiers are extensions of player will.

**Architecture Characteristics**:
- **State Representation**: 64-bit bitfield allowing orthogonal state composition
- **Decision Making**: Explicit player commands only
- **Prerequisite Handling**: Automatic action chaining (e.g., prone → stand → reload → fire)
- **Visual Feedback**: Action markers show current execution

**Advantages**:
- Predictable behavior
- Player has full control
- No "AI did something stupid" frustration
- Easy to debug and balance

**Disadvantages**:
- Heavy micromanagement burden
- Unrealistic (real soldiers take initiative)
- Doesn't scale to large unit counts
- No emergent gameplay from AI interactions

### 14.2.2 CloseCombatFree: Behavioral State Triggers

CloseCombatFree implements minimal AI through state-based triggers rather than autonomous decision-making.

```cpp
// States exist but minimal logic
if (unitStatus == "DEFENDING") {
    // Will fire at enemies in range
    // No complex decision making
}

if (unitStatus == "AMBUSHING") {
    // Will fire when enemy enters detection radius
    // No flanking or repositioning
}
```

**Design Philosophy**: Simple state triggers provide basic automation while maintaining player control.

**Architecture Characteristics**:
- **State Representation**: String-based unitStatus property
- **Decision Making**: State-based triggers (Defend, Ambush)
- **Prerequisite Handling**: Animation-driven completion
- **Visual Feedback**: Order markers with color coding

**Available States**:
| State | AI Behavior | Trigger |
|-------|-------------|---------|
| READY | None | Default |
| DEFENDING | Fire at enemies in arc | Player command |
| AMBUSHING | Fire when enemy detected | Player command |
| MOVING | Follow path | Player order |
| FIRING | Execute attack | Player order |

**Limitations**:
- No target selection AI
- No cover seeking
- No suppression response
- No threat assessment

### 14.2.3 OpenCombat: Reactive Perception + Proactive Planning

OpenCombat implements the most sophisticated AI of the three clones, featuring a "Feeling" system that drives emotional states and reactive behaviors.

```rust
fn tick_soldiers(&mut self) {
    for soldier_idx in self.all_soldiers() {
        let soldier = self.soldier(soldier_idx);
        
        // Check for threats
        if let Some(threat) = self.find_nearest_enemy(soldier) {
            if self.can_see(soldier_idx, threat) {
                // Override to engage
                self.change_behavior(
                    soldier_idx, 
                    Behavior::EngageSoldier(threat)
                );
            }
        }
        
        // Check if under fire
        if soldier.under_fire == Feeling::Danger {
            // Find cover
            if let Some(cover_pos) = self.find_cover(soldier) {
                self.change_behavior(
                    soldier_idx,
                    Behavior::Hide(cover_pos.direction)
                );
            }
        }
    }
}
```

**Design Philosophy**: Soldiers have self-preservation instincts but remain loyal to player intent when not threatened.

**Architecture Characteristics**:
- **State Representation**: Three-tier hierarchy (Phase → Behavior → Gesture)
- **Decision Making**: Threat evaluation with behavior overrides
- **Prerequisite Handling**: Gesture timing system
- **Emotional Model**: Feeling enum with intensity levels

**The Feeling System**:
```rust
enum Feeling {
    UnderFire(intensity: u32),  // 0-200 scale
}

const UNDER_FIRE_TICK: u32 = 10;       // Decay per frame
const UNDER_FIRE_DANGER: u32 = 150;    // Threshold for danger response
const UNDER_FIRE_WARNING: u32 = 100;   // Threshold for caution

fn update_feeling(soldier: &mut Soldier, world: &World) {
    // Increase from nearby explosions
    for explosion in &world.recent_explosions {
        let distance = distance(soldier.position, explosion.position);
        if distance < 5.0 {
            soldier.under_fire.increase(150);
        } else if distance < 10.0 {
            soldier.under_fire.increase(100);
        } else {
            soldier.under_fire.increase(50);
        }
    }
    
    // Increase from bullet impacts
    for bullet in &world.recent_bullets {
        let distance = distance(soldier.position, bullet.impact);
        if distance < 3.0 {
            soldier.under_fire.increase(100);
        } else if distance < 10.0 {
            soldier.under_fire.increase(35);
        } else {
            soldier.under_fire.increase(1);
        }
    }
    
    // Natural decay
    soldier.under_fire.decrease(UNDER_FIRE_TICK);
}
```

**Behavior Hierarchy**:
```
┌─────────────────────────────────────────────────────────────────────────┐
│                    OPENCOMBAT THREE-TIER SYSTEM                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PHASE (Global)                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Placement    - Pre-battle setup                                 │   │
│  │ • Battle       - Active combat                                  │   │
│  │ • End          - Post-battle resolution                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  BEHAVIOR (Tactical - Seconds)                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ Movement:    MoveTo, MoveFastTo, SneakTo, DriveTo               │   │
│  │ Combat:      Idle, Defend, Hide, SuppressFire                  │   │
│  │ Engagement:  EngageSoldier                                      │   │
│  │ Terminal:    Dead, Unconscious                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  GESTURE (Physical - Milliseconds)                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Idle                                                          │   │
│  │ • Reloading(duration, weapon_class)                               │   │
│  │ • Aiming(duration, weapon_class)                                │   │
│  │ • Firing(duration, weapon_class)                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**AI Behaviors**:

| Situation | Response | Priority |
|-----------|----------|----------|
| Under heavy fire | Seek cover or return fire | 100 (Survival) |
| Enemy spotted | Engage if in range | 50 (Combat) |
| Squad leader killed | Promote new leader | 75 (Cohesion) |
| Out of ammo | Reload (blocking) | 90 (Readiness) |
| Wounded | Continue or retreat (morale check) | 80 (Self-preservation) |

### 14.2.4 Comparative Summary

| Aspect | OpenCombat-SDL | CloseCombatFree | OpenCombat |
|--------|---------------|-----------------|------------|
| **AI Philosophy** | Pure obedience | State triggers | Reactive + proactive |
| **Autonomy Level** | None | Minimal | Tactical |
| **Decision System** | Player commands only | State-based | Threat evaluation |
| **Self-Preservation** | No | No | Yes (Feeling system) |
| **Cover Seeking** | Manual | No | Automatic |
| **Target Selection** | Manual | Basic range | Dynamic evaluation |
| **Implementation** | C++ bitfield | C++ strings | Rust enums |
| **Complexity** | Low | Very Low | Medium |

---

## 14.3 Perception Systems

### 14.3.1 What Can AI Perceive?

In tactical wargames, AI perception defines the boundary between fair challenge and frustrating omniscience. The perception system determines what information AI-controlled units can access about the game world.

#### **Visual Perception**
What the AI can "see":
- Enemy units within line of sight
- Terrain features and obstacles
- Weapon effects (muzzle flashes, tracers)
- Movement and posture changes

#### **Auditory Perception**
What the AI can "hear":
- Gunfire and explosions (direction and distance)
- Vehicle engines
- Footsteps (close range)
- Shouted orders and radio chatter

#### **Tactile/Environmental Perception**
What the AI can "feel":
- Nearby explosions (concussion)
- Bullet impacts (suppression)
- Ground vibrations (vehicle movement)
- Temperature (fire, smoke)

#### **Knowledge-Based Perception**
What the AI "knows":
- Last known enemy positions
- Squad member locations
- Objective locations
- Terrain hazards

### 14.3.2 Line of Sight (LoS) Calculations

The foundation of visual perception is accurate line of sight calculation. All three clones implement LoS with varying sophistication:

**CloseCombatFree Implementation**:
```cpp
bool CcfEngineHelpers::isObstacleInLOS(QList<QObject *> items, qreal x1, qreal y1,
                                        qreal x2, qreal y2, QObject *currentUnit) {
    qreal distance = targetDistance(x1, y1, x2, y2);
    qreal a = (y2 - y1) / (x2 - x1);  // Slope
    qreal b = y1 - (a * x1);           // Intercept
    
    // Sample points along the line
    for (int i = 0; i < distance; ++i) {
        qreal x = (x2 >= x1) ? x1 + i : x1 - i;
        qreal y = (a * x) + b;
        
        // Check collision with all items
        for (int j = 0; j < items.length(); ++j) {
            QObject *item = items.at(j);
            if (item == currentUnit) continue;
            
            // AABB collision detection
            if (pointInRect(x, y, item)) {
                return true;  // Obstacle found
            }
        }
    }
    return false;
}
```

**Optimization Strategies**:
1. **Spatial Partitioning**: Divide map into grids for faster queries
2. **Ray Marching**: Sample at fixed intervals rather than every pixel
3. **Height Fields**: Use terrain elevation data for over/under checks
4. **Caching**: Store LoS results for static geometry

### 14.3.3 OpenCombat's "Feeling" System

OpenCombat's most innovative AI feature is the Feeling system, which abstracts sensory input into emotional states that drive behavior.

```rust
enum Feeling {
    UnderFire(intensity: u32),  // 0-200 scale
}

struct Soldier {
    under_fire: Feeling,
    last_known_enemies: Vec<EnemyContact>,
    suppression_level: f32,
}
```

**Sensory Input Sources**:

| Source | Effect on Feeling | Range |
|--------|-------------------|-------|
| Explosion < 5m | +150 intensity | Immediate danger |
| Explosion 5-10m | +100 intensity | Nearby threat |
| Explosion > 10m | +50 intensity | Distant threat |
| Bullet impact < 3m | +100 intensity | Direct fire |
| Bullet impact 3-10m | +35 intensity | Close fire |
| Bullet impact > 10m | +1 intensity | Distant fire |

**Behavior Triggers**:
```rust
fn evaluate_threat_response(soldier: &Soldier) -> Option<Behavior> {
    match soldier.under_fire.intensity {
        150..=200 => {
            // Critical danger - seek cover immediately
            find_immediate_cover(soldier)
        }
        100..=149 => {
            // Warning level - proceed with caution
            if soldier.has_good_cover {
                None  // Stay in cover
            } else {
                find_better_cover(soldier)
            }
        }
        _ => None,  // No immediate threat
    }
}
```

### 14.3.4 Information Sharing (Squad Coordination)

Real soldiers communicate threats and observations. AI systems can model this through information sharing mechanisms:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    INFORMATION SHARING MODELS                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  MODEL 1: Perfect Shared Knowledge (Unrealistic)                         │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                            │
│  │Soldier A│←──→│Soldier B│←──→│Soldier C│   All know what others see │
│  └────┬────┘    └────┬────┘    └────┬────┘                            │
│       └────────────────┴───────────────┘                                  │
│                    Shared Memory                                        │
│                                                                         │
│  MODEL 2: Communication Range Limited (Realistic)                       │
│  ┌─────────┐         ┌─────────┐         ┌─────────┐                    │
│  │Soldier A│←───────→│Soldier B│←───────→│Soldier C│   Only share      │
│  │ (Sees)  │         │ (Range) │         │ (Knows) │   within range     │
│  └─────────┘         └─────────┘         └─────────┘                    │
│       │                    │                    │                       │
│       └────────────────────┴────────────────────┘                       │
│                    Within 50m range                                       │
│                                                                         │
│  MODEL 3: Chain of Command (Hierarchical)                                 │
│  ┌─────────┐                                                            │
│  │ Leader  │←── Knows all squad sightings                               │
│  └────┬────┘                                                            │
│       │                                                                 │
│   ┌───┴───┬───┐                                                         │
│   ↓       ↓   ↓                                                         │
│ ┌───┐   ┌───┐ ┌───┐                                                     │
│ │A  │   │B  │ │C  │  Soldiers report to leader                          │
│ └───┘   └───┘ └───┘                                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Implementation Example**:
```rust
fn share_contact_info(squad: &mut Squad, world: &World) {
    let leader = world.soldier_mut(squad.leader);
    
    // Collect all sightings from squad members
    let mut all_contacts: Vec<EnemyContact> = Vec::new();
    
    for member_idx in &squad.members {
        let member = world.soldier(*member_idx);
        for contact in &member.visible_enemies {
            all_contacts.push(contact.clone());
        }
    }
    
    // Distribute shared knowledge to all members
    for member_idx in &squad.members {
        let member = world.soldier_mut(*member_idx);
        for contact in &all_contacts {
            if !member.knows_about(contact) {
                member.add_contact(contact.clone());
            }
        }
    }
}
```

### 14.3.5 Fog of War for AI

To ensure fair play, AI should operate under the same fog of war constraints as human players:

```rust
struct FogOfWar {
    revealed_tiles: HashSet<GridPosition>,
    last_seen: HashMap<EntityId, (Position, Time)>,
    suspicion_level: HashMap<GridPosition, f32>,
}

impl FogOfWar {
    fn can_see(&self, entity: &Entity, target: &Entity) -> bool {
        // Check if target position was ever revealed
        if !self.revealed_tiles.contains(&target.position.to_grid()) {
            return false;
        }
        
        // Check if position is currently visible
        if !has_line_of_sight(entity.position, target.position) {
            // Can only see if we have recent memory
            if let Some((_, time)) = self.last_seen.get(&target.id) {
                if time.elapsed() < Duration::from_secs(30) {
                    return true;  // Memory of recent position
                }
            }
            return false;
        }
        
        true
    }
    
    fn update(&mut self, viewer: &Entity, world: &World) {
        // Reveal tiles within view radius
        for pos in iterate_view_cone(viewer.position, viewer.facing, viewer.view_distance) {
            self.revealed_tiles.insert(pos);
        }
        
        // Update last seen positions for visible enemies
        for enemy in world.visible_enemies(viewer) {
            self.last_seen.insert(enemy.id, (enemy.position, Time::now()));
        }
    }
}
```

**AI Cheating Prevention**:
- AI cannot target enemies it cannot see
- AI uses last known position for area fire only
- AI must scout to reveal unknown areas
- AI accuracy penalties for firing at suspected vs. confirmed targets

---

## 14.4 Decision Making

### 14.4.1 Overview of Decision Architectures

Modern game AI employs several architectural patterns for decision-making, each with distinct advantages for tactical scenarios:

| Architecture | Best For | Complexity | Emergence |
|--------------|----------|------------|-----------|
| **Behavior Trees** | Reactive behaviors, clear structure | Medium | Low |
| **GOAP** | Multi-step planning, complex goals | High | High |
| **Utility AI** | Fuzzy decisions, scoring actions | Medium | Medium |
| **Finite State Machines** | Simple, discrete states | Low | Low |
| **HTN** | Hierarchical task decomposition | High | Medium |

### 14.4.2 Behavior Trees

Behavior Trees organize AI logic as a tree of hierarchical behaviors with standardized execution patterns.

**Node Types**:

```pseudocode
// Selector: Tries children until one succeeds
class Selector extends BTNode
    children: List<BTNode>
    
    function Execute(context)
        for child in children
            result = child.Execute(context)
            if result == Success
                return Success
            else if result == Running
                return Running
        return Failure

// Sequence: Executes children until one fails
class Sequence extends BTNode
    children: List<BTNode>
    
    function Execute(context)
        for child in children
            result = child.Execute(context)
            if result == Failure
                return Failure
            else if result == Running
                return Running
        return Success

// Condition: Checks world state
class Condition extends BTNode
    predicate: Function<Context, bool>
    
    function Execute(context)
        if predicate(context)
            return Success
        else
            return Failure

// Action: Performs game action
class Action extends BTNode
    action: Function<Context, Status>
    
    function Execute(context)
        return action(context)
```

**Example: Soldier Behavior Tree**:

```pseudocode
class SoldierBehaviorTree
    function BuildTree()
        return Selector([
            // Priority 1: React to immediate danger
            Sequence([
                Condition(IsUnderHeavyFire),
                Selector([
                    Sequence([
                        Condition(HasNearbyCover),
                        Action(MoveToCover)
                    ]),
                    Action(ReturnFire)
                ])
            ]),
            
            // Priority 2: Engage visible enemies
            Sequence([
                Condition(CanSeeEnemy),
                Action(EngageTarget)
            ]),
            
            // Priority 3: Follow orders
            Sequence([
                Condition(HasPendingOrder),
                Action(ExecuteCurrentOrder)
            ]),
            
            // Priority 4: Idle behavior
            Action(IdleScan)
        ])
```

**Advantages**:
- Visual and intuitive structure
- Easy to debug and modify
- Good performance (O(depth) per tick)
- Industry standard (Unreal Engine, Unity)

**Disadvantages**:
- Can become complex for long-term planning
- Reactive rather than proactive by default
- Requires manual structuring of all possibilities

### 14.4.3 Goal-Oriented Action Planning (GOAP)

GOAP flips the decision-making process: instead of checking conditions top-down, the AI plans backwards from goals.

```pseudocode
class WorldState
    facts: Map<String, Value>
    
    function DistanceTo(otherState)
        differences = 0
        for key in facts.keys
            if facts[key] != otherState.facts[key]
                differences += 1
        return differences

class Action
    name: String
    cost: float
    preconditions: Map<String, Value>
    effects: Map<String, Value>
    
    function IsValid(state)
        for key, value in preconditions
            if state.Get(key) != value
                return false
        return true
    
    function Apply(state)
        newState = state.Clone()
        for key, value in effects
            newState.Set(key, value)
        return newState

class GOAPPlanner
    function Plan(currentState, goalState, availableActions)
        openSet = PriorityQueue()
        openSet.Add(Node(currentState, [], 0))
        
        while not openSet.empty()
            current = openSet.Pop()
            
            if current.state == goalState
                return current.actions
            
            for action in availableActions
                if action.IsValid(current.state)
                    newState = action.Apply(current.state)
                    newActions = current.actions + [action]
                    cost = current.cost + action.cost
                    heuristic = newState.DistanceTo(goalState)
                    
                    openSet.Add(Node(newState, newActions, cost + heuristic))
        
        return null  // No plan found
```

**Example: Soldier GOAP Domain**:

```pseudocode
// World State Facts
enum Fact
    HasWeaponLoaded
    IsInCover
    IsSafe
    CanSeeEnemy
    EnemyDead
    IsReloading

// Available Actions
actions = [
    Action(
        name: "Reload",
        cost: 3.0,
        preconditions: {IsReloading: false},
        effects: {HasWeaponLoaded: true}
    ),
    
    Action(
        name: "MoveToCover",
        cost: 2.0,
        preconditions: {IsInCover: false},
        effects: {IsInCover: true, IsSafe: true}
    ),
    
    Action(
        name: "EngageEnemy",
        cost: 1.0,
        preconditions: {
            CanSeeEnemy: true,
            HasWeaponLoaded: true
        },
        effects: {EnemyDead: true}
    ),
    
    Action(
        name: "TakeCover",
        cost: 0.5,
        preconditions: {IsInCover: false, IsUnderFire: true},
        effects: {IsInCover: true, IsSafe: true}
    )
]

// Example planning
function ReactToContact(soldier, world)
    currentState = WorldState({
        HasWeaponLoaded: soldier.weapon.loaded,
        IsInCover: soldier.isInCover,
        IsSafe: not soldier.underFire,
        CanSeeEnemy: CanSeeEnemy(soldier, world)
    })
    
    goalState = WorldState({
        IsSafe: true,
        EnemyDead: true
    })
    
    plan = GOAPPlanner.Plan(currentState, goalState, actions)
    
    if plan
        ExecutePlan(plan)
    else
        ExecuteAction(MoveToCover)
```

**Advantages**:
- Emergent behavior from action combinations
- Handles complex multi-step planning
- Easy to add new actions without restructuring
- Proactive rather than purely reactive

**Disadvantages**:
- Higher computational cost (A* search)
- Can generate unrealistic plans
- Requires careful cost tuning
- Debugging planned sequences can be difficult

### 14.4.4 Utility AI (Scoring Actions)

Utility AI evaluates all possible actions and selects the one with the highest score based on current context.

```pseudocode
class UtilityAI
    actions: List<ScoredAction>
    
    function SelectAction(context): Action
        bestAction = null
        bestScore = -infinity
        
        for action in actions
            score = action.CalculateScore(context)
            if score > bestScore
                bestScore = score
                bestAction = action
        
        return bestAction

class ScoredAction
    name: String
    considerations: List<Consideration>
    
    function CalculateScore(context): float
        score = 1.0
        
        for consideration in considerations
            score *= consideration.Evaluate(context)
            if score < 0.01  // Early exit for very low scores
                return 0.0
        
        return score

class Consideration
    input: Function<Context, float>  // Raw value from world
    curve: ResponseCurve             // How to transform value
    weight: float                    // Importance multiplier
    
    function Evaluate(context): float
        rawValue = input(context)
        curvedValue = curve.Evaluate(rawValue)
        return curvedValue * weight
```

**Example: Target Selection**:

```pseudocode
class EngageTargetAction extends ScoredAction
    considerations = [
        // Distance consideration (closer is better)
        Consideration(
            input: DistanceToTarget,
            curve: InverseLinear(0, 100),  // Score 1.0 at 0m, 0.0 at 100m
            weight: 1.0
        ),
        
        // Threat consideration (higher threat is better target)
        Consideration(
            input: TargetThreatLevel,
            curve: Linear(0, 100),  // Score 0.0 at low threat, 1.0 at high
            weight: 0.8
        ),
        
        // Visibility consideration (must be visible)
        Consideration(
            input: CanSeeTarget,
            curve: Step(0.9),  // 0.0 if < 0.9 visibility, 1.0 otherwise
            weight: 1.0  // Critical - must see to engage
        ),
        
        // Cover consideration (prefer targets not in heavy cover)
        Consideration(
            input: TargetCoverQuality,
            curve: InverseLinear(0, 100),  // Lower cover = higher score
            weight: 0.5
        )
    ]

// Example usage
function SelectTarget(soldier, visibleEnemies)
    engageAction = EngageTargetAction()
    
    bestTarget = null
    bestScore = 0.0
    
    for enemy in visibleEnemies
        context = EngagementContext(soldier, enemy)
        score = engageAction.CalculateScore(context)
        
        if score > bestScore
            bestScore = score
            bestTarget = enemy
    
    return bestTarget
```

**Response Curves**:

```pseudocode
enum CurveType
    Linear          // y = x
    InverseLinear   // y = 1 - x
    Step            // y = 0 if x < threshold, else 1
    SCurve          // Smooth S-shaped curve
    Exponential     // y = x^2
    Logarithmic     // y = log(x + 1) / log(2)
```

**Advantages**:
- Natural handling of fuzzy decisions
- Easy to balance via curve tuning
- Considers all options simultaneously
- Graceful degradation (always picks something)

**Disadvantages**:
- Can be computationally expensive (evaluate all actions)
- Harder to debug than behavior trees
- Can oscillate between similar-scoring actions
- Requires careful normalization of inputs

### 14.4.5 Finite State Machines

Finite State Machines (FSM) represent the simplest decision architecture, where AI exists in one of several discrete states with defined transitions.

```pseudocode
class FiniteStateMachine
    currentState: State
    states: Map<StateId, State>
    
    function Update(context)
        transition = currentState.CheckTransitions(context)
        if transition != null
            ChangeState(transition.targetState)
        
        currentState.Execute(context)
    
    function ChangeState(newState)
        currentState.OnExit()
        currentState = states[newState]
        currentState.OnEnter()

class State
    id: StateId
    onEnter: Action
    onExecute: Action
    onExit: Action
    transitions: List<Transition>
    
    function CheckTransitions(context): Transition
        for transition in transitions
            if transition.condition(context)
                return transition
        return null

class Transition
    condition: Function<Context, bool>
    targetState: StateId
```

**CloseCombatFree's FSM**:

```cpp
// Simplified state transitions
void UpdateUnitState(Unit* unit, World* world) {
    QString currentStatus = unit->getUnitStatus();
    
    if (currentStatus == "READY") {
        // Check for queued orders
        if (unit->hasPendingOrders()) {
            unit->processNextOrder();
        }
    }
    else if (currentStatus == "DEFENDING") {
        // Check for enemies in defense arc
        if (EnemyInRange(unit, world)) {
            unit->changeStatus("FIRING");
            unit->acquireTarget();
        }
    }
    else if (currentStatus == "AMBUSHING") {
        // Wait for enemy to enter trigger zone
        if (EnemyEnteredAmbushZone(unit, world)) {
            unit->changeStatus("FIRING");
            unit->fireAtWill();
        }
    }
    else if (currentStatus == "MOVING") {
        // Check if movement complete
        if (unit->reachedDestination()) {
            unit->changeStatus("READY");
            unit->continueQueue();
        }
    }
}
```

**State Transition Diagram**:

```
                    ┌─────────────┐
                    │    READY    │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
         ↓                 ↓                 ↓
   ┌──────────┐    ┌──────────┐    ┌──────────┐
   │  MOVING  │    │ DEFENDING│    │ AMBUSHING│
   └────┬─────┘    └────┬─────┘    └────┬─────┘
        │               │               │
        │               │               │
        ↓               ↓               ↓
   ┌──────────┐    ┌──────────┐    ┌──────────┐
   │  READY   │←───│  FIRING  │←───│  FIRING  │
   └──────────┘    └──────────┘    └──────────┘
```

**Advantages**:
- Simple to understand and implement
- Predictable behavior
- Easy to debug
- Low computational cost

**Disadvantages**:
- Limited expressiveness
- State explosion for complex behaviors
- No memory of past states
- Difficult to handle concurrent behaviors

### 14.4.6 Hierarchical Task Networks (HTN)

HTN decomposes high-level goals into increasingly specific tasks until reaching primitive actions.

```pseudocode
class HTNPlanner
    function Plan(domain, currentState, goalTask)
        plan = []
        tasksToProcess = [goalTask]
        
        while tasksToProcess not empty
            currentTask = tasksToProcess.Pop()
            
            if IsPrimitive(currentTask)
                plan.Add(currentTask)
            else
                // Find applicable decomposition method
                method = FindApplicableMethod(domain, currentTask, currentState)
                if method == null
                    return null  // Planning failed
                
                // Add subtasks to process (in reverse order for stack)
                for subtask in Reverse(method.subtasks)
                    tasksToProcess.Push(subtask)
        
        return plan

class Task
    name: String
    preconditions: List<Condition>

class PrimitiveTask extends Task
    effects: List<Effect>
    cost: float

class CompoundTask extends Task
    methods: List<Method>

class Method
    preconditions: List<Condition>
    subtasks: List<Task>
```

**Example: Assault Position HTN**:

```pseudocode
CompoundTask: AssaultPosition(target)
    Method 1: CoordinatedAssault
        Preconditions: HasSquadSupport, NotSuppressed
        Subtasks:
            1. SuppressTarget(target)
            2. MoveToCover(nearTarget)
            3. EngageEnemy(target)
    
    Method 2: SoloAssault
        Preconditions: None
        Subtasks:
            1. MoveFastTo(target)
            2. EngageEnemy(target)

CompoundTask: SuppressTarget(target)
    Method 1: MachineGunSuppression
        Preconditions: HasMachineGun
        Subtasks:
            1. PositionForFieldOfFire(target)
            2. OpenFire(target)
            3. MaintainFire(duration: 10s)
    
    Method 2: RifleSuppression
        Preconditions: None
        Subtasks:
            1. TakeKnee
            2. OpenFire(target)
            3. MaintainFire(duration: 5s)

PrimitiveTask: MoveToCover(nearTarget)
    Preconditions: NotInCover
    Effects: IsInCover, PositionChanged
    Cost: Distance / Speed

// Planning example
plan = HTNPlanner.Plan(
    domain = MilitaryDomain(),
    currentState = soldier.CurrentState(),
    goalTask = AssaultPosition(enemyBuilding)
)

// Might produce:
// 1. PositionForFieldOfFire(enemyBuilding)
// 2. OpenFire(enemyBuilding)
// 3. MaintainFire(10s)
// 4. MoveToCover(enemyBuilding)
// 5. EngageEnemy(enemyBuilding)
```

**Advantages**:
- Natural expression of tactical doctrine
- Hierarchical structure mirrors military organization
- Easy to add new tactics as methods
- Can represent complex multi-step plans

**Disadvantages**:
- Complex implementation
- Planning can be expensive
- Requires domain expertise to write methods
- Debugging plan failures can be difficult

### 14.4.7 Comparative Analysis

| Architecture | Best For | Complexity | Performance | Debugging |
|--------------|----------|------------|-------------|-----------|
| **Behavior Trees** | Reactive AI, clear behaviors | Medium | O(depth) | Visual, easy |
| **GOAP** | Complex planning, emergent tactics | High | O(actions^depth) | Hard |
| **Utility AI** | Fuzzy decisions, continuous scoring | Medium | O(actions) | Moderate |
| **Finite State Machines** | Simple, discrete behaviors | Low | O(1) | Easy |
| **HTN** | Hierarchical tactics, military doctrine | High | O(tasks) | Moderate |

**Recommendation for Tactical Wargames**:
- Use **Behavior Trees** for individual soldier reactive behaviors
- Use **GOAP** or **HTN** for squad-level tactical planning
- Use **Utility AI** for target selection and position evaluation
- Combine: GOAP generates high-level plans, Behavior Trees execute individual actions

---

## 14.5 Tactical AI

### 14.5.1 Pathfinding with Cover

Tactical pathfinding differs from standard pathfinding by incorporating combat considerations into route selection.

```pseudocode
function FindTacticalPath(start, goal, world, soldier)
    openSet = PriorityQueue()
    openSet.Add(Node(start, [], 0))
    
    while not openSet.empty()
        current = openSet.Pop()
        
        if current.position == goal
            return ReconstructPath(current)
        
        for neighbor in GetNeighbors(current.position)
            // Calculate base movement cost
            movementCost = TerrainCost(neighbor) * Distance(current.position, neighbor)
            
            // Add cover bonus/penalty
            coverValue = EvaluateCover(neighbor, world.enemyPositions)
            coverCost = (1.0 - coverValue) * COVER_WEIGHT
            
            // Add exposure penalty (visible to enemies)
            if IsVisibleFromAnyEnemy(neighbor, world)
                coverCost += EXPOSURE_PENALTY
            
            // Add suppression risk
            suppressionRisk = CalculateSuppressionRisk(neighbor, world)
            coverCost += suppressionRisk * SUPPRESSION_WEIGHT
            
            totalCost = movementCost + coverCost
            
            if not visited.Contains(neighbor) or totalCost < visited[neighbor].cost
                visited[neighbor] = Node(neighbor, current, totalCost)
                openSet.Add(visited[neighbor])
    
    return null  // No path found
```

**Cover Evaluation**:

```pseudocode
function EvaluateCover(position, enemyPositions)
    if enemyPositions.empty()
        return 0.0  // No enemies, no cover needed
    
    totalCover = 0.0
    
    for enemyPos in enemyPositions
        // Check if cover blocks line of sight
        if HasLineOfSight(enemyPos, position)
            coverValue = 0.0
        else
            // Calculate how much cover blocks
            coverValue = CalculateCoverPercentage(enemyPos, position)
        
        // Weight by distance (closer enemies more important)
        distance = Distance(position, enemyPos)
        weight = 1.0 / (1.0 + distance / 50.0)  // 50m reference distance
        
        totalCover += coverValue * weight
    
    return totalCover / enemyPositions.Count()
```

**OpenCombat's Direction Cost System** (for vehicle pathfinding):

```rust
const COST_AHEAD: i32 = 0;        // Moving forward
const COST_DIAGONAL: i32 = 10;    // 45° turn
const COST_CORNER: i32 = 20;      // 90° turn
const COST_BACK_CORNER: i32 = 30; // 135° turn
const COST_BACK: i32 = 50;        // 180° turn (full about-face)

fn pathfind_with_direction_cost(soldier: &Soldier, destination: WorldPoint, map: &Map) -> WorldPaths {
    let start = soldier.world_point.to_grid();
    let end = destination.to_grid();
    
    a_star(start, end, |node| {
        let terrain = map.terrain_at(node);
        let base_cost = terrain.pedestrian_cost;
        
        // Add direction change cost
        let direction_cost = calculate_turn_cost(
            soldier.looking_direction,
            node.direction
        );
        
        base_cost + direction_cost
    })
}
```

### 14.5.2 Position Evaluation

Evaluating positions for tactical value involves multiple factors:

```pseudocode
class PositionEvaluator
    function EvaluatePosition(position, soldier, world): float
        score = 0.0
        
        // Factor 1: Cover quality
        coverScore = EvaluateCover(position, world.enemyPositions)
        score += coverScore * COVER_WEIGHT
        
        // Factor 2: Line of sight to enemies
        visibleEnemies = CountVisibleEnemies(position, world)
        score += visibleEnemies * VISIBILITY_WEIGHT
        
        // Factor 3: Field of fire
        fieldOfFire = CalculateFieldOfFire(position, world)
        score += fieldOfFire * FIELD_OF_FIRE_WEIGHT
        
        // Factor 4: Retreat routes
        retreatOptions = CountRetreatRoutes(position, world)
        score += retreatOptions * RETREAT_WEIGHT
        
        // Factor 5: Proximity to objective
        if world.hasObjective
            distanceToObjective = Distance(position, world.objective)
            score += (1.0 / (1.0 + distanceToObjective / 100.0)) * OBJECTIVE_WEIGHT
        
        // Factor 6: Proximity to squad
        if soldier.hasSquad
            distanceToSquad = DistanceToSquad(position, soldier.squad)
            score += (1.0 / (1.0 + distanceToSquad / 30.0)) * COHESION_WEIGHT
        
        // Factor 7: Suppression exposure
        suppressionRisk = CalculateSuppressionRisk(position, world)
        score -= suppressionRisk * SUPPRESSION_WEIGHT
        
        return score
```

**Cover Types**:

| Cover Type | Protection | Visibility | Movement Impact |
|------------|------------|------------|-----------------|
| **Hard Cover** (walls, rocks) | 100% from facing | Prone only | Blocks movement |
| **Soft Cover** (bushes, fences) | 50-75% | Crouched or prone | Slows movement |
| **Concealment** (tall grass) | 0% (just hides) | All postures | Normal movement |
| **Elevation** (rooftops, hills) | Varies | Enhanced visibility | Climbing cost |

### 14.5.3 Fire and Maneuver

The fundamental tactical principle of fire and maneuver (one element suppresses while another moves) can be implemented through coordinated squad AI:

```pseudocode
class FireAndManeuverCoordinator
    function CoordinateAssault(squad, objective, world)
        // Split squad into elements
        fireTeam = SelectFireTeam(squad, hasMachineGun: true)
        maneuverTeam = SelectManeuverTeam(squad, fireTeam)
        
        // Phase 1: Establish base of fire
        for soldier in fireTeam
            position = FindBestSuppressPosition(soldier, objective, world)
            soldier.Order(MoveTo(position))
        
        // Phase 2: Suppress objective
        WaitForAllInPosition(fireTeam)
        
        for soldier in fireTeam
            soldier.Order(SuppressFire(objective))
        
        // Phase 3: Maneuver element advances
        for soldier in maneuverTeam
            flankingPosition = FindFlankingPosition(soldier, objective, world)
            soldier.Order(MoveFastTo(flankingPosition))
        
        // Phase 4: Assault
        WaitForAllInPosition(maneuverTeam)
        
        for soldier in maneuverTeam
            soldier.Order(EngageAt(objective))
        
        // Phase 5: Consolidate
        WaitForEngagementComplete(maneuverTeam)
        
        for soldier in squad
            coverPosition = FindCoverNear(objective, world)
            soldier.Order(Defend(coverPosition))
```

### 14.5.4 Suppression Tactics

Suppression is a core tactical mechanic that affects both friendly and enemy AI behavior.

```pseudocode
class SuppressionSystem
    function ApplySuppression(position, radius, intensity, duration)
        affectedUnits = FindUnitsInRadius(position, radius)
        
        for unit in affectedUnits
            if IsFriendly(unit)
                continue  // Don't suppress friendlies
            
            // Calculate suppression effect based on distance
            distance = Distance(position, unit.position)
            falloff = 1.0 - (distance / radius)
            suppressionAmount = intensity * falloff
            
            // Apply to unit
            unit.suppression += suppressionAmount
            unit.under_fire.increase(suppressionAmount * 10)
            
            // Trigger suppression response if threshold exceeded
            if unit.suppression > SUPPRESSION_THRESHOLD
                TriggerSuppressionResponse(unit)
    
    function TriggerSuppressionResponse(unit)
        // Pinning behavior
        if unit.suppression > PINNING_THRESHOLD
            unit.ChangeBehavior(Hide(nearestCover))
            unit.canMove = false
            unit.accuracy *= 0.3
        
        // Seek cover
        else if unit.suppression > COVER_SEEK_THRESHOLD
            cover = FindNearestCover(unit)
            if cover != null
                unit.ChangeBehavior(MoveTo(cover))
        
        // Reduced effectiveness
        else
            unit.accuracy *= 0.7
            unit.movementSpeed *= 0.8
```

**Suppression Mechanics**:

| Suppression Level | Effect on AI |
|-------------------|--------------|
| **None (0-25)** | Normal behavior |
| **Light (25-50)** | Accuracy -30%, movement -20% |
| **Medium (50-75)** | Accuracy -50%, seeks cover |
| **Heavy (75-90)** | Pinned, cannot move, accuracy -70% |
| **Severe (90-100)** | Broken morale, may retreat |

### 14.5.5 Coordinated Attacks

Advanced AI can coordinate multi-unit attacks for tactical advantage:

```pseudocode
class CoordinatedAttackPlanner
    function PlanCoordinatedAttack(units, target, world)
        // Analyze target
        targetCover = EvaluateTargetCover(target, world)
        targetStrength = EvaluateTargetStrength(target, world)
        
        // Select tactic based on force ratio and cover
        if targetCover > 0.7 and units.Count() > targetStrength * 1.5
            return PlanFlankingAttack(units, target, world)
        else if units.Count() > targetStrength * 2.0
            return PlanFrontalAssault(units, target, world)
        else
            return PlanFireAndManeuver(units, target, world)
    
    function PlanFlankingAttack(units, target, world)
        // Find flanking routes
        leftFlank = FindFlankingRoute(units[0], target, world, leftSide: true)
        rightFlank = FindFlankingRoute(units[1], target, world, leftSide: false)
        
        // Synchronize timing
        leftTime = EstimateTravelTime(units[0], leftFlank)
        rightTime = EstimateTravelTime(units[1], rightFlank)
        
        if leftTime > rightTime
            delay = leftTime - rightTime
            units[1].AddDelayedOrder(MoveTo(rightFlank), delay)
            units[0].Order(MoveTo(leftFlank))
        else
            delay = rightTime - leftTime
            units[0].AddDelayedOrder(MoveTo(leftFlank), delay)
            units[1].Order(MoveTo(rightFlank))
        
        // Both engage simultaneously
        for unit in units
            unit.AddOrder(EngageTarget(target))
```

---

## 14.6 Strategic AI

### 14.6.1 Higher-Level Decision Making

While tactical AI controls individual units and squads, strategic AI manages the broader battle plan.

```pseudocode
class StrategicAI
    function MakeStrategicDecision(commander, battlefield, resources)
        // Evaluate current situation
        situation = AnalyzeSituation(battlefield)
        
        // Generate candidate strategies
        strategies = [
            Strategy(name: "Assault", action: PlanAssault),
            Strategy(name: "Defend", action: PlanDefense),
            Strategy(name: "Flank", action: PlanFlanking),
            Strategy(name: "Retreat", action: PlanRetreat)
        ]
        
        // Score each strategy
        bestStrategy = null
        bestScore = -infinity
        
        for strategy in strategies
            score = strategy.action.Evaluate(commander, battlefield, resources)
            if score > bestScore
                bestScore = score
                bestStrategy = strategy
        
        // Execute chosen strategy
        return bestStrategy.action.Execute(commander, battlefield, resources)
```

**Strategic Considerations**:

| Factor | Weight | Evaluation |
|--------|--------|------------|
| Force Ratio | 1.0 | Friendly vs enemy strength |
| Position Advantage | 0.8 | Cover, elevation, terrain |
| Morale | 0.7 | Unit condition and experience |
| Supply Status | 0.6 | Ammunition and reinforcement availability |
| Time Pressure | 0.5 | Mission time limits |
| Casualty Tolerance | 0.4 | Acceptable loss threshold |

### 14.6.2 Resource Allocation

Strategic AI manages limited resources across the battlefield:

```pseudocode
class ResourceAllocator
    function AllocateResources(units, availableResources, priorities)
        allocations = Map<Unit, Resource>()
        
        // Phase 1: Critical needs (ammunition, medical)
        for unit in units
            if unit.ammoLevel < CRITICAL_AMMO_THRESHOLD
                allocations[unit] = Resource(type: Ammunition, priority: 100)
            
            if unit.casualties > 0 and not unit.hasMedicalSupport
                allocations[unit] = Resource(type: Medical, priority: 90)
        
        // Phase 2: Reinforce weak sectors
        weakSectors = IdentifyWeakSectors(units)
        for sector in weakSectors
            reinforcements = CalculateNeededReinforcements(sector)
            allocations[sector.commander] = Resource(
                type: Reinforcements, 
                amount: reinforcements,
                priority: 70
            )
        
        // Phase 3: Support main effort
        mainEffort = IdentifyMainEffort(units)
        allocations[mainEffort] = Resource(
            type: FireSupport,
            priority: 80
        )
        
        return allocations
```

### 14.6.3 Mission Planning

Strategic AI decomposes mission objectives into tactical operations:

```pseudocode
class MissionPlanner
    function PlanMission(objective, availableForces, battlefield)
        plan = MissionPlan()
        
        // Phase 1: Reconnaissance
        plan.AddPhase(
            name: "Phase 1: Recon",
            tasks: [
                Task(type: Scout, position: objective.approaches),
                Task(type: Identify, target: enemyPositions)
            ]
        )
        
        // Phase 2: Approach and positioning
        plan.AddPhase(
            name: "Phase 2: Positioning",
            tasks: [
                Task(type: MoveTo, position: assaultPositions),
                Task(type: Establish, objective: BaseOfFire)
            ]
        )
        
        // Phase 3: Assault
        plan.AddPhase(
            name: "Phase 3: Assault",
            tasks: [
                Task(type: Suppress, target: objective),
                Task(type: Assault, objective: objective),
                Task(type: Consolidate, position: objective)
            ]
        )
        
        // Assign forces to phases
        for phase in plan.phases
            phase.assignedForces = AllocateForces(phase, availableForces)
        
        return plan
```

### 14.6.4 Reinforcement Learning (Optional Discussion)

While traditional AI uses handcrafted rules, reinforcement learning (RL) offers the potential for AI that learns optimal tactics through experience.

**RL for Tactical Decisions**:

```pseudocode
// Simplified RL concept (not implemented in clones)
class TacticalRLAgent
    state: TacticalState      // Position, cover, enemies, etc.
    action: TacticalAction    // Move, Shoot, Hide, etc.
    reward: float             // Casualties inflicted, survival, objective progress
    
    function ChooseAction(state): Action
        // Epsilon-greedy: explore or exploit
        if random() < epsilon
            return RandomAction()  // Explore
        else
            return QTable.BestAction(state)  // Exploit
    
    function Learn(state, action, reward, nextState)
        // Q-learning update
        oldQ = QTable.Get(state, action)
        maxNextQ = QTable.MaxQ(nextState)
        newQ = oldQ + alpha * (reward + gamma * maxNextQ - oldQ)
        QTable.Set(state, action, newQ)
```

**Challenges for Tactical RL**:
1. **State Space Size**: Tactical situations have enormous state spaces
2. **Credit Assignment**: Determining which action led to victory
3. **Training Time**: Requires millions of simulated battles
4. **Explainability**: RL agents are black boxes
5. **Determinism**: RL introduces non-determinism (problematic for multiplayer)

**Practical RL Applications**:
- **Offline Training**: Train AI against itself in simulation
- **Behavioral Cloning**: Learn from human player replays
- **Hybrid Approach**: RL tunes parameters of scripted behaviors

---

## 14.7 Determinism in AI

### 14.7.1 Why Determinism Matters

Determinism—the property that the same inputs always produce the same outputs—is crucial for:

1. **Multiplayer Synchronization**: All clients must see identical game states
2. **Replay Systems**: Recordings must play back identically
3. **Debugging**: Bugs must be reproducible
4. **Fairness**: AI shouldn't exploit randomness unfairly

**OpenCombat's Deterministic Design**:

```rust
// Deterministic random number generator
struct DeterministicRNG {
    seed: u64,
}

impl DeterministicRNG {
    fn new(seed: u64) -> Self {
        Self { seed }
    }
    
    fn next(&mut self) -> u64 {
        // Linear congruential generator (deterministic)
        self.seed = self.seed.wrapping_mul(6364136223846793005).wrapping_add(1);
        self.seed
    }
    
    fn random_float(&mut self) -> f32 {
        (self.next() as f32) / (u64::MAX as f32)
    }
}

// Usage in AI
fn make_random_decision(rng: &mut DeterministicRNG, options: &[Action]) -> Action {
    let index = (rng.random_float() * options.len() as f32) as usize;
    options[index]
}
```

### 14.7.2 Random Seed Management

For deterministic behavior, all random decisions must use a controlled seed:

```pseudocode
class DeterministicAI
    seed: u64
    rng: DeterministicRNG
    
    function Initialize(seed: u64)
        self.seed = seed
        self.rng = DeterministicRNG(seed)
    
    function MakeDecision(context): Action
        // All randomness goes through rng
        if ShouldAddNoise(rng)
            return SelectRandomly(rng, context.validActions)
        else
            return SelectBest(context.validActions)
    
    function AdvanceSeed()
        // Advance seed deterministically
        self.seed = Hash(self.seed, frameNumber)
        self.rng = DeterministicRNG(self.seed)
```

**Synchronization Strategy**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DETERMINISTIC MULTIPLAYER SYNC                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  SERVER                                          CLIENT                 │
│  ───────                                         ──────                 │
│                                                                         │
│  Frame N:                                    Frame N:                   │
│  ┌─────────────┐                            ┌─────────────┐            │
│  │ Receive     │                            │ Receive     │            │
│  │ Player      │                            │ Player      │            │
│  │ Inputs      │                            │ Inputs      │            │
│  └──────┬──────┘                            └──────┬──────┘            │
│         │                                          │                    │
│         ▼                                          ▼                    │
│  ┌─────────────┐                            ┌─────────────┐            │
│  │ Seed: 1234  │                            │ Seed: 1234  │            │
│  │ AI Decision │                            │ AI Decision │            │
│  │ (Determin.) │                            │ (Determin.) │            │
│  └──────┬──────┘                            └──────┬──────┘            │
│         │                                          │                    │
│         │ Both produce identical results           │                    │
│         ▼                                          ▼                    │
│  ┌─────────────┐                            ┌─────────────┐            │
│  │ State Hash  │◄──────────────────────────►│ State Hash  │            │
│  │ 0xA7B3...   │      Compare               │ 0xA7B3...   │            │
│  └─────────────┘                            └─────────────┘            │
│                                                                         │
│  If hashes match: simulation is synchronized                           │
│  If mismatch: client must re-synchronize from server                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 14.7.3 Sequencing Guarantees

For determinism, AI decisions must be processed in a consistent order:

```rust
fn tick_ai_system(world: &mut World, rng: &mut DeterministicRNG) {
    // Process soldiers in deterministic order (by index)
    let mut soldier_indices: Vec<SoldierIndex> = world.all_soldiers().collect();
    soldier_indices.sort();  // Ensure consistent ordering
    
    for soldier_idx in soldier_indices {
        // Each AI decision advances the RNG
        let decision = make_ai_decision(
            world.soldier(soldier_idx),
            world,
            rng
        );
        
        apply_decision(world, soldier_idx, decision);
    }
}
```

**Determinism Checklist**:

- [ ] Use fixed-point math or consistent floating-point operations
- [ ] Seed random number generators deterministically
- [ ] Process entities in sorted order
- [ ] Avoid unordered data structures (HashMap iteration)
- [ ] Use consistent time steps
- [ ] Synchronize floating-point precision across platforms

---

## 14.8 AI Difficulty Levels

### 14.8.1 Accuracy Modifiers

Adjusting AI accuracy provides a straightforward difficulty tuning mechanism:

```pseudocode
class DifficultySettings
    function CalculateAimAccuracy(baseAccuracy, difficulty): float
        match difficulty
            Easy ->
                return baseAccuracy * 0.6  // -40% accuracy
            Normal ->
                return baseAccuracy * 0.9  // -10% accuracy
            Hard ->
                return baseAccuracy * 1.0  // Normal accuracy
            Expert ->
                return baseAccuracy * 1.1  // +10% accuracy
            Insane ->
                return baseAccuracy * 1.25 // +25% accuracy
```

### 14.8.2 Information Cheating

Controlling AI information access affects difficulty dramatically:

```pseudocode
class InformationAccessController
    function CanSeeEnemy(ai, enemy, difficulty): bool
        // Check base visibility
        baseVisible = HasLineOfSight(ai, enemy)
        
        match difficulty
            Easy ->
                // AI only sees what player sees
                return baseVisible and enemy.revealedToPlayer
            
            Normal ->
                // Fair play - same LoS rules
                return baseVisible
            
            Hard ->
                // AI can "hear" nearby enemies
                if baseVisible
                    return true
                distance = Distance(ai, enemy)
                if distance < HEARING_RANGE and enemy.isMoving
                    return true  // AI heard movement
                return false
            
            Expert ->
                // AI has perfect memory of last known positions
                if baseVisible
                    ai.memory[enemy.id] = enemy.position
                    return true
                if ai.memory.Contains(enemy.id)
                    and TimeSince(ai.memory[enemy.id]) < MEMORY_DURATION
                    return true  // AI "remembers" enemy position
                return false
```

### 14.8.3 Decision Delays

Adding delays to AI decisions simulates human reaction time:

```pseudocode
class DecisionDelayController
    function GetDecisionDelay(difficulty): Duration
        match difficulty
            Easy -> return Duration::from_millis(800)  // Slow reactions
            Normal -> return Duration::from_millis(400)  // Human-like
            Hard -> return Duration::from_millis(200)  // Fast reactions
            Expert -> return Duration::from_millis(50)  // Near-instant
    
    function ShouldMakeDecisionNow(ai, difficulty): bool
        if not ai.decisionTimer.IsReady()
            return false
        
        ai.decisionTimer.Reset(GetDecisionDelay(difficulty))
        return true
```

### 14.8.4 Tactical Sophistication

Higher difficulty levels can use more sophisticated tactics:

```pseudocode
class TacticalSophisticationController
    function GetTacticalOptions(difficulty): TacticalComplexity
        match difficulty
            Easy ->
                return TacticalComplexity {
                    usesCover: true,
                    usesFlanking: false,
                    coordinatesSquads: false,
                    suppresses: false,
                    retreatsWhenOvermatched: true,
                    usesSmoke: false,
                }
            
            Normal ->
                return TacticalComplexity {
                    usesCover: true,
                    usesFlanking: true,
                    coordinatesSquads: false,
                    suppresses: true,
                    retreatsWhenOvermatched: true,
                    usesSmoke: false,
                }
            
            Hard ->
                return TacticalComplexity {
                    usesCover: true,
                    usesFlanking: true,
                    coordinatesSquads: true,
                    suppresses: true,
                    retreatsWhenOvermatched: true,
                    usesSmoke: true,
                }
            
            Expert ->
                return TacticalComplexity {
                    usesCover: true,
                    usesFlanking: true,
                    coordinatesSquads: true,
                    suppresses: true,
                    retreatsWhenOvermatched: true,
                    usesSmoke: true,
                    usesRecon: true,
                    adaptsToPlayerTactics: true,
                }
```

**Difficulty Level Summary**:

| Difficulty | Accuracy | Info Access | Decision Speed | Tactics |
|------------|----------|-------------|----------------|---------|
| **Easy** | -40% | Same as player | 800ms delay | Basic cover only |
| **Normal** | -10% | Fair play | 400ms delay | Cover + flanking |
| **Hard** | Normal | Extended hearing | 200ms delay | Coordinated squads |
| **Expert** | +10% | Perfect memory | 50ms delay | Full tactical playbook |
| **Insane** | +25% | Near-perfect info | Instant | Adaptive + predictive |

---

## 14.9 Implementation Guide

### 14.9.1 Recommended Architecture for New Projects

Based on the analysis of the three Close Combat clones, here is a recommended hybrid architecture:

```
┌─────────────────────────────────────────────────────────────────────────┐
│           RECOMMENDED AI ARCHITECTURE FOR TACTICAL WARGAMES              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  DATA LAYER                                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Unit definitions (JSON/YAML)                                    │   │
│  │ • Behavior configurations                                         │   │
│  │ • Difficulty settings                                             │   │
│  │ • Tactical doctrine scripts                                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  STRATEGIC AI                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ HTN or GOAP Planner                                               │   │
│  │ • Mission planning                                                │   │
│  │ • Resource allocation                                             │   │
│  │ • High-level tactics                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  TACTICAL AI                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ Squad Coordination System                                         │   │
│  │ • Formation management                                            │   │
│  │ • Fire and maneuver                                             │   │
│  │ • Synchronization                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  UNIT AI                                                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ Behavior Trees + Utility AI                                       │   │
│  │ • Reactive behaviors (trees)                                      │   │
│  │ • Target selection (utility)                                      │   │
│  │ • Position evaluation (utility)                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  PERCEPTION SYSTEM                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Line of sight calculations                                      │   │
│  │ • Fog of war integration                                          │   │
│  │ • Information sharing (squad)                                   │   │
│  │ • Threat evaluation (feeling system)                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  EXECUTION LAYER                                                        │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ • Action queuing                                                  │   │
│  │ • Animation integration                                           │   │
│  │ • State machines                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 14.9.2 Code Examples in Pseudocode

**Complete Soldier AI System**:

```pseudocode
// Main AI update loop
class SoldierAI
    soldier: Soldier
    behaviorTree: BehaviorTree
    perception: PerceptionSystem
    memory: AIMemory
    
    function Update(deltaTime, world)
        // 1. Perception update
        perception.Update(soldier, world)
        
        // 2. Update memory
        UpdateMemory(perception, memory)
        
        // 3. Evaluate threats
        threats = EvaluateThreats(perception, memory)
        
        // 4. Run behavior tree
        context = AIContext(soldier, perception, memory, threats)
        behaviorTree.Execute(context)
        
        // 5. Execute actions
        soldier.ExecuteCurrentAction(deltaTime)

// Behavior tree definition
class SoldierBehaviorTree
    function Build()
        root = Selector([
            // Priority 1: Survival
            BuildSurvivalBranch(),
            
            // Priority 2: Combat
            BuildCombatBranch(),
            
            // Priority 3: Orders
            BuildOrdersBranch(),
            
            // Priority 4: Idle
            BuildIdleBranch()
        ])
        return root
    
    function BuildSurvivalBranch()
        return Sequence([
            Condition(IsUnderHeavyFire),
            Selector([
                Sequence([
                    Condition(HasNearbyCover),
                    Action(SeekCover)
                ]),
                Action(ReturnFire)
            ])
        ])
    
    function BuildCombatBranch()
        return Sequence([
            Condition(CanSeeEnemy),
            Selector([
                Sequence([
                    Condition(IsInGoodPosition),
                    Action(EngageTarget)
                ]),
                Sequence([
                    Condition(NeedsBetterPosition),
                    Action(MoveToBetterPosition)
                ]),
                Action(SuppressArea)
            ])
        ])

// Target selection using utility AI
class TargetSelector
    function SelectBestTarget(soldier, visibleEnemies, world)
        bestTarget = null
        bestScore = 0.0
        
        for enemy in visibleEnemies
            score = 0.0
            
            // Distance consideration (inverse - closer is better)
            distance = Distance(soldier, enemy)
            score += (1.0 / (1.0 + distance / 50.0)) * 1.0
            
            // Threat consideration
            threat = enemy.CalculateThreatLevel()
            score += (threat / 100.0) * 0.8
            
            // Vulnerability consideration
            if not enemy.IsInCover()
                score += 0.5
            
            // Weapon effectiveness
            effectiveness = CalculateWeaponEffectiveness(soldier.weapon, distance)
            score += effectiveness * 0.6
            
            if score > bestScore
                bestScore = score
                bestTarget = enemy
        
        return bestTarget

// Cover evaluation
class CoverEvaluator
    function EvaluateCover(position, soldier, world)
        score = 0.0
        
        // Check protection from known enemies
        for enemy in soldier.knownEnemies
            if HasLineOfSight(enemy.position, position)
                score -= 1.0  // Penalty for exposure
            else
                coverValue = CalculateCoverPercentage(enemy.position, position)
                score += coverValue
        
        // Check if position offers fields of fire
        visibleEnemies = CountVisibleEnemies(position, world)
        score += visibleEnemies * 0.3
        
        // Check retreat routes
        retreatRoutes = CountRetreatRoutes(position, world)
        score += retreatRoutes * 0.2
        
        // Check proximity to squad
        if soldier.hasSquad
            squadDistance = DistanceToSquad(position, soldier.squad)
            score += (1.0 / (1.0 + squadDistance / 30.0)) * 0.4
        
        return score
```

**Squad Coordination**:

```pseudocode
class SquadAI
    squad: Squad
    tacticalGoal: TacticalGoal
    
    function Update(world)
        match tacticalGoal.type
            MoveTo(destination) ->
                ExecuteMovementFormation(destination)
            
            Engage(enemySquad) ->
                ExecuteFireAndManeuver(enemySquad)
            
            Defend(area) ->
                ExecuteDefensePosture(area)
            
            Retreat(rallyPoint) ->
                ExecuteFightingRetreat(rallyPoint)
    
    function ExecuteFireAndManeuver(enemySquad)
        // Split squad into fire and maneuver elements
        (fireTeam, maneuverTeam) = SplitSquad(squad)
        
        // Position fire team
        for soldier in fireTeam
            coverPosition = FindBestSuppressPosition(soldier, enemySquad)
            soldier.Order(MoveTo(coverPosition))
        
        // Wait for fire team in position
        WaitUntilInPosition(fireTeam)
        
        // Begin suppression
        for soldier in fireTeam
            soldier.Order(SuppressFire(CenterOf(enemySquad)))
        
        // Maneuver team advances
        for soldier in maneuverTeam
            flankPosition = FindFlankingPosition(soldier, enemySquad)
            soldier.Order(MoveFastTo(flankPosition))
        
        // Wait for maneuver team
        WaitUntilInPosition(maneuverTeam)
        
        // Transition to direct engagement
        for soldier in squad.members
            soldier.Order(EngageSquad(enemySquad))
```

### 14.9.3 Testing AI Behavior

Systematic testing is essential for AI quality:

```pseudocode
class AITestSuite
    function RunAllTests()
        TestCoverSeeking()
        TestSuppressionResponse()
        TestFlankingBehavior()
        TestRetreatLogic()
        TestSquadCoordination()
        TestDeterminism()
    
    function TestCoverSeeking()
        // Setup: Soldier under fire with nearby cover
        soldier = CreateSoldier(position: (100, 100))
        cover = CreateCover(position: (120, 100))
        enemy = CreateEnemy(position: (50, 100))
        
        // Simulate enemy fire
        FireAt(soldier, enemy)
        
        // Update AI
        for i in 0..10
            soldier.AI.Update()
        
        // Verify: Soldier should move to cover
        assert(IsInCover(soldier, cover))
    
    function TestDeterminism()
        // Setup: Same scenario, same seed
        world1 = CreateWorld(seed: 12345)
        world2 = CreateWorld(seed: 12345)
        
        // Run identical simulations
        for i in 0..100
            Update(world1)
            Update(world2)
        
        // Verify: States should be identical
        assert(world1.Hash() == world2.Hash())
    
    function TestSuppressionResponse()
        // Setup: Squad under heavy suppression
        squad = CreateSquad(size: 6)
        mg = CreateMachineGun()
        
        // Suppress with machine gun
        for i in 0..30
            SuppressFire(mg, squad.Center())
        
        // Verify: Squad should seek cover or retreat
        assert(CountInCover(squad) > 3 or IsRetreating(squad))
```

**Visual Debugging Tools**:

```pseudocode
class AIDebugVisualizer
    function DrawDebugInfo(renderer, world)
        for soldier in world.soldiers
            // Draw perception radius
            renderer.DrawCircle(soldier.position, soldier.perceptionRange, Color.Yellow)
            
            // Draw LoS to visible enemies
            for enemy in soldier.visibleEnemies
                if HasLineOfSight(soldier, enemy)
                    renderer.DrawLine(soldier.position, enemy.position, Color.Green)
                else
                    renderer.DrawLine(soldier.position, enemy.position, Color.Red)
            
            // Draw current behavior
            renderer.DrawText(soldier.position, soldier.currentBehavior.ToString())
            
            // Draw cover evaluation
            for cover in FindNearbyCover(soldier, radius: 50)
                score = EvaluateCover(cover, soldier, world)
                color = Lerp(Color.Red, Color.Green, score)
                renderer.DrawCircle(cover, 5, color)
```

### 14.9.4 Key Takeaways

1. **Start Simple**: Begin with FSM or simple behavior trees, add complexity incrementally
2. **Separate Concerns**: Keep perception, decision-making, and execution distinct
3. **Test Determinism**: If multiplayer matters, test determinism from day one
4. **Visualize Everything**: Debug visualizers are essential for AI development
5. **Use the Right Tool**: Behavior trees for reactivity, GOAP/HTN for planning, Utility for fuzzy decisions
6. **Balance Fairness**: AI should be challenging but not omniscient
7. **Modulate Difficulty**: Multiple knobs (accuracy, info, speed, tactics) for fine-tuning
8. **Model Emotion**: Systems like OpenCombat's "Feeling" add believability
9. **Coordinate at Squad Level**: Individual AI + squad coordination = emergent tactics
10. **Profile Early**: AI can become expensive; profile and optimize hotspots

---

## Chapter Summary

This chapter examined AI systems in tactical wargames through the lens of three Close Combat clones, each representing a different point on the sophistication spectrum:

- **OpenCombat-SDL** demonstrates that no AI can be a valid design choice when player control is paramount
- **CloseCombatFree** shows how simple state triggers provide basic automation without complexity
- **OpenCombat** illustrates how reactive systems with emotional modeling create believable soldier behaviors

The key architectural decisions for tactical AI include:
1. **Perception systems** that balance fairness with challenge
2. **Decision architectures** appropriate to the complexity needs (Behavior Trees, GOAP, Utility AI)
3. **Tactical coordination** for squad-level operations
4. **Strategic planning** for higher-level battle management
5. **Determinism** for multiplayer and replay systems
6. **Difficulty modulation** through multiple tunable parameters

For new implementations, a hybrid approach combining systems-oriented core simulation with behavior trees for reactive behaviors and utility AI for scoring decisions offers the best balance of power, flexibility, and maintainability.

---

**Next**: Chapter 15 - Multiplayer and Networking Systems
