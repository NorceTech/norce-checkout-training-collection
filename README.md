# Norce Checkout Training Collection

This repository contains a structured **Bruno API Collection**. Bruno is a fast and GIT-friendly client for testing and documenting API flows.

The collection provides a runnable environment for developers to learn and test the integration points between **Norce Commerce**, **Norce Checkout**, and various **Payment Service Providers (PSPs)**, such as Adyen and Walley.

The requests cover the entire order lifecycle, from basket creation in Commerce to final payment configuration in Checkout.

---
➡️ **Learn more:** For official documentation and API references, please visit [docs.norce.io](https://docs.norce.io).
---

## 🚀 Getting Started

### 1. Prerequisites

To run this collection, you must have:

1.  **Bruno API Client:** Installed on your machine.
2.  **Required Credentials:** You need access details for the following systems:
    * **Norce Checkout** (API Keys and token access)
    * **Norce Commerce** (Client ID and Secret for token generation)
    * **Walley** (Client and API Secrets — *optional*)
    * **Adyen** (API Keys and HMAC Secrets — *optional*)

### 2. Setup

1.  **Load the Collection:**
    * Open the Bruno application.
    * Click `Open Bruno Project` and select the `Norce Checkout Training` folder.

2.  **Configure Environment Secrets:**
    The collection uses the `Playground.bru` environment file to define variables, including those marked as secrets. You **must** fill in the actual secret values locally before running any requests.

    * In Bruno, open the **Playground** environment (e.g., by clicking the Edit icon next to the environment name).
    * Fill in the actual secret values for all variables (marked with the orange lock icon in the UI):
        * `token`
        * `commerceIdentityClientId`
        * `commerceIdentityClientSecret`
        * `adyenApiKey`, `adyenClientKey`, `adyenMerchantAccount`, `adyenWebhookHmacKey`
        * `walleyScope`, `walleyApiSecret`, `walleyClientId`

    * **Save** and **Activate** the environment.

## 🗺️ Collection Structure and Flow

The collection is logically separated into three main folders representing the central transaction stages.

### 1. Arrange Basket [Commerce]

This folder focuses on creating and preparing the shopping basket in **Norce Commerce** before it is imported into Checkout.

| Request | Purpose |
| :--- | :--- |
| `[Identity] Fetch Access Token` | **CRITICAL:** Acquires the necessary `commerce_token` (Client Credentials Grant) to authenticate all subsequent Commerce requests. |
| `[Shopping] Create basket` | Initializes a new shopping basket and extracts the `basketId`. |
| `[Shopping] Insert basket item` | Adds a product to the basket. |
| `[Shopping] Update Buyer` / `[Shopping] Update Payment Method` / `[Shopping] Update Delivery Method` | Sets customer and logistic details required for successful checkout. |

### 2. Configuration [Checkout]

This folder contains requests used for setting up the **Norce Checkout Channel Configurations**.

* The folder is subdivided by channel: `nc-adyen-se`, `nc-nonpsp-se`, and `nc-walley-se`.
* Requests like `Configure Adyen Adapter` demonstrate how sensitive PSP API keys are securely passed to the Norce Adapters on the server-side.

### 3. Purchase [Checkout] (Flow Testing)

These folders demonstrate the full runtime flow of an order using the configured adapters. The typical sequence includes:

| Request | Endpoint | Function |
| :--- | :--- | :--- |
| **Step 1:** `[Norce Adapter] Create Checkout Order` | `POST /api/v1/orders` | Imports the prepared Commerce basket into Norce Checkout, retrieving the `orderId`. |
| **Step 2:** `[Order] Get Order` | `GET /api/v0/checkout/orders/{orderId}` | Retrieves the current state of the Checkout Order. |
| **Step 3:** `[PSP Adapter] Initiate Payment` | `POST /.../payments` | Starts the payment session, returning data necessary for the client-side UI (e.g., `paymentMethodsResponse`). |
| **Step 4:** `[PSP Adapter] Complete Payment` | `POST /.../complete` | Simulates the final action required to move the order state to `completed` after successful authorization (often simulating a server-side Webhook). |

## 🔒 Security and Compliance Notes

* **API Gateway:** All API calls mimic a secure backend proxy. Headers like `x-merchant` and the Bearer `token` are used for authentication and scope enforcement within the Norce API environment.
* **Tokenization:** Payment requests handle tokens and should **never** expose raw payment details (PCI data).
* **History Flattening:** The repository's Git history has been explicitly flattened and rewritten to ensure that no credential values were inadvertently committed and exposed.
