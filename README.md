# Ghost Hunters

## Game Microservices – Service Boundaries

This project is built using a **microservices architecture** to ensure modularity, scalability, and independent deployment of each component. Each service is implemented in a language chosen to best fit its specific responsibilities, allowing the team to leverage **polyglot programming** for optimal performance and maintainability.

We selected **JavaScript (Node.js)**, **PHP**, and **Python (FastAPI)** for this project:  
- **JavaScript (Node.js)** is used for services requiring high concurrency and fast development cycles, such as **User Management** and **Ghost AI**. Its event-driven model and rich ecosystem make it ideal for real-time operations and rapid prototyping.  
- **PHP** is used for data-centric, transactional services such as **Shop** and **Journal**. Its strong MySQL integration and stability make it ideal for managing structured CRUD operations.  
- **Python (FastAPI)** is chosen for services that benefit from asynchronous communication and rapid iteration, such as **Lobby**, **Map**, **Ghost**, **Location**, **Inventory**, and **Chat**. Its modern async capabilities and lightweight design enable efficient request handling and event coordination.

This combination of technologies allows us to **balance speed of development, runtime performance, and maintainability**, ensuring each service meets the unique demands of the game while allowing teams to work efficiently within their expertise.

---

## Services Overview

---

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
- Each ghost runs as an independent process or thread (implemented via Node.js worker threads or clusters).
- Processes contextual data, such as:
  - Map layout.
  - Difficulty settings.
  - Player sanity levels.
  - Movable/interactable objects.
  - Player targeting and attack decisions.
- Relays ghost state changes to the Game or Lobby Service.

### Service Boundaries:
- Encapsulates all ghost decision-making logic.
- Does not store user data or handle session management.
- Operates independently, only pushing relevant updates to other systems.

### Interfaces / Consumers:

**Provides APIs/events for:**
- Ghost state changes (hiding, haunting, interacting).
- AI decisions related to player interactions.

**Consumed by:**
- Game Service (to broadcast ghost activity to players).
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
- Independent from gameplay logic — it provides catalog information and updates only.

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

### Technologies and Communication Patterns
- **Language:** PHP  
- **Framework:** Laravel / Slim  
- **Database:** MySQL  
- **Communication:** REST (HTTP/JSON)

### Responsibilities:
- Allows players to record:
  - Symptoms observed during exploration.  
  - The ghost type they believe they encountered.  
- Stores journal entries per user and session.  
- Validates journal entries at the end of a session by comparing them with the actual ghost type.  
- Awards in-game currency for accurate records (via User Management Service).

### Service Boundaries:
- Encapsulates only journal-related data and validation logic.  
- Does not manage user authentication, ghost AI, or currency directly (delegates rewards to User Management Service).  
- Independent of session management; only consumes final game results to validate entries.

### Interfaces / Consumers:

**Provides APIs for:**
- Creating and updating journal entries.  
- Retrieving entries by user or session.  
- Validating journal results at game end.  

**Consumed by:**
- **Lobby Service** – to identify player journals per session.  
- **User Management Service** – to reward players for correct ghost guesses.  
- **Game Service** – to compare journal entries with the true ghost type.

---

## 5. Lobby Service

### Technologies and Communication Patterns
- **Language:** Python  
- **Framework:** FastAPI  
- **Database:** PostgreSQL  
- **Communication:** REST (HTTP/JSON)

### Responsibilities:
- Manages **active game sessions**, storing:
  - Difficulty level.  
  - Players in the session with sanity levels and death states.  
  - Items brought into the session and current ownership.  
  - The ghost type associated with the session.  
  - The map currently used in the lobby.  
- Ensures items are removed or expired when a player dies.

### Service Boundaries:
- Encapsulates only **session-related state and metadata**.  
- Does not handle:
  - User authentication (**User Management Service**).  
  - Ghost behavior (**Ghost AI Service**).  
  - Item catalog (**Shop Service**).  
- Acts as the **single source of truth** for ongoing or paused sessions.

### Interfaces / Consumers:

**Provides APIs for:**
- Creating, updating, and resuming sessions.  
- Managing session players (join, leave, update status).  
- Tracking sanity, deaths, and item ownership.  
- Associating maps and ghost types with sessions.

**Consumed by:**
- **Game Service** – to execute gameplay logic.  
- **User Management Service** – to sync player profiles and stats.  
- **Shop Service** & **Inventory Service** – to validate item usage in sessions.  
- **Ghost AI Service** – to retrieve lobby configuration for ghost behavior.

---

## 6. Map Service

