# Ghost Hunters

## Game Microservices – Service Boundaries

This project is built using a **microservices architecture** to ensure modularity, scalability, and independent deployment of each component. Each service is implemented in a language chosen to best fit its specific responsibilities, allowing the team to leverage **polyglot programming** for optimal performance and maintainability.

We selected **JavaScript (Node.js)**, **C# (.NET)**, and **Python** for this project:  
- **JavaScript (Node.js)** is used for services requiring high concurrency and fast development cycles, such as user management and ghost AI. Its event-driven model and rich ecosystem make it ideal for real-time operations and rapid prototyping.  
- **C# (.NET)** is used for services that demand strong type safety, reliability, and transactional consistency, such as Shop and Journal. It provides excellent performance under load and robust tooling for enterprise-grade APIs.  
- **Python** is chosen for services that benefit from flexibility, simplicity, and fast iteration, such as Lobby and Map. Its async capabilities and mature libraries allow efficient handling of multiple game sessions and dynamic content management.

This combination of technologies allows us to **balance speed of development, runtime performance, and maintainability**, ensuring each service meets the unique demands of the game while allowing teams to work efficiently within their expertise.


## Services Overview


## 1. User Management Service

### Responsibilities:
- Manage user accounts and authentication (email, username, password).
- Store and update player metadata (level, in-game currency).
- Implement a friend system (add, remove, list friends).

### Service Boundaries:
- Handles only user-related data and social relationships.
- Does not manage game session logic, AI behavior, or inventory.
- Provides identity and social graph for other services to consume.

### Interfaces / Consumers:

**Provides APIs for:**
- User authentication and profile management.
- Currency balance queries and updates.
- Friend system operations.

**Consumed by:**
- Lobby Service (to fetch player info for sessions).
- Inventory Service (currency validation for purchases).
- Game Service (to load player profile into active sessions).


---

## 2. Ghost AI Service

### Responsibilities:
- Controls ghost AI behavior within each game lobby.
- Each ghost runs as an independent Thread/Actor (can be simulated with Node.js worker threads or clustered processes).
- Processes contextual data, such as:
  - Map layout.
  - Difficulty settings.
  - Player sanity levels.
  - Movable/interactable objects.
  - Player targeting and attack decisions.
- Relays ghost state changes to the Game Service.

### Service Boundaries:
- Encapsulates all ghost decision-making logic.
- Does not store user data or session management.
- Operates independently of other services, but shares state updates as needed.

### Interfaces / Consumers:

**Provides APIs/events for:**
- Ghost state changes (hiding, haunting, interacting).
- AI decisions relevant to player interactions.

**Consumed by:**
- Game Service (to update lobby state and broadcast ghost activity to players).
- Lobby Service (to synchronize ghost type and behavior with session state).


--- 


## 3. Shop Service

### Responsibilities:
- Provides a catalog of purchasable items:
  - Title, description, durability (number of sessions usable).
  - Current price and full price history.
- Ensures items and prices are accessible to players during gameplay.
- Allows dynamic price updates with historical tracking.

### Service Boundaries:
- Handles only item-related data and price management.
- Does not manage player ownership (handled by Inventory Service).
- Independent from gameplay logic, only provides catalog information and updates.

### Interfaces / Consumers:

**Provides APIs for:**
- Listing items with descriptions, durability, and prices.
- Retrieving price history for each item.
- Updating prices.

**Consumed by:**
- Inventory Service (to validate item purchases).
- Lobby Service (to load selected items into sessions).
- Game Service (to display purchasable content).


---

## 4. Journal Service

### Responsibilities:
- Allows players to record:
  - Symptoms observed during exploration.
  - Type of ghost they believe they encountered.
- Stores journal entries per user and session.
- Validates journal entries at the end of a session by comparing them to the actual ghost type.
- Awards in-game currency for accurate records.

