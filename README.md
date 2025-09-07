# Ghost Hunters

## Game Microservices – Service Boundaries

This project is structured around a microservices architecture to ensure modularity, scalability, and independent deployment of each component. Each service is responsible for a specific domain of the game and communicates with others where required.

## Services Overview
1. User Management Service
=======

## 1. User Management Service

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

- ### Processes:

   - Map layout.
   - Difficulty settings.
   - Player sanity levels.
   - Movable/interactable objects.
   - Player targeting and attack decisions.

- Sends AI state updates to the Game Service.

### Service Boundaries:

- Does not manage user data.

- Encapsulates all ghost logic independent of the game state store.

### Interfaces/Consumers:

- Provides APIs/events for ghost state changes (hiding, haunting, interacting).

### Consumed by:

- Game Service (to update lobby and broadcast changes to players).

---

## 3. Shop Service

### Responsibilities:

- Provides catalog of purchasable items (title, description, durability, price).

- Maintains price history.

### Service Boundaries:

- Handles item data only.

###Consumers:

 Game Service, Inventory Service.

---

## 4. Journal Service

### Responsibilities:

- Allows users to record symptoms and ghost guesses.

- Awards currency based on correct entries after games.

### Service Boundaries:

- Handles journal entries only.

### Consumers:

- Game Service for reward processing.

---

## 5. Lobby Service

### Responsibilities:

- Tracks active game sessions, players in them, their sanity, death status.

- Manages items brought into the session and their holders.

- Tracks ghost type and map for each session.

### Service Boundaries:

- Maintains session state and metadata only.

### Consumers:

- Game Service to coordinate gameplay.

---

## 6. Game Service (Coordinator)

### Responsibilities:

- Orchestrates game sessions, lobbies, and player interactions.

- Communicates with all other services to manage real-time gameplay.

### Service Boundaries:

- Does not store domain-specific data (users, ghosts, items).

### Consumers:

-Players/Clients.

- Players via Game Service.
