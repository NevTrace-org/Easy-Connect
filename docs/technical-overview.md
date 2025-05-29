# Technical Overview Document – Easy Connect

## 1. Project Overview

**Easy Connect** is a web-based system that simplifies interaction with smart contracts on the Qubic blockchain. It allows users to create customized alerts based on specific contract events and receive real-time notifications via webhooks or no-code platforms like Make and Zapier.

## 2. Goals and Objectives

-   Provide a user-friendly interface for configuring alerts based on smart contract data.
-   Support complex conditional logic on smart contract fields.
-   Enable seamless integration with third-party services using webhooks.
-   Abstract the complexities of Qubic to empower non-expert users.

## 3. Functional Requirements

-   Users can:

    -   Create, edit, disable alerts.
    -   Configure alert conditions based on smart contract method arguments.
    -   Define "from" Qubic addresses as sources for monitoring (optional).
    -   Test notifications for a better integration.
    -   Browse a pricing page and subscribe to a plan via a payment provider (Stripe or similar).

-   Backend must:

    -   Continuously monitor specified smart contracts.
    -   Evaluate each transaction against user-defined conditions.
    -   Trigger webhooks when conditions are met.

## 4. Non-Functional Requirements

-   **Performance**: Support multiple users and frequent smart contract activity.
-   **Security**: Authenticated access, secure data transmission.
-   **Reliability**: High uptime and error resilience.
-   **Scalability**: Designed to scale horizontally with user growth.

## 5. Proposed Architecture

-   **Frontend**: Angular 18, Tailwind CSS, Cognito authentication.
-   **Backend**: .NET 8 Web API with RPC interaction layer for Qubic.
-   **Database**: PostgreSQL for alerts, conditions, and audit logs.
-   **Monitoring Service**: Background worker that polls the Qubic network.
-   **Notification System**: Asynchronous webhook dispatcher.

## 6. Technology Stack

-   **Frontend**: Angular 18, Tailwind, RxJS, Cognito, Signals, and standalone Stores
-   **Backend**: C#, .NET 8, ASP.NET Web API, Qubic RPC
-   **Database**: PostgreSQL

## 7. Frontend Design

-   Modular structure using Angular feature modules:

    -   `auth`, `pages`, `shared`, `core`, `models`, `state`

-   State Management:

    -   Uses Angular Signals and standalone Stores to manage shared model state across components.

-   Routing:

    -   `/`
    -   `/create-alert`
    -   `/alert/:id`
    -   `/alert/:id/notifications`
    -   `/pricing` → leads to subscription and external Payment Provider flow

-   Services:

    -   API wrappers for alerts, conditions, and notifications

## 8. Backend Design

-   REST API Controllers:

    -   ### AlertController

        Manages all operations related to alerts. It provides endpoints to create, update, delete, and retrieve alerts by user or ID. It also allows updating specific fields such as the smart contract method or the webhook URL.

    -   ### ConditionController

        Handles the creation, updating, and deletion of alert conditions. Each condition defines a rule that must be satisfied for an alert to be triggered.

    -   ### FromAddressController

        Manages "from" addresses linked to an alert. Allows adding or removing addresses that are considered valid sources for incoming transactions associated with alert evaluation.

    -   ### NotificationController

        Provides an endpoint to retrieve all notifications triggered by a specific alert. These records represent past executions where an alert condition matched and the webhook was invoked.

    -   ### ContractController

        Exposes an endpoint to retrieve all registered contracts in the system. These are used as the basis for configuring alerts.

    -   ### HealthController

        Includes a basic health check endpoint (`GET /`) that returns “Ok” to verify that the system is running. Also includes a protected endpoint (`/create-seed-data`) used to populate the database with sample contract data.

-   Background workers:

    -   ### QueueSender

        The `QueueSender` is a background worker responsible for monitoring smart contract transactions on the Qubic network. It evaluates each transaction against all active alerts configured by users. When a match is found, it serializes the relevant alert data and sends a message to an AWS SQS queue. This service does not persist notifications or execute webhooks—it simply identifies matches and delegates processing to the listener.

    -   ### QueueListener
        The `QueueListener` is implemented as an AWS Lambda function triggered by messages arriving in the SQS queue. For each message, it deserializes the content, performs the webhook notification to the user, and stores the resulting notification in the PostgreSQL database. It acts as the final step in the alert notification pipeline, ensuring both delivery and persistence.

## 9. Backend Technical Architecture Overview

### Backend Architecture Overview

The backend follows a Clean Architecture approach, organized into layered projects that promote separation of concerns and maintainability:

-   **0. Domain (`EasyConnect.Domain`)**  
    Contains the core domain entities, value objects, and business rules. This layer is independent of any external framework or technology and represents the heart of the application logic.

-   **1. Infrastructure (`EasyConnect.Infrastructure`)**  
    Provides implementations for infrastructure concerns such as database access (e.g., Entity Framework), logging, and external service integrations. This layer depends on the domain layer and exposes concrete implementations of interfaces defined in it.

-   **2. Application**

    -   **`EasyConnect.Application`**: Contains application services, use cases, DTOs, and interfaces for external dependencies. It orchestrates domain logic and defines the operations that the system supports.
    -   **`EasyConnect.RPC`**: Encapsulates all interaction logic with the Qubic network via RPC. This layer is responsible for deserializing smart contract transactions and converting them into usable models.

-   **3. Presentation (`EasyConnect.API`)**  
    Exposes the application’s functionality through a RESTful API built with ASP.NET Core. It handles request routing, input validation, authentication, and calls into the application layer.

-   **4. Integration**
    -   **`EasyConnect.QueueSender`**: A background service that monitors the blockchain and enqueues messages into SQS when an alert condition is matched.
    -   **`EasyConnect.QueueListener`**: An AWS Lambda function that consumes messages from SQS, sends webhook notifications, and stores the results in the database.

This architecture enables testability, scalability, and clear boundaries between business rules, application logic, infrastructure, and external integration.

## 10. API Design

RESTful API using standard HTTP verbs. Token-based authentication via AWS Cognito.

-   JSON request/response

## 11. Security Considerations

-   All API endpoints require a valid JWT token issued by AWS Cognito.
-   Frontend interceptors automatically attach tokens to requests and redirect unauthenticated users.
-   Input validation and DTO-level constraints are enforced both in the frontend and backend using data annotations.
-   CORS is configured to restrict API access to allowed frontend domains.
-   Background workers and Lambda functions validate incoming SQS payloads and log any anomalies.
-   Logs and error traces avoid exposing sensitive data and are restricted to internal access.
-   Alerts and conditions are always linked to the authenticated user's identity to prevent cross-account access.
-   Role-based access (if expanded in the future) can be enforced using Cognito groups or custom claims.
