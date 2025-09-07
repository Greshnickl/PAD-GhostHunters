# Ghost Hunters

## Game Microservices – Service Boundaries

This project is structured around a microservices architecture to ensure modularity, scalability, and independent deployment of each component. Each service is responsible for a specific domain of the game and communicates with others where required.

## Services Overview
1. User Management Service

### Responsibilities:

- Manage user accounts and authentication (email, username, password).

- Store and update player metadata (level, in-game currency).

- Implement a friend system (add, remove, list friends).

### Service Boundaries:

- Handles only user-related data and relationships.

- Does not directly interact with game logic or AI.

### Interfaces/Consumers:

### Provides APIs for:

- User authentication & profile data.

- Friend lists (for social and cooperative play).

### Consumed by:

- Game Service (for retrieving player info during active sessions).

---

## 2. Ghost AI Service

### Responsibilities:

- Controls the game’s ghost AI behavior.

- Each ghost runs as a separate Thread/Actor with its own decision-making.

### Processes:

   -- Map layout.
   -- Difficulty settings.
   -- Player sanity levels.
   -- Movable/interactable objects.
   -- Player targeting and attack decisions.

- Sends AI state updates to the Game Service.

### Service Boundaries:

- Does not manage user data.

- Encapsulates all ghost logic independent of the game state store.

### Interfaces/Consumers:

- Provides APIs/events for ghost state changes (hiding, haunting, interacting).

### Consumed by:

- Game Service (to update lobby and broadcast changes to players).
