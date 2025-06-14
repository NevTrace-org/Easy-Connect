![Easy Connect logo](../easyconnect-logotype.jpg)

# Easy Connect – Functional Specification

## Overview

Easy Connect is a web application that allows users to monitor activity on Qubic smart contracts by creating configurable alerts. Each alert can be linked to a specific contract method, filtered by transaction content, and connected to a webhook that will be triggered when the alert condition is met. The application is built with Angular 18, featuring a custom authentication flow using AWS Cognito, and provides a clean, modular user experience for both configuration and usage.

The frontend consists of the following main sections:

- **Dashboard**
- **Create Alert**
- **Edit Alert**
- **View Notifications**
- **Pricing**
- **Authentication Flow**

Each section is described below in terms of its purpose and functional behavior.

---

## Dashboard (`/`)

The dashboard serves as the main landing page after login. It provides an overview of the user's alert activity and offers quick navigation to key features.

### Purpose

- To display all alerts created by the logged-in user.
- To allow easy access to alert management actions and notifications.

### Key Features

- **Alert Table**: Lists all user alerts with the following columns:
  - Alert Name
  - Short description
  - Date
  - Status (with color-coded indicator)
  - Notificatons sent
  - Actions
- **Actions per Alert**:
  - Edit the alert configuration (redirect to edit alert page)
  - Toggle the alert as active/inactive (alerts are not deleted)
  - Navigate to the notification history for that alert
- **Status Indicators**: Color tags visually represent the current state of each alert (e.g. green for active, grey for inactive).

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/2%20Dashboard.png" />

---

## Create Alert (`/create-alert`)

This screen allows users to initiate the creation of a new alert by specifying basic information that identifies the alert.

### Purpose

To let the user define a new alert and provide its name and description before continuing to configure the contract origin, conditions, sources, and webhook.

### Key Features

- **Form Fields**:

  - **Name**: A required input field for naming the alert.
  - **Description**: An optional field where the user can describe the purpose or scope of the alert.

- **Primary Action**:

  - **Create Button**: Validates the inputs and initializes the alert in the backend. After successful creation, the user is redirected to the full alert editor screen (`/alert/:id`) to complete the configuration.

- **UX Notes**:
  - The form uses a clean, minimal layout for focus.
  - No configuration of logic or conditions is required at this stage—this step is meant to name and register the alert only.

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/3%20New%20alert.png" />

---

## Edit Alert (`/alert/:id`)

This screen allows users to fully configure an alert after creation. It includes detailed controls to define the alert's logic, monitoring source, and delivery method.

### Purpose

To enable users to modify an existing alert’s parameters, including contract filters, conditions, data sources, and notification configuration.

### Key Features

- **Editable Fields**:

  - **Name**: Allows renaming the alert.
  - **Description**: Editable description for internal use.

- **Visual Status Tag**:

  - Clearly indicates whether the alert is currently active or inactive (with colored badge at the top).
  - Alerts can be activated or deactivated but not deleted.

- **Origin Section**:

  - **Contract**: Select the smart contract this alert listens to.
  - **Method**: Choose the method of interest within the contract.
  - **Warning Message**: A warning is displayed when the user changes the contract or method, informing that this might cause inconsistencies in incoming data. However, the user is allowed to proceed.

- **Conditions Section**:

  - Add one or more filter conditions to restrict the alert.
  - Each condition includes:
    - **Field/Variable Name** (e.g., AssetName)
    - **Operator**: equals, contains, greater than, etc.
    - **Value** to compare

- **Sources Section**:

  - Add one or more **specific wallet addresses** to monitor as the source of transactions.
  - The alert will only trigger if the transaction originates from one of the listed addresses.
  - Useful for tracking activity from **individual or multiple known identities** (e.g., user wallets, bots, or trusted entities).

- **Notification Section**:

  - **Webhook URL**: Required field to define where the alert payload will be sent.
  - **Send Test Notification**: Allows users to validate delivery and structure of payload before or after enabling the alert.

  - **Notification Test Flow**:
    When the user clicks the "Send test notification" button:

    - A confirmation modal appears displaying a sample payload that would be sent to the configured webhook. This modal allows the user to verify the structure and content of the payload before proceeding.

    - Once the user confirms (clicking "Yes, send it!"), the application performs the test request. The outcome is shown in a response modal, indicating whether the webhook responded successfully or an error occurred. The modal includes:

      - HTTP status and message

      - Headers and body of the webhook response (if available)

      - Clear visual feedback: success or failure indicator

    - This functionality helps users debug their webhook integration and ensures their endpoint can handle the actual structure of alert notifications.

### Auto-Save Behavior

