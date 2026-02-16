# Sequence Diagrams - API Calls

## 🔄 Overview

These diagrams show the interaction flow between layers for key API operations.

---

## 1️⃣ User Registration
```mermaid
sequenceDiagram
    actor Client
    participant API as Presentation Layer
    participant Facade as Facade Pattern
    participant User as Business Logic
    participant Repo as Persistence Layer
    participant DB as Database

    Client->>API: POST /api/v1/users
    activate API
    
    API->>API: Validate input data
    API->>Facade: register_user(data)
    activate Facade
    
    Facade->>User: create_user(first_name, last_name, email, password)
    activate User
    
    User->>User: Validate email uniqueness
    User->>User: Hash password
    User->>User: Generate UUID
    User->>User: Set timestamps
    
    User->>Repo: save(user)
    activate Repo
    
    Repo->>DB: INSERT INTO users
    activate DB
    DB-->>Repo: Success
    deactivate DB
    
    Repo-->>User: user_id
    deactivate Repo
    
    User-->>Facade: User object
    deactivate User
    
    Facade-->>API: User data
    deactivate Facade
    
    API->>API: Format response (exclude password)
    API-->>Client: 201 Created {user_data}
    deactivate API
```

**Steps:**
1. Client sends registration data
2. API validates input format
3. Facade orchestrates user creation
4. Business logic validates and processes data
5. Repository persists to database
6. Response returns created user (without password)

---

## 2️⃣ Place Creation
```mermaid
sequenceDiagram
    actor Client
    participant API as Presentation Layer
    participant Facade as Facade Pattern
    participant Place as Business Logic
    participant User as User Model
    participant Repo as Persistence Layer
    participant DB as Database

    Client->>API: POST /api/v1/places
    activate API
    Note over Client,API: Authorization: Bearer token
    
    API->>API: Authenticate user
    API->>Facade: create_place(user_id, place_data)
    activate Facade
    
    Facade->>User: verify_user_exists(user_id)
    activate User
    User->>Repo: get(user_id)
    activate Repo
    Repo->>DB: SELECT FROM users
    activate DB
    DB-->>Repo: user_record
    deactivate DB
    Repo-->>User: User object
    deactivate Repo
    User-->>Facade: User exists
    deactivate User
    
    Facade->>Place: create(title, description, price, lat, lon, owner_id)
    activate Place
    
    Place->>Place: Validate coordinates
    Place->>Place: Validate price > 0
    Place->>Place: Generate UUID
    Place->>Place: Set timestamps
    
    Place->>Repo: save(place)
    activate Repo
    Repo->>DB: INSERT INTO places
    activate DB
    DB-->>Repo: Success
    deactivate DB
    Repo-->>Place: place_id
    deactivate Repo
    
    Place-->>Facade: Place object
    deactivate Place
    
    Facade-->>API: Place data
    deactivate Facade
    
    API-->>Client: 201 Created {place_data}
    deactivate API
```

**Steps:**
1. Client sends place data with authentication
2. API authenticates user token
3. Facade verifies user exists
4. Business logic validates place data
5. Repository persists place
6. Response returns created place

---

## 3️⃣ Review Submissionету
```mermaid
sequenceDiagram
    actor Client
    participant API as Presentation Layer
    participant Facade as Facade Pattern
    participant Review as Review Model
    participant Place as Place Model
    participant User as User Model
    participant Repo as Persistence Layer
    participant DB as Database

    Client->>API: POST /api/v1/places/{place_id}/reviews
    activate API
    Note over Client,API: Authorization: Bearer token
    
    API->>API: Authenticate user
    API->>Facade: create_review(user_id, place_id, rating, comment)
    activate Facade
    
    Facade->>Place: verify_place_exists(place_id)
    activate Place
    Place->>Repo: get(place_id)
    activate Repo
    Repo->>DB: SELECT FROM places
    activate DB
    DB-->>Repo: place_record
    deactivate DB
    Repo-->>Place: Place object
    deactivate Repo
    Place-->>Facade: Place exists
    deactivate Place
    
    Facade->>Review: create(place_id, user_id, rating, comment)
    activate Review
    
    Review->>Review: Validate rating (1-5)
    Review->>Review: Check not own place
    Review->>Review: Generate UUID
    Review->>Review: Set timestamps
    
    Review->>Repo: save(review)
    activate Repo
    Repo->>DB: INSERT INTO reviews
    activate DB
    DB-->>Repo: Success
    deactivate DB
    Repo-->>Review: review_id
    deactivate Repo
    
    Review-->>Facade: Review object
    deactivate Review
    
    Facade-->>API: Review data
    deactivate Facade
    
    API-->>Client: 201 Created {review_data}
    deactivate API
```

**Steps:**
1. Client submits review with authentication
2. API authenticates user
3. Facade verifies place exists
4. Business logic validates review data
5. System checks user doesn't review own place
6. Repository persists review
7. Response returns created review

---

## 4️⃣ Fetching List of Places
```mermaid
sequenceDiagram
    actor Client
    participant API as Presentation Layer
    participant Facade as Facade Pattern
    participant Place as Business Logic
    participant Repo as Persistence Layer
    participant DB as Database

    Client->>API: GET /api/v1/places?page=1&limit=10
    activate API
    
    API->>API: Parse query parameters
    API->>Facade: get_places(filters, pagination)
    activate Facade
    
    Facade->>Place: list_all(filters, page, limit)
    activate Place
    
    Place->>Place: Build filter criteria
    Place->>Place: Calculate offset
    
    Place->>Repo: find_all(criteria, offset, limit)
    activate Repo
    
    Repo->>DB: SELECT FROM places WHERE... LIMIT... OFFSET...
    activate DB
    DB-->>Repo: place_records[]
    deactivate DB
    
    Repo->>DB: SELECT COUNT(*) FROM places
    activate DB
    DB-->>Repo: total_count
    deactivate DB
    
    Repo-->>Place: List[Place], total
    deactivate Repo
    
    Place-->>Facade: Places with metadata
    deactivate Place
    
    Facade-->>API: {places: [], total: N, page: 1}
    deactivate Facade
    
    API->>API: Format response with pagination
    API-->>Client: 200 OK {data, pagination}
    deactivate API
```

**Steps:**
1. Client requests places list with pagination
2. API parses query parameters
3. Facade processes the request
4. Business logic builds filter criteria
5. Repository queries database with pagination
6. Repository gets total count for pagination
7. Response returns places with pagination metadata

---

## 🎯 Common Patterns

### Authentication Flow
```
Client → API: Request with token
API → API: Verify JWT token
API → Facade: Continue with user_id
```

### Error Handling
```
Layer → Layer: Operation
alt Success
    Layer ← Layer: Result
else Failure
    Layer ← Layer: Error
    API → Client: Error response
end
```

### Validation Pattern
```
1. API: Format validation
2. Facade: Business rule validation
3. Model: Entity-specific validation
4. Repository: Database constraints
```

## 📊 Response Codes

- **200 OK**: Successful GET request
- **201 Created**: Successful POST request
- **400 Bad Request**: Invalid input data
- **401 Unauthorized**: Missing/invalid authentication
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Resource not found
- **500 Internal Server Error**: Server error