# Chapter 15: Multiplayer Architecture for Tactical Wargames

## Synchronization, Determinism, and Network Architecture

---

*"Adding multiplayer to a single-player game is like trying to turn a monologue into a conversation while ensuring both speakers say exactly the same words at exactly the same time—without either knowing what the other will say next."*

---

## 15.1 Why Multiplayer Changes Everything

Multiplayer is not a feature you add to a game; it is an architectural commitment that fundamentally alters how every system operates. The three Close Combat clones examined in this book illustrate this divide: OpenCombat-SDL and CloseCombatFree are single-player only, while OpenCombat was designed from inception for networked play. This chapter explores why that design decision has such profound implications.

### 15.1.1 The Determinism Requirement

In single-player games, the simulation state exists in one place: the player's machine. The game can use floating-point arithmetic, system timers, and platform-specific random number generators without concern. Multiplayer shatters this assumption.

**The Core Problem**: When two players execute the same sequence of actions, they must arrive at exactly the same game state. If Player A calculates that a bullet hits at position (100.5, 200.3) while Player B calculates (100.500001, 200.299999), their simulations diverge. This divergence compounds exponentially—within seconds, the games are desynchronized and players are experiencing different realities.

**OpenCombat's Solution**: The Rust-based OpenCombat achieves determinism through:

```rust
// Fixed-point arithmetic for positions
pub struct WorldPoint {
    pub x: FixedPoint,  // 16.16 fixed-point, not float
    pub y: FixedPoint,
}

// Seeded RNG for combat
pub struct DeterministicRNG {
    state: u64,  // Same seed = same sequence across all platforms
}

impl DeterministicRNG {
    pub fn new(seed: u64) -> Self {
        Self { state: seed }
    }
    
    pub fn next(&mut self) -> u64 {
        // xorshift algorithm - identical on all platforms
        self.state ^= self.state << 13;
        self.state ^= self.state >> 7;
        self.state ^= self.state << 17;
        self.state
    }
}
```

**Why This Matters**: In deterministic simulation, only player inputs need to traverse the network. The simulation itself runs independently on each machine, producing identical results. This is the foundation of all multiplayer synchronization strategies.

### 15.1.2 Latency Challenges

Tactical wargames present unique latency challenges compared to fast-paced action games:

| Game Type | Acceptable Latency | Input Frequency | Latency Compensation |
|-----------|-------------------|-----------------|---------------------|
| First-Person Shooter | < 100ms | Continuous | Client prediction |
| Fighting Game | < 50ms | Discrete frames | Rollback netcode |
| RTS Game | 150-300ms | Discrete commands | Command delay |
| Tactical Wargame | 200-500ms | Intermittent | Deterministic lockstep |

**The Tactical Advantage**: Slow-paced tactical games actually have an advantage here. A player issuing a "move to position" command expects a delay while the unit acknowledges and begins moving. The natural pace of tactical decision-making (seconds to minutes between significant actions) accommodates network latency far better than twitch-based games.

However, this creates a design constraint: the game must feel responsive at 200-500ms latencies. This requires careful input acknowledgment and feedback systems:

```pseudocode
// Immediate feedback loop
onPlayerIssueOrder(order) {
    // 1. Show acknowledgment immediately (no network wait)
    ui.showOrderMarker(order.target)
    playAcknowledgmentSound()
    
    // 2. Send to server
    network.sendOrder(order)
    
    // 3. Optimistically update local state (client-side prediction)
    predictedState.apply(order)
}

onServerConfirmOrder(confirmedOrder) {
    // 4. Reconcile if prediction was wrong
    if predictedState != serverState {
        state = serverState  // Authoritative server always wins
    }
}
```

### 15.1.3 Synchronization Complexity

Synchronizing game state across multiple machines requires solving three distinct problems:

**Problem 1: State Consistency**
All clients must maintain identical game state. This requires:
- Deterministic simulation (same inputs → same outputs)
- Synchronized random number generators
- Fixed timestep updates
- No platform-specific behaviors

**Problem 2: Bandwidth Optimization**
A tactical wargame with 500 units has thousands of state variables. Transmitting complete state 60 times per second is impossible.

**Problem 3: Latency Hiding**
Players must feel in control even when their commands take 200ms to reach the server and another 200ms for results to return.

### 15.1.4 Security Concerns

Single-player games trust the client completely. Multiplayer games trust nothing.

**The Cheating Spectrum**:

| Cheat Type | Single-Player Impact | Multiplayer Impact | Detection Difficulty |
|------------|---------------------|-------------------|---------------------|
| Wallhacks (see through walls) | None | Severe | Hard |
| Aimbots (perfect accuracy) | None | Severe | Medium |
| Speed hacks | None | Severe | Easy |
| Map hacks (reveal fog of war) | None | Severe | Hard |
| Replay editing | None | Moderate | Easy |

**OpenCombat's Security Model**:

```
┌─────────────────────────────────────────────────────────────┐
│                    SECURITY ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CLIENT (Dumb Terminal)                                     │
│  ├── Can only SEND commands                                 │
│  ├── Can only RECEIVE state updates                         │
│  └── Cannot modify game state directly                      │
│                                                             │
│  SERVER (Authority)                                         │
│  ├── Validates ALL commands                                 │
│  ├── Computes ALL state changes                             │
│  └── Is source of truth for game state                      │
│                                                             │
│  NETWORK (Encrypted)                                        │
│  ├── Commands authenticated                                 │
│  ├── State updates signed                                   │
│  └── Replay verification possible                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

This server-authoritative architecture prevents all client-side cheating. The client is literally incapable of modifying game state—all it can do is request actions that the server validates and executes.

---

## 15.2 The Deterministic Simulation Approach (OpenCombat)

OpenCombat represents the state-of-the-art for tactical wargame multiplayer architecture. Its approach—deterministic simulation with message-based state updates—provides a blueprint for any developer building networked tactical games.

### 15.2.1 Message-Based State Updates

In OpenCombat, game state never changes directly. All modifications flow through a message system:

```rust
// Core message enum - every state change is a message
pub enum BattleStateMessage {
    // Soldier-level updates
    Soldier(SoldierIndex, SoldierMessage),
    
    // Global game phase
    SetPhase(Phase),
    
    // Combat events
    PushBulletFire(BulletFire),
    PushExplosion(Explosion),
    
    // Terrain changes
    SetTerrainBlocking { grid_point: GridPoint, blocking: bool },
    
    // Entity lifecycle
    SoldierSpawn(Soldier),
    SoldierDespawn(SoldierIndex),
}

// Per-soldier messages
pub enum SoldierMessage {
    SetWorldPosition(WorldPoint),
    SetBehavior(Behavior),
    SetGesture(Gesture),
    SetOrientation(Angle),
    TakeDamage(f32),
    SetHealth(f32),
    SetUnderFire(u32),
}

// State application - the ONLY way to modify state
impl BattleState {
    pub fn apply(&mut self, msg: BattleStateMessage) {
        match msg {
            BattleStateMessage::Soldier(idx, soldier_msg) => {
                if let Some(soldier) = self.soldiers.get_mut(idx.0) {
                    soldier.apply(soldier_msg);
                }
            }
            BattleStateMessage::SetPhase(phase) => {
                self.phase = phase;
            }
            // ... other messages
        }
    }
}
```

**Why Messages?**

1. **Network Transparency**: Messages serialize naturally for network transmission
2. **Replay Capability**: Recording messages captures complete game history
3. **Debugging**: Message log shows exactly what changed and when
4. **Testing**: Can inject messages to test specific scenarios
5. **Determinism**: Same message sequence always produces same state

### 15.2.2 Fixed Timestep Simulation

OpenCombat uses a fixed timestep to ensure determinism:

```rust
pub const TICK_RATE: u32 = 60;  // 60 FPS
pub const TICK_DURATION_MICROS: u64 = 1_000_000 / TICK_RATE as u64;  // 16,666 µs