The **Edit Alert** screen implements **auto-save** for all editable sections, ensuring that any changes made by the user are automatically persisted without requiring manual submission.

Each section — **Origin**, **Conditions**, **Sources**, and **Notification** — is individually monitored for changes. Upon detecting an update (e.g., modifying a field or adding a condition), the system triggers an auto-save request.

At the top of each section box, a **save status indicator** is shown, reflecting one of the following states:

- **Saved** – Changes were successfully persisted.
- **Saving...** – A save operation is currently in progress.
- **Error** – An error occurred while saving (e.g., network failure or invalid input).

This user experience ensures real-time persistence and minimizes the risk of data loss during alert configuration.

### Business Rules

- Alerts must always be tied to:

  - A valid smart contract
  - A specific method within the contract
  - At least one condition

- Alerts:

  - **Cannot be deleted**, only deactivated — to preserve historical notification data.
  - Are **editable only by their owner**.

- Webhook notifications include:
  - The full triggering transaction payload

### Enable/Disable Alerts

A **toggle button** is available at the top of the screen to **activate or deactivate** the alert. When the user attempts to activate an alert, the system performs a validation check to ensure all **minimum requirements** are met:

- At least one condition is defined
- A valid webhook URL is configured
- A smart contract and method are selected

If any of these requirements are missing, a **modal dialog** will be displayed, informing the user of the incomplete configuration and preventing activation until resolved.

**Note:** This validation and modal warning behavior is also applied in the **Dashboard** screen when enabling or disabling an alert directly from the list.

### Future Improvements: Timeframe Filtering (Proposed)

> We are currently evaluating the possibility of allowing **timeframe-based filtering** for alerts in future versions.  
> This would enable users to define temporal windows during which alerts are active or relevant.
>
> While this feature is not yet implemented, users can still **filter notifications externally** by leveraging the `timestamp` included in the payload — for example, using Make and Google Sheets workflows.

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/4.1%20Edit-Alert.jpg" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/4.2%20Edit-Alert-%20origin.jpg" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/4.3%20Edit-Alert-%20condition.jpg" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/4.4%20Edit-Alert-source.jpg" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/4.5%20Edit-Alert-webhook.jpg" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/4.6%20Modal-Send.png" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/4.7%20Modal-Date-Sent.jpg" />

---

## View Notifications (`/alert/:id/notifications`)

This screen allows users to review the webhook notifications triggered by their alerts. It provides a clear breakdown of each notification attempt, including the payload sent and the response received.

### Key Features

- **Tabular Display**

  - **URL**: The destination webhook endpoint.
  - **Date**: Timestamp of when the webhook was triggered.
  - **Success**: Status of the delivery (e.g., success or error).
  - **Milliseconds**: Time taken for the webhook request to complete.

- **Expandable Rows**
  - Each row can be expanded to show:
    - **Data Sent**: Full JSON payload that was delivered to the webhook. This includes:
      - Raw transaction data
      - Parsed transaction fields (like `AssetName`, `Price`, `IssuerAddress`, etc.)
    - **Webhook Response**: Raw HTTP response returned by the webhook endpoint.
- **Traceability**
  - Displays both the data sent and the exact response from the receiving server.
  - Useful for debugging failed alerts or confirming that the receiver processed the data correctly.

### Additional Notes

- This screen makes it explicit what was **sent** by the system (outbound payload) and what was **received** in response from the webhook (inbound reply).
- Helps users identify configuration issues or failed deliveries at a glance.

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/5.1%20Results.png" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/5.2%20Results-displayed.png" />

---

## Pricing (`/pricing`)

The Pricing screen displays the available subscription plans and their respective limits. It provides users with a clear comparison between the free and paid tiers, encouraging upgrade when needed.

### Key Features

- Two-column layout comparing the **Free** and **Premium** plans:
  - **Free Plan (Freemium):**
    - Up to **200 notifications** per month.
    - Ideal for testing and personal use.
    - No credit card required.
  - **Advance Plan:**
    - Up to **1,000 notifications** per month.
    - Designed for power users and production usage.
    - Access to priority support and extended features (optional).
- Visual indicators to highlight the benefits of upgrading.
- “Subscribe” button in the Premium column that redirects the user to an external payment provider (e.g., Stripe) to complete the subscription process.
- Responsive layout for both desktop and mobile views.

### Behavior

- Users can view the current plan they are subscribed to.
- Plan selection and upgrades are handled via secure redirection to the external payment gateway.
- Once the payment is confirmed, the user's subscription level is updated in the system automatically.

### Disclaimer:

- The pricing plans and their respective limits described above are provisional and subject to change.
- We are continuously refining the service, and adjustments may be made to better reflect usage patterns and operational costs.

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/6.1%20Pricing.png" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/6.2%20Pricing.png" />