### Service Boundaries:
- Encapsulates only journal-related data and validation.
- Does not manage user authentication, ghost AI, or currency itself (delegates currency update to User Management Service).
- Independent of session management, only consumes final game state to validate entries.

### Interfaces / Consumers:

**Provides APIs for:**
- Creating and updating journal entries.
- Retrieving entries by user or session.
- Validating journal results at game end.

**Consumed by:**
- Lobby Service (to know which player journals to check).
- User Management Service (to reward currency for correct guesses).
- Game Service (to compare journal entries with actual ghost).


---

## 5. Lobby Service

### Responsibilities:
- Manages **active game sessions**, storing:
  - Difficulty level.
  - Players in the session, with:
    - Sanity levels.
    - Death status.
  - Items brought into the session and their current holders.
  - Ghost type for the session.
  - Map currently used in the lobby.
- Ensures items are expired along with dead players (removal from session inventory).

### Service Boundaries:
- Encapsulates only **session-related state and metadata**.
- Does not manage:
  - User authentication (delegated to User Management Service).
  - Ghost behavior (delegated to Ghost AI Service).
  - Item catalog (delegated to Shop Service).
- Acts as the **single source of truth** for an ongoing or paused session.

### Interfaces / Consumers:

**Provides APIs for:**
- Creating, updating, and resuming game sessions.
- Managing session players (join, leave, update status).
- Tracking sanity, deaths, and item ownership.
- Associating map and ghost type with a session.

**Consumed by:**
- Game Service (to run the actual gameplay loop).
- User Management Service (to sync player profiles and stats).
- Shop Service & Inventory Service (to validate items used in the lobby).
- Ghost AI Service (to get lobby info for ghost behavior).

 

---

## 6. Map Service

### Responsibilities:
- Provides tools for users to **create and manage maps**, including:
  - Houses with rooms connected to each other.
  - Placement of objects inside rooms.
  - Definition of hiding places (inaccessible to ghosts).
- Stores predefined maps and allows dynamic map creation (custom maps).
- Provides APIs for retrieving map layouts for gameplay.

### Service Boundaries:
- Focuses only on **map structure and object placement**.
- Does not:
  - Handle ghost AI (only provides hiding spots).
  - Manage players or items in the map (handled by Lobby & Inventory).
  - Track session state (delegated to Lobby Service).
- Acts as a **map provider** to other services.

### Interfaces / Consumers:

**Provides APIs for:**
- Creating new houses (with rooms and objects).
- Modifying or shuffling object placement.
- Querying hiding spots and room connectivity.
- Retrieving stored maps.

**Consumed by:**
- Lobby Service (to assign a map to a session).
- Ghost AI Service (to know where hiding spots and objects are).
- Game Service (to render maps for players).


---

## 7. Ghost Service

### Responsibilities:
- Acts as an encyclopedia of all ghost types in the game world.
- Stores and provides information about ghost **Type A symptoms**:
  - Breaking mirrors, moving objects, creating cold spots, etc.
  - Visible to players for guessing purposes.
- Stores **Type B symptoms**:
  - Hunting behavior based on player sanity or group status.
  - Used for players to discover through observation.
- Provides ghost data for gameplay mechanics and player education.

### Service Boundaries:
- Encapsulates only ghost-related information and symptoms.
- Does not manage user data, session states, or player inventory.
- Independent of game sessions; only consumed by other services for ghost info.

### Interfaces / Consumers:

**Provides APIs for:**
- Retrieving ghost types and symptoms (A & B).
- Listing ghosts available in specific maps or sessions.

**Consumed by:**
- Lobby Service (to know which ghost is active in the session).
- Game Service (for gameplay mechanics and player feedback).
- Journal Service (to validate player observations).

### Trade-offs:
- **Pros:** Centralized ghost information ensures consistency across the game. Easy to expand with new ghost types or symptoms.  
- **Cons:** Requires synchronization with session state if ghost behavior changes dynamically. Large data sets may require caching for performance.  