### Technologies and Communication Patterns
- **Language:** Python  
- **Framework:** FastAPI  
- **Database:** PostgreSQL  
- **Communication:** REST (HTTP/JSON)

### Responsibilities:
- Manages creation and configuration of maps, including:
  - Houses, rooms, and their connectivity.  
  - Placement of interactive objects.  
  - Definition of hiding spots (safe zones).  
- Stores both predefined and dynamically generated maps.  
- Provides APIs to retrieve layouts during gameplay.

### Service Boundaries:
- Focused exclusively on **map structure and object placement**.  
- Does not:
  - Handle ghost AI logic (only exposes map and hiding data).  
  - Manage players or inventory (handled by Lobby & Inventory Services).  
  - Track session state (delegated to Lobby Service).  
- Acts as a **map provider** for all other gameplay services.

### Interfaces / Consumers:

**Provides APIs for:**
- Creating new houses with rooms and objects.  
- Editing or randomizing object placement.  
- Querying hiding spots and room connectivity.  
- Retrieving stored maps for game rendering.

**Consumed by:**
- **Lobby Service** – to assign a map to each session.  
- **Ghost AI Service** – to understand room layout and hiding spots.  
- **Game Service** – to visualize map data for players.

---

## 7. Ghost Service

### Technologies and Communication Patterns
- **Language:** Python  
- **Framework:** FastAPI  
- **Database:** PostgreSQL  
- **Communication:** REST (HTTP/JSON)

### Responsibilities:
- Acts as an encyclopedia of all ghost types in the game world.  
- Stores and provides information about ghost **Type A symptoms**:  
  - Breaking mirrors, moving objects, creating cold spots, etc.  
  - Visible to players for deduction purposes.  
- Stores **Type B symptoms**:  
  - Hunting behavior depending on player sanity or group status.  
  - Used for advanced observation and discovery.  
- Provides ghost-related data for gameplay mechanics and player education.

### Service Boundaries:
- Encapsulates only ghost-related information and symptoms.  
- Does not manage users, sessions, or player inventory.  
- Independent of session state; only consumed by other services for ghost data.

### Interfaces / Consumers:

**Provides APIs for:**
- Retrieving ghost types and associated symptoms (Type A & Type B).  
- Listing ghosts available for specific maps or sessions.  

**Consumed by:**
- **Lobby Service** – to know which ghost type is active in the session.  
- **Game Service** – for in-game behavior logic and player feedback.  
- **Journal Service** – to validate player observations.

### Trade-offs:
- **Pros:** Centralized ghost data ensures consistency and easy expansion for new ghost types.  
- **Cons:** Requires synchronization with session data if ghost attributes change dynamically. Caching or local replication may be needed for performance.

---

## 8. Location Service

### Technologies and Communication Patterns
- **Language:** Python  
- **Framework:** FastAPI  
- **Database:** PostgreSQL  
- **Communication:** REST (HTTP/JSON)

### Responsibilities:
- Tracks player movements in real time.  
- Records:
  - Current room or zone location.  
  - Interactions with items or objects.  
  - Player state (alone/in group, speaking, hiding, visible).  
  - Timestamped activity logs.  
- Provides up-to-date positional data to other gameplay systems.

### Service Boundaries:
- Responsible solely for tracking player locations and states.  
- Does not handle authentication, session control, or ghost AI logic.  
- Operates independently while feeding real-time data to session management and gameplay logic.

### Interfaces / Consumers:

**Provides APIs for:**
- Querying current player location and state.  
- Subscribing to live updates during sessions.  

**Consumed by:**
- **Lobby Service** – for item interactions and status syncing.  
- **Ghost AI Service** – for decision-making based on player proximity.  
- **Game Service** – for player visibility, collisions, and events.

### Trade-offs:
- **Pros:** Dedicated service ensures accurate, scalable real-time tracking.  
- **Cons:** Frequent updates can impact performance; requires caching or event batching to maintain consistency.

---

### Architecture Diagram

![Architecture diagram](diagrams/architect-diagram.png)

---

# Ghost Hunters – Technologies and Communication Patterns

This project follows a **polyglot microservices architecture** to ensure modularity, scalability, and independence between services.  
Each microservice uses a technology stack that best aligns with its core responsibilities and performance needs.  
All services communicate primarily through **REST APIs (HTTP/JSON)** for synchronous operations and optionally through **message queues** (e.g., RabbitMQ or Kafka) for asynchronous updates.

---

## 1. User Management Service (JavaScript / Node.js)

