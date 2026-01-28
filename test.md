# Mermaid Diagrams for Furniture Rent Service

## 1. Class Diagram

```mermaid
classDiagram
    class FurnitureRentService {
        -FurnitureRepository furnitureRepository
        +NewFurnitureRentService(repo) FurnitureRentService
        +Rent(furniture Furniture) error
    }
    
    class FurnitureRepository {
        <<interface>>
        +IsAvailable(furnitureId int) bool
        +Reserve(furnitureId int) error
    }
    
    class FurnitureRepositoryImpl {
        -DatabaseConnection dbConnection
        +NewFurnitureRepositoryImpl(db) FurnitureRepositoryImpl
        +IsAvailable(furnitureId int) bool
        +Reserve(furnitureId int) error
        -executeQuery(sql string) Rows
        -updateRecord(sql string) error
    }
    
    class MockFurnitureRepository {
        -map~int,bool~ availabilityMap
        -map~int,bool~ reservedItems
        -map~int,int~ methodCallCount
        +NewMockFurnitureRepository() MockFurnitureRepository
        +IsAvailable(furnitureId int) bool
        +Reserve(furnitureId int) error
        +SetupMockAvailability(id int, available bool)
        +ClearMockData()
        +GetReservationCallCount(id int) int
        +WasReserved(id int) bool
    }
    
    class DatabaseConnection {
        -db sql.DB
        +NewDatabaseConnection(dsn) DatabaseConnection
        +Connect() error
        +Query(sql string) Rows
        +Update(sql string) Result
        +Close() error
    }
    
    class Furniture {
        +int ID
        +string Name
    }
    
    FurnitureRentService --> FurnitureRepository : depends on
    FurnitureRentService --> Furniture : uses
    FurnitureRepositoryImpl ..|> FurnitureRepository : implements
    MockFurnitureRepository ..|> FurnitureRepository : implements
    FurnitureRepositoryImpl --> DatabaseConnection : uses
```

## 2. Sequence Diagram - Successful Rent Flow

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant FurnitureRentService
    participant FurnitureRepository
    participant DatabaseConnection
    
    Client->>+FurnitureRentService: Rent(furniture)
    FurnitureRentService->>+FurnitureRepository: IsAvailable(furnitureId)
    FurnitureRepository->>+DatabaseConnection: Query(SELECT available...)
    DatabaseConnection-->>-FurnitureRepository: Rows (available=true)
    FurnitureRepository-->>-FurnitureRentService: true
    
    FurnitureRentService->>+FurnitureRepository: Reserve(furnitureId)
    FurnitureRepository->>+DatabaseConnection: Update(SET reserved=true...)
    DatabaseConnection-->>-FurnitureRepository: Result (1 row affected)
    FurnitureRepository-->>-FurnitureRentService: nil (success)
    
    FurnitureRentService-->>-Client: nil (success)
```

## 3. Sequence Diagram - Failed Rent Flow (Not Available)

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant FurnitureRentService
    participant FurnitureRepository
    participant DatabaseConnection
    
    Client->>+FurnitureRentService: Rent(furniture)
    FurnitureRentService->>+FurnitureRepository: IsAvailable(furnitureId)
    FurnitureRepository->>+DatabaseConnection: Query(SELECT available...)
    DatabaseConnection-->>-FurnitureRepository: Rows (available=false)
    FurnitureRepository-->>-FurnitureRentService: false
    
    FurnitureRentService-->>-Client: error: "furniture not available"
    
    Note over FurnitureRentService,FurnitureRepository: Reserve() is never called
```

## 4. Flowchart - Rent Process

```mermaid
flowchart TD
    A[Client calls Rent] --> B{Is Furniture Available?}
    B -->|Yes| C[Call Reserve]
    B -->|No| D[Return Error: Not Available]
    
    C --> E{Reserve Successful?}
    E -->|Yes| F[Return Success nil]
    E -->|No| G[Return Error: Reserve Failed]
    
    D --> H[End]
    F --> H
    G --> H
    
    style A fill:#e1f5fe
    style F fill:#c8e6c9
    style D fill:#ffcdd2
    style G fill:#ffcdd2
```

## 5. Flowchart - Detailed System Architecture

```mermaid
flowchart TB
    subgraph Client Layer
        CL[Client Application]
    end
    
    subgraph Service Layer
        FRS[FurnitureRentService]
    end
    
    subgraph Repository Layer
        FR{FurnitureRepository\nInterface}
        FRI[FurnitureRepositoryImpl\nProduction]
        MFR[MockFurnitureRepository\nTesting]
    end
    
    subgraph Data Layer
        DC[DatabaseConnection]
        DB[(Database)]
    end
    
    subgraph Test Environment
        TM[Mock Data\navailabilityMap\nreservedItems]
    end
    
    CL --> FRS
    FRS --> FR
    FR -.-> FRI
    FR -.-> MFR
    FRI --> DC
    DC --> DB
    MFR --> TM
    
    style FR fill:#fff3e0,stroke:#ff9800
    style FRI fill:#e8f5e9,stroke:#4caf50
    style MFR fill:#fce4ec,stroke:#e91e63
```

## 6. State Diagram - Furniture States

```mermaid
stateDiagram-v2
    [*] --> Available: Furniture Added
    
    Available --> Reserved: Rent() Success
    Available --> Available: Rent() Failed\n(validation error)
    
    Reserved --> Available: Return Furniture
    Reserved --> Maintenance: Damage Reported
    
    Maintenance --> Available: Repair Complete
    Maintenance --> Retired: Beyond Repair
    
    Retired --> [*]
    
    note right of Available: IsAvailable() = true
    note right of Reserved: IsAvailable() = false
```

## 7. Test Flow Diagram

```mermaid
flowchart LR
    subgraph Test Setup
        A[Create MockRepository] --> B[SetupMockAvailability]
        B --> C[Create FurnitureRentService]
    end
    
    subgraph Test Execution
        C --> D[Call Rent]
        D --> E{Check Result}
    end
    
    subgraph Test Verification
        E --> F[Assert Error/No Error]
        F --> G[WasReserved?]
        G --> H[GetReservationCallCount]
        H --> I[ClearMockData]
    end
    
    style A fill:#bbdefb
    style D fill:#fff9c4
    style F fill:#c8e6c9
```

## 8. Component Interaction Overview

```mermaid
flowchart TB
    subgraph Production
        direction TB
        P1[FurnitureRentService] --> P2[FurnitureRepositoryImpl]
        P2 --> P3[DatabaseConnection]
        P3 --> P4[(MySQL/PostgreSQL)]
    end
    
    subgraph Testing
        direction TB
        T1[FurnitureRentService] --> T2[MockFurnitureRepository]
        T2 --> T3[In-Memory Maps]
    end
    
    INT{FurnitureRepository\nInterface}
    
    P2 -.->|implements| INT
    T2 -.->|implements| INT
    P1 -->|depends on| INT
    T1 -->|depends on| INT
    
    style INT fill:#ffecb3,stroke:#ffa000,stroke-width:2px
```

These diagrams provide a complete visualization of your Go furniture rental system architecture, workflows, and testing strategy.
