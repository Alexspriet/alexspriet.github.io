flowchart LR
  subgraph Clients [Clients (Players)]
    A[Player A]
    B[Player B]
    C[Player C]
    D[Player D]
  end

  subgraph Network[ ]
    direction TB
    STUN[STUN Server]
    TURN_EU[TURN Cluster - EU]
    TURN_NA[TURN Cluster - NA]
    MM[Matchmaker (EOS/PlayFab/custom)]
    AUTH[Auth / Accounts]
    ORCH[Orchestrator (GameLift/Multiplay/K8s)]
    POOL[Fallback Game Pool]
    LOGS[Telemetry & Logging]
    DB[User DB / Leaderboards]
    AC[Anti-cheat Service]
  end

  A -- STUN probe --> STUN
  B -- STUN probe --> STUN
  C -- STUN probe --> STUN
  D -- STUN probe --> STUN

  STUN -- result --> MM
  AUTH -- accounts --> MM
  MM -- host-selection --> A
  MM -- host-feedback --> ORCH

  %% P2P attempts
  A -- direct P2P --> B
  A -- direct P2P --> C
  A -- direct P2P --> D

  %% TURN fallback per-client when direct P2P fails
  B -- fallback alloc --> TURN_EU
  C -- fallback alloc --> TURN_NA

  TURN_EU -- relay --> A
  TURN_NA -- relay --> A

  %% Orchestrator fallback path
  MM -- trigger fallback --> ORCH
  ORCH -- spawn --> POOL
  POOL -- authoritative --> A
  POOL -- authoritative --> B
  POOL -- authoritative --> C
  POOL -- authoritative --> D

  %% Observability
  A -- telemetry --> LOGS
  B -- telemetry --> LOGS
  C -- telemetry --> LOGS
  D -- telemetry --> LOGS
  POOL -- telemetry --> LOGS
  TURN_EU -- metrics --> LOGS
  TURN_NA -- metrics --> LOGS

  LOGS -- aggregated --> DB
  DB -- leaderboards --> MM
  AC -- signals --> ORCH

  click MM "" "Matchmaker endpoint"