<img width="1946" alt="2 Dashboard" src="https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/6.3%20Pricing.png" />

---

## Authentication (`/auth/...`)

The application uses a customized authentication flow based on AWS Cognito. While the underlying logic is handled by Cognito, the UI has been tailored to match the style and UX of the rest of the application.

### Routing

- `/auth/login`
- `/auth/signup`
- `/auth/send-password-code`
- `/auth/password-reset`

### Login Screen (`/auth/login`)

- Allows existing users to log in using their email and password.
- Authentication tokens (JWT) are received from Cognito upon success and stored securely.
- Invalid attempts are clearly indicated with error messages.

<img width="1946" alt="2 https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/1.1%20Login.jpg" />

### Signup Screen (`/auth/signup`)

- New users can create an account by entering:
  - Email
  - Password
  - Confirmation password
- Upon form submission, a verification code is sent to the user’s email.
- The user must confirm the code to complete registration.

### Send Password Code Screen (`/auth/send-password-code`)

- For users who have forgotten their password.
- Requests the user's email and triggers Cognito to send a reset code.

### Password Reset Screen (`/auth/password-reset`)

- After receiving the reset code via email, the user can enter:
  - Email
  - Code
  - New password
  - Confirmation of new password
- Successfully completes the password reset via Cognito.

<img width="1946" alt="2 https://github.com/NevTrace-org/Easy-Connect/blob/main/ui/1.2%20Reset%20Password.jpg" />

### Notes

- All forms include validation (required fields, password strength, etc.).
- Security is enforced on the frontend using route guards to prevent access to protected pages without a valid session.
- Cognito handles email delivery, password policies, and token generation.

## Make Integration Examples

### Purpose

This section provides examples of how Easy Connect webhook data can be integrated with Make.com workflows. The data structure varies depending on the smart contract and method being monitored.

### Example 1: QX Contract - Bid Order Tracking

#### Scenario

Monitor all bid orders on the QX decentralized exchange and log them to Google Sheets.

#### Easy Connect Alert Configuration

- **Contract**: QX
- **Method**: AddToBidOrder
- **Condition**: Any bid (no specific filters)
- **Webhook**: Make webhook URL

#### Sample Webhook Payload

The webhook sends data in the following structure:

- **ProcedureName**: "AddToBidOrder"
- **InputType**: 6
- **RawData**: Contains transaction details including sourceId, destId, amount, tickNumber, and txId
- **ParsedData**: Contains processed information including IssuerAddress, AssetName, Price, and NumberOfShares

#### Detailed Make Setup Process

1. **Create New Scenario**

- Log into Make.com and click "Create a new scenario"
- Search for "Webhooks" in the apps directory
- Select "Custom Webhook" as your trigger module

2. **Configure Webhook Trigger**

- Click "Add" to create a new webhook
- Copy the generated webhook URL (example: `https://hook.integromat.com/abc123def456`)
- Paste this URL into Easy Connect's webhook field
- Click "Send Test Notification" in Easy Connect to establish data structure

3. **Add Google Sheets Module**

- Click the "+" button after the webhook trigger
- Search for "Google Sheets" and select it
- Choose "Add a Row" action
- Authorize your Google account if not already connected

4. **Configure Spreadsheet Connection**

- Select your target spreadsheet from the dropdown
- Choose the specific worksheet tab
- If the spreadsheet doesn't exist, create one with these column headers:
  - Date | Asset | Price | Quantity | Bidder Address | Transaction ID

5. **Map Data Fields**

- **Date Column**: Use formatDate(timestamp; "YYYY-MM-DD HH:mm:ss") to convert Unix timestamp
- **Asset Column**: Map to ParsedData.AssetName
- **Price Column**: Map to ParsedData.Price
- **Quantity Column**: Map to ParsedData.NumberOfShares
- **Bidder Address Column**: Map to ParsedData.IssuerAddress
- **Transaction ID Column**: Map to RawData.transaction.txId

6. **Test the Integration**

- Click "Run once" in Make to start listening
- Send another test notification from Easy Connect
- Verify that a new row appears in your Google Sheets with the correct data

7. **Activate Scenario**

- Click the "ON" toggle in Make to activate continuous monitoring
- Enable your alert in Easy Connect for live data processing

#### Result

Every QX bid order automatically creates a new row in the tracking spreadsheet with properly formatted data.

### Example 2: QX Contract - Ask Order Notifications

#### Scenario

Send Slack notifications when someone places a sell order (ask) for assets above a certain price threshold.

#### Easy Connect Alert Configuration

- **Contract**: QX
- **Method**: AddToAskOrder
- **Condition**: Price greater than 100000
- **Webhook**: Make webhook URL