pub struct SimulationLoop {
    last_tick: Instant,
    accumulator: Duration,
}

impl SimulationLoop {
    pub fn run(&mut self, state: &mut BattleState) {
        let now = Instant::now();
        let delta = now - self.last_tick;
        self.last_tick = now;
        
        self.accumulator += delta;
        
        // Fixed timestep: consume time in discrete chunks
        while self.accumulator >= Duration::from_micros(TICK_DURATION_MICROS) {
            // Collect all messages from this tick
            let messages = collect_messages();
            
            // Apply messages deterministically (sorted by priority)
            let sorted_messages = sort_by_priority(messages);
            
            for msg in sorted_messages {
                state.apply(msg);
            }
            
            // Run simulation systems
            update_physics(state);
            update_ai(state);
            update_combat(state);
            
            self.accumulator -= Duration::from_micros(TICK_DURATION_MICROS);
            state.frame_i += 1;
        }
    }
}
```

**Critical Rule**: Simulation logic must never depend on real time, frame rate, or platform-specific behaviors. Only the tick count matters.

### 15.2.3 Replay System

Deterministic simulation enables perfect replay:

```rust
pub struct Replay {
    initial_state: BattleState,
    messages: Vec<TimedMessage>,  // (frame_index, message)
    random_seed: u64,
}

pub struct TimedMessage {
    pub frame: u64,
    pub message: BattleStateMessage,
}

impl Replay {
    pub fn record(state: &BattleState) -> Self {
        Self {
            initial_state: state.clone(),
            messages: Vec::new(),
            random_seed: state.rng_seed,
        }
    }
    
    pub fn add_message(&mut self, frame: u64, msg: BattleStateMessage) {
        self.messages.push(TimedMessage { frame, message: msg });
    }
    
    pub fn playback(&self) -> BattleState {
        let mut state = self.initial_state.clone();
        state.rng = DeterministicRNG::new(self.random_seed);
        
        let mut msg_iter = self.messages.iter();
        let mut next_msg = msg_iter.next();
        
        loop {
            // Apply all messages for this frame
            while let Some(msg) = next_msg {
                if msg.frame <= state.frame_i {
                    state.apply(msg.message.clone());
                    next_msg = msg_iter.next();
                } else {
                    break;
                }
            }
            
            // Run simulation tick
            update_physics(&mut state);
            update_ai(&mut state);
            
            state.frame_i += 1;
            
            // Check for end conditions
            if next_msg.is_none() && state.phase == Phase::End {
                break;
            }
        }
        
        state
    }
}
```

**Replay Benefits**:
- **Bug Reproduction**: Record a bug, send replay to developers
- **Tournament Validation**: Verify no cheating occurred
- **After-Action Reviews**: Analyze battles frame-by-frame
- **Spectator Mode**: Delayed broadcast of gameplay

### 15.2.4 Save/Restore Capability

Deterministic simulation makes save/load trivial:

```rust
// Save = snapshot state + message history since snapshot
pub struct SaveGame {
    pub snapshot: BattleState,
    pub messages_since_snapshot: Vec<TimedMessage>,
    pub save_time: SystemTime,
}

impl SaveGame {
    pub fn create(state: &BattleState, replay: &Replay) -> Self {
        Self {
            snapshot: state.clone(),
            messages_since_snapshot: replay.messages.clone(),
            save_time: SystemTime::now(),
        }
    }
    
    pub fn restore(&self) -> BattleState {
        let mut state = self.snapshot.clone();
        
        // Replay messages to restore exact state
        for timed_msg in &self.messages_since_snapshot {
            if timed_msg.frame > self.snapshot.frame_i {
                state.apply(timed_msg.message.clone());
            }
        }
        
        state
    }
}
```

This approach is superior to traditional save/load because:
- **Compact**: Only need one full state + recent messages
- **Reliable**: No serialization version hell (messages define the schema)
- **Network-friendly**: Same format works for both save and network sync

### 15.2.5 Lockstep vs Deterministic Simulation

Two related but distinct approaches are often confused:

**Lockstep Synchronization**:
- All clients wait for all inputs before simulating the next frame
- Used by: Age of Empires, StarCraft (original)
- Pros: Guaranteed synchronization, simple
- Cons: Latency affects everyone (slowest player sets the pace)

```
Frame 1: All players send commands → Wait for all → Simulate
Frame 2: All players send commands → Wait for all → Simulate
...
```

**Deterministic Simulation (OpenCombat's approach)**:
- Clients simulate continuously, inputs arrive when they arrive
- Server authority resolves conflicts
- Used by: Overwatch, Rocket League, OpenCombat
- Pros: Responsive, scales to high latency
- Cons: Requires prediction and reconciliation

```
Frame 1: Client A sends command → Simulates immediately
         Client B sends command → Simulates immediately
         Server validates → Broadcasts result
Frame 2: Both clients reconcile if prediction was wrong
```

**OpenCombat's Hybrid Approach**:

```rust
pub enum SyncMode {
    // Single-player: No network, immediate application
    Local,
    
    // Multiplayer client: Predict locally, reconcile with server
    Client { server_authority: bool },
    
    // Multiplayer server: Authoritative simulation
    Server,
}

pub struct MultiplayerState {
    mode: SyncMode,
    
    // For clients: track predictions
    predicted_state: BattleState,
    server_state: BattleState,
    
    // For server: authoritative truth
    authoritative_state: BattleState,
    client_inputs: HashMap<PlayerId, Vec<PlayerInput>>,
}
```

---

## 15.3 Non-Deterministic Approaches

While deterministic simulation is the gold standard for competitive multiplayer, other approaches have valid use cases.

### 15.3.1 State Synchronization (Snapshot Interpolation)

In this approach, the server periodically sends complete (or delta-compressed) state snapshots to clients:

```rust
pub struct StateSnapshot {
    pub frame: u64,
    pub timestamp: u64,
    pub soldiers: Vec<SoldierSnapshot>,
    pub projectiles: Vec<ProjectileSnapshot>,
}

pub struct SoldierSnapshot {
    pub index: SoldierIndex,
    pub position: WorldPoint,
    pub behavior: Behavior,
    pub health: f32,
}

// Server sends snapshots at 20 Hz (every 3 simulation ticks)
pub const SNAPSHOT_RATE: u32 = 20;

impl Server {
    pub fn broadcast_snapshot(&self, state: &BattleState) {
        let snapshot = StateSnapshot {
            frame: state.frame_i,
            timestamp: now_micros(),
            soldiers: state.soldiers.iter().map(|s| s.to_snapshot()).collect(),
            projectiles: state.bullets.iter().map(|b| b.to_snapshot()).collect(),
        };
        
        // Compress and send to all clients
        let compressed = compress_snapshot(&snapshot);
        for client in &self.connected_clients {
            self.network.send(client, &compressed);
        }
    }
}
```

**When to Use**: 
- Games with complex physics (vehicle simulators)
- When determinism is impossible (external physics engines)
- Lower player counts where bandwidth is less constrained

**Trade-offs**:
- (+) Simple to implement
- (-) High bandwidth requirements
- (-) No replay capability (can't reconstruct from snapshots)
- (-) Harder to debug (no message history)

### 15.3.2 Event Replication

Similar to deterministic simulation but without the determinism guarantee:

```rust
pub struct Event {
    pub timestamp: u64,
    pub event_type: EventType,
    pub reliable: bool,  // TCP for critical events
}

pub enum EventType {
    WeaponFired { soldier: SoldierIndex, target: WorldPoint },
    Explosion { position: WorldPoint, radius: f32 },
    SoldierDied { soldier: SoldierIndex, killer: SoldierIndex },
}

