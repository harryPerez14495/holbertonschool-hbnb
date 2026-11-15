# HBnB Evolution — Sequence Diagrams for API Calls

## Purpose

These diagrams describe four API calls and their interactions across
the Presentation, Business Logic, and Persistence layers.

## Participants

| Participant | Responsibility |
| --- | --- |
| Client | Sends HTTP requests and receives responses. |
| ServiceAPI | Presentation Layer: reads request data, calls the facade, and builds HTTP responses. |
| HBnBFacade | Business Logic Layer: coordinates use cases and domain models. |
| Repository | Persistence Layer: encapsulates storage access. |
| Database | Persistence Layer: stores and retrieves entity data. |

Facade self-messages summarize its internal coordination of the
domain models. Models validate their own state.

The API paths and repository method names below are proposed interfaces.

## 1. User Registration

The client supplies registration data. The facade coordinates User
construction, input validation, password hashing, and persistence.

```mermaid
sequenceDiagram
    autonumber
    actor Client
    box Presentation Layer
        participant API as ServiceAPI
    end
    box Business Logic Layer
        participant Facade as HBnBFacade
    end
    box Persistence Layer
        participant Repo as Repository
        participant DB as Database
    end

    Client->>API: POST /api/v1/users/ with registration data
    API->>Facade: register_user(data)
    Facade->>Facade: Construct and validate User(data)
    alt Invalid user data
        Facade-->>API: Validation error
        API-->>Client: 400 Bad Request
    else Valid user data
        Facade->>Repo: add(user)
        Repo->>DB: Store user
        DB-->>Repo: Stored user
        Repo-->>Facade: User created
        Facade->>Facade: user.to_dict()
        Facade-->>API: User data without password hash
        API-->>Client: 201 Created with user JSON
    end
```

### Notes

- The constructor initializes a UUID4, created_at, and updated_at.
- The supplied password is stored as a hash.
- Public user data excludes password_hash.
- Invalid data returns 400 without storing the user.
- Successful registration returns 201.

## 2. Place Creation

The client provides property data and an owner ID. The facade validates
the input and retrieves the owner before constructing the Place.

```mermaid
sequenceDiagram
    autonumber
    actor Client
    box Presentation Layer
        participant API as ServiceAPI
    end
    box Business Logic Layer
        participant Facade as HBnBFacade
    end
    box Persistence Layer
        participant Repo as Repository
        participant DB as Database
    end

    Client->>API: POST /api/v1/places/ with owner_id and data
    API->>Facade: create_place(owner_id, data)
    Facade->>Facade: Validate place input
    alt Invalid place data
        Facade-->>API: Validation error
        API-->>Client: 400 Bad Request
    else Valid place input
        Facade->>Repo: get(User, owner_id)
        Repo->>DB: Find owner by ID
        DB-->>Repo: Owner or no result
        Repo-->>Facade: Owner or no result
        alt Owner not found
            Facade-->>API: Owner not found
            API-->>Client: 404 Not Found
        else Owner exists
            Facade->>Facade: Construct Place(data, owner)
            Facade->>Repo: add(place)
            Repo->>DB: Store place with owner reference
            DB-->>Repo: Stored place
            Repo-->>Facade: Place created
            Facade->>Facade: place.to_dict()
            Facade-->>API: Place data
            API-->>Client: 201 Created with place JSON
        end
    end
```

### Notes

- Place data includes title, description, price, latitude, and longitude.
- Each Place must reference exactly one existing owner.
- Construction initializes its UUID4 and audit timestamps.
- Invalid input returns 400.
- A missing owner returns 404.
- Successful creation returns 201.
- This example creates a place without initial amenities.
  Later associations use add_amenity().

## 3. Review Submission

The client supplies a user ID, place ID, rating, and comment.
The facade validates the input and retrieves the related entities.

```mermaid
sequenceDiagram
    autonumber
    actor Client
    box Presentation Layer
        participant API as ServiceAPI
    end
    box Business Logic Layer
        participant Facade as HBnBFacade
    end
    box Persistence Layer
        participant Repo as Repository
        participant DB as Database
    end

    Client->>API: POST /api/v1/reviews/ with user_id, place_id and data
    API->>Facade: create_review(user_id, place_id, data)
    Facade->>Facade: Validate rating and comment
    alt Invalid review data
        Facade-->>API: Validation error
        API-->>Client: 400 Bad Request
    else Valid review input
        Facade->>Repo: get(User, user_id)
        Repo->>DB: Find author by ID
        DB-->>Repo: User or no result
        Repo-->>Facade: User or no result
        Facade->>Repo: get(Place, place_id)
        Repo->>DB: Find place by ID
        DB-->>Repo: Place or no result
        Repo-->>Facade: Place or no result
        alt User or place not found
            Facade-->>API: Related entity not found
            API-->>Client: 404 Not Found
        else User and place exist
            Facade->>Facade: Construct Review(data, user, place)
            Facade->>Facade: place.add_review(review)
            Facade->>Repo: add(review)
            Repo->>DB: Store review with user and place references
            DB-->>Repo: Stored review
            Repo-->>Facade: Review created
            Facade->>Facade: review.to_dict()
            Facade-->>API: Review data
            API-->>Client: 201 Created with review JSON
        end
    end
```

### Notes

- Each Review has exactly one author and one associated Place.
- Construction initializes its UUID4 and audit timestamps.
- place.add_review(review) maintains the domain relationship.
- Persistence stores the review and its user and place references.
- Invalid review data returns 400.
- A missing user or place returns 404.
- Successful submission returns 201.

## 4. Fetching a List of Places

The client supplies optional search criteria through query parameters.
The facade validates the filters and retrieves matching places.

```mermaid
sequenceDiagram
    autonumber
    actor Client
    box Presentation Layer
        participant API as ServiceAPI
    end
    box Business Logic Layer
        participant Facade as HBnBFacade
    end
    box Persistence Layer
        participant Repo as Repository
        participant DB as Database
    end

    Client->>API: GET /api/v1/places/ with query filters
    API->>Facade: list_places(filters)
    Facade->>Facade: Validate filter values
    alt Invalid filters
        Facade-->>API: Validation error
        API-->>Client: 400 Bad Request
    else Valid filters
        Facade->>Repo: find(Place, filters)
        Repo->>DB: Find matching places
        DB-->>Repo: Matching records or empty result
        Repo-->>Facade: Place objects or empty list
        Facade->>Facade: Serialize places using to_dict()
        Facade-->>API: List of place data
        API-->>Client: 200 OK with JSON array
    end
```

### Notes

- The repository applies the validated search criteria.
- Matching places are serialized into a JSON array.
- A valid query with no matches returns [] with status 200.
- Invalid filters return 400.
- Reading places preserves their IDs and timestamps.

## Architecture Consistency

- ServiceAPI delegates application operations to HBnBFacade.
- The facade coordinates domain behavior and repository operations.
- Repositories encapsulate database access.
- Results return through the facade to the API.
- Persistence messages are conceptual. Database technology will be
  selected in a later project part.

## Response Summary

| API call | Success | Error paths shown |
| --- | --- | --- |
| User registration | 201 with public user data | 400 for invalid data |
| Place creation | 201 with place data | 400 for invalid data; 404 for a missing owner |
| Review submission | 201 with review data | 400 for invalid data; 404 for a missing user or place |
| Place listing | 200 with an array | 400 for invalid filters |