#### Detailed Make Setup Process

1. **Setup Webhook Trigger** (same as Example 1)

2. **Add Filter Module**

- Click "+" after webhook trigger
- Add a "Filter" module (Tools section)
- Set condition: ParsedData.Price Greater than 100000
- This ensures only high-value orders trigger Slack notifications

3. **Add Slack Module**

- Click "+" after the filter
- Search for "Slack" and select it
- Choose "Create a Message" action
- Authorize your Slack workspace

4. **Configure Slack Message**

- **Channel**: Select target channel (e.g., #trading-alerts)
- **Message Text**: Create a formatted message template including asset name, price per share, total shares, seller address, transaction ID, and timestamp
- **Username**: "Easy Connect Bot" (optional)
- **Icon**: Add emoji or custom icon (optional)

5. **Test and Activate**

- Test with sample data to ensure Slack message formatting looks correct
- Activate scenario for live monitoring

#### Result

Team receives instant, well-formatted Slack notifications only for significant sell orders above the threshold.

### Example 3: Multi-Contract Portfolio Monitoring

#### Scenario

Track a specific wallet's activity across multiple contracts and maintain a comprehensive activity log.

#### Easy Connect Alert Configuration

Multiple alerts with the same webhook:

- **Alert 1**: QX contract, any method, source filter for specific wallet
- **Alert 2**: Quottery contract, any method, source filter for same wallet
- **Webhook**: Same Make webhook URL for all alerts

#### Detailed Make Setup Process

1. **Setup Webhook Trigger** (receives data from multiple alerts)

2. **Add Router Module**

- Click "+" after webhook trigger
- Add "Router" module (Flow Control section)
- This will split the flow based on which contract sent the data

3. **Configure Router Paths**

   **Path 1 - QX Transactions:**

- **Filter Condition**: ProcedureName Contains "AddToBidOrder" OR "AddToAskOrder" OR "IssueAsset"
- **Action**: Google Sheets "Add Row" to "QX Trading Log"
- **Mapping**: Date, Action, Asset, Price, Shares, Address

**Path 2 - Quottery Transactions:**

- **Filter Condition**: ProcedureName Contains "IssueBet" OR "JoinBet" OR "FinalizeBet"
- **Action**: Google Sheets "Add Row" to "Betting Activity Log"
- **Mapping**: Date, Action, Bet Details, Amount, Address

4. **Add Master Log Path**

- After the router, add another Google Sheets module
- Connect to "Master Activity Log" spreadsheet
- Map general fields that exist in all transactions: Date, Contract, Action, Wallet, Transaction ID

#### Result

Comprehensive monitoring of a wallet's activity across the entire Qubic ecosystem with specialized logs for each contract type.

### Testing Your Integration

#### Step-by-Step Testing Process

1. **Create Test Alert**: Set up alert with very broad conditions to capture test data

- Use "Any" conditions initially to see what data structure you receive
- This helps you understand available fields before mapping

2. **Send Test Notification**: Use Easy Connect's test feature to send sample payload

- Always test before going live to avoid errors in production
- Review the JSON structure in Make's data inspector

3. **Verify Make Reception**: Check that Make receives and parses the data correctly

- Look for the green checkmark indicating successful data reception
- Review the parsed data structure in Make's execution history

4. **Map Data Fields**: Configure your Make modules using the actual field names

- Use Make's variable picker to avoid typos in field names
- Pay attention to nested objects (e.g., ParsedData.AssetName)

5. **Test Actions**: Verify data flows correctly to your destination (sheets, email, etc.)

- Check that Google Sheets rows are created with correct data
- Verify Slack messages format properly
- Test email notifications if configured

6. **Refine Conditions**: Adjust alert conditions to match your specific monitoring needs

- Start broad, then narrow down based on actual data patterns
- Use filters in Make for additional data processing

7. **Activate Live Monitoring**: Enable the alert for real-time data processing

- Turn on the Make scenario first
- Then activate the Easy Connect alert
- Monitor for the first few real transactions to ensure everything works

#### Common Data Variations

- **Field Names**: Vary by contract method (e.g., AssetName vs BetDescription)
- **Data Types**: Some fields are numbers, others are strings or addresses
- **Optional Fields**: Not all methods include the same data fields
- **Array Fields**: Some methods return arrays (e.g., OptionDescriptions in Quottery)

#### Troubleshooting Tips

- **Missing Data**: Check that webhook URL is correct and alert is active
- **Wrong Format**: Use Make's date and number formatting functions
- **Duplicate Entries**: Ensure alerts aren't triggering multiple times for the same transaction
- **Failed Connections**: Verify all third-party authorizations (Google, Slack, etc.) are still valid
