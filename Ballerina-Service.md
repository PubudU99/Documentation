# Ballerina Backend Service for CST Process

## Overview
The Ballerina API service, hosted on Choreo, is designed to trigger CI/CD pipelines in Azure. The service provides endpoints to manage build processes, retrieve customer data, and handle webhook notifications.

---

## Endpoints

### **Build Management**

#### **POST - `/builds`**
**Description:** Initiates a build process and retrives information about the build.

**Request Body:**
```json
[
  {
    "productBaseVersion": "string",
    "productName": "string"
  }
]
```

**Response Body:**
```json
{
  "uuid": "<UUID string if cstAvailable = true>",
  "reason": "Reason if the cstAvailable = false",
  "cstAvailable": true/false
}
```

---

#### **GET - `/builds/{cicdId}`**
**Description:** Retrieves the build details for the given UUID.

**Response Body:**
```json
{
  "cdBuild": [
    {
      "consoleErrorUrl": "string",
      "customer": "string",
      "status": "string"
    }
  ],
  "ciBuild": [
    {
      "consoleErrorUrl": "string",
      "product": "string",
      "status": "string",
      "version": "string"
    }
  ],
  "id": "string"
}
```

---

#### **POST - `/builds/{cicdId}/re-trigger`**
**Description:** Re-triggers the CI/CD process for the given UUID.

---

### **Customer Management**

#### **GET - `/customer`**
**Description:** Retrieves the list of customers which is eligible for CSTs.

**Response Body:**
```json
[
  {
    "id": 1,
    "customer_key": "cagipsub",
    "environment": "Development",
    "product_name": "wso2am",
    "product_base_version": "3.2.0",
    "u2_level": "26"
  },
  {
    "id": 2,
    "customer_key": "AAA",
    "environment": "Staging",
    "product_name": "wso2am",
    "product_base_version": "4.2.0",
    "u2_level": "24"
  }
]
```

---

#### **POST - `/customer`**
**Description:** Adds new entry when a customer promoted to high values customer.

**Request Body:**
```json
[
  {
    "customerKey": "string",
    "environment": "string",
    "productBaseVersion": "string",
    "productName": "string",
    "u2Level": "string"
  }
]
```

---

### **Webhook Endpoints**

#### **POST - `/builds/ci/status`**
**Description:** Handles status updates and update the database in the Azure for CI builds.

#### **POST - `/builds/cd/status`**
**Description:** Handles status updates and update the database in the Azure for CD builds.

#### **POST - `/builds/cd/trigger`**
**Description:** Triggers CD pipeline when CI pipeline is over.


---

## Authentication
The service has a token endpoint and a public endpoint to securely access resources. An application has been created and subscribed to a app and providing client credentials for authentication.

---

## Key Features
- **Pipeline Triggering:** Facilitates CI/CD pipeline operations via well-defined endpoints.
- **Customer Data Management:** Enables retrieval and addition of customer-related information.
- **Webhook Support:** Updates Database for CI/CD build statuses and triggers.

---

## Notes
- Ensure proper authentication by utilizing the subscribed application’s client credentials.
- Verify endpoint functionality with test data before integrating into production workflows.

---