---

## 8. Location Service

### Responsibilities:
- Tracks player movements in real-time.
- Records:
  - Current room location.
  - Items the player interacts with.
  - Player state (alone/in group, speaking, hiding, visible).
  - Timestamps for each update.
- Provides fresh location data to other services for gameplay mechanics.

### Service Boundaries:
- Only responsible for location tracking and real-time player states.
- Does not manage user accounts, game sessions, or AI behavior.
- Operates independently but provides data for session and game logic.

### Interfaces / Consumers:

**Provides APIs for:**
- Querying current player location and state.
- Subscribing to location updates (for live sessions).

**Consumed by:**
- Lobby Service (to manage session state and item interactions).
- Ghost AI Service (to make decisions based on player positions).
- Game Service (for real-time player visibility and events).

### Trade-offs:
- **Pros:** Dedicated service ensures accurate real-time tracking. Can scale independently to handle many players.  
- **Cons:** High frequency updates may require efficient storage and caching. Needs careful handling to avoid stale data or race conditions.

---

### Architecture diagram

![Architecture diagram](diagrams/architect-diagram.png)

---

# Ghost Hunters – Technologies and Communication Patterns

This project follows a **polyglot microservices architecture** to ensure modularity, scalability, and independence of each team.  
Each microservice is implemented in a language and technology stack that best matches its business logic and performance requirements.  
Services communicate via **REST APIs** for synchronous operations and **message queues (e.g., RabbitMQ/Kafka)** for asynchronous updates.

---

## 1. User Management Service (JavaScript / Node.js)

### Technologies:
- **Node.js + Express** for REST APIs.
- **MongoDB** for user and friend data (flexible schema for social graph).
- **JWT** for authentication and authorization.
- **Redis** for caching sessions and currency balances.

### Communication Patterns:
- **REST API** for CRUD operations on users and friends.
- **Event publishing** (via Kafka) when currency updates or new friends are added (consumed by Inventory and Lobby services).

### Motivation & Trade-offs:
- **Pros:** Node.js handles concurrent HTTP requests efficiently, rich ecosystem for auth (Passport.js, JWT).  
- **Cons:** Single-threaded, CPU-heavy tasks (hashing) need offloading to worker threads.  
- **Fit:** Ideal for high-volume user interactions and real-time friend updates.

---

## 2. Ghost AI Service (JavaScript / Node.js)

### Technologies:
- **Node.js** with **Worker Threads** or **Cluster** for running ghost AI as independent actors.
- **Socket.IO (WebSockets)** for real-time updates to the Game Service.
- **In-memory state store** (Redis) for ghost decisions.

### Communication Patterns:
- **Event-driven**: broadcasts ghost state changes (hiding, haunting, attacking) to the Game Service.  
- **REST API** for debug tools and querying ghost states.

### Motivation & Trade-offs:
- **Pros:** Event-driven model + WebSockets are perfect for real-time AI-driven interactions.  
- **Cons:** Node.js is weaker in CPU-heavy computations; may require scaling ghost workers horizontally.  
- **Fit:** Excellent for fast iteration on ghost logic and broadcasting AI state changes in real time.

---

## 3. Shop Service (C# / .NET 7)

### Technologies:
- **ASP.NET Core Web API** for catalog endpoints.
- **SQL Server** for item and price history persistence.
- **Entity Framework Core** for ORM.
- **gRPC** for efficient communication with Inventory Service.

### Communication Patterns:
- **REST API** for item catalog queries from players.  
- **gRPC** for low-latency communication with Inventory Service (price validation).  
- **Event publishing** (via RabbitMQ) when prices change, consumed by Game Service.

### Motivation & Trade-offs:
- **Pros:** .NET provides high performance, strong typing, and enterprise-level tooling.  
- **Cons:** More heavy-weight than scripting languages; slower prototyping.  
- **Fit:** Perfect for financial transactions and historical price tracking where consistency is critical.

