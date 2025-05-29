![Easy Connect logo](../easyconnect-logo.jpg)

# Queue System and RPC Integration – Deep Dive

## Overview

This document provides a high-level explanation of Easy Connect’s alert processing architecture, which leverages an asynchronous queue system and integrates with the Qubic blockchain through a dedicated RPC layer.

---

## 1. QueueSender (Lambda)

### Purpose

-   Monitors new smart contract transactions from the Qubic blockchain.
-   Evaluates each transaction against user-defined alerts.
-   If a transaction matches, it triggers a webhook and sends a message to a queue for further processing.

### Flow

1. Fetches new transaction data from Qubic.
2. Converts raw data into structured internal models.
3. Evaluates alerts against each transaction.
4. If matched:
    - Sends a message to the queue with alert and transaction details.

---

## 2. Alert Evaluation Logic

### Purpose

Applies user-defined filtering rules to blockchain transaction data.

### Behavior

-   Compares transaction method and input fields to configured alert conditions.
-   Each condition includes a field, operator, and expected value.
-   Supports various data types like strings, numbers, booleans, and addresses.

---

## 3. Queue Management

### Purpose

Ensures that alert match data is serialized and pushed to a message queue for asynchronous processing.

### Behavior

-   Serializes the relevant information from alert evaluations.
-   Sends the data to an external queueing system (e.g., AWS SQS).
-   Preserves compatibility across backend components for further processing.

---

## 4. QueueListener (Lambda)

### Purpose

Consumes alert match messages from the queue and completes the alert workflow.

### Flow

1. Deserializes the incoming message.
2. Sends the notification payload to the user’s configured webhook.
3. Records the result of the notification attempt in the database.
4. Handles success/failure and ensures error visibility.

---

## 5. Webhook Notification Delivery

### Role

Responsible for sending alert data to third-party services via HTTP.

### Behavior

-   Constructs the payload from the triggering transaction.
-   Sends an HTTP request to the webhook endpoint.
-   Captures delivery metrics including status, time, and response content.
-   Handles errors gracefully and logs issues for diagnostics.

---

## 6. RPC Integration

### Overview

Interacts with the Qubic blockchain to retrieve and interpret smart contract activity.

### Behavior

-   Requests the latest transaction blocks.
-   Extracts and decodes input data from each transaction.
-   Maps decoded data to internal models for condition evaluation.
-   Records raw and parsed blockchain data for auditability.

---

## Summary

This queue-based architecture enables Easy Connect to operate efficiently and reliably. It decouples transaction monitoring from notification delivery, supports real-time responsiveness, and ensures clear traceability for alert execution through persistent logging and structured payload handling.