### Technologies:
- **Node.js + Express** for RESTful endpoints.  
- **PostgreSQL** for structured user and friend data.  
- **JWT** for authentication and authorization.  
- **Redis (optional)** for caching active sessions or frequently accessed data.

### Communication Patterns:
- **REST API** for user and friend CRUD operations.  
- **Event publishing** when user or currency data changes (consumed by Inventory and Lobby services).

### Motivation & Trade-offs:
- **Pros:** Excellent for handling concurrent HTTP requests; rich middleware ecosystem (Passport.js, JWT).  
- **Cons:** CPU-heavy tasks (e.g., password hashing) require worker threads.  
- **Fit:** Best suited for high-volume user interactions and social updates.

---

## 2. Ghost AI Service (JavaScript / Node.js)

### Technologies:
- **Node.js (Express + Worker Threads)** for running ghost AI as independent actors.  
- **Socket.IO (WebSockets)** for real-time updates to the Game Service.  
- **Redis (in-memory cache)** for storing ghost state and decisions.

### Communication Patterns:
- **Event-driven** – broadcasts ghost state changes (hiding, haunting, attacking) to the Game Service.  
- **REST API** – used for debugging and querying ghost states.  

### Motivation & Trade-offs:
- **Pros:** Node.js with WebSockets is ideal for real-time AI-driven interaction and low-latency communication.  
- **Cons:** Node.js is single-threaded; CPU-heavy tasks (AI loops) need horizontal scaling or worker threads.  
- **Fit:** Excellent for fast iteration and live AI logic that reacts instantly to player events.

---

## 3. Shop Service (PHP / Laravel)

### Technologies:
- **Laravel Framework** for item catalog endpoints.  
- **MySQL** for item data and price history persistence.  
- **Eloquent ORM** for database interaction.  
- **Event broadcasting** for notifying other services of price or catalog updates.

### Communication Patterns:
- **REST API** – provides item catalog queries for players.  
- **Internal event system** – emits updates to Inventory Service and Lobby Service when prices or durability change.  
- **Optional async messaging** (RabbitMQ/Kafka) for consistent updates across gameplay systems.

### Motivation & Trade-offs:
- **Pros:** Laravel offers rapid development, built-in ORM, and great MySQL integration.  
- **Cons:** Slightly heavier runtime than Node.js for highly concurrent workloads.  
- **Fit:** Ideal for managing item catalogs, durability systems, and price tracking with strong data consistency.

---

## 4. Journal Service (PHP / Laravel)

### Technologies:
- **Laravel Framework** for CRUD journal operations.  
- **MySQL** for storing player journals linked to sessions.  
- **Eloquent ORM** for structured journal entries.  
- **Background jobs (Laravel Queues)** for validating journals and triggering rewards post-session.

### Communication Patterns:
- **REST API** – allows players to submit and query journal entries.  
- **Asynchronous messaging** – communicates with the User Management Service to award currency after validation.  
- **REST API calls** – communicates with the Game Service to fetch actual ghost type for comparison.

### Motivation & Trade-offs:
- **Pros:** Strong schema and validation, straightforward integration with MySQL, and built-in background jobs.  
- **Cons:** Limited concurrency for real-time workloads; more suited to structured post-session operations.  
- **Fit:** Excellent for reliable storage and validation of user-generated session data.

---

## 5. Lobby Service (Python / FastAPI)

### Technologies:
- **FastAPI** for async REST endpoints managing lobbies.  
- **PostgreSQL** for session state persistence (players, sanity, items, ghosts, maps).  
- **SQLAlchemy ORM** for schema mapping.  
- **Celery + Redis** for background task scheduling and cleanup.

### Communication Patterns:
- **REST API** – handles lobby creation and updates.  
- **Event-driven updates** – syncs lobby state with Ghost AI and User Management services.  
- **WebSockets** – pushes real-time lobby updates to connected clients.

### Motivation & Trade-offs:
- **Pros:** Lightweight, async-ready, and easy to scale horizontally; clean syntax for Python developers.  
- **Cons:** Requires good async design for high load; slightly less performant in raw speed than Node.js.  
- **Fit:** Perfect for real-time session coordination and player state management across multiple lobbies.

---

## 6. Map Service (Python / FastAPI)

### Technologies:
- **FastAPI** for REST endpoints managing map structures.  
- **PostgreSQL (JSONB)** for flexible storage of maps, rooms, and hiding places.  
- **Pydantic Models** for input validation and schema enforcement.

### Communication Patterns:
- **REST API** – for CRUD operations on maps.  
- **Event publishing** – triggers updates when new maps are created or edited (consumed by Lobby and Ghost AI Services).  
- **Read-only API** – used by the Game Service to load map data for sessions.