// Server broadcasts events as they happen
impl Server {
    pub fn on_weapon_fired(&mut self, event: WeaponFiredEvent) {
        // Apply to server state
        self.state.apply_weapon_fired(event);
        
        // Broadcast to clients
        let event = Event {
            timestamp: now_micros(),
            event_type: EventType::WeaponFired { 
                soldier: event.soldier,
                target: event.target,
            },
            reliable: false,  // UDP is fine for fire events
        };
        
        self.broadcast_event(event);
    }
}
```

**When to Use**:
- Cooperative games where perfect sync isn't critical
- Games with forgiving gameplay (RPGs, action games)
- Rapid prototyping before implementing determinism

**Trade-offs**:
- (+) Lower bandwidth than snapshots
- (+) Simpler than deterministic simulation
- (-) Desyncs are inevitable and hard to resolve
- (-) No replay or verification possible

### 15.3.3 Hybrid Approaches

Many successful games combine approaches:

```rust
pub struct HybridNetworkSystem {
    // Critical state: Deterministic simulation + messages
    deterministic_messages: Vec<BattleStateMessage>,
    
    // Visual state: Interpolated from snapshots
    last_snapshot: StateSnapshot,
    target_snapshot: StateSnapshot,
    interpolation_factor: f32,
    
    // Event state: Event replication for effects
    pending_events: Vec<Event>,
}

impl HybridNetworkSystem {
    pub fn update(&mut self, dt: f32) {
        // 1. Apply deterministic messages (authoritative)
        for msg in &self.deterministic_messages {
            self.state.apply(msg.clone());
        }
        
        // 2. Interpolate visual state (smooth rendering)
        self.interpolation_factor += dt * SNAPSHOT_RATE as f32;
        if self.interpolation_factor >= 1.0 {
            self.last_snapshot = self.target_snapshot.clone();
            self.target_snapshot = receive_next_snapshot();
            self.interpolation_factor = 0.0;
        }
        
        // 3. Process events (visual/audio feedback)
        for event in &self.pending_events {
            self.handle_event(event);
        }
    }
    
    pub fn get_render_state(&self) -> RenderState {
        // Interpolate between snapshots for smooth visuals
        RenderState::interpolate(
            &self.last_snapshot,
            &self.target_snapshot,
            self.interpolation_factor
        )
    }
    
    pub fn get_simulation_state(&self) -> &BattleState {
        // Use deterministic state for gameplay logic
        &self.state
    }
}
```

### 15.3.4 When Non-Deterministic Is Acceptable

Consider non-deterministic approaches when:

1. **Cooperative PvE**: Players work together against AI, not competing
2. **Casual Multiplayer**: Perfect sync isn't critical for enjoyment
3. **Rapid Prototyping**: Get multiplayer working quickly, add determinism later
4. **External Physics**: Using PhysX, Havok, or other non-deterministic engines
5. **Massive Scale**: 1000+ players where deterministic sync is impossible

**OpenCombat-SDL Conversion Example**:

If converting OpenCombat-SDL to multiplayer, a non-deterministic approach might be faster to implement initially:

```cpp
// Quick-and-dirty multiplayer for OpenCombat-SDL
class NetworkedWorld : public World {
public:
    void Update(float dt) {
        // 1. Receive remote state updates
        while (auto update = network.ReceiveStateUpdate()) {
            ApplyRemoteState(update);
        }
        
        // 2. Run local simulation (non-deterministic)
        World::Update(dt);
        
        // 3. Send our state to others
        auto state = SerializeState();
        network.BroadcastState(state);
    }
    
private:
    void ApplyRemoteState(const StateUpdate& update) {
        // Naive: Just overwrite remote entities
        for (auto& soldierUpdate : update.soldiers) {
            auto* soldier = FindSoldier(soldierUpdate.id);
            if (soldier && soldier->GetSide() != localPlayerSide) {
                soldier->SetPosition(soldierUpdate.position);
                soldier->SetHealth(soldierUpdate.health);
            }
        }
    }
};
```

This would work for cooperative play but would fail for competitive due to desync issues.

---

## 15.4 Network Architecture Patterns

Choosing the right network topology is as important as choosing the right synchronization method.

### 15.4.1 Client-Server

The industry standard for competitive multiplayer:

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT-SERVER MODEL                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                      ┌─────────────┐                        │
│                      │   SERVER    │                        │
│                      │  (Authori-  │                        │
│                      │   tative)   │                        │
│                      └──────┬──────┘                        │
│                             │                               │
│         ┌───────────────────┼───────────────────┐          │
│         │                   │                   │          │
│    ┌────┴────┐        ┌────┴────┐        ┌────┴────┐     │
│    │ Client  │        │ Client  │        │ Client  │     │
│    │   A     │◄──────►│         │◄──────►│    C    │     │
│    └─────────┘        └─────────┘        └─────────┘     │
│                                                             │
│  Communication: Only clients ↔ server                       │
│  Clients never communicate directly                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**OpenCombat Implementation**:

```rust
// Server
pub struct GameServer {
    socket: zmq::Socket,  // ZeroMQ for networking
    clients: HashMap<PlayerId, ClientConnection>,
    state: BattleState,
    tick_timer: Interval,
}

impl GameServer {
    pub async fn run(&mut self) {
        loop {
            tokio::select! {
                // 1. Receive commands from clients
                command = self.receive_command() => {
                    self.process_command(command);
                }
                
                // 2. Periodic state broadcast
                _ = self.tick_timer.tick() => {
                    self.broadcast_state_delta();
                }
            }
        }
    }
    
    fn process_command(&mut self, cmd: ClientCommand) {
        // Validate command (anti-cheat)
        if !self.validate_command(&cmd) {
            self.send_error(cmd.player_id, "Invalid command");
            return;
        }
        
        // Convert to message and apply
        let msg = cmd.to_message();
        self.state.apply(msg);
    }
}

// Client
pub struct GameClient {
    server_socket: zmq::Socket,
    local_state: BattleState,
    prediction_state: BattleState,
}

impl GameClient {
    pub fn issue_command(&self, cmd: PlayerCommand) {
        // 1. Predict locally for responsiveness
        let predicted_msg = cmd.to_message();
        self.prediction_state.apply(predicted_msg);
        
        // 2. Send to server
        self.server_socket.send(&serialize(cmd), 0).unwrap();
    }
    
    pub fn on_state_update(&mut self, update: StateUpdate) {
        // Apply authoritative server state
        self.local_state = update.state;
        
        // Check if prediction was correct
        if self.prediction_state != self.local_state {
            // Reconciliation: Snap to server state
            self.prediction_state = self.local_state.clone();
        }
    }
}
```

**Pros**:
- (+) Authoritative server prevents cheating
- (+) Clients can join/leave easily
- (+) Scales to many players
- (+) Simple mental model

**Cons**:
- (-) Server infrastructure required
- (-) Latency for all actions
- (-) Server is single point of failure

### 15.4.2 Peer-to-Peer

All clients are equal; no central server:

```
┌─────────────────────────────────────────────────────────────┐
│                     PEER-TO-PEER MODEL                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    ┌─────────┐───────────────────────────┐─────────┐      │
│    │ Peer A  │◄─────────────────────────►│ Peer B  │      │
│    └────┬────┘                           └────┬────┘      │
│         │                                     │            │
│         └─────────────────────────────────────┘            │
│                       ▲                                    │
│                       │                                    │
│                  ┌────┴────┐                              │
│                  │ Peer C  │                              │
│                  └─────────┘                              │
│                                                             │
│  Communication: All peers ↔ all other peers                │
│  Requires deterministic lockstep for synchronization       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**When to Use**:
- Fighting games (GGPO, Rollback Netcode)
- Small player counts (2-4 players)
- LAN play
- When server infrastructure is unavailable

**Tactical Wargame Considerations**:

Peer-to-peer is generally NOT recommended for tactical wargames because:
- Cheating is trivial (players control the simulation)
- Desyncs are hard to resolve
- Joining mid-game is nearly impossible
- Hard to scale beyond 4-6 players

### 15.4.3 Relay Server

A lightweight server that forwards messages without simulation:

