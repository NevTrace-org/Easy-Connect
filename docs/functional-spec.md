# Easy Connect – Functional Specification

## Overview

Easy Connect is a web application that allows users to monitor activity on Qubic smart contracts by creating configurable alerts. Each alert can be linked to a specific contract method, filtered by transaction content, and connected to a webhook that will be triggered when the alert condition is met. The application is built with Angular 18, featuring a custom authentication flow using AWS Cognito, and provides a clean, modular user experience for both configuration and usage.

The frontend consists of the following main sections:

-   **Dashboard**
-   **Create Alert**
-   **Edit Alert**
-   **View Notifications**
-   **Pricing**
-   **Authentication Flow**

Each section is described below in terms of its purpose and functional behavior.

---

## Dashboard (`/`)

The dashboard serves as the main landing page after login. It provides an overview of the user's alert activity and offers quick navigation to key features.

### Purpose

-   To display all alerts created by the logged-in user.
-   To allow easy access to alert management actions and notifications.

### Key Features

-   **Alert Table**: Lists all user alerts with the following columns:
    -   Alert Name
    -   Short description
    -   Date
    -   Status (with color-coded indicator)
    -   Actions
-   **Actions per Alert**:
    -   Edit the alert configuration (redirect to edit alert page)
    -   Toggle the alert as active/inactive (alerts are not deleted)
    -   Navigate to the notification history for that alert
-   **Status Indicators**: Color tags visually represent the current state of each alert (e.g. green for active, red for inactive).

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/2%20Dashboard.png" />

---

## Create Alert (`/create-alert`)

This screen allows users to initiate the creation of a new alert by specifying basic information that identifies the alert.

### Purpose

To let the user define a new alert and provide its name and description before continuing to configure the contract origin, conditions, sources, and webhook.

### Key Features

-   **Form Fields**:

    -   **Name**: A required input field for naming the alert.
    -   **Description**: An optional field where the user can describe the purpose or scope of the alert.

-   **Primary Action**:

    -   **Create Button**: Validates the inputs and initializes the alert in the backend. After successful creation, the user is redirected to the full alert editor screen (`/alert/:id`) to complete the configuration.

-   **UX Notes**:
    -   The form uses a clean, minimal layout for focus.
    -   No configuration of logic or conditions is required at this stage—this step is meant to name and register the alert only.

---

## Edit Alert (`/alert/:id`)

This screen allows users to fully configure an alert after creation. It includes detailed controls to define the alert's logic, monitoring source, and delivery method.

### Purpose

To enable users to modify an existing alert’s parameters, including contract filters, conditions, data sources, and notification configuration.

### Key Features

-   **Editable Fields**:

    -   **Name**: Allows renaming the alert.
    -   **Description**: Editable description for internal use.

-   **Visual Status Tag**:

    -   Clearly indicates whether the alert is currently active or inactive (with colored badge at the top).
    -   Alerts can be activated or deactivated but not deleted.

-   **Origin Section**:

    -   **Contract**: Select the smart contract this alert listens to.
    -   **Method**: Choose the method of interest within the contract.
    -   **Warning Message**: A warning is displayed when the user changes the contract or method, informing that this might cause inconsistencies in incoming data. However, the user is allowed to proceed.

-   **Conditions Section**:

    -   Add one or more filter conditions to restrict the alert.
    -   Each condition includes:
        -   **Field/Variable Name** (e.g., AssetName)
        -   **Operator**: equals, contains, greater than, etc.
        -   **Value** to compare

-   **Sources Section**:

    -   Add specific identity addresses that act as the origin of monitored transactions.
    -   Each source filters alerts based on the transaction's sender address.

-   **Notification Section**:

    -   **Webhook URL**: Required field to define where the alert payload will be sent.
    -   **Send Test Notification**: Allows users to validate delivery and structure of payload before or after enabling the alert.

    -   **Notification Test Flow**:
        When the user clicks the "Send test notification" button:

        -   A confirmation modal appears displaying a sample payload that would be sent to the configured webhook. This modal allows the user to verify the structure and content of the payload before proceeding.

        -   Once the user confirms (clicking "Yes, send it!"), the application performs the test request. The outcome is shown in a response modal, indicating whether the webhook responded successfully or an error occurred. The modal includes:

            -   HTTP status and message

            -   Headers and body of the webhook response (if available)

            -   Clear visual feedback: success or failure indicator

        -   This functionality helps users debug their webhook integration and ensures their endpoint can handle the actual structure of alert notifications.

### Auto-Save Behavior

The **Edit Alert** screen implements **auto-save** for all editable sections, ensuring that any changes made by the user are automatically persisted without requiring manual submission.

Each section — **Origin**, **Conditions**, **Sources**, and **Notification** — is individually monitored for changes. Upon detecting an update (e.g., modifying a field or adding a condition), the system triggers an auto-save request.

At the top of each section box, a **save status indicator** is shown, reflecting one of the following states:

