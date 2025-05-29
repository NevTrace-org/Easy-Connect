![Easy Connect logo](../easyconnect-logo.jpg)

# Easy Connect Backend - Technical Documentation

## Table of Contents

- [1. Architecture Overview](#1-architecture-overview)
- [2. Technology Stack](#2-technology-stack)
- [3. Clean Architecture Implementation](#3-clean-architecture-implementation)
- [4. Database Design](#4-database-design)
- [5. Domain Layer](#5-domain-layer)
- [6. Application Layer](#6-application-layer)
- [7. Infrastructure Layer](#7-infrastructure-layer)
- [8. Presentation Layer](#8-presentation-layer)
- [9. Integration Layer](#9-integration-layer)
- [10. Blockchain Integration & RPC Layer](#10-blockchain-integration--rpc-layer)
- [11. Message Queue System](#11-message-queue-system)
- [12. Security & Authentication](#12-security--authentication)
- [13. API Design Patterns](#13-api-design-patterns)
- [14. Data Flow Architecture](#14-data-flow-architecture)
- [15. Deployment Architecture](#15-deployment-architecture)

---

## 1. Architecture Overview

Easy Connect follows **Clean Architecture** principles to build a scalable blockchain monitoring platform. The system enables users to create smart contract alerts and receive real-time notifications via webhooks when specific conditions are met.

### Core Architectural Principles

- **Dependency Inversion**: Inner layers define interfaces, outer layers implement them
- **Separation of Concerns**: Each layer has distinct responsibilities
- **SOLID Principles**: Applied throughout the codebase
- **Domain-Driven Design**: Business logic centered in the domain layer
- **Event-Driven Architecture**: Asynchronous processing via message queues

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 🌐 PRESENTATION LAYER                       │
│  EasyConnect.API (REST API + AWS Lambda)                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                📋 APPLICATION LAYER                         │
│  • EasyConnect.Application (CQRS + Use Cases)              │
│  • EasyConnect.RPC (Blockchain Integration)                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│               🗄️ INFRASTRUCTURE LAYER                      │
│  EasyConnect.Infrastructure (EF Core + PostgreSQL)         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                💼 DOMAIN LAYER                              │
│  EasyConnect.Domain (Entities + Business Rules)            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│               🔄 INTEGRATION LAYER                          │
│  • EasyConnect.QueueSender (AWS Lambda)                    │
│  • EasyConnect.QueueListener (AWS Lambda)                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Technology Stack

### Backend Technologies

| Component              | Technology            | Version | Purpose                |
| ---------------------- | --------------------- | ------- | ---------------------- |
| **Runtime**            | .NET                  | 8.0     | Core framework         |
| **API Framework**      | ASP.NET Core          | 8.0     | REST API development   |
| **Database**           | PostgreSQL            | Latest  | Primary data store     |
| **ORM**                | Entity Framework Core | 8.0     | Data access layer      |
| **Cloud Platform**     | AWS                   | -       | Infrastructure hosting |
| **Serverless**         | AWS Lambda            | -       | Background processing  |
| **Message Queue**      | AWS SQS               | -       | Asynchronous messaging |
| **Authentication**     | AWS Cognito           | -       | User management        |
| **Object Mapping**     | Mapster               | 7.4.0   | DTO mapping            |
| **API Documentation**  | Swagger/OpenAPI       | 8.1.0   | API documentation      |
| **JSON Serialization** | Newtonsoft.Json       | 13.0.3  | JSON processing        |

### Development Tools

- **IDE**: Visual Studio 2022
- **Version Control**: Git
- **Database Migrations**: EF Core Migrations
- **API Testing**: Swagger UI
- **Deployment**: AWS Lambda Tools

---

## 3. Clean Architecture Implementation

### Project Structure

```
EasyConnect/
├── EasyConnect.Domain/              # 🏛️ Core business entities
├── EasyConnect.Application/         # 📋 Use cases and business logic
├── EasyConnect.Infrastructure/      # 🗄️ Data access and external services
├── EasyConnect.API/                 # 🌐 REST API controllers
├── EasyConnect.RPC/                 # 🔗 Blockchain integration
├── EasyConnect.QueueSender/         # 📤 Alert processing Lambda
└── EasyConnect.QueueListener/       # 📥 Webhook execution Lambda
```

### Dependency Flow

```
API ──────────────────────────────────────────┐
│                                              │
├─► Application ──► Domain ◄── Infrastructure │
│                                              │
└─► RPC ──────────────────────────────────────┘

QueueSender ──► Application ──► Infrastructure
│
└─► RPC

QueueListener ──► Application ──► Infrastructure
```

### Layer Responsibilities

| Layer              | Responsibilities                                  | Dependencies                 |
| ------------------ | ------------------------------------------------- | ---------------------------- |
| **Domain**         | Business entities, value objects, domain services | None                         |
| **Application**    | Use cases, DTOs, interfaces, business workflows   | Domain only                  |
| **Infrastructure** | Data persistence, external APIs, configurations   | Application + Domain         |
| **Presentation**   | HTTP endpoints, request/response handling         | Application                  |
| **Integration**    | Background jobs, message processing               | Application + Infrastructure |

---

## 4. Database Design

### Entity Relationship Diagram

```mermaid
erDiagram
    NETWORKS ||--o{ RPC_CALLS : contains
    ADDRESSES ||--o{ CONTRACTS : has
    CONTRACTS ||--o{ METHODS : contains
    METHODS ||--o{ VARIABLES : has
    METHODS ||--o{ ALERTS : monitors
    ALERTS ||--o{ CONDITIONS : defines
    ALERTS ||--o{ NOTIFICATIONS : generates
    ALERTS ||--o{ ALERT_FROM_ADDRESSES : filters
    ALERTS ||--o{ ALERT_TO_ADDRESSES : targets
    VARIABLES ||--o{ CONDITIONS : evaluates
    ADDRESSES ||--o{ ALERT_FROM_ADDRESSES : sources
    ADDRESSES ||--o{ ALERT_TO_ADDRESSES : destinations

    NETWORKS {
        uuid id PK
        string name
        string url
        boolean deleted
        datetime created
        datetime updated
    }

    ADDRESSES {
        uuid id PK
        string identity UK
        boolean deleted
        datetime created
        datetime updated
    }

    CONTRACTS {
        uuid id PK
        uuid address_id FK
        string name
        text abi
        string git_url
        boolean deleted
        datetime created
        datetime updated
    }

    METHODS {
        uuid id PK
        uuid contract_id FK
        string name
        text signature
        boolean deleted
        datetime created
        datetime updated
    }

    VARIABLES {
        uuid id PK
        uuid method_id FK
        string name
        int type
        boolean deleted
        datetime created
        datetime updated
    }

    ALERTS {
        uuid id PK
        uuid user_id
        uuid method_id FK
        string name
        string description
        string webhook
        boolean is_active
        boolean deleted
        datetime created
        datetime updated
    }

    CONDITIONS {
        uuid id PK
        uuid alert_id FK
        uuid variable_id FK
        string value
        int type
        boolean deleted
        datetime created
        datetime updated
    }

    NOTIFICATIONS {
        uuid id PK
        uuid alert_id FK
        string webhook_url
        text data
        datetime date
        text response
        int status_code
        boolean succeeded
        text exception_message
        bigint elapsed_milliseconds
        boolean deleted
        datetime created
        datetime updated
    }
```

### Database Schema Design Principles

#### 1. **Audit Trail Pattern**

Every entity inherits from base `Entity` class:

```csharp
public abstract class Entity
{
    public Guid Id { get; set; }
    public bool Deleted { get; set; }        // Soft delete
    public DateTime Created { get; set; }     // Creation timestamp
    public DateTime? Updated { get; set; }    // Last modification
}
```

#### 2. **Soft Delete Implementation**

- Logical deletion via `Deleted` flag
- Global query filter prevents deleted records from queries
- Maintains data integrity and audit history

#### 3. **Schema Organization**

- Dedicated schema: `easy_connect`
- Consistent naming: snake_case for tables/columns
- Unique constraints on business keys
- Proper foreign key relationships

### Key Database Relationships

#### **Core Business Entities**

- `Contracts` ↔ `Methods`: One-to-Many
- `Methods` ↔ `Variables`: One-to-Many
- `Methods` ↔ `Alerts`: One-to-Many
- `Alerts` ↔ `Conditions`: One-to-Many

#### **Address Filtering**

- `AlertFromAddress`: Many-to-Many (Alert ↔ Address)
- `AlertToAddress`: Many-to-Many (Alert ↔ Address)

#### **Notification Tracking**

- `Alerts` ↔ `Notifications`: One-to-Many
- Stores webhook delivery results and metrics

---

## 5. Domain Layer

The Domain layer contains the core business entities and rules. It has no dependencies on other layers.

### Core Entities

#### **Alert Entity**

```
CLASS Alert EXTENDS Entity:
    PROPERTIES:
        - UserId: Unique identifier for user ownership
        - Name: Human-readable alert name
        - Description: Optional alert description
        - Webhook: URL endpoint for notifications
        - IsActive: Boolean flag to enable/disable alert

    RELATIONSHIPS:
        - Method: Smart contract method to monitor
        - Conditions: List of evaluation criteria
        - Notifications: History of sent notifications
        - FromAddresses: Source address filters (optional)
        - ToAddresses: Destination address filters (optional)

    BUSINESS RULES:
        - Constructor sets IsActive = false by default
        - UserId enforces user isolation
        - Cannot be physically deleted (soft delete only)
        - Must have valid method and webhook to activate
```

#### **Contract Entity**

```
CLASS Contract EXTENDS Entity:
    PROPERTIES:
        - Name: Smart contract display name
        - AddressId: Reference to blockchain address
        - ABI: Application Binary Interface definition
        - GitUrl: Source code repository link

    RELATIONSHIPS:
        - Address: Blockchain address entity
        - Methods: Collection of callable contract methods

    BUSINESS RULES:
        - Constructor initializes empty Methods collection
        - One-to-one relationship with Address
        - ABI stores contract interface definition
```

#### **Condition Entity**

```
CLASS Condition EXTENDS Entity:
    PROPERTIES:
        - AlertId: Reference to parent alert
        - VariableId: Reference to method variable
        - Value: Expected value for comparison
        - Type: Comparison operator (Equals, GreaterThan, etc.)

    RELATIONSHIPS:
        - Variable: Method parameter to evaluate
        - Alert: Parent alert that owns this condition

    BUSINESS RULES:
        - Constructor validates required fields
        - Type determines comparison logic
        - Value stored as string, converted at evaluation time
```

### Domain Enums

#### **ConditionType**

```
ENUM ConditionType:
    - GreaterThan: Numeric comparison (>)
    - LessThan: Numeric comparison (<)
    - Equals: Exact value match
    - Contains: String substring search
    - NotEquals: Value inequality check
```

#### **VariableType**

```
ENUM VariableType:
    - String: Text data type
    - Integer: General numeric type
    - Int64: 64-bit signed integer
    - UInt64: 64-bit unsigned integer
    - Int32: 32-bit signed integer
    - UInt32: 32-bit unsigned integer
    - Decimal: Floating-point numbers
    - Boolean: True/false values
    - Address: Base32 encoded blockchain addresses
    - ByteArray: Raw binary data
```

### Business Rules Implementation

#### **Entity Lifecycle**

- All entities have audit fields (Created, Updated, Deleted)
- Soft delete prevents data loss
- Business invariants enforced in constructors

#### **Alert Business Rules**

- Alerts start inactive by default
- Cannot be physically deleted (audit trail)
- Must have valid method and webhook to activate
- User isolation enforced via UserId

---

## 6. Application Layer

The Application layer implements use cases using the **CQRS pattern** (Command Query Responsibility Segregation).

### CQRS Implementation

#### **Commands** (Write Operations)

```
Application/
├── Alerts/
│   ├── Commands/
│   │   ├── Create.cs
│   │   ├── Update.cs
│   │   ├── Delete.cs
│   │   ├── UpdateMethod.cs
│   │   └── UpdateWebhook.cs
│   └── Queries/
│       ├── Get.cs
│       ├── GetAll.cs
│       └── GetByUser.cs
└── Conditions/
    ├── Commands/
    │   ├── Create.cs
    │   ├── Update.cs
    │   └── Delete.cs
```

#### **Command Pattern Implementation**

```
CLASS Create (Alert Creation Command):
    DEPENDENCIES:
        - IQubikDbContext: Database context for persistence

    METHOD Execute(userId, createAlertDto):
        1. Create new Alert entity with provided data
        2. Set user ownership via userId
        3. Add entity to database context
        4. Save changes to database
        5. Map entity to DTO and return

    VALIDATION:
        - UserId must be valid
        - Name is required
        - Description is optional
```

#### **Query Pattern Implementation**

```
CLASS GetByUser (Alert Query by User):
    DEPENDENCIES:
        - IQubikDbContext: Database context for data access

    METHOD Execute(userId):
        1. Query alerts table filtered by userId
        2. Include related entities (Method, Contract, Conditions, Variables)
        3. Apply soft delete filter automatically
        4. Map entities to DTOs using Mapster
        5. Return collection of AlertDto objects

    PERFORMANCE:
        - Uses Include() for eager loading
        - Filters at database level
        - Returns only non-deleted records
```

### Data Transfer Objects (DTOs)

#### **AlertDto**

```
CLASS AlertDto (Data Transfer Object):
    PROPERTIES:
        - Id: Unique alert identifier
        - Name: Alert display name
        - Description: Alert description
        - Webhook: Notification endpoint URL
        - IsActive: Current status flag
        - MethodId: Referenced method identifier

    NAVIGATION PROPERTIES:
        - Method: Referenced smart contract method
        - Conditions: Alert evaluation criteria
        - FromAddresses: Source address filters
        - Notifications: Historical notifications

    PURPOSE:
        - Transfer data between layers
        - Hide internal entity complexity
        - Provide flat structure for API responses
```

### Interface Abstraction

#### **IQubikDbContext**

```
INTERFACE IQubikDbContext (Database Context Contract):
    PROPERTIES:
        - Alerts: Alert entity collection
        - Addresses: Address entity collection
        - Notifications: Notification entity collection
        - Conditions: Condition entity collection
        - Contracts: Contract entity collection
        - Methods: Method entity collection
        - Variables: Variable entity collection
        - AlertsFromAddress: Alert-Address relationship collection
        - AlertsToAddress: Alert-Address relationship collection
        - Networks: Network entity collection
        - RPCCalls: RPC call audit collection

    METHODS:
        - SaveChangesAsync(): Persist changes to database

    PURPOSE:
        - Abstract database operations
        - Enable dependency injection
        - Support unit testing with mocks
        - Provide transaction boundaries
```

### Object Mapping Strategy

#### **Mapster Configuration**

```
CLASS DependencyInjection (Service Registration):
    METHOD AddApplication():
        1. Register Mapster mapping services
        2. Configure automatic DTO mapping
        3. Return configured service collection

    BENEFITS:
        - High performance object mapping
        - Convention-based configuration
        - Minimal setup required
        - Compile-time code generation
        - Better performance than AutoMapper
```

---

## 7. Infrastructure Layer

The Infrastructure layer implements external concerns like data persistence and configuration.

### Entity Framework Configuration

#### **DbContext Implementation**

```
CLASS QubikDbContext IMPLEMENTS IQubikDbContext:
    INHERITANCE: DbContext (Entity Framework)

    CONFIGURATION:
        - Default schema: "easy_connect"
        - Apply entity configurations via Fluent API
        - Configure soft delete global query filters
        - Set up audit field automatic handling

    METHOD OnModelCreating():
        1. Set default schema for all entities
        2. Apply individual entity configurations
        3. Configure soft delete filters for all entities
        4. Set up relationships and constraints

    METHOD SaveChangesAsync():
        1. Iterate through tracked entity changes
        2. Set Created/Updated timestamps automatically
        3. Convert physical deletes to soft deletes
        4. Execute base save changes
        5. Return number of affected records

    FEATURES:
        - Automatic audit trail
        - Soft delete implementation
        - UTC timestamp handling
        - Change tracking optimization
```

#### **Entity Configurations**

```
CLASS AlertConfiguration IMPLEMENTS IEntityTypeConfiguration<Alert>:
    METHOD Configure(entityBuilder):
        1. Apply base entity configuration (table name, keys, indexes)
        2. Configure property constraints:
           - UserId: Required, 36 character limit
           - Name: Required, 100 character limit
           - Description: Optional, 500 character limit
           - Webhook: Optional, 200 character limit
           - IsActive: Required boolean
        3. Define relationships:
           - One-to-many with Notifications (cascade delete)
           - One-to-many with Conditions (cascade delete)
           - Many-to-one with Method (no action on delete)
           - Many-to-many with Addresses via junction tables
        4. Set foreign key constraints and delete behaviors

    PURPOSE:
        - Centralize entity configuration
        - Define database schema via code
        - Ensure data integrity constraints
        - Configure relationship behaviors
```

### Migration Strategy

#### **Database Migrations Timeline**

1. **Initial Migration** (`20250418073849_Initial`)

   - Core schema creation
   - Base entities and relationships

2. **Contract Address Fix** (`20250418150324_ContractAddressRelationship`)

   - Fixed Contract-Address relationship
   - Added proper foreign key constraints

3. **User Isolation** (`20250430141451_UserIdInAlerts`)

   - Added UserId to Alerts table
   - Enables multi-tenant isolation

4. **Method Flexibility** (`20250430142025_MethodIdNullableOnAlerts`)
   - Made MethodId nullable in Alerts
   - Allows alert creation before method selection

### Data Seeding

#### **Contract Seeding Strategy**

```
CLASS DataSeed (Database Initialization):
    METHOD SeedContracts(dbContext):
        1. Check if Qubic network already exists
        2. Create Qubic network entity if not found
        3. Define smart contract specifications:
           - Quottery Contract (betting system)
           - QX Contract (asset exchange)
        4. For each contract:
           - Create contract entity with metadata
           - Define methods (IssueBet, JoinBet, etc.)
           - Define variables for each method with types
           - Set up proper relationships
        5. Add all entities to context and save

    CONTRACT DEFINITIONS:
        - Quottery: Betting system with 4 procedures
        - QX: Asset exchange with 7 procedures

    PURPOSE:
        - Initialize system with known contracts
        - Provide base data for alert configuration
        - Enable immediate system functionality
        - Support development and testing
```

### Dependency Injection Configuration

```
CLASS DependencyInjection (Infrastructure Registration):
    METHOD AddInfrastructure(services, configuration):
        1. Resolve database connection string:
           - Check environment variable first
           - Fallback to configuration file
        2. Configure Entity Framework:
           - Use PostgreSQL provider
           - Enable connection retry logic
           - Configure performance optimizations
        3. Register database context as scoped service
        4. Return configured service collection

    CONFIGURATION FEATURES:
        - Environment-aware connection strings
        - Production vs Development settings
        - Connection pooling optimization
        - Retry policies for transient failures
```

---

## 8. Presentation Layer

The Presentation layer exposes the application functionality through REST API endpoints.

### API Controller Design

#### **Base Controller Pattern**

```
CLASS V1ControllerBase EXTENDS ControllerBase:
    ATTRIBUTES:
        - ApiController: Enables automatic model validation
        - ApiVersion: Specifies version 1.0
        - Route: Template "api/v{version:apiVersion}/[controller]"

    METHOD GetUserId():
        1. Extract JWT claims from current HTTP context
        2. Find NameIdentifier claim (user ID)
        3. Parse and return as Guid
        4. Used for user isolation in all operations

    PURPOSE:
        - Provide common functionality for all controllers
        - Enforce consistent routing patterns
        - Handle user authentication extraction
        - Enable API versioning
```

#### **Alert Controller Implementation**

```
CLASS AlertController EXTENDS V1ControllerBase:
    DEPENDENCIES:
        - IQubikDbContext: Database access

    ENDPOINTS:

    POST /alerts (CreateAlert):
        1. Extract user ID from JWT token
        2. Create new alert using Create command
        3. Return 201 Created with alert DTO
        4. Include location header for new resource

    GET /alerts (GetAlertsByUser):
        1. Extract user ID from JWT token
        2. Query alerts filtered by user
        3. Return 200 OK with alert collection

    PATCH /alerts/method (UpdateMethod):
        1. Extract user ID from JWT token
        2. Update alert method using UpdateMethod command
        3. Clear existing conditions (business rule)
        4. Return 204 No Content

    FEATURES:
        - User isolation enforced on all operations
        - Command/Query pattern implementation
        - RESTful HTTP status codes
        - DTO-based request/response handling
```

### API Versioning Strategy

#### **Version Configuration**

```
API VERSIONING CONFIGURATION:
    DEFAULT VERSION: 1.0
    VERSION SOURCES:
        - URL segment: /api/v1.0/alerts
        - HTTP header: X-Api-Version

    FEATURES:
        - Assume default version when unspecified
        - Report available versions in response headers
        - Support multiple version readers simultaneously
        - Generate versioned Swagger documentation

    URL PATTERN:
        - Template: "api/v{version:apiVersion}/[controller]"
        - Example: "api/v1.0/alerts"
        - Automatic version substitution in documentation
```

### Swagger/OpenAPI Configuration

#### **Multi-Version Documentation**

```
CLASS ConfigureSwaggerOptions IMPLEMENTS IConfigureOptions<SwaggerGenOptions>:
    DEPENDENCIES:
        - IApiVersionDescriptionProvider: Discovers available API versions

    METHOD Configure(swaggerOptions):
        1. Iterate through all discovered API versions
        2. For each version, create Swagger document:
           - Title: "Easy Connect API"
           - Version: API version string
           - Description: Version-specific description
           - Contact information
        3. Configure Swagger UI to display all versions

    FEATURES:
        - Automatic version discovery
        - Version-specific documentation
        - Consistent metadata across versions
        - Developer contact information
        - Supports API evolution strategy
```

### AWS Lambda Integration

#### **Lambda Entry Point**

```csharp
public class LambdaEntryPoint : Amazon.Lambda.AspNetCoreServer.APIGatewayProxyFunction
{
    protected override void Init(IWebHostBuilder builder)
    {
        builder.UseStartup<Startup>();
    }
}
```

#### **Local Development Entry Point**

```csharp
public class LocalEntryPoint
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

---

## 9. Integration Layer

The Integration layer handles background processing and external system communication through AWS Lambda functions.

### Queue-Based Architecture

```
Blockchain ──► QueueSender ──► SQS Queue ──► QueueListener ──► Webhook
             (Monitor)                      (Process)        (Notify)
```

### QueueSender Lambda

#### **Function Class Implementation**

The QueueSender Function class serves as the main entry point for blockchain monitoring operations. It implements a scheduled Lambda function that executes every 5 minutes to check for new blockchain transactions.

**Core Responsibilities:**

- Configure application services using dependency injection
- Retrieve active alerts from the database
- Fetch current blockchain tick information from RPC
- Determine starting point for transaction retrieval based on last processed tick
- Process and parse smart contract transactions
- Evaluate each transaction against active alert conditions
- Send matching alerts to SQS queue for webhook delivery
- Store RPC call metadata for audit trail

**Service Configuration Process:**
The function loads environment variables from a .env file and builds a configuration object from environment variables. It then creates a service provider with proper dependency injection setup, including database context and application services registration.

**Transaction Processing Flow:**
The function retrieves the latest tick from the Qubic RPC endpoint and determines the starting tick for processing. If no previous RPC calls exist, it processes the last 1 million ticks. Otherwise, it continues from the last processed tick. The function then fetches all QX contract transactions in paginated batches and parses them using the QX contract parser.

#### **Alert Evaluation Engine**

The AlertEvaluator class contains static methods for matching blockchain transactions against user-defined alert conditions.

**Transaction Matching Logic:**
The evaluation engine first checks if the alert's method name matches the transaction type. It then iterates through each condition defined in the alert and uses reflection to extract the corresponding property value from the parsed transaction. The extracted value is compared against the expected value using the specified condition type (equals, greater than, contains, etc.).

**Type Conversion System:**
The engine includes a robust type conversion system that handles different variable types including strings, integers, addresses, and binary data. It properly converts string values to the expected data types based on the variable type definition.

**Condition Evaluation:**
The system supports multiple comparison operators including equality, inequality, greater than, less than, and string containment. For comparable types, it uses the IComparable interface for numeric comparisons.

### QueueListener Lambda

#### **SQS Message Processing Function**

The QueueListener Function processes SQS messages containing alert match notifications and handles webhook delivery.

**Batch Processing Implementation:**
The function receives SQS events containing multiple message records and processes them in batches. For each message, it deserializes the alert match data and processes the webhook notification. Failed messages are marked for retry by adding them to the batch failure response, which triggers SQS retry logic.

**Error Handling Strategy:**
The function implements comprehensive error handling with try-catch blocks around individual message processing. Exceptions are logged with detailed error information, and failed messages are properly marked for SQS retry handling.

#### **Webhook Delivery Service**

The WebhookCaller class handles HTTP webhook delivery with comprehensive monitoring and error tracking.

**Delivery Process:**
The service serializes the transaction data to JSON format and makes HTTP POST requests to the configured webhook URLs. It tracks delivery metrics including response time, HTTP status codes, and response content.

**Performance Monitoring:**
The service uses Stopwatch to measure delivery time and captures detailed information about each webhook call including success/failure status, response data, and any exception messages. All delivery attempts are stored in the database for audit and debugging purposes.

**Notification Creation:**
For each webhook call, the service creates a Notification entity that stores the complete delivery details including the sent payload, webhook response, delivery status, and performance metrics.

### Queue Management

#### **SQS Queue Manager Implementation**

The QueueManager class handles message serialization and SQS queue operations.

**Message Serialization:**
The queue manager serializes alert match data into JSON format using Newtonsoft.Json with type name handling enabled. This ensures that complex object hierarchies, including parsed transaction data, are properly preserved across the queue boundary.

**Queue Operations:**
The manager creates SQS message requests with the serialized payload and sends them to the configured SQS queue URL retrieved from environment variables. It uses the AWS SDK for .NET to handle all SQS interactions.

---

## 10. Blockchain Integration & RPC Layer

The RPC layer handles communication with the Qubic blockchain and smart contract parsing.

### RPC Client Architecture

#### **Main RPC Client Implementation**

The RPCClient class provides a high-level interface for interacting with the Qubic blockchain through HTTP RPC calls.

**Core Functionality:**
The client uses an internal HTTP client wrapper to make REST API calls to the Qubic RPC endpoint. It provides methods for retrieving the current blockchain tick and fetching transaction data for specific smart contracts.

**Current Tick Retrieval:**
The GetCurrentTick method queries the latest tick information from the blockchain, which is used to determine the processing window for new transactions.

**Transaction Fetching Strategy:**
The GetAllQxTransfers method implements a pagination strategy to retrieve all transactions for the QX contract starting from a specified tick. It continues fetching pages until all available transactions are retrieved, handling large result sets efficiently.

**Pagination Handling:**
The client processes transaction results in pages with a configurable page size (250 transactions per page). It continues fetching subsequent pages until reaching the total page count returned by the API.

### Smart Contract Parsing

#### **Contract Transaction Structure**

The ContractTransaction generic class provides a typed wrapper for blockchain transactions with parsed data.

**Generic Implementation:**
The class uses generic type parameters to support different contract types and parsed data formats. It stores both the raw blockchain transaction data and the parsed, structured data for easy access.

**Property Mapping:**
The structure provides properties for the procedure name (derived from the enum), input type enumeration, raw transaction data, and parsed transaction data with strong typing.

#### **QX Contract Implementation**

The QxContract class handles parsing and processing of QX smart contract transactions.

**Contract Metadata:**
The class includes metadata about the smart contract including the GitHub repository URL and display name for documentation and debugging purposes.

**Transaction Parsing Process:**
The ParseTransactions method processes a collection of raw blockchain transactions and converts them into strongly-typed contract transactions. It uses a parser mapping system to determine the appropriate parser for each transaction type.

**Parser Resolution:**
For each transaction, the system converts the input type to the corresponding enum value and looks up the appropriate parser class from the parser mapping dictionary. It then creates an instance of the parser and processes the transaction data.

**Error Handling:**
The parsing process includes comprehensive error handling that logs parsing errors but continues processing remaining transactions to ensure system resilience.

#### **Transaction Parser Mapping System**

The QxTransactionParsers static class maintains a mapping between transaction types and their corresponding parser implementations.

**Parser Dictionary:**
The system uses a static dictionary that maps QxProcedureType enum values to their corresponding parser class types. This enables dynamic parser selection based on transaction type.

**Supported Operations:**
The mapping includes parsers for all QX contract operations including asset issuance, ownership transfers, order management (ask/bid), and share management rights transfers.

### Transaction Parsing Examples

#### **QX Add To Ask Order Parser Implementation**

The QxAddToAskOrder parser handles asset selling order transactions on the QX exchange.

**Byte Array Processing:**
The parser processes a 56-byte payload containing the transaction data. It validates the payload length and throws exceptions for invalid data to ensure data integrity.

**Field Extraction:**
The parser extracts the issuer address (32 bytes converted to Base32), asset name (8 bytes as ASCII string), price (8-byte integer), and number of shares (8-byte integer) from specific byte positions in the payload.

**Data Validation:**
The parser includes validation logic to ensure proper data format and removes null characters from string fields to handle C-style string termination.

#### **Quottery Issue Bet Parser Implementation**

The QuotteryIssueBetInput parser handles complex betting transaction data from the Quottery smart contract.

**Complex Data Structure:**
The parser processes a 600-byte payload containing multiple data sections including bet descriptions, option descriptions, oracle provider information, fees, dates, and betting parameters.

**Array Processing:**
The parser handles arrays of data including 8 option descriptions, 8 oracle providers, and 8 oracle fees. It processes each array element sequentially and advances the byte offset appropriately.

**Custom Date Decoding:**
The parser implements a custom date decoding algorithm for Quottery-specific date format. It extracts year, month, day, hour, minute, and second from a compressed 32-bit integer format.

**Base32 Encoding:**
The parser uses a custom Base32 encoding utility to convert raw byte arrays into human-readable addresses and identifiers following Qubic's address format standards.

---

## 11. Message Queue System

### Queue Architecture Design

```
┌─────────────────┐    ┌─────────────┐    ┌─────────────────┐
│   QueueSender   │───▶│  SQS Queue  │───▶│  QueueListener  │
│   (Producer)    │    │             │    │   (Consumer)    │
└─────────────────┘    └─────────────┘    └─────────────────┘
        │                                          │
        ▼                                          ▼
┌─────────────────┐                    ┌─────────────────┐
│   Blockchain    │                    │   Webhook API   │
│   Monitoring    │                    │   Integration   │
└─────────────────┘                    └─────────────────┘
```

### Message Flow Design

#### **1. Alert Matching Process (QueueSender)**

```
1. Fetch latest blockchain transactions from RPC
2. Parse smart contract data into structured objects
3. Evaluate each transaction against active alerts
4. For matches: Send message to SQS queue
5. Store RPC call metadata in database
```

#### **2. Webhook Delivery Process (QueueListener)**

```
1. Receive SQS message batch
2. Deserialize alert match data
3. Call webhook endpoint with transaction payload
4. Record notification result in database
5. Handle delivery failures with retry logic
```

### Error Handling Strategy

#### **Message Processing Resilience**

The message processing system implements comprehensive error handling to ensure system reliability and data consistency.

**Individual Message Processing:**
Each SQS message is processed individually within a try-catch block to isolate failures. When a message fails to process, it's marked for retry without affecting other messages in the batch.

**Batch Response Management:**
The system builds batch responses that indicate which messages failed processing. Failed messages are added to the BatchItemFailures collection, which instructs SQS to retry those specific messages while acknowledging successful ones.

**Retry Logic Implementation:**
SQS handles automatic message retries based on the queue configuration. Messages that fail processing multiple times are eventually moved to a dead letter queue for manual investigation.

#### **Webhook Delivery Resilience**

The webhook delivery system includes multiple layers of resilience to handle network failures and endpoint unavailability.

**Timeout Configuration:**
Each webhook call is configured with a 30-second maximum timeout to prevent hanging requests from blocking the system.

**Automatic Retries:**
Failed webhook deliveries are automatically retried through the SQS retry mechanism. The QueueListener will reprocess failed messages according to the queue's retry policy.

**Dead Letter Queue:**
Messages that fail processing after multiple retry attempts are moved to a dead letter queue for manual investigation and recovery.

**Comprehensive Monitoring:**
All webhook delivery attempts are logged with detailed metrics including response times, status codes, and error messages for operational monitoring and debugging.

---

## 13. API Design Patterns

### RESTful API Design

#### **Resource Naming Conventions**

```
GET    /api/v1/alerts              # Get user alerts
POST   /api/v1/alerts              # Create new alert
GET    /api/v1/alerts/{id}         # Get specific alert
PUT    /api/v1/alerts              # Update alert
DELETE /api/v1/alerts/{id}         # Delete alert
PATCH  /api/v1/alerts/method       # Update alert method
PATCH  /api/v1/alerts/webhook      # Update alert webhook

GET    /api/v1/contracts           # Get available contracts
GET    /api/v1/notifications/from-alert/{alertId}  # Get alert notifications

POST   /api/v1/conditions          # Create condition
PUT    /api/v1/conditions          # Update condition
DELETE /api/v1/conditions/{id}     # Delete condition
```

#### **HTTP Status Code Usage**

The API follows RESTful conventions for HTTP status codes to provide clear communication about operation results.

**Success Responses:**

- 200 OK: Used for successful GET requests and general successful operations
- 201 Created: Used for successful POST requests that create new resources
- 204 No Content: Used for successful PUT, PATCH, and DELETE operations that don't return data

**Error Responses:**

- 400 Bad Request: Used for invalid request data or validation failures
- 401 Unauthorized: Used when authentication is required but missing or invalid
- 403 Forbidden: Used when the user lacks permission for the requested operation
- 404 Not Found: Used when the requested resource doesn't exist
- 500 Internal Server Error: Used for unexpected server-side errors

**Location Headers:**
For resource creation operations, the API includes Location headers pointing to the newly created resource URL following REST best practices.

### Request/Response Patterns

#### **Command Pattern Integration**

The API controllers integrate seamlessly with the CQRS command pattern implemented in the application layer.

**Command Execution Flow:**
Each controller action creates the appropriate command or query object, injects the database context dependency, and executes the operation with user context. The results are then mapped to appropriate HTTP responses.

**User Context Propagation:**
All operations include user context extracted from JWT tokens to ensure proper data isolation and security. The base controller provides utility methods for extracting user information from authentication claims.

#### **Validation Strategy**

The API implements multi-layer validation including data annotations, model state validation, and business rule validation.

**Data Annotations:**
DTOs include validation attributes such as Required, MaxLength, and Range to enforce basic data constraints at the model binding level.

**Model State Validation:**
Controllers automatically validate model state and return appropriate error responses for invalid requests without executing business logic.

**Business Validation:**
Additional business rule validation occurs in the application layer commands and queries to ensure data consistency and business invariant enforcement.

### Error Handling Strategy

#### **Consistent Error Response Format**

The API implements standardized error response formats to provide consistent client experience across all endpoints.

**Error Response Structure:**
All error responses include standard fields such as error message, details, timestamp, and optional validation errors for comprehensive client-side error handling.

**Exception Categories:**
The system categorizes exceptions into different types including validation errors, business rule violations, authorization failures, and system errors, each with appropriate HTTP status codes.

**Logging Integration:**
All errors are logged with appropriate detail levels including request context, user information, and stack traces for debugging while avoiding sensitive data exposure.

---

## 14. Data Flow Architecture

### Complete System Data Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │───▶│   API Gateway   │───▶│  EasyConnect    │
│   (Angular)     │    │   (AWS Lambda)  │    │     API         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                                                       ▼
                                               ┌─────────────────┐
                                               │   PostgreSQL    │
                                               │   Database      │
                                               └─────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Qubic RPC     │───▶│  QueueSender    │───▶│   SQS Queue     │
│   Blockchain    │    │   (Lambda)      │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                                                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  External       │◄───│ QueueListener   │◄───│   PostgreSQL    │
│  Webhooks       │    │   (Lambda)      │    │   Database      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Alert Processing Flow

#### **1. Alert Creation Flow**

```
User Request → API Controller → Application Command → Domain Entity → Database
     │
     └─► Response DTO ◄── Mapster ◄── Saved Entity
```

#### **2. Blockchain Monitoring Flow**

```
Scheduled Trigger → QueueSender Lambda → RPC Client → Qubic Blockchain
     │
     ├─► Parse Transactions → Smart Contract Parsers
     │
     ├─► Evaluate Alerts → Alert Matching Engine
     │
     ├─► Send to Queue → SQS Message
     │
     └─► Store RPC Call → Database Audit Log
```

#### **3. Webhook Delivery Flow**

```
SQS Message → QueueListener Lambda → Deserialize Alert Data
     │
     ├─► HTTP Client → External Webhook → Response
     │
     └─► Store Notification → Database (Success/Failure + Metrics)
```

### Database Transaction Patterns

#### **Unit of Work Pattern Implementation**

The system implements the Unit of Work pattern through Entity Framework's DbContext, which manages transaction boundaries and change tracking.

**Transaction Scope:**
Each command operation runs within a single transaction scope that encompasses all related database operations. Changes are committed atomically to ensure data consistency.

**Change Tracking:**
Entity Framework tracks entity state changes automatically and generates appropriate SQL commands during SaveChanges operations.

**Optimistic Concurrency:**
The system uses optimistic concurrency control through entity timestamps to handle concurrent updates without explicit locking.

#### **Change Tracking & Audit Implementation**

The system implements comprehensive audit trailing through automatic change tracking in the database context.

**Audit Field Management:**
The SaveChangesAsync override automatically sets Created and Updated timestamps for all entity changes. New entities receive both timestamps, while modified entities only update the Updated field.

**Soft Delete Implementation:**
Delete operations are converted to soft deletes by setting the Deleted flag and updating the timestamp. The entity state is changed from Deleted to Modified to prevent physical deletion.

**Global Query Filters:**
Soft delete functionality is enforced through global query filters that automatically exclude deleted entities from all queries without requiring explicit filtering in application code.End('\0');
Price = BitConverter.ToInt64(payload, 40);
NumberOfShares = BitConverter.ToInt64(payload, 48);
}
}

````

#### **Quottery Issue Bet Parser**
```csharp
public class QuotteryIssueBetInput : IParsedTransaction
{
    public string BetDescription { get; set; }
    public List<string> OptionDescriptions { get; set; }
    public List<string> OracleProviders { get; set; }
    public List<uint> OracleFees { get; set; }
    public DateTime CloseDate { get; set; }
    public DateTime EndDate { get; set; }
    public ulong AmountPerSlot { get; set; }
    public uint MaxSlotPerOption { get; set; }
    public uint NumberOfOptions { get; set; }

    public void Parse(string inputHex)
    {
        byte[] payload = Convert.FromHexString(inputHex);
        if (payload.Length != 600)
            throw new ArgumentException("Payload must be exactly 600 bytes.");

        int offset = 0;

        // Parse bet description (32 bytes -> Base32)
        byte[] betDescBytes = payload.Skip(offset).Take(32).ToArray();
        BetDescription = DecodingUtils.EncodeBase32(betDescBytes);
        offset += 32;

        // Parse option descriptions (8 x 32 bytes)
        OptionDescriptions = new List<string>();
        for (int i = 0; i < 8; i++)
        {
            byte[] optionBytes = payload.Skip(offset).Take(32).ToArray();
            OptionDescriptions.Add(DecodingUtils.EncodeBase32(optionBytes));
            offset += 32;
        }

        // Parse oracle providers (8 x 32 bytes)
        OracleProviders = new List<string>();
        for (int i = 0; i < 8; i++)
        {
            byte[] oracleBytes = payload.Skip(offset).Take(32).ToArray();
            OracleProviders.Add(DecodingUtils.EncodeBase32(oracleBytes));
            offset += 32;
        }

        // Parse oracle fees (8 x 4 bytes)
        OracleFees = new List<uint>();
        for (int i = 0; i < 8; i++)
        {
            OracleFees.Add(BitConverter.ToUInt32(payload, offset));
            offset += 4;
        }

        // Parse dates with custom decoding
        uint rawCloseDate = BitConverter.ToUInt32(payload, offset);
        CloseDate = DecodeQtryDate(rawCloseDate);
        offset += 4;

        uint rawEndDate = BitConverter.ToUInt32(payload, offset);
        EndDate = DecodeQtryDate(rawEndDate);
        offset += 4;

        // Parse remaining fields
        AmountPerSlot = BitConverter.ToUInt64(payload, offset);
        offset += 8;
        MaxSlotPerOption = BitConverter.ToUInt32(payload, offset);
        offset += 4;
        NumberOfOptions = BitConverter.ToUInt32(payload, offset);
    }

    private DateTime DecodeQtryDate(uint raw)
    {
        int year = (int)(raw >> 26) + 2000 + 24; // Base year = 2024
        int month = (int)(raw >> 22 & 0b1111);
        int day = (int)(raw >> 17 & 0b11111);
        int hour = (int)(raw >> 12 & 0b11111);
        int minute = (int)(raw >> 6 & 0b111111);
        int second = (int)(raw & 0b111111);

        try
        {
            return new DateTime(year, month, day, hour, minute, second, DateTimeKind.Utc);
        }
        catch
        {
            return DateTime.MinValue;
        }
    }
}
````

---

## 11. Message Queue System

### Queue Architecture Design

```
┌─────────────────┐    ┌─────────────┐    ┌─────────────────┐
│   QueueSender   │───▶│  SQS Queue  │───▶│  QueueListener  │
│   (Producer)    │    │             │    │   (Consumer)    │
└─────────────────┘    └─────────────┘    └─────────────────┘
        │                                          │
        ▼                                          ▼
┌─────────────────┐                    ┌─────────────────┐
│   Blockchain    │                    │   Webhook API   │
│   Monitoring    │                    │   Integration   │
└─────────────────┘                    └─────────────────┘
```

### Message Flow Design

#### **1. Alert Matching Process (QueueSender)**

```
1. Fetch latest blockchain transactions from RPC
2. Parse smart contract data into structured objects
3. Evaluate each transaction against active alerts
4. For matches: Send message to SQS queue
5. Store RPC call metadata in database
```

#### **2. Webhook Delivery Process (QueueListener)**

```
1. Receive SQS message batch
2. Deserialize alert match data
3. Call webhook endpoint with transaction payload
4. Record notification result in database
5. Handle delivery failures with retry logic
```

### Error Handling Strategy

#### **SQS Batch Processing**

```csharp
public async Task<SQSBatchResponse> FunctionHandler(SQSEvent evnt, ILambdaContext context)
{
    var response = new SQSBatchResponse();

    foreach (var record in evnt.Records)
    {
        try
        {
            await ProcessMessage(record);
        }
        catch (Exception ex)
        {
            // Mark message for retry
            response.BatchItemFailures.Add(new SQSBatchResponse.BatchItemFailure
            {
                ItemIdentifier = record.MessageId
            });
        }
    }

    return response;
}
```

#### **Webhook Delivery Resilience**

- **Timeout Configuration**: 30-second maximum per webhook call
- **Retry Logic**: SQS handles automatic retries for failed messages
- **Dead Letter Queue**: For permanently failed messages
- **Monitoring**: All delivery attempts logged with metrics

---

## 12. Security & Authentication

### AWS Cognito Integration

#### **JWT Authentication Configuration**

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://cognito-idp.eu-west-3.amazonaws.com/eu-west-3_DV0Wghhmw";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidIssuer = "https://cognito-idp.eu-west-3.amazonaws.com/eu-west-3_DV0Wghhmw",
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidateAudience = false // Cognito access tokens don't include 'aud' claim
        };

        options.Events = new JwtBearerEvents
        {
            OnAuthenticationFailed = context =>
            {
                // Log authentication failures
                return Task.CompletedTask;
            },
            OnTokenValidated = context =>
            {
                // Log successful authentications
                return Task.CompletedTask;
            }
        };
    });
```

#### **Authorization Policies**

```csharp
services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

### User Isolation Strategy

#### **User Context Extraction**

```csharp
public abstract class V1ControllerBase : ControllerBase
{
    public Guid GetUserId()
    {
        return Guid.Parse(User.FindFirst(ClaimTypes.NameIdentifier)?.Value);
    }
}
```

#### **Data Access Security**

```csharp
public async Task<IEnumerable<AlertDto>> Execute(Guid userId)
{
    var alerts = await _context.Alerts
        .Where(x => x.UserId == userId) // User isolation
        .Include(x => x.Method)
        .ToListAsync();

    return alerts.Adapt<IEnumerable<AlertDto>>();
}
```

### CORS Configuration

#### **Development vs Production CORS**

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseCors(options =>
        {
            options.AllowAnyHeader();
            options.AllowAnyMethod();
            options.AllowAnyOrigin();
        });
    }
    else
    {
        app.UseCors("ProductionPolicy");
    }
}
```

### Environment Variable Security

#### **Configuration Management**

```csharp
public class Variables
{
    private IConfiguration Configuration { get; set; }

    public string CognitoAuthority
    {
        get
        {
            var authority = Environment.GetEnvironmentVariable("CognitoAuthority");
            if (string.IsNullOrEmpty(authority))
            {
                authority = Configuration["Aws:Cognito:Authority"];
            }
            return authority;
        }
    }

    // Similar pattern for other sensitive configuration
}
```

---

## 13. API Design Patterns

### RESTful API Design

#### **Resource Naming Conventions**

```
GET    /api/v1/alerts              # Get user alerts
POST   /api/v1/alerts              # Create new alert
GET    /api/v1/alerts/{id}         # Get specific alert
PUT    /api/v1/alerts              # Update alert
DELETE /api/v1/alerts/{id}         # Delete alert
PATCH  /api/v1/alerts/method       # Update alert method
PATCH  /api/v1/alerts/webhook      # Update alert webhook

GET    /api/v1/contracts           # Get available contracts
GET    /api/v1/notifications/from-alert/{alertId}  # Get alert notifications

POST   /api/v1/conditions          # Create condition
PUT    /api/v1/conditions          # Update condition
DELETE /api/v1/conditions/{id}     # Delete condition
```

#### **HTTP Status Code Usage**

```csharp
[HttpPost]
public async Task<ActionResult<AlertDto>> CreateAlert([FromBody] CreateAlertDto dto)
{
    var result = await command.Execute(this.GetUserId(), dto);
    return CreatedAtAction(nameof(CreateAlert), new { id = result.Id }, result); // 201
}

[HttpPut]
public async Task<ActionResult> UpdateAlert([FromBody] UpdateAlertDto dto)
{
    await command.Execute(this.GetUserId(), dto);
    return NoContent(); // 204
}

[HttpDelete("{alertId}")]
public async Task<ActionResult> Delete([FromRoute] Guid alertId)
{
    await command.Execute(this.GetUserId(), alertId);
    return NoContent(); // 204
}
```

### Request/Response Patterns

#### **Command DTOs**

```csharp
public class CreateAlertDto
{
    [Required]
    public string Name { get; set; }

    public string Description { get; set; }
}

public class UpdateMethodOnAlertDto
{
    [Required]
    public Guid Id { get; set; }

    [Required]
    public Guid MethodId { get; set; }
}
```

#### **Response DTOs**

```csharp
public class AlertDto
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public string Webhook { get; set; }
    public bool IsActive { get; set; }
    public Guid MethodId { get; set; }

    // Nested objects for related data
    public MethodDto? Method { get; set; }
    public ICollection<ConditionDto> Conditions { get; set; }
    public ICollection<AlertFromAddressDto> FromAddresses { get; set; }
}
```

### Error Handling Strategy

#### **Global Exception Handling**

```csharp
// Implement custom middleware for consistent error responses
public class GlobalExceptionMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        try
        {
            await next(context);
        }
        catch (Exception ex)
        {
            await HandleExceptionAsync(context, ex);
        }
    }

    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        var response = new
        {
            Error = "An error occurred while processing your request.",
            Details = exception.Message,
            Timestamp = DateTime.UtcNow
        };

        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";

        await context.Response.WriteAsync(JsonConvert.SerializeObject(response));
    }
}
```

---

## 14. Data Flow Architecture

### Complete System Data Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │───▶│   API Gateway   │───▶│  EasyConnect    │
│   (Angular)     │    │   (AWS Lambda)  │    │     API         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                                                       ▼
                                               ┌─────────────────┐
                                               │   PostgreSQL    │
                                               │   Database      │
                                               └─────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Qubic RPC     │───▶│  QueueSender    │───▶│   SQS Queue     │
│   Blockchain    │    │   (Lambda)      │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                                                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  External       │◄───│ QueueListener   │◄───│   PostgreSQL    │
│  Webhooks       │    │   (Lambda)      │    │   Database      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Alert Processing Flow

#### **1. Alert Creation Flow**

```
User Request → API Controller → Application Command → Domain Entity → Database
     │
     └─► Response DTO ◄── Mapster ◄── Saved Entity
```

#### **2. Blockchain Monitoring Flow**

```
Scheduled Trigger → QueueSender Lambda → RPC Client → Qubic Blockchain
     │
     ├─► Parse Transactions → Smart Contract Parsers
     │
     ├─► Evaluate Alerts → Alert Matching Engine
     │
     ├─► Send to Queue → SQS Message
     │
     └─► Store RPC Call → Database Audit Log
```

#### **3. Webhook Delivery Flow**

```
SQS Message → QueueListener Lambda → Deserialize Alert Data
     │
     ├─► HTTP Client → External Webhook → Response
     │
     └─► Store Notification → Database (Success/Failure + Metrics)
```

### Database Transaction Patterns

#### **Unit of Work Pattern**

```csharp
public async Task<AlertDto> Execute(Guid userId, CreateAlertDto dto)
{
    // Single transaction boundary
    var alert = new Alert(userId, dto.Name, dto.Description);
    await _context.Alerts.AddAsync(alert);
    await _context.SaveChangesAsync(); // Commit transaction

    return alert.Adapt<AlertDto>();
}
```

#### **Change Tracking & Audit**

```csharp
public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
{
    foreach (EntityEntry<Entity> entry in ChangeTracker.Entries<Entity>())
    {
        switch (entry.State)
        {
            case EntityState.Added:
                entry.Entity.Created = DateTime.UtcNow;
                entry.Entity.Updated = DateTime.UtcNow;
                break;
            case EntityState.Modified:
                entry.Entity.Updated = DateTime.UtcNow;
                break;
            case EntityState.Deleted:
                entry.State = EntityState.Modified; // Soft delete
                entry.Entity.Deleted = true;
                entry.Entity.Updated = DateTime.UtcNow;
                break;
        }
    }

    return await base.SaveChangesAsync(cancellationToken);
}
```

---

## 15. Deployment Architecture

### AWS Infrastructure Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS CLOUD INFRASTRUCTURE                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   API Gateway   │───▶│  Lambda Function│                │
│  │                 │    │  (EasyConnect   │                │
│  │  - REST API     │    │      API)       │                │
│  │  - CORS         │    └─────────────────┘                │
│  │  - Rate Limit   │                                        │
│  └─────────────────┘                                        │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   CloudWatch    │───▶│ Lambda Function │                │
│  │   Events        │    │  (QueueSender)  │                │
│  │  - Cron: 5 min  │    └─────────────────┘                │
│  └─────────────────┘             │                         │
│                                   ▼                         │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │      SQS        │───▶│ Lambda Function │                │
│  │    Queue        │    │ (QueueListener) │                │
│  │  - Batch Size   │    └─────────────────┘                │
│  │  - DLQ          │                                        │
│  └─────────────────┘                                        │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   Cognito       │    │      RDS        │                │
│  │  User Pool      │    │   PostgreSQL    │                │
│  │  - JWT Tokens   │    │  - Multi-AZ     │                │
│  │  - User Mgmt    │    │  - Encrypted    │                │
│  └─────────────────┘    └─────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

### Lambda Function Configuration

#### **API Lambda Settings**

```json
{
  "function-name": "EasyConnectAPI",
  "runtime": "dotnet8",
  "memory-size": 512,
  "timeout": 30,
  "environment-variables": {
    "DDBBConnectionString": "encrypted-connection-string",
    "CognitoAuthority": "https://cognito-idp.eu-west-3.amazonaws.com/...",
    "ASPNETCORE_ENVIRONMENT": "Production"
  }
}
```

#### **QueueSender Lambda Settings**

```json
{
  "function-name": "EasyConnectQueueSender",
  "runtime": "dotnet8",
  "memory-size": 1024,
  "timeout": 900,
  "schedule": "rate(5 minutes)",
  "environment-variables": {
    "DDBBConnectionString": "encrypted-connection-string",
    "SQS_QUEUE_URL": "https://sqs.eu-west-3.amazonaws.com/..."
  }
}
```

#### **QueueListener Lambda Settings**

```json
{
  "function-name": "EasyConnectQueueListener",
  "runtime": "dotnet8",
  "memory-size": 1024,
  "timeout": 30,
  "trigger": "SQS",
  "batch-size": 10,
  "environment-variables": {
    "DDBBConnectionString": "encrypted-connection-string"
  }
}
```

### Database Deployment Strategy

#### **PostgreSQL RDS Configuration**

```yaml
Engine: PostgreSQL 15
Instance: db.t3.micro (dev) / db.t3.medium (prod)
Storage: 20GB SSD (auto-scaling enabled)
Multi-AZ: Yes (production)
Backup: 7-day retention
Encryption: At rest + in transit
Schema: easy_connect
Connection Pooling: Enabled
```

#### **Migration Strategy**

```bash
# Development
dotnet ef migrations add MigrationName
dotnet ef database update

# Production
dotnet ef script-migration --idempotent --output migration.sql
# Execute via CI/CD pipeline
```

### Environment Configuration

#### **Development Environment**

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=easyconnect_dev;Username=dev;Password=dev123"
  },
  "Aws": {
    "Cognito": {
      "Authority": "https://cognito-idp.eu-west-3.amazonaws.com/eu-west-3_DV0Wghhmw",
      "Audience": "dev-client-id",
      "Issuer": "https://cognito-idp.eu-west-3.amazonaws.com/eu-west-3_DV0Wghhmw"
    }
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

#### **Production Environment Variables**

```bash
# Database
DDBBConnectionString="Server=rds-endpoint;Database=easyconnect_prod;Username=prod_user;Password=encrypted_password"

# Authentication
CognitoAuthority="https://cognito-idp.eu-west-3.amazonaws.com/eu-west-3_DV0Wghhmw"
CognitoAudience="prod-client-id"
CognitoIssuer="https://cognito-idp.eu-west-3.amazonaws.com/eu-west-3_DV0Wghhmw"

# Queue
SQS_QUEUE_URL="https://sqs.eu-west-3.amazonaws.com/437794636416/easy-connect-alerts"

# Environment
ASPNETCORE_ENVIRONMENT="Production"
```

### CI/CD Pipeline Architecture

#### **Deployment Pipeline Flow**

```
Developer Push → GitHub → GitHub Actions → Build & Test → Deploy to AWS

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Code Commit   │───▶│  GitHub Actions │───▶│  AWS Deployment │
│                 │    │                 │    │                 │
│ • Feature Branch│    │ • Build         │    │ • Lambda Deploy │
│ • Pull Request  │    │ • Unit Tests    │    │ • RDS Migration │
│ • Main Branch   │    │ • Integration   │    │ • Env Variables │
└─────────────────┘    │ • Security Scan │    │ • Health Check  │
                       └─────────────────┘    └─────────────────┘
```

#### **GitHub Actions Workflow**

```yaml
name: Deploy EasyConnect Backend

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: 8.0.x

      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore

      - name: Test
        run: dotnet test --no-build --verbosity normal

      - name: Run EF Migrations
        run: dotnet ef database update --project EasyConnect.Infrastructure
        env:
          ConnectionStrings__Default: "Server=localhost;Database=easyconnect_test;Username=postgres;Password=postgres"

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-west-3

      - name: Deploy API Lambda
        run: |
          cd EasyConnect.API
          dotnet lambda deploy-serverless

      - name: Deploy QueueSender Lambda
        run: |
          cd EasyConnect.QueueSender
          dotnet lambda deploy-function

      - name: Deploy QueueListener Lambda
        run: |
          cd EasyConnect.QueueListener
          dotnet lambda deploy-function

      - name: Run Database Migrations
        run: |
          dotnet ef script-migration --idempotent --output migration.sql --project EasyConnect.Infrastructure
          # Execute via AWS RDS Query Editor or psql
```

### Monitoring & Observability

#### **CloudWatch Logging Strategy**

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddLogging(builder =>
        {
            builder.AddConsole();
            builder.AddAWSProvider(); // CloudWatch integration
        });
    }
}
```

#### **Application Metrics**

```csharp
public class AlertController : V1ControllerBase
{
    private readonly ILogger<AlertController> _logger;

    [HttpPost]
    public async Task<ActionResult<AlertDto>> CreateAlert([FromBody] CreateAlertDto dto)
    {
        _logger.LogInformation("Creating alert for user {UserId}: {AlertName}",
                              GetUserId(), dto.Name);

        try
        {
            var result = await command.Execute(GetUserId(), dto);
            _logger.LogInformation("Alert created successfully: {AlertId}", result.Id);
            return CreatedAtAction(nameof(CreateAlert), new { id = result.Id }, result);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to create alert for user {UserId}", GetUserId());
            throw;
        }
    }
}
```

#### **CloudWatch Dashboards**

```yaml
Metrics to Monitor:
  - Lambda Invocations (API, QueueSender, QueueListener)
  - Lambda Duration & Memory Usage
  - Lambda Error Rate
  - SQS Queue Depth & Processing Rate
  - RDS Connection Count & Query Performance
  - API Gateway Request Count & Latency
  - Custom Business Metrics (Alerts Created, Webhooks Sent)

Alarms:
  - Lambda Error Rate > 5%
  - SQS Queue Depth > 1000 messages
  - RDS CPU > 80%
  - API Gateway 5xx Errors > 10/minute
```

### Security & Compliance

#### **IAM Roles & Policies**

```yaml
Lambda Execution Roles:
  EasyConnectAPI:
    Policies:
      - AWSLambdaBasicExecutionRole
      - Custom: RDS Access
      - Custom: Cognito User Pool Access

  EasyConnectQueueSender:
    Policies:
      - AWSLambdaBasicExecutionRole
      - Custom: RDS Access
      - Custom: SQS Send Message
      - Custom: HTTP Outbound (RPC calls)

  EasyConnectQueueListener:
    Policies:
      - AWSLambdaBasicExecutionRole
      - Custom: RDS Access
      - Custom: SQS Consume Messages
      - Custom: HTTP Outbound (Webhooks)
```

#### **Data Encryption Strategy**

```yaml
At Rest:
  - RDS: Encrypted with AWS KMS
  - SQS: Server-side encryption enabled
  - Lambda Environment Variables: Encrypted

In Transit:
  - HTTPS/TLS 1.2+ for all API communications
  - PostgreSQL SSL connections required
  - Webhook calls over HTTPS only

Secrets Management:
  - Database credentials: AWS Secrets Manager
  - API keys: Environment variables
  - JWT tokens: Short-lived with refresh mechanism
```

### Performance Optimization

#### **Database Performance**

```sql
-- Index Strategy
CREATE INDEX CONCURRENTLY idx_alerts_user_id ON easy_connect.alerts(user_id) WHERE NOT deleted;
CREATE INDEX CONCURRENTLY idx_alerts_active ON easy_connect.alerts(is_active) WHERE NOT deleted;
CREATE INDEX CONCURRENTLY idx_notifications_alert_date ON easy_connect.notifications(alert_id, date) WHERE NOT deleted;
CREATE INDEX CONCURRENTLY idx_conditions_alert_id ON easy_connect.conditions(alert_id) WHERE NOT deleted;

-- Query Optimization
-- Use Include() for related data loading
-- Implement query result caching where appropriate
-- Use AsNoTracking() for read-only queries
```

#### **Lambda Cold Start Optimization**

```csharp
// Startup.cs - Minimize cold start time
public void ConfigureServices(IServiceCollection services)
{
    // Use Singleton where possible
    services.AddSingleton<IRPCClient, RPCClient>();

    // Configure EF Core for Lambda
    services.AddDbContext<QubikDbContext>(options =>
    {
        options.UseNpgsql(connectionString, npgsql =>
        {
            npgsql.EnableRetryOnFailure(3);
        });
        options.EnableSensitiveDataLogging(false);
        options.EnableServiceProviderCaching();
        options.EnableQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery);
    });
}
```

#### **Memory & Resource Management**

```csharp
// Queue processing optimization
public async Task<SQSBatchResponse> FunctionHandler(SQSEvent evnt, ILambdaContext context)
{
    const int MAX_CONCURRENT_WEBHOOKS = 10;
    var semaphore = new SemaphoreSlim(MAX_CONCURRENT_WEBHOOKS);

    var tasks = evnt.Records.Select(async record =>
    {
        await semaphore.WaitAsync();
        try
        {
            return await ProcessMessage(record);
        }
        finally
        {
            semaphore.Release();
        }
    });

    var results = await Task.WhenAll(tasks);
    return BuildBatchResponse(results);
}
```

### Scalability Considerations

#### **Auto-Scaling Configuration**

```yaml
Lambda Functions:
  Concurrency:
    - API: Reserved 100, Provisioned 10
    - QueueSender: Reserved 5, Provisioned 1
    - QueueListener: Reserved 50, Provisioned 5

  Scaling Metrics:
    - Invocation count
    - Duration
    - Error rate
    - Queue depth (for QueueListener)

SQS Configuration:
  - Visibility Timeout: 6 x Lambda Timeout
  - Message Retention: 14 days
  - Dead Letter Queue: After 3 retries
  - Batch Size: 10 messages per Lambda invocation

RDS Scaling:
  - Read Replicas: For analytics/reporting
  - Connection Pooling: PgBouncer or RDS Proxy
  - Storage Auto-scaling: Enabled
```

#### **Cost Optimization Strategies**

```yaml
Lambda Optimization:
  - Right-size memory allocation based on profiling
  - Use ARM64 Graviton2 processors (better cost/performance)
  - Minimize package size and dependencies
  - Implement intelligent retry logic to reduce failed invocations

Database Optimization:
  - Use appropriate instance types (Burstable for dev, General Purpose for prod)
  - Implement connection pooling
  - Regular maintenance (VACUUM, ANALYZE)
  - Archive old notification data

Monitoring Costs:
  - Set up billing alerts
  - Regular cost analysis and optimization reviews
  - Use AWS Cost Explorer for trend analysis
```

---

## Conclusion

This technical documentation provides a comprehensive blueprint for implementing the Easy Connect backend system. The architecture leverages modern cloud-native patterns including:

### **Key Architectural Benefits**

1. **Scalability**: Serverless functions scale automatically based on demand
2. **Reliability**: Message queues provide fault tolerance and retry mechanisms
3. **Maintainability**: Clean Architecture ensures separation of concerns
4. **Security**: Multi-layered security with AWS Cognito and proper IAM policies
5. **Observability**: Comprehensive logging and monitoring throughout the system
6. **Cost Efficiency**: Pay-per-use model with optimized resource allocation

### **Development Best Practices Implemented**

- **Domain-Driven Design**: Business logic centralized in domain entities
- **CQRS Pattern**: Clear separation between read and write operations
- **Dependency Injection**: Loose coupling and testable components
- **Repository Pattern**: Abstracted data access layer
- **Error Handling**: Comprehensive exception handling and retry logic
- **Configuration Management**: Environment-specific settings
- **Database Migrations**: Version-controlled schema changes
- **API Versioning**: Backward compatibility for API evolution

### **Future Enhancement Opportunities**

1. **Caching Layer**: Redis for frequently accessed data
2. **Event Sourcing**: Full audit trail of domain events
3. **CQRS Read Models**: Optimized projections for complex queries
4. **Microservices**: Split into smaller, domain-specific services
5. **GraphQL**: Alternative API paradigm for flexible data fetching
6. **Real-time Updates**: WebSocket connections for live notifications
7. **Analytics Dashboard**: Business intelligence and usage metrics
8. **Multi-tenancy**: Support for enterprise customers

This architecture provides a solid foundation for building a production-ready blockchain monitoring platform that can scale to support thousands of users and millions of transactions while maintaining high availability and performance.
