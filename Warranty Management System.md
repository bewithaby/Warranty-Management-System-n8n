# Warranty Management System (n8n Workflow)

This repository contains an n8n workflow that powers a simple car battery warranty management system.  
It allows customers to:

- Register their battery warranty
- Check if a serial number is registered
- View warranty status as Active, Expired, or Not Registered

All UI pages are rendered directly from n8n via HTML responses.

---

## Features

- Single landing page with:
  - Warranty registration form
  - Warranty status check form
- Registration flow with:
  - Duplicate registration check
  - Data storage in n8n DataTable
  - Success and "already registered" pages
- Status check flow with:
  - Serial number lookup in DataTable
  - Warranty Active page
  - Warranty Expired page (with days since expiry)
  - Not Registered page prompting the user to register

---

## Architecture Overview

### Main Components

1. **Warranty Page Load**
   - Type: `Webhook` + `Respond to Webhook`
   - Path: `/webhook/battery-warranty-system`
   - Serves the main HTML page that contains:
     - "Warranty Registration" form
     - "Check Warranty Status" form
   - Uses client side JavaScript to:
     - Open a popup window with a loading screen
     - Call registration or status endpoints
     - Replace the popup content with the HTML returned from n8n

2. **Warranty Registration Button Click**
   - Type: `Webhook`
   - Method: `POST`
   - Path: `/webhook/warranty-registration-click`
   - Receives JSON body:

     ```json
     {
       "serialNumber": "1234567899",
       "customerName": "John Doe",
       "phoneNumber": "9876543210",
       "email": "user@example.com",
       "purchaseDate": "2025-07-17"
     }
     ```

   - Flow:
     1. `Search Existing Registration` (DataTable - get by `SerialNumber`)
     2. `Registration Exists?` (IF node)
        - If serial number found: return **Registration Exists Page**
        - If not found:
          - `Create New Registration` (DataTable - create)
          - Return **Registration Success** HTML page

3. **Warranty Status Check**
   - Type: `Webhook`
   - Method: `POST`
   - Path: `/webhook/warranty-status-checker`
   - Receives JSON body:

     ```json
     {
       "serialNumber": "1234567890"
     }
     ```

   - Flow:
     1. `Find Serial Number` (DataTable - get by `SerialNumber`)
     2. `Serial Number Exists?` (IF node)
        - If no record: return **Serial Number Not Found** page
        - If record exists:
          - Check `ExpiryDate` relative to `$now`
          - If within warranty: return **Warranty Active** page
          - If past expiry: return **Warranty Expired** page

---

## Data Model

All registration and status data is stored in an n8n DataTable named **`Warranty Registration V1`**.

**Columns:**

- `SerialNumber` (string)
- `CustomerName` (string)
- `Email` (string)
- `Phone` (string)
- `PurchaseDate` (dateTime)
- `ExpiryDate` (dateTime)

`ExpiryDate` is set during the "Create New Registration" step, for example using an expression based on `PurchaseDate` and a 24 month warranty period.

---

## Endpoints

Assuming the default n8n local URL `http://localhost:5678`:

- `GET  /webhook/battery-warranty-system`  
  Landing page with registration and status forms

- `POST /webhook/warranty-registration-click`  
  Register a new battery warranty

- `POST /webhook/warranty-status-checker`  
  Check warranty status for a serial number

---

## Frontend Behavior

The HTML and JavaScript for the main page are embedded directly in a `Respond to Webhook` node.

Key behaviors:

- On submit of **Registration Form**:
  - Opens a new window with a "Processing" loading screen
  - Sends a `POST` request to `/webhook/warranty-registration-click`
  - Receives a full HTML response
  - Replaces the popup content with the returned HTML (either success or already registered)

- On submit of **Status Check Form**:
  - Opens a new window with a "Checking Status" loading screen
  - Sends a `POST` request to `/webhook/warranty-status-checker`
  - Receives a full HTML response
  - Replaces the popup content with the returned HTML  
    (active, expired, or not registered page)

The result pages are fully styled, mobile friendly HTML screens rendered from different `Respond to Webhook` nodes.

---

## Importing the Workflow into n8n

1. Open your n8n instance.
2. Click **Workflows** at the top.
3. Click **Import from File**.
4. Select `Warranty Management System.json` from this repository.
5. Save the workflow and toggle it to **Active**.

---

## Local Setup

1. Run n8n locally:

   ```bash
   n8n