-   **Saved** – Changes were successfully persisted.
-   **Saving...** – A save operation is currently in progress.
-   **Error** – An error occurred while saving (e.g., network failure or invalid input).

This user experience ensures real-time persistence and minimizes the risk of data loss during alert configuration.

### Business Rules

-   Alerts must always be tied to:

    -   A valid smart contract
    -   A specific method within the contract
    -   At least one condition

-   Alerts:

    -   **Cannot be deleted**, only deactivated — to preserve historical notification data.
    -   Are **editable only by their owner**.

-   Webhook notifications include:
    -   The full triggering transaction payload

### Enable/Disable Alerts

A **toggle button** is available at the top of the screen to **activate or deactivate** the alert. When the user attempts to activate an alert, the system performs a validation check to ensure all **minimum requirements** are met:

-   At least one condition is defined
-   A valid webhook URL is configured
-   A smart contract and method are selected

If any of these requirements are missing, a **modal dialog** will be displayed, informing the user of the incomplete configuration and preventing activation until resolved.

**Note:** This validation and modal warning behavior is also applied in the **Dashboard** screen when enabling or disabling an alert directly from the list.

---

## View Notifications (`/alert/:id/notifications`)

This screen allows users to review the webhook notifications triggered by their alerts. It provides a clear breakdown of each notification attempt, including the payload sent and the response received.

TODO HACER YO LAS CAPTURAS DE LA WEB DE PRE PARA INCLUIRLAS AQUÍ

### Key Features

-   **Tabular Display**

    -   **URL**: The destination webhook endpoint.
    -   **Date**: Timestamp of when the webhook was triggered.
    -   **Success**: Status of the delivery (e.g., success or error).
    -   **Milliseconds**: Time taken for the webhook request to complete.

-   **Expandable Rows**
    -   Each row can be expanded to show:
        -   **Data Sent**: Full JSON payload that was delivered to the webhook. This includes:
            -   Raw transaction data
            -   Parsed transaction fields (like `AssetName`, `Price`, `IssuerAddress`, etc.)
        -   **Webhook Response**: Raw HTTP response returned by the webhook endpoint.
-   **Traceability**
    -   Displays both the data sent and the exact response from the receiving server.
    -   Useful for debugging failed alerts or confirming that the receiver processed the data correctly.

### Additional Notes

-   This screen makes it explicit what was **sent** by the system (outbound payload) and what was **received** in response from the webhook (inbound reply).
-   Helps users identify configuration issues or failed deliveries at a glance.

---

## Pricing (`/pricing`)

The Pricing screen displays the available subscription plans and their respective limits. It provides users with a clear comparison between the free and paid tiers, encouraging upgrade when needed.

### Key Features

-   Two-column layout comparing the **Free** and **Premium** plans:
    -   **Free Plan (Freemium):**
        -   Up to **200 notifications** per month.
        -   Ideal for testing and personal use.
        -   No credit card required.
    -   **Advance Plan:**
        -   Up to **1,000 notifications** per month.
        -   Designed for power users and production usage.
        -   Access to priority support and extended features (optional).
-   Visual indicators to highlight the benefits of upgrading.
-   “Subscribe” button in the Premium column that redirects the user to an external payment provider (e.g., Stripe) to complete the subscription process.
-   Responsive layout for both desktop and mobile views.

TODO INCLUIR QUE PUEDEN CAMBIAR LOS PRECIOS Y LOS NÚMEROS DE ALERTAS, TAMBIÉN EL NÚMERO DE ALERTAS

### Behavior

-   Users can view the current plan they are subscribed to.
-   Plan selection and upgrades are handled via secure redirection to the external payment gateway.
-   Once the payment is confirmed, the user's subscription level is updated in the system automatically.

---

## Authentication (`/auth/...`)

The application uses a customized authentication flow based on AWS Cognito. While the underlying logic is handled by Cognito, the UI has been tailored to match the style and UX of the rest of the application.

### Routing

-   `/auth/login`
-   `/auth/signup`
-   `/auth/send-password-code`
-   `/auth/password-reset`

### Login Screen (`/auth/login`)

-   Allows existing users to log in using their email and password.
-   Authentication tokens (JWT) are received from Cognito upon success and stored securely.
-   Invalid attempts are clearly indicated with error messages.

### Signup Screen (`/auth/signup`)

-   New users can create an account by entering:
    -   Email
    -   Password
    -   Confirmation password
-   Upon form submission, a verification code is sent to the user’s email.
-   The user must confirm the code to complete registration.

### Send Password Code Screen (`/auth/send-password-code`)

-   For users who have forgotten their password.
-   Requests the user's email and triggers Cognito to send a reset code.

### Password Reset Screen (`/auth/password-reset`)

-   After receiving the reset code via email, the user can enter:
    -   Email
    -   Code
    -   New password
    -   Confirmation of new password
-   Successfully completes the password reset via Cognito.

### Notes

-   All forms include validation (required fields, password strength, etc.).
-   Security is enforced on the frontend using route guards to prevent access to protected pages without a valid session.
-   Cognito handles email delivery, password policies, and token generation.