```
┌─────────────────────────────────────────────────────────────┐
│                      RELAY SERVER MODEL                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                      ┌─────────────┐                        │
│                      │   RELAY     │                        │
│                      │   SERVER    │                        │
│                      │  (No sim)   │                        │
│                      └──────┬──────┘                        │
│                             │                               │
│         ┌───────────────────┼───────────────────┐          │
│         │                   │                   │          │
│    ┌────┴────┐        ┌────┴────┐        ┌────┴────┐     │
│    │ Client  │        │ Client  │        │ Client  │     │
│    │(Authori-│        │(Authori-│        │(Authori-│     │
│    │  tative)│◄──────►│  tative)│◄──────►│  tative)│     │
│    └─────────┘        └─────────┘        └─────────┘     │
│                                                             │
│  All clients run full simulation                            │
│  Relay only forwards inputs                                 │
│  Requires lockstep synchronization                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Implementation**:

```rust
// Relay is just a message forwarder
pub struct RelayServer {
    clients: Vec<ClientConnection>,
}

impl RelayServer {
    pub fn on_message(&self, from: ClientId, msg: GameMessage) {
        // Broadcast to all other clients
        for client in &self.clients {
            if client.id != from {
                client.send(&msg);
            }
        }
    }
}

// Clients run full simulation deterministically
pub struct P2PClient {
    relay: RelayConnection,
    state: BattleState,
    pending_inputs: Vec<PlayerInput>,
}

impl P2PClient {
    pub fn tick(&mut self) {
        // 1. Collect local input
        let input = collect_input();
        self.pending_inputs.push(input);
        
        // 2. Send to relay (which forwards to others)
        self.relay.send(InputMessage { input });
        
        // 3. Wait for all players' inputs (lockstep)
        let all_inputs = self.wait_for_all_inputs();
        
        // 4. Apply all inputs deterministically
        for input in all_inputs {
            let msg = input.to_message();
            self.state.apply(msg);
        }
        
        // 5. Run simulation
        update_simulation(&mut self.state);
    }
}
```

**When to Use**:
- Very low player counts (2-3 players)
- When you want to minimize server costs
- Games with simple state (chess, turn-based)

**Trade-offs**:
- (+) Cheap to host (relay uses minimal resources)
- (+) No server-side cheating possible (all clients equal)
- (-) Requires perfect lockstep
- (-) Desyncs are catastrophic
- (-) Can't handle late joiners

### 15.4.4 Authoritative Server Requirements

For competitive tactical wargames, authoritative server is non-negotiable:

**Must-Have Features**:

```rust
pub struct AuthoritativeServer {
    // 1. State validation
    pub fn validate_command(&self, cmd: &Command, state: &BattleState) -> bool {
        // Check if player owns this unit
        if !self.player_owns_unit(cmd.player_id, cmd.unit_id) {
            return false;
        }
        
        // Check if command is valid in current state
        if !self.is_valid_command(cmd, state) {
            return false;
        }
        
        // Check rate limiting (prevent spam)
        if self.is_rate_limited(cmd.player_id) {
            return false;
        }
        
        true
    }
    
    // 2. State verification (detect desyncs)
    pub fn verify_client_state(&self, client_hash: u64) -> bool {
        let server_hash = self.compute_state_hash();
        client_hash == server_hash
    }
    
    // 3. Anti-cheat validation
    pub fn detect_cheating(&self, player_id: PlayerId, cmd: &Command) -> bool {
        // Check for impossible movements (speed hacks)
        if cmd.movement_distance > MAX_POSSIBLE_DISTANCE {
            return true;
        }
        
        // Check for impossible vision (wallhacks)
        if cmd.target_visible && !self.line_of_sight(cmd.player, cmd.target) {
            return true;
        }
        
        false
    }
    
    // 4. Replay recording
    pub fn record_message(&mut self, msg: &BattleStateMessage) {
        self.replay_log.push(TimedMessage {
            frame: self.state.frame_i,
            message: msg.clone(),
        });
    }
}
```

---

## 15.5 Dealing with Latency

Latency is the enemy of multiplayer games. Tactical wargames have advantages here, but still require careful engineering.

### 15.5.1 Input Delay Techniques

Accepting a fixed input delay provides consistency:

```rust
pub struct InputDelaySystem {
    local_input_delay_frames: u32,  // e.g., 6 frames = 100ms at 60fps
    input_queue: VecDeque<(u32, PlayerInput)>,  // (frame, input)
}

impl InputDelaySystem {
    pub fn queue_input(&mut self, input: PlayerInput) {
        let execution_frame = self.current_frame + self.local_input_delay_frames;
        self.input_queue.push_back((execution_frame, input));
        
        // Send to server immediately, but scheduled for future frame
        self.network.send(DelayedInput {
            frame: execution_frame,
            input,
        });
    }
    
    pub fn get_inputs_for_frame(&mut self, frame: u32) -> Vec<PlayerInput> {
        let mut inputs = Vec::new();
        
        // Collect all inputs scheduled for this frame
        while let Some((input_frame, input)) = self.input_queue.front() {
            if *input_frame == frame {
                inputs.push(self.input_queue.pop_front().unwrap().1);
            } else if *input_frame < frame {
                // Missed input! Use default or last input
                self.input_queue.pop_front();
            } else {
                break;
            }
        }
        
        inputs
    }
}
```

**In Practice**:
- 6-frame delay (100ms) is usually imperceptible in tactical games
- Gives time for network transit before input execution
- All players see the same delayed input, ensuring consistency

### 15.5.2 Client-Side Prediction

Predict the result of player inputs locally for immediate feedback:

```rust
pub struct PredictionSystem {
    // The authoritative server state (delayed)
    server_state: BattleState,
    
    // Our predicted state (immediate)
    predicted_state: BattleState,
    
    // Inputs we've sent but not yet confirmed
    pending_inputs: Vec<PlayerInput>,
}

impl PredictionSystem {
    pub fn on_player_input(&mut self, input: PlayerInput) {
        // 1. Send to server
        self.network.send(input.clone());
        
        // 2. Predict result immediately
        let msg = input.to_message();
        self.predicted_state.apply(msg.clone());
        
        // 3. Remember we predicted this
        self.pending_inputs.push(input);
    }
    
    pub fn on_server_state(&mut self, server_state: BattleState) {
        // 1. Update authoritative state
        self.server_state = server_state;
        
        // 2. Reapply pending inputs to server state
        self.predicted_state = server_state.clone();
        for input in &self.pending_inputs {
            let msg = input.to_message();
            self.predicted_state.apply(msg);
        }
        
        // 3. Check for misprediction
        if self.predicted_state != self.server_state {
            // Server correction occurred
            // Options:
            // A) Snap to server state (jarring but accurate)
            // B) Blend to server state over several frames (smoother)
            self.blend_to_server_state();
        }
    }
    
    fn blend_to_server_state(&mut self) {
        // Gradually interpolate from predicted to server state
        // This hides the correction
        for soldier in &mut self.predicted_state.soldiers {
            let server_soldier = self.server_state.get_soldier(soldier.index);
            soldier.position = lerp(soldier.position, server_soldier.position, 0.3);
        }
    }
}
```

**Visual Feedback**:

Players need to know what's predicted vs confirmed:

```rust
pub fn render_unit(unit: &Unit, is_predicted: bool) {
    let color = if is_predicted {
        // Semi-transparent for predicted positions
        Color::rgba(1.0, 1.0, 1.0, 0.7)
    } else {
        // Solid for confirmed positions
        Color::WHITE
    };
    
    draw_sprite(unit.sprite, unit.position, color);
    
    if is_predicted {
        // Show predicted path
        draw_dashed_line(unit.position, unit.target_position);
    }
}
```

### 15.5.3 Server Reconciliation

When the server corrects client prediction, handle it gracefully:

```rust
pub enum ReconciliationStrategy {
    // Snap immediately (jarring, accurate)
    Snap,
    
    // Blend over time (smooth, may show wrong intermediate state)
    Blend { duration_ms: u32 },
    
