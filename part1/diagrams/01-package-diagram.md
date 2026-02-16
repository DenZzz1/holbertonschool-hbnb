# High-Level Package Diagram

## 📦 Architecture Overview

This diagram illustrates the three-layer architecture of the HBnB application and the communication via the Facade Pattern.

## 🎨 Diagram
```mermaid
graph TB
    subgraph PresentationLayer["🎨 Presentation Layer"]
        API[API Endpoints]
        Services[Services Layer]
    end

    subgraph FacadePattern["🎭 Facade Pattern"]
        Facade[HBnB Facade]
    end

    subgraph BusinessLogicLayer["💼 Business Logic Layer"]
        UserModel[User Model]
        PlaceModel[Place Model]
        ReviewModel[Review Model]
        AmenityModel[Amenity Model]
    end

    subgraph PersistenceLayer["💾 Persistence Layer"]
        Repository[Repository Interface]
        Database[(Database)]
    end

    API --> Facade
    Services --> Facade
    Facade --> UserModel
    Facade --> PlaceModel
    Facade --> ReviewModel
    Facade --> AmenityModel
    UserModel --> Repository
    PlaceModel --> Repository
    ReviewModel --> Repository
    AmenityModel --> Repository
    Repository --> Database

    style PresentationLayer fill:#e1f5ff
    style BusinessLogicLayer fill:#fff4e1
    style PersistenceLayer fill:#f0e1ff
    style FacadePattern fill:#e1ffe1
```

## 📋 Layer Descriptions

### Presentation Layer
- **Responsibility**: Handle HTTP requests and responses
- **Components**: API endpoints, Services
- **Technologies**: REST API, JSON

### Business Logic Layer
- **Responsibility**: Implement business rules and entity management
- **Components**: User, Place, Review, Amenity models
- **Patterns**: Domain-Driven Design

### Persistence Layer
- **Responsibility**: Data storage and retrieval
- **Components**: Repository pattern, Database connection
- **Technologies**: To be defined in Part 3

## 🎭 Facade Pattern

The Facade provides a unified interface that:
- Simplifies communication between layers
- Decouples presentation from business logic
- Centralizes orchestration logic
- Improves maintainability

## 🔄 Communication Flow
```
Client → API → Facade → Business Logic → Repository → Database
Client ← API ← Facade ← Business Logic ← Repository ← Database
```