### Motivation & Trade-offs:
- **Pros:** Excellent for handling flexible map data structures and dynamic content.  
- **Cons:** Large, complex map computations might require caching or pre-processing.  
- **Fit:** Ideal for content-driven features like custom map creation and map-based ghost logic.

---

## 7. Ghost Service

### Technologies:
- **Python (FastAPI)** for REST API endpoints and service logic.  
- **PostgreSQL** to store ghost types, symptoms, and related metadata.  
- **Redis (optional)** for caching frequently accessed ghost data.  
- **Celery Workers / Task Queues** for batch updates or background operations (e.g., ghost catalog updates).

### Communication Patterns:
- **REST API** – for retrieving ghost types, Type A/B symptoms, and behaviors.  
- **Event-driven messaging (RabbitMQ/Kafka)** – to notify the Lobby or Game Service about ghost-related updates or additions.

### Motivation & Trade-offs:
- **Pros:** Python and FastAPI offer excellent readability and fast iteration speed; perfect for maintaining a growing encyclopedia of ghost data.  
- **Cons:** For high-frequency data access, caching is essential; also requires periodic cleanup of cached data to avoid inconsistencies.  
- **Fit:** Well-suited for maintaining structured ghost knowledge and supporting other gameplay services that depend on ghost data.

---

## 8. Location Service

### Technologies:
- **Python (FastAPI)** for REST API endpoints and WebSocket connections.  
- **Redis** for real-time tracking of player positions and states.  
- **PostgreSQL** for historical tracking (player routes, session archives).  
- **WebSockets (via FastAPI / Starlette)** for broadcasting player locations to connected services.

### Communication Patterns:
- **REST API** – for querying player positions, movement logs, and current state.  
- **WebSockets** – for real-time updates and event broadcasting during sessions.  
- **Event-driven (RabbitMQ/Kafka)** – for pushing location-change events to Lobby and Ghost AI Services.

### Motivation & Trade-offs:
- **Pros:** Asynchronous FastAPI + Redis allows scalable, low-latency tracking of live player data.  
- **Cons:** High update frequency requires efficient caching and batching; may need rate limiting for heavy load.  
- **Fit:** Perfect for enabling responsive ghost AI behavior and session coordination.

---

## 9. Inventory Service

### Technologies:
- **Python (FastAPI)** for REST endpoints handling inventory logic.  
- **PostgreSQL** for persisting player-owned items, durability, and usage data.  
- **SQLAlchemy ORM** for relational data management.  
- **Celery Workers** for background tasks such as durability reduction and inventory cleanup.

### Communication Patterns:
- **REST API** – for CRUD operations on player inventories.  
- **Event-driven messaging** – listens to **Shop Service** for catalog updates and **Lobby Service** for session item usage.  
- **WebSockets (optional)** – for real-time inventory updates to connected clients.

### Responsibilities:
- Manages all player-owned items (equipment, consumables, tools).  
- Updates item durability based on usage in active sessions.  
- Validates purchases using data from the **Shop Service** and **User Management Service**.  
- Automatically removes expired items after session completion.

### Motivation & Trade-offs:
- **Pros:** Strong data integrity, modular validation through service interaction.  
- **Cons:** Synchronization complexity between services when handling multiple updates simultaneously.  
- **Fit:** Ideal for maintaining accurate player inventories and enforcing durability and purchase rules.

---

## 10. Chat Service

### Technologies:
- **Python (FastAPI + WebSockets)** for real-time communication channels.  
- **PostgreSQL** for storing message logs and chat history.  
- **Redis Pub/Sub** for efficient message distribution across nodes.  
- **Socket.IO / WebSockets** for instant communication between players.

### Communication Patterns:
- **WebSockets** – for real-time chat (voice/text events).  
- **REST API** – for fetching past chat history or archived logs.  
- **Event-driven (Pub/Sub)** – for handling broadcast messages across multiple rooms or sessions.

### Responsibilities:
- Enables **in-game chat** between players in the same session (local or radio).  
- Manages message delivery, persistence, and access control.  
- Filters messages and manages room contexts (public lobby, private groups, ghost communication).  
- Supports asynchronous message archiving for post-session review.

### Motivation & Trade-offs:
- **Pros:** WebSockets provide seamless real-time communication; Pub/Sub allows easy scaling across instances.  
- **Cons:** Requires careful connection management; scaling voice or large chat rooms can strain performance.  
- **Fit:** Excellent for enabling immersive, real-time multiplayer interaction within game sessions.

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