    // Rewind and replay (complex, most accurate)
    RewindReplay,
}

impl PredictionSystem {
    pub fn reconcile(&mut self, server_state: BattleState, strategy: ReconciliationStrategy) {
        match strategy {
            ReconciliationStrategy::Snap => {
                self.predicted_state = server_state;
            }
            
            ReconciliationStrategy::Blend { duration_ms } => {
                self.target_state = server_state;
                self.blend_start_time = now();
                self.blend_duration = duration_ms;
                self.is_blending = true;
            }
            
            ReconciliationStrategy::RewindReplay => {
                // Complex: rewind to last server state, reapply all inputs
                self.rewind_and_replay(server_state);
            }
        }
    }
}
```

### 15.5.4 Entity Interpolation

For remote entities (other players' units), interpolate between known positions:

```rust
pub struct InterpolationSystem {
    // Buffer of received states
    state_buffer: VecDeque<(u64, BattleState)>,  // (timestamp, state)
    render_time_offset_ms: i64,  // Render 100ms behind latest
}

impl InterpolationSystem {
    pub fn on_state_update(&mut self, timestamp: u64, state: BattleState) {
        self.state_buffer.push_back((timestamp, state));
        
        // Keep buffer size reasonable
        while self.state_buffer.len() > 10 {
            self.state_buffer.pop_front();
        }
    }
    
    pub fn get_interpolated_state(&self, now: u64) -> BattleState {
        let render_time = now as i64 - self.render_time_offset_ms;
        
        // Find states before and after render_time
        let (before, after) = self.find_surrounding_states(render_time as u64);
        
        if let (Some((t1, s1)), Some((t2, s2))) = (before, after) {
            // Interpolate between s1 and s2
            let t = (render_time - t1 as i64) as f32 / (t2 - t1) as f32;
            BattleState::interpolate(s1, s2, t)
        } else {
            // Not enough data, use latest
            self.state_buffer.back().unwrap().1.clone()
        }
    }
}
```

### 15.5.5 Specific Challenges for Slow-Paced Tactical Games

Tactical wargames have unique latency considerations:

**Challenge 1: Long Actions**
A "sneak to position" order might take 30 seconds to complete. Players need feedback during this time:

```rust
pub struct LongActionFeedback {
    // Show progress indicators
    pub fn render_movement_progress(soldier: &Soldier) {
        let progress = soldier.distance_traveled / soldier.total_distance;
        draw_progress_bar(soldier.position, progress);
        
        // Show ETA
        let eta_seconds = soldier.remaining_distance / soldier.speed;
        draw_text(format!("ETA: {}s", eta_seconds), soldier.position);
    }
}
```

**Challenge 2: Multiple Simultaneous Orders**
Players issue orders to many units. Show acknowledgment for each:

```rust
pub fn on_order_issued(order: &Order) {
    // Visual feedback
    for unit in &order.target_units {
        spawn_acknowledgment_effect(unit.position);
        play_acknowledgment_sound(unit.unit_type);
    }
    
    // UI feedback
    ui.show_order_summary(order);
    
    // Cancel previous order marker if overriding
    if order.overrides_previous {
        ui.clear_order_markers(order.target_units);
    }
}
```

**Challenge 3: Fog of War Synchronization**
Fog of war requires careful handling:

```rust
pub struct FogOfWarSync {
    // Each client only knows what their side can see
    local_visibility: VisibilityMap,
    
    // Server tracks true state
    server_visibility: VisibilityMap,
}

impl FogOfWarSync {
    pub fn should_send_entity_to_player(
        &self,
        entity: &Entity,
        player_side: Side
    ) -> bool {
        // Only send entities the player can actually see
        self.server_visibility.is_visible_to(entity.position, player_side)
    }
}
```

---

## 15.6 Synchronization Strategies

### 15.6.1 State Snapshots

Send complete state periodically as a baseline:

```rust
pub struct SnapshotSync {
    full_snapshot_interval: Duration,  // Every 5 seconds
    last_full_snapshot: Instant,
}

impl SnapshotSync {
    pub fn update(&mut self, state: &BattleState) {
        if self.last_full_snapshot.elapsed() >= self.full_snapshot_interval {
            let snapshot = create_full_snapshot(state);
            let compressed = compress(&snapshot);
            self.broadcast(compressed);
            self.last_full_snapshot = Instant::now();
        }
    }
}

fn create_full_snapshot(state: &BattleState) -> StateSnapshot {
    StateSnapshot {
        frame: state.frame_i,
        soldiers: state.soldiers.iter().map(|s| s.to_snapshot()).collect(),
        projectiles: state.projectiles.iter().map(|p| p.to_snapshot()).collect(),
        terrain: state.terrain.to_snapshot(),
    }
}
```

### 15.6.2 Delta Compression

Only send what changed:

```rust
pub struct DeltaCompressor {
    last_sent_state: BattleState,
}

impl DeltaCompressor {
    pub fn compute_delta(&self, current: &BattleState) -> StateDelta {
        let mut delta = StateDelta::new();
        
        // Check each soldier
        for (idx, soldier) in current.soldiers.iter().enumerate() {
            let last = &self.last_sent_state.soldiers[idx];
            
            if soldier.position != last.position {
                delta.position_changes.push((idx, soldier.position));
            }
            
            if soldier.health != last.health {
                delta.health_changes.push((idx, soldier.health));
            }
            
            if soldier.behavior != last.behavior {
                delta.behavior_changes.push((idx, soldier.behavior.clone()));
            }
        }
        
        delta
    }
    
    pub fn estimate_bandwidth(&self, delta: &StateDelta) -> usize {
        // Calculate bytes needed
        delta.position_changes.len() * 12 +  // 12 bytes per position
        delta.health_changes.len() * 8 +      // 8 bytes per health
        delta.behavior_changes.len() * 16     // 16 bytes per behavior
    }
}
```

**Bandwidth Comparison**:

| Method | Snapshot Size | 20 Hz Bandwidth |
|--------|--------------|-----------------|
| Full snapshots | 50 KB | 1000 KB/s |
| Delta compression | 2-5 KB | 40-100 KB/s |
| Event sourcing | 0.5-1 KB | 10-20 KB/s |

### 15.6.3 Event Sourcing

OpenCombat's approach—send events, not state:

```rust
// Instead of sending "Soldier at (100, 200)", send "Soldier moved to (100, 200)"
pub enum GameEvent {
    SoldierMoved { soldier: SoldierIndex, from: WorldPoint, to: WorldPoint },
    SoldierFired { soldier: SoldierIndex, target: WorldPoint, weapon: WeaponType },
    SoldierTookDamage { soldier: SoldierIndex, amount: f32, source: DamageSource },
    Explosion { position: WorldPoint, radius: f32, damage: f32 },
}

// Bandwidth-efficient because events are small
// And clients can reconstruct state from event log
```

**Event Sourcing vs Delta Compression**:

- **Delta compression**: "What is the current state?"
- **Event sourcing**: "How did we get here?"

Event sourcing is superior for replay and debugging but requires deterministic simulation.

### 15.6.4 OpenCombat's Approach: Deterministic Simulation + Input Sync

OpenCombat combines the best approaches:

```rust
pub struct OpenCombatSync {
    // 1. Deterministic simulation core
    state: BattleState,
    
    // 2. Event sourcing for replay
    event_log: Vec<TimedMessage>,
    
    // 3. Delta compression for network efficiency
    delta_compressor: DeltaCompressor,
    
    // 4. Periodic full snapshots for recovery
    snapshot_timer: Timer,
}