---

## 4. Journal Service (C# / .NET 7)

### Technologies:
- **ASP.NET Core Web API** for CRUD journal entries.
- **SQL Server** for storing session-based journal records.
- **Entity Framework Core** for data modeling.
- **Background workers** for validating journal results at session end.

### Communication Patterns:
- **REST API** for players to submit and query journals.  
- **Async messaging** with User Management Service to award currency after validation.  
- **REST/gRPC** with Game Service to fetch actual ghost type for comparison.

### Motivation & Trade-offs:
- **Pros:** Strong consistency model, robust schema for structured journal entries.  
- **Cons:** More complex setup compared to Python/JS.  
- **Fit:** Reliable choice for validation-heavy workflows where correctness matters (awarding rewards).

---

## 5. Lobby Service (Python / FastAPI)

### Technologies:
- **FastAPI** for high-performance async REST APIs.
- **PostgreSQL** for session state (players, sanity, items, ghosts, maps).
- **SQLAlchemy** for ORM.
- **Celery + Redis** for background session cleanup tasks.

### Communication Patterns:
- **REST API** for creating and managing lobbies.  
- **Event-driven messaging** (via Kafka) to sync state with Ghost AI and User Management services.  
- **WebSockets** for real-time lobby updates to clients.

### Motivation & Trade-offs:
- **Pros:** Python’s FastAPI is very fast to develop, async support is strong for managing multiple lobbies.  
- **Cons:** Slower raw performance vs C#/Go; needs scaling under heavy load.  
- **Fit:** Great for coordinating state across multiple players in a lobby.

---

## 6. Map Service (Python / FastAPI)

### Technologies:
- **FastAPI** for REST APIs managing maps.
- **PostgreSQL** (JSONB) for flexible map storage (rooms, objects, hiding places).
- **Pydantic** for schema validation of user-created maps.

### Communication Patterns:
- **REST API** for CRUD operations on maps.  
- **Event publishing** when new maps are created (consumed by Lobby and Ghost AI).  
- **Read-only API** consumed by Game Service to load maps.

### Motivation & Trade-offs:
- **Pros:** Python is excellent for handling JSON-like structures; very flexible for user-generated content.  
- **Cons:** Handling large map computations may need optimization or caching.  
- **Fit:** Perfect for CRUD-heavy, content-driven service where flexibility matters more than raw speed.

---

## 7. Ghost Service

### Technologies:
- **Node.js (Express / Fastify)** for REST API endpoints and service logic.
- **PostgreSQL** (or any relational DB) to store ghost types and symptoms.
- **Redis** (optional) for caching frequently accessed ghost data.
- **Worker threads / Bull (Queue)** for batch updates or complex ghost behavior processing.

### Communication Patterns:
- **REST API** for retrieving ghost types and symptoms.
- **Event-driven (Message Queue, e.g., Kafka/RabbitMQ)** for notifying Lobby or Game Service about ghost updates or new ghost additions.

### Motivation & Trade-offs:
- **Pros:** Node.js provides fast I/O, easy JSON handling, and event-driven concurrency for multiple requests.  
- **Cons:** CPU-intensive tasks (complex ghost AI logic) may block the event loop; requires worker threads or separate services for heavy computations.

---

## 8. Location Service

### Technologies:
- **Node.js (Express / Fastify)** for REST API endpoints.
- **Redis** for fast in-memory storage of player locations and real-time state.
- **PostgreSQL** for historical location logs and session tracking.
- **WebSockets (Socket.IO)** for real-time broadcasting of player locations.
- **Bull / Queue workers** for background tasks such as cleanup or aggregation.

### Communication Patterns:
- **REST API** for querying player positions and state.
- **WebSockets** for real-time broadcasting of location updates to Lobby and Ghost AI.
- **Event-driven (Message Queue)** for notifying other services when player location changes trigger game events.

