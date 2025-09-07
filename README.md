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