impl OpenCombatSync {
    pub fn server_tick(&mut self) {
        // 1. Collect all inputs from clients
        let inputs = self.collect_inputs();
        
        // 2. Convert to messages
        let messages = inputs.into_iter().map(|i| i.to_message()).collect();
        
        // 3. Apply deterministically
        for msg in messages {
            self.state.apply(msg.clone());
            self.event_log.push(TimedMessage {
                frame: self.state.frame_i,
                message: msg,
            });
        }
        
        // 4. Run simulation
        self.update_simulation();
        
        // 5. Broadcast state delta
        let delta = self.delta_compressor.compute_delta(&self.state);
        self.broadcast(delta);
        
        // 6. Send full snapshot periodically
        if self.snapshot_timer.finished() {
            let snapshot = create_full_snapshot(&self.state);
            self.broadcast(snapshot);
        }
    }
}
```

---

## 15.7 Security and Anti-Cheat

### 15.7.1 Authoritative Server

The foundation of security:

```rust
pub struct SecurityLayer {
    // All game logic runs here, nowhere else
    state: BattleState,
}

impl SecurityLayer {
    pub fn process_command(&mut self, cmd: Command, player: PlayerId) -> Result<(), SecurityError> {
        // 1. Authentication
        if !self.authenticate(player) {
            return Err(SecurityError::Unauthorized);
        }
        
        // 2. Authorization
        if !self.authorize(player, &cmd) {
            return Err(SecurityError::Forbidden);
        }
        
        // 3. Validation
        if !self.validate(&cmd) {
            self.log_suspicious_activity(player, &cmd);
            return Err(SecurityError::InvalidCommand);
        }
        
        // 4. Rate limiting
        if self.is_rate_limited(player) {
            return Err(SecurityError::RateLimited);
        }
        
        // 5. Execute
        self.execute(cmd);
        Ok(())
    }
}
```

### 15.7.2 Replay Validation

Record everything, verify later:

```rust
pub struct ReplayValidator {
    replay: Replay,
}

impl ReplayValidator {
    pub fn validate(&self) -> ValidationResult {
        let mut state = self.replay.initial_state.clone();
        
        for timed_msg in &self.replay.messages {
            // Replay the message
            state.apply(timed_msg.message.clone());
            
            // Run one tick
            update_simulation(&mut state);
            
            // Verify no anomalies
            if !self.verify_state(&state) {
                return ValidationResult::Invalid(format!(
                    "Anomaly at frame {}", timed_msg.frame
                ));
            }
        }
        
        ValidationResult::Valid
    }
    
    fn verify_state(&self, state: &BattleState) -> bool {
        // Check for impossible values
        for soldier in &state.soldiers {
            if soldier.health < 0.0 || soldier.health > 100.0 {
                return false;
            }
            
            if soldier.position.x.is_nan() || soldier.position.y.is_nan() {
                return false;
            }
        }
        
        true
    }
}
```

### 15.7.3 State Hash Verification

Periodically verify all clients are synchronized:

```rust
pub struct SyncVerifier {
    last_verification_frame: u64,
    verification_interval: u64,
}

impl SyncVerifier {
    pub fn should_verify(&self, frame: u64) -> bool {
        frame - self.last_verification_frame >= self.verification_interval
    }
    
    pub fn compute_state_hash(&self, state: &BattleState) -> u64 {
        let mut hasher = XxHash64::default();
        
        // Hash critical state
        for soldier in &state.soldiers {
            hasher.write_u64(soldier.position.x.as_bits());
            hasher.write_u64(soldier.position.y.as_bits());
            hasher.write_u32(soldier.health.to_bits());
        }
        
        hasher.write_u64(state.frame_i);
        
        hasher.finish()
    }
    
    pub fn verify_sync(&self, client_hashes: HashMap<PlayerId, u64>, server_hash: u64) {
        for (player_id, client_hash) in client_hashes {
            if client_hash != server_hash {
                println!("DESYNC DETECTED: Player {} at frame {}", 
                    player_id, self.last_verification_frame);
                
                // Request client state for debugging
                self.request_client_state(player_id);
            }
        }
    }
}
```

### 15.7.4 Encryption

Protect network traffic:

```rust
pub struct EncryptedConnection {
    cipher: ChaCha20Poly1305,
}

impl EncryptedConnection {
    pub fn send_encrypted(&self, msg: &GameMessage) -> Vec<u8> {
        let plaintext = serialize(msg);
        let nonce = generate_nonce();
        let ciphertext = self.cipher.encrypt(&nonce, plaintext.as_ref())
            .expect("encryption failure");
        
        // Prepend nonce for decryption
        let mut result = nonce.to_vec();
        result.extend(ciphertext);
        result
    }
    
    pub fn receive_encrypted(&self, data: &[u8]) -> Result<GameMessage, DecryptError> {
        let (nonce_bytes, ciphertext) = data.split_at(12);
        let nonce = Nonce::from_slice(nonce_bytes);
        
        let plaintext = self.cipher.decrypt(nonce, ciphertext)?;
        
        Ok(deserialize(&plaintext)?)
    }
}
```

---

## 15.8 Spectator Mode and Replays

### 15.8.1 Replay File Format

```rust
pub struct ReplayFile {
    // Header
    pub version: u32,
    pub game_version: String,
    pub map_name: String,
    pub game_mode: GameMode,
    pub player_info: Vec<PlayerInfo>,
    pub initial_state: BattleState,
    pub random_seed: u64,
    
    // Body - compressed message stream
    pub messages: Vec<TimedMessage>,
    
    // Optional: full snapshots for seeking
    pub snapshots: Vec<(u64, StateSnapshot)>,  // (frame, snapshot)
}

impl ReplayFile {
    pub fn save(&self, path: &Path) -> Result<(), SaveError> {
        let file = File::create(path)?;
        let mut writer = BufWriter::new(file);
        
        // Write header
        writer.write_u32::<BigEndian>(self.version)?;
        writer.write_all(self.game_version.as_bytes())?;
        
        // Write compressed messages
        let compressed = compress_messages(&self.messages);
        writer.write_u64::<BigEndian>(compressed.len() as u64)?;
        writer.write_all(&compressed)?;
        
        Ok(())
    }
    
    pub fn load(path: &Path) -> Result<Self, LoadError> {
        let file = File::open(path)?;
        let mut reader = BufReader::new(file);
        
        let version = reader.read_u32::<BigEndian>()?;
        if version != CURRENT_VERSION {
            return Err(LoadError::VersionMismatch);
        }
        
        // Read and decompress messages
        let compressed_len = reader.read_u64::<BigEndian>()?;
        let mut compressed = vec![0u8; compressed_len as usize];
        reader.read_exact(&mut compressed)?;
        let messages = decompress_messages(&compressed)?;
        
        Ok(ReplayFile {
            version,
            messages,
            // ... other fields
        })
    }
}
```

### 15.8.2 Deterministic Replay Requirements

For perfect replay:

```rust
pub struct ReplayRequirements {
    // 1. Fixed timestep
    pub tick_rate: u32,  // Must be constant
    
    // 2. Seeded RNG
    pub random_seed: u64,  // Same seed = same random numbers
    
    // 3. Deterministic physics
    pub use_fixed_point: bool,  // No floating-point accumulation
    
    // 4. No external inputs
    pub no_user_input_during_replay: bool,
    
    // 5. Same game version
    pub exact_version_match: bool,
}

pub fn verify_replay_compatibility(
    replay: &ReplayFile,
    current_version: &str
) -> Result<(), ReplayError> {
    if replay.game_version != current_version {
        // Check if versions are compatible
        if !are_versions_compatible(&replay.game_version, current_version) {
            return Err(ReplayError::VersionMismatch);
        }
    }
    
    Ok(())
}
```

### 15.8.3 Spectator Features

```rust
pub struct SpectatorSystem {
    replay: Replay,
    current_frame: u64,
    playback_speed: f32,
    is_playing: bool,
}

impl SpectatorSystem {
    pub fn seek_to_frame(&mut self, frame: u64) {
        // Find nearest snapshot before target frame
        let snapshot = self.find_nearest_snapshot(frame);
        
        // Restore from snapshot
        self.state = snapshot.state.clone();
        self.current_frame = snapshot.frame;
        
        // Replay messages up to target
        for msg in &self.replay.messages {
            if msg.frame > self.current_frame && msg.frame <= frame {
                self.state.apply(msg.message.clone());
                self.current_frame = msg.frame;
            }
        }
    }
    
