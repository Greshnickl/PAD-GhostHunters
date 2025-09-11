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