### Motivation & Trade-offs:
- **Pros:** Node.js + Redis + WebSockets allows efficient real-time tracking. Flexible and easy to maintain schema for dynamic player state.  
- **Cons:** High-frequency updates can stress the event loop; requires scaling horizontally or using worker threads for heavy workloads.

---

# Ghost Hunters – API Reference (v1)

**Base URL:** `/api/v1`  
**Auth:** Bearer JWT in `Authorization: Bearer <token>` (except register/login)  
**Content-Type:** `application/json`  
**Timestamps:** ISO-8601 UTC

### Error model (uniform)
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Item not found",
    "details": { "field": "itemId" }
  }
}
```

---

## User Management Service

### POST `/users/register` – Consumed by Gateway
**Description**  
Creates a new user account with **initial currency** (e.g., `100`).

**Payload**
```json
{
  "email": "player@example.com",
  "username": "PlayerOne",
  "password": "SecurePassword123"
}
```

**Response**
```json
{
  "user": {
    "id": "uuid-here",
    "email": "player@example.com",
    "username": "PlayerOne",
    "level": 1,
    "currency": 100,
    "createdAt": "2025-09-09T10:00:00Z"
  },
  "accessToken": "jwt-token-here",
  "refreshToken": "jwt-refresh-here"
}
```

### POST `/users/login`
**Description**  
Authenticates a user and returns access/refresh tokens.

**Payload**
```json
{ "email": "player@example.com", "password": "SecurePassword123" }
```

**Response**
```json
{
  "user": { "id": "uuid", "username": "PlayerOne", "level": 1, "currency": 100 },
  "accessToken": "jwt-token-here",
  "refreshToken": "jwt-refresh-here"
}
```

### POST `/users/refresh`
**Description**  
Exchanges a valid refresh token for a new access token.

**Payload**
```json
{ "refreshToken": "jwt-refresh-here" }
```

**Response**
```json
{ "accessToken": "new-jwt-token-here" }
```

### GET `/users/{id}`
**Description**  
Returns public profile and currency/wallet.

**Response**
```json
{
  "id": "uuid",
  "email": "player@example.com",
  "username": "PlayerOne",
  "level": 5,
  "currency": 245
}
```

### PATCH `/users/{id}`
**Description**  
Update username or level (self or admin).

**Payload**
```json
{ "username": "NewName", "level": 6 }
```

**Response**
```json
{ "id": "uuid", "username": "NewName", "level": 6 }
```

### GET `/users/{id}/friends`
**Description**  
List friends for a user.

**Response**
```json
{
  "friends": [
    { "id": "uuid-a", "username": "Alfa", "level": 3 },
    { "id": "uuid-b", "username": "Bravo", "level": 2 }
  ]
}
```

### POST `/users/{id}/friends`
**Description**  
Send a friend request.

**Payload**
```json
{ "friendId": "uuid-of-other-user" }
```

**Response**
```json
{ "status": "pending" }
```

### DELETE `/users/{id}/friends/{friendId}`
**Description**  
Remove a friend (or cancel pending request).

**Response**
```json
{ "deleted": true }
```

---

## Ghost AI Service

### POST `/ghostai/lobbies/{lobbyId}/spawn`
**Description**  
Starts a dedicated AI worker for the lobby.

**Payload**
```json
{
  "ghostTypeId": "uuid-ghost",
  "mapId": "uuid-map",
  "difficulty": "NORMAL"
}
```

**Response**
```json
{ "taskId": "uuid-task", "status": "started" }
```

### POST `/ghostai/lobbies/{lobbyId}/update-context`
**Description**  
Pushes live context (players, sanity, positions, objects) to AI.

**Payload**
```json
{
  "players": [
    { "userId": "uuid-u1", "sanity": 72.5, "roomId": "uuid-r1", "alone": true }
  ],
  "objects": [
    { "id": "uuid-obj1", "state": "BROKEN" }
  ]
}
```

**Response**
```json
{ "accepted": true, "queuedAt": "2025-09-09T10:01:00Z" }
```

### DELETE `/ghostai/lobbies/{lobbyId}`
**Description**  
Stops the AI worker for a lobby.

**Response**
```json
{ "stopped": true }
```

---

## Shop Service

### GET `/items`
**Description**  
List items with current price.

**Response**
```json
{
  "total": 2,
  "page": 1,
  "pageSize": 20,
  "items": [
    {
      "id": "uuid-i1",
      "title": "EMF Reader",
      "description": "Detects electromagnetic fields.",
      "durability": 5,
      "price": { "amount": 50, "currency": "CRD" }
    },
    {
      "id": "uuid-i2",
      "title": "Thermometer",
      "description": "Measures temperature.",
      "durability": 10,
      "price": { "amount": 30, "currency": "CRD" }
    }
  ]
}
```

### GET `/items/{id}`
**Description**  
Get one item with current price.

**Response**
```json
{
  "id": "uuid-i1",
  "title": "EMF Reader",
  "description": "Detects electromagnetic fields.",
  "durability": 5,
  "price": { "amount": 50, "currency": "CRD" }
}
```

### GET `/items/{id}/prices`
**Description**  
Returns price history.

**Response**
```json
{
  "history": [
    { "price": { "amount": 50, "currency": "CRD" }, "since": "2025-09-01T00:00:00Z" },
    { "price": { "amount": 45, "currency": "CRD" }, "since": "2025-08-01T00:00:00Z" }
  ]
}
```

### POST `/items`
**Description**  
Create a new shop item.

**Payload**
```json
{
  "title": "UV Flashlight",
  "description": "Reveals traces.",
  "durability": 8,
  "price": { "amount": 40, "currency": "CRD" }
}
```

**Response**
```json
{ "id": "uuid-new-item" }
```

### PATCH `/items/{id}/price`
**Description**  
Update the active price.

**Payload**
```json
{ "price": { "amount": 55, "currency": "CRD" } }
```

**Response**
```json
{ "id": "uuid-i1", "price": { "amount": 55, "currency": "CRD" } }
```

---

## Journal Service

### POST `/journals`
**Description**  
Create a journal for a user in a lobby.

**Payload**
```json
{
  "lobbyId": "uuid-l1",
  "userId": "uuid-u1",
  "observations": [
    { "symptom": "Freezing temperature", "evidence": "Thermo at -5°C" }
  ],
  "guessGhostTypeId": "uuid-ghost"
}
```

**Response**
```json
{ "journalId": "uuid-j1" }
```

### GET `/journals/{journalId}`
**Description**  
Get one journal with entries.

**Response**
```json
{
  "id": "uuid-j1",
  "lobbyId": "uuid-l1",
  "userId": "uuid-u1",
  "observations": [
    { "symptom": "Freezing temperature", "evidence": "Thermo at -5°C" }
  ],
  "guessGhostTypeId": "uuid-ghost",
  "submittedAt": "2025-09-09T10:02:00Z"
}
```

### GET `/lobbies/{lobbyId}/journals`
**Description**  
List journals for a lobby.

**Response**
```json
{
  "journals": [
    { "id": "uuid-j1", "userId": "uuid-u1" },
    { "id": "uuid-j2", "userId": "uuid-u2" }
  ]
}
```

### POST `/journals/{journalId}/finalize`
**Description**  
Finalize and score a journal.

**Payload**
```json
{ "actualGhostTypeId": "uuid-ghost-real" }
```

**Response**
```json
{ "awarded": { "amount": 120, "currency": "CRD" } }
```

---

## Lobby Service

### POST `/lobbies`
**Description**  
Create a lobby and spawn Ghost AI.

**Payload**
```json
{
  "hostUserId": "uuid-host",
  "mapId": "uuid-map",
  "difficulty": "HARD",
  "maxPlayers": 4
}
```

**Response**
```json
{
  "id": "uuid-l1",
  "difficulty": "HARD",
  "mapId": "uuid-map",
  "players": [
    { "userId": "uuid-host", "sanity": 100.0, "dead": false, "items": [] }
  ],
  "status": "open"
}
```

### POST `/lobbies/{id}/join`
**Description**  
Join a lobby.

**Payload**
```json
{ "userId": "uuid-u2" }
```

**Response**
```json
{ "id": "uuid-l1", "players": [{ "userId": "uuid-host" }, { "userId": "uuid-u2" }] }
```

### POST `/lobbies/{id}/leave`
**Description**  
Leave a lobby.

**Payload**
```json
{ "userId": "uuid-u2" }
```

**Response**
```json
{ "left": true }
```

### PATCH `/lobbies/{id}/players/{userId}`
**Description**  
Update player state.

**Payload**
```json
{ "sanity": 45.3, "dead": false }
```

**Response**
```json
{ "userId": "uuid-u2", "sanity": 45.3, "dead": false }
```

### POST `/lobbies/{id}/items/bring`
**Description**  
Bring an item into lobby.

**Payload**
```json
{ "userId": "uuid-u2", "inventoryId": "uuid-inv-1" }
```

**Response**
```json
{ "added": true }
```

### GET `/lobbies/{id}`
**Description**  
Get current lobby state.

**Response**
```json
{
  "id": "uuid-l1",
  "difficulty": "HARD",
  "mapId": "uuid-map",
  "players": [
    { "userId": "uuid-host", "sanity": 88.2, "dead": false },
    { "userId": "uuid-u2", "sanity": 45.3, "dead": false }
  ],
  "status": "active"
}
```

---

## Map Service

### GET `/maps`
**Description**  
List maps.

**Response**
```json
{
  "total": 1,
  "page": 1,
  "pageSize": 20,
  "maps": [
    { "id": "uuid-map", "name": "Willow Street House" }
  ]
}
```

### GET `/maps/{id}`
**Description**  
Get map structure.

**Response**
```json
{
  "id": "uuid-map",
  "name": "Willow Street House",
  "rooms": [{ "id": "uuid-r1", "name": "Kitchen" }],
  "connections": [{ "from": "uuid-r1", "to": "uuid-r2" }],
  "objects": [{ "id": "uuid-obj1", "roomId": "uuid-r1", "type": "Mirror" }],
  "hidingSpots": [{ "id": "uuid-h1", "roomId": "uuid-r2", "meta": { "cover": "car" } }]
}
```

### POST `/maps`
**Description**  
Create a map.

**Payload**
```json
{
  "name": "Bleasdale Farmhouse",
  "rooms": [{ "name": "Living Room" }, { "name": "Basement" }]
}
```

**Response**
```json
{ "mapId": "uuid-new-map" }
```

### PATCH `/maps/{id}`
**Description**  
Update a map.

**Payload**
```json
{ "name": "Bleasdale Farmhouse (v2)" }
```

**Response**
```json
{ "id": "uuid-map", "name": "Bleasdale Farmhouse (v2)" }
```

---

## Ghost Service

### GET `/ghosts`
**Description**  
List ghost types.

**Response**
```json
{
  "ghosts": [
    {
      "id": "uuid-ghost",
      "name": "Banshee",
      "typeASymptoms": ["Screams", "Breaks mirrors"]
    }
  ]
}
```

### GET `/ghosts/{id}`
**Description**  
Get one ghost type.

**Response**
```json
{
  "id": "uuid-ghost",
  "name": "Banshee",
  "typeASymptoms": ["Screams", "Breaks mirrors"]
}
```

### POST `/ghosts`
**Description**  
Create a ghost type.

**Payload**
```json
{
  "name": "Yurei",
  "typeASymptoms": ["Freezing", "Ghost orbs"]
}
```

**Response**
```json
{ "id": "uuid-new-ghost" }
```

### PATCH `/ghosts/{id}`
**Description**  
Update a ghost type.

**Payload**
```json
{ "name": "Yurei (v2)", "typeASymptoms": ["Freezing", "Orbs", "Footsteps"] }
```

**Response**
```json
{ "id": "uuid-ghost", "name": "Yurei (v2)" }
```

---

## Location Service

### POST `/location/track`
**Description**  
Append a location sample for a user.

**Payload**
```json
{
  "userId": "uuid-u1",
  "lobbyId": "uuid-l1",
  "roomId": "uuid-r2",
  "isSpeaking": true,
  "group": ["uuid-u2"],
  "isHiding": false,
  "at": "2025-09-09T10:05:30Z"
}
```

**Response**
```json
{ "accepted": true }
```

### GET `/location/lobbies/{lobbyId}/users/{userId}/latest`
**Description**  
Get the latest known location for a user.

**Response**
```json
{
  "roomId": "uuid-r2",
  "isAlone": false,
  "lastSeenAt": "2025-09-09T10:05:30Z"
}
```

---

### Notes
- All POST/DELETE operations that affect wallet/inventory/lobby can accept `Idempotency-Key` header.
- Pagination: `?page` (default 1), `?pageSize` (default 20). Responses include `total`, `page`, `pageSize`.


---


# GitHub Workflow

To ensure consistency and maintain high-quality contributions, we follow a structured workflow in this project.  
This section describes how branching, merging, reviews, and versioning are handled.

---

## 1. Branching Strategy

We use two primary branches:

- **`main`** → Stable production-ready branch.  
- **`dev`** → Integration branch where features are merged before release.

### Branch Naming Convention

- **Feature branches:** `feature/<short-description>`  
  - Example: `feature/user-authentication`
- **Bugfix branches:** `bugfix/<issue-number>-<short-description>`  
  - Example: `bugfix/123-fix-lobby-crash`
- **Hotfix branches (critical fixes to main):** `hotfix/<short-description>`  
  - Example: `hotfix/fix-login-token`

---

## 2. Merging Strategy

- All changes must be proposed via **Pull Requests (PRs)** into the `development` branch.  
- PRs into `main` happen only from `development` once a release is ready.  
- **Fast-forward merges are not allowed** → We use **Squash and Merge** for clean history.  

---

## 3. Pull Requests (PRs)

Each PR must include:

- **Title** → Clear and descriptive (e.g., `Add Ghost AI decision-making module`)  
- **Description** → What was done, why, and how it was tested.  
- **Linked Issues** → Reference issues with `Closes #issueNumber`.  
- **Checklist**:
  - [ ] Code compiles successfully  
  - [ ] Tests are written/updated  
  - [ ] Documentation updated (if needed)  

---

## 4. Code Reviews & Approvals

- At least **2 approvals** are required for merging a PR.  
- **Self-approval is not allowed**.  
- **Requested changes must be resolved** before merging.  

---

## 5. Test Coverage

- All new features must include **unit and integration tests**.  
- Minimum **80% coverage** enforced via CI (GitHub Actions).  
- Tests are run automatically on each PR.  

---

## 6. Versioning

- We follow **Semantic Versioning (SemVer)**:  
  - **MAJOR** version → Incompatible API changes  
  - **MINOR** version → Backward-compatible new features  
  - **PATCH** version → Backward-compatible bug fixes  

Example: `v2.1.4` → Major release 2, minor update 1, patch 4.  

---

## 7. Continuous Integration / Deployment (CI/CD)

- **CI** runs on every PR → Builds, runs tests, checks coverage.  
- **CD** is triggered from `main` → Deploys stable release.  

---

## 8. Release Workflow

1. Work is merged into `development`.  
2. Once stable → Create a **release branch**: `release/x.y.z`.  
3. Test and validate → Merge into `main`.  
4. Tag release with version number.  
5. Deploy automatically via GitHub Actions.  