    pub fn set_playback_speed(&mut self, speed: f32) {
        self.playback_speed = speed.clamp(0.25, 4.0);
    }
    
    pub fn render_spectator_ui(&self) {
        // Timeline scrubber
        ui.slider("Timeline", self.current_frame, 0..self.replay.total_frames);
        
        // Playback controls
        if ui.button("Play/Pause") { self.toggle_playback(); }
        ui.button("<< Rewind 10s");
        ui.button("Fast Forward 10s >>");
        
        // Speed control
        ui.slider("Speed", self.playback_speed, [0.25, 0.5, 1.0, 2.0, 4.0]);
        
        // Unit inspection
        if let Some(unit) = ui.hovered_unit() {
            ui.show_unit_details(unit);
        }
    }
}
```

---

## 15.9 Implementation Guide

### 15.9.1 Making a Single-Player Game Multiplayer-Ready

Converting OpenCombat-SDL to multiplayer would require these steps:

**Phase 1: Extract Deterministic Core**

```cpp
// Before: Everything in World class
class World {
    void Update(float dt);  // Non-deterministic, uses real time
};

// After: Separate simulation from rendering
class Simulation {
    void Tick(uint32_t frame);  // Deterministic, uses frame count
};

class Renderer {
    void Render(const SimulationState& state, float interpolation);
};
```

**Phase 2: Message-Based State Updates**

```cpp
// Before: Direct modification
void Soldier::TakeDamage(float amount) {
    health -= amount;
}

// After: Message-based
void Soldier::TakeDamage(float amount) {
    auto msg = std::make_shared<DamageMessage>(
        this->GetId(), amount
    );
    MessageQueue::GetInstance()->Post(msg);
}

void BattleState::Apply(const DamageMessage& msg) {
    auto* soldier = GetSoldier(msg.soldier_id);
    if (soldier) {
        soldier->health -= msg.amount;
    }
}
```

**Phase 3: Replace Floating-Point**

```cpp
// Before: Float positions
struct Position {
    float x, y;
};

// After: Fixed-point positions
struct Position {
    FixedPoint x, y;  // 16.16 fixed-point
    
    Position operator+(const Position& other) const {
        return {x + other.x, y + other.y};
    }
};
```

**Phase 4: Add Seeded RNG**

```cpp
// Before: Platform RNG
int RollDice() {
    return rand() % 6 + 1;
}

// After: Seeded RNG
class DeterministicRNG {
    uint64_t state;
    
public:
    uint64_t Next() {
        state ^= state << 13;
        state ^= state >> 7;
        state ^= state << 17;
        return state;
    }
    
    int RollDice() {
        return (Next() % 6) + 1;
    }
};
```

**Phase 5: Add Network Layer**

```cpp
// Server
class GameServer {
    Simulation simulation;
    NetworkSocket socket;
    
    void Run() {
        while (running) {
            // Receive commands
            auto commands = socket.ReceiveCommands();
            for (auto& cmd : commands) {
                if (ValidateCommand(cmd)) {
                    auto msg = cmd.ToMessage();
                    simulation.Apply(msg);
                }
            }
            
            // Tick simulation
            simulation.Tick();
            
            // Broadcast state
            auto delta = simulation.ComputeDelta();
            socket.Broadcast(delta);
        }
    }
};

// Client
class GameClient {
    Simulation localSimulation;
    Simulation predictedSimulation;
    NetworkSocket serverConnection;
    
    void IssueCommand(Command cmd) {
        // Predict locally
        auto msg = cmd.ToMessage();
        predictedSimulation.Apply(msg);
        
        // Send to server
        serverConnection.Send(cmd);
    }
    
    void OnStateUpdate(StateUpdate update) {
        localSimulation = update.state;
        
        // Reconcile
        if (predictedSimulation != localSimulation) {
            predictedSimulation = localSimulation;
        }
    }
};
```

### 15.9.2 Architecture Decisions for Deterministic Simulation

```
DECISION MATRIX: Multiplayer Architecture

┌──────────────────────────────────────────────────────────────┐
│ Question: What type of multiplayer?                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Competitive PvP → Deterministic + Authoritative Server     │
│  ├─ OpenCombat approach                                     │
│  ├─ Fixed timestep                                          │
│  ├─ Message-based sync                                      │
│  └─ Server validation                                       │
│                                                              │
│  Cooperative PvE → Deterministic or Non-deterministic       │
│  ├─ Either approach works                                   │
│  ├─ Non-deterministic is easier                             │
│  └─ Deterministic enables replay                            │
│                                                              │
│  Casual/Social → Non-deterministic                          │
│  ├─ State snapshots                                         │
│  ├─ Event replication                                       │
│  └─ Simpler implementation                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 15.9.3 Testing Determinism

```rust
#[test]
fn test_deterministic_simulation() {
    // Create two identical simulations
    let mut sim1 = create_test_simulation();
    let mut sim2 = create_test_simulation();
    
    // Same seed
    sim1.rng = DeterministicRNG::new(12345);
    sim2.rng = DeterministicRNG::new(12345);
    
    // Same inputs
    let inputs = generate_test_inputs(1000);
    
    // Run both simulations
    for input in inputs {
        sim1.apply(input.clone());
        sim2.apply(input);
        
        sim1.tick();
        sim2.tick();
        
        // States must match exactly
        assert_eq!(sim1.state_hash(), sim2.state_hash(),
            "Desync at frame {}", sim1.frame);
    }
}

#[test]
fn test_cross_platform_determinism() {
    // Run simulation on different platforms
    let results = vec![
        run_on_platform(Platform::Windows),
        run_on_platform(Platform::Linux),
        run_on_platform(Platform::MacOS),
    ];
    
    // All results must match
    for i in 1..results.len() {
        assert_eq!(results[0], results[i],
            "Platform {} differs from Windows", i);
    }
}

#[test]
fn test_floating_point_consistency() {
    // Test that floating-point operations are consistent
    let a = 0.1_f32;
    let b = 0.2_f32;
    let c = a + b;
    
    // This will pass on all platforms with IEEE 754 compliance
    assert_eq!(c, 0.30000001192092896);
}
```

---

## 15.10 Case Study: Converting OpenCombat-SDL to Multiplayer

### 15.10.1 What Would Need to Change

**Current Architecture** (Single-Player):

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenCombat-SDL                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   World Class                        │   │
│  │  ├─ Direct state modification                       │   │
│  │  ├─ Floating-point positions                        │   │
│  │  ├─ Platform RNG                                    │   │
│  │  └─ Real-time updates                               │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                  │
│                    ┌──────┴──────┐                          │
│                    │ Rendering   │                          │
│                    └─────────────┘                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Target Architecture** (Multiplayer):

```
┌─────────────────────────────────────────────────────────────┐
│              OpenCombat-SDL Multiplayer                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐          ┌─────────────────────────────┐  │
│  │   SERVER     │          │         CLIENT              │  │
│  │              │          │                             │  │
│  │ Authoritative│◄────────►│  ┌─────────────────────┐   │  │
│  │ Simulation   │  Network │  │  Predicted State    │   │  │
│  │              │          │  └─────────────────────┘   │  │
│  │ Message-     │          │            │               │  │
│  │ driven       │          │  ┌─────────┴─────────┐     │  │
│  └──────────────┘          │  │  Reconciliation   │     │  │
│                            │  └─────────┬─────────┘     │  │
│                            │            │               │  │
│                            │  ┌─────────┴─────────┐     │  │
│                            │  │  Render State     │     │  │
│                            │  │  (Interpolated)   │     │  │
│                            │  └───────────────────┘     │  │
│                            └─────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 15.10.2 State Management Modifications

**1. Convert State to Message-Based**:

```cpp
// Current: Direct state in Soldier class
class Soldier {
    State _currentState;
    
public:
    void SetState(unsigned int state) {
        _currentState.Set(state);  // Immediate, non-reversible
    }
};

// Required: Message-based state
class Soldier {
    State _currentState;
    
public:
    void RequestStateChange(unsigned int state) {
        auto msg = std::make_shared<StateChangeMessage>(
            GetId(), state
        );
        g_GameState->PostMessage(msg);  // Queued, reversible
    }
    
    void ApplyMessage(const StateChangeMessage& msg) {
        _currentState.Set(msg.newState);
    }
};
```

**2. Add Three-Tier State Hierarchy**:

```cpp
// Current: 64-bit bitfield (simple but limiting)
class State {
    uint64_t _bits;
};

// Required: Hierarchical with capability overlay
struct UnitState {
    Phase phase;              // Global game phase
    Behavior behavior;        // Tactical behavior
    Gesture gesture;          // Physical action
    uint64_t capabilities;    // Bitfield for orthogonal states
};

enum class Phase {
    Placement,
    Battle,
    End
};

enum class Behavior {
    Idle,
    MoveTo(WorldPath),
    Defend(Angle),
    Engage(SoldierIndex),
    Dead
};

enum class Gesture {
    Idle,
    Reloading(uint64_t endFrame),
    Aiming(uint64_t endFrame),
    Firing(uint64_t endFrame)
};
```

**3. Centralize State in BattleState**:

```cpp
// Current: State distributed in objects
class World {
    std::vector<Soldier*> _soldiers;
    std::vector<Vehicle*> _vehicles;
    
    void Update() {
        for (auto* s : _soldiers) s->Simulate();
        for (auto* v : _vehicles) v->Simulate();
    }
};

// Required: Centralized state
class BattleState {
    std::vector<Soldier> soldiers;  // Contiguous storage
    std::vector<Vehicle> vehicles;
    uint64_t frame_i;
    
    void Apply(const BattleStateMessage& msg);
    void Tick();  // Fixed timestep
};
```

### 15.10.3 Serialization Requirements

**Current**: No serialization (single-player only)

**Required**: Full serialization for network sync:

```cpp
// Binary serialization for network efficiency
class Serializer {
public:
    template<typename T>
    void Write(const T& value);
    
    void WriteString(const std::string& str);
    void WriteBytes(const std::vector<uint8_t>& bytes);
};

// Example: Serialize BattleStateMessage
void SerializeBattleStateMessage(
    const BattleStateMessage& msg,
    Serializer& ser
) {
    ser.Write<uint8_t>(static_cast<uint8_t>(msg.type));
    
    switch (msg.type) {
        case MessageType::SoldierState:
            ser.Write<uint32_t>(msg.soldier_index);
            ser.Write<uint8_t>(static_cast<uint8_t>(msg.behavior));
            ser.Write<float>(msg.position.x);
            ser.Write<float>(msg.position.y);
            break;
            
        case MessageType::CombatEvent:
            ser.Write<uint32_t>(msg.attacker);
            ser.Write<uint32_t>(msg.target);
            ser.Write<float>(msg.damage);
            break;
            
        // ... other message types
    }
}

// Deserialize
BattleStateMessage DeserializeMessage(Deserializer& des) {
    auto type = static_cast<MessageType>(des.Read<uint8_t>());
    
    switch (type) {
        case MessageType::SoldierState:
            return SoldierStateMessage {
                .soldier_index = des.Read<uint32_t>(),
                .behavior = static_cast<Behavior>(des.Read<uint8_t>()),
                .position = {
                    des.Read<float>(),
                    des.Read<float>()
                }
            };
            
        // ... other cases
    }
}
```

**Bandwidth Optimization**:

```cpp
// Delta compression: only send what changed
class DeltaCompressor {
    BattleState last_sent_state;
    
public:
    std::vector<BattleStateMessage> ComputeDelta(
        const BattleState& current
    ) {
        std::vector<BattleStateMessage> deltas;
        
        for (size_t i = 0; i < current.soldiers.size(); i++) {
            const auto& curr = current.soldiers[i];
            const auto& last = last_sent_state.soldiers[i];
            
            if (curr.position != last.position) {
                deltas.push_back(SoldierPositionMessage {
                    i, curr.position
                });
            }
            
            if (curr.behavior != last.behavior) {
                deltas.push_back(SoldierBehaviorMessage {
                    i, curr.behavior
                });
            }
            
            // Only include health changes if significant
            if (std::abs(curr.health - last.health) > 1.0f) {
                deltas.push_back(SoldierHealthMessage {
                    i, curr.health
                });
            }
        }
        
        last_sent_state = current.clone();
        return deltas;
    }
};
```

### 15.10.4 Conversion Timeline

Converting OpenCombat-SDL to multiplayer would require approximately:

**Phase 1: Core Refactoring (2-3 months)**
- Extract deterministic simulation core
- Implement message-based state updates
- Replace floating-point with fixed-point
- Add seeded RNG

**Phase 2: Network Layer (1-2 months)**
- Implement client-server architecture
- Add serialization/deserialization
- Implement delta compression
- Add basic prediction and reconciliation

**Phase 3: Security & Polish (1-2 months)**
- Add server-side validation
- Implement anti-cheat measures
- Add replay recording
- Optimize bandwidth usage

**Total: 4-7 months for a small team (2-3 developers)**

---

## 15.11 Conclusion

Multiplayer transforms every aspect of tactical wargame architecture. The lessons from this chapter can be summarized in five principles:

### The Five Principles of Multiplayer Architecture

**1. Determinism is Non-Negotiable**
For competitive multiplayer, deterministic simulation is the only viable foundation. Without it, synchronization becomes impossible and exploits inevitable.

**2. Server Authority is Sacred**
Never trust the client. The server must validate every action, compute every state change, and serve as the single source of truth.

**3. Messages Over State**
Transmit intentions (messages/events), not results (state snapshots). This enables replay, reduces bandwidth, and simplifies debugging.

**4. Embrace Latency**
Work with latency, not against it. Input delay, client prediction, and server reconciliation can make 200ms feel responsive.

**5. Security is Architecture**
Anti-cheat isn't a feature you add later; it's a consequence of your architectural choices. Authoritative server, replay validation, and state verification must be designed in from the start.

### The Multiplayer Decision Matrix

| If Your Game Is... | Use This Approach |
|-------------------|-------------------|
| Competitive PvP | Deterministic + Authoritative Server (OpenCombat model) |
| Cooperative PvE | Deterministic recommended, non-deterministic acceptable |
| Casual/Social | Non-deterministic (snapshots or events) |
| LAN-only | Peer-to-peer with lockstep |
| Single-player now, multiplayer later | Design for determinism from day one |

### Final Recommendation

If you are building a tactical wargame and even *considering* multiplayer in the future, follow OpenCombat's architectural patterns from the beginning. Retrofitting determinism into a non-deterministic codebase is a herculean effort that often fails. The patterns in this chapter—message-driven state updates, fixed timestep simulation, three-tier state hierarchy, and server-authoritative architecture—are not just "best practices." They are survival requirements for multiplayer tactical games.

The choice between single-player and multiplayer is not a feature toggle. It is an architectural commitment that shapes every system in your game. Choose wisely, commit fully, and build with confidence.

---

**Key Takeaways**:

1. **Deterministic simulation** enables synchronization, replay, and verification
2. **Message-based updates** provide network transparency and debuggability  
3. **Server-authoritative** architecture prevents cheating
4. **Latency compensation** techniques make slow-paced games feel responsive
5. **Retrofitting multiplayer** is significantly harder than designing for it initially

---

*This chapter examined multiplayer architecture through the lens of the three Close Combat clones, with particular focus on OpenCombat's production-ready deterministic simulation system. The patterns presented here represent the current state of the art for tactical wargame networking.*

*Version: 1.0 (February 2026)*