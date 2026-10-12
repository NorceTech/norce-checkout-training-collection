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
    * **Kustom** (API username and password — *optional*)

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
        * `kustomUsername`, `kustomPassword`

    * **Save** and **Activate** the environment.

3.  **Set Your Own Webhook URL:**
    The environment variable `webhooksiteurl` points to a shared [webhook.site](https://webhook.site) session. The committed URL has expired and answers 404, which makes the validation and hook scenarios fail. Create your own session at [webhook.site](https://webhook.site) and update this variable locally. To run the scenarios without inspecting the calls, `https://200.returnco.de` answers 200 with an empty body.

    For the hook scenario (Non-PSP Additional Operations → *3. Notifications and Hooks*), configure your webhook.site URL to respond with an empty body or `[]`. Checkout treats any other non-JSON response from a hook as an error and rolls back the state change.

4.  **Kustom snippet pages (optional):**
    `web/checkout.html` and `web/confirmation.html` render the Kustom checkout and confirmation snippets. Open them locally in a browser and paste the response from `[Kustom Adapter] Initiate Payment` or `[Kustom Adapter] Get Payment`.

## 🗺️ Collection Structure and Flow

The collection has three kinds of folders: basket preparation in Commerce, purchase flows in Checkout and channel configuration.

### 1. Arrange Basket [Commerce] / One-Click Basket [Commerce]

`Arrange Basket` builds the basket step by step. `One-Click Basket` creates a complete basket in one request. Both focus on creating and preparing the shopping basket in **Norce Commerce** before it is imported into Checkout.

| Request | Purpose |
| :--- | :--- |
| `[Shopping] Create basket` | Initializes a new shopping basket and extracts the `basketId`. Authentication is handled automatically via OAuth2 Client Credentials configured at the folder level. |
| `[Shopping] Insert basket item` | Adds a product to the basket. |
| `[Shopping] Update Buyer` / `[Shopping] Update Payment Method` / `[Shopping] Update Delivery Method` | Sets customer and logistic details required for successful checkout. |

### 2. Configuration [Checkout]

This folder contains requests used for setting up the **Norce Checkout Channel Configurations**.

* The folder is subdivided by channel: `nc-walley-se`, `nc-nonpsp-se`, `nc-adyen-se` and `nc-kustom-se`.
* Requests like `Configure Adyen Adapter` demonstrate how sensitive PSP API keys are securely passed to the Norce Adapters on the server-side.

### 3. Purchase [Checkout] (Flow Testing)

One folder per adapter: `Purchase Non-PSP [Checkout] Simple`, `Purchase Non-PSP [Checkout] Additional Operations`, `Purchase Walley [Checkout]`, `Purchase Adyen [Checkout]` and `Purchase Kustom [Checkout]`. Start each flow with a fresh basket. Use `Arrange Basket` for flows that complete the order: the One-Click basket has no delivery address, so Commerce rejects it at completion. The typical sequence is:

| Request | Endpoint | Function |
| :--- | :--- | :--- |
| **Step 1:** `[Norce Adapter] Create Checkout Order` | `POST /api/v1/orders` | Imports the prepared Commerce basket into Norce Checkout, retrieving the `orderId`. |
| **Step 2:** `[Order] Get Order` | `GET /api/v1/checkout/orders/{orderId}` | Retrieves the current state of the Checkout Order. |
| **Step 3:** `[PSP Adapter] Initiate Payment` | `POST /.../payments` | Starts the payment session, returning data necessary for the client-side UI (e.g., `paymentMethodsResponse` or `htmlSnippet`). |
| **Step 4 (Non-PSP only):** `[Non-PSP Adapter] Complete Payment` | `POST /.../complete` | Moves the order to `completed`. Answers `202 Accepted`. |

For Walley, Adyen and Kustom the purchase is completed in the provider's checkout, not in Bruno.

`Purchase Non-PSP [Checkout] Additional Operations` contains three self-contained scenarios in subfolders:

1. **Update Order** — update cart, customer addresses and payment method, then complete.
2. **Validations** — register a passing and a failing external validation and run `Validate Order`.
3. **Notifications and Hooks** — register a notification and a state hook, complete the payment and inspect the calls in webhook.site.

Hooks and validations can only be added while the order is in `checkout` state.

## 🔒 Security and Compliance Notes

* **API Gateway:** All API calls mimic a secure backend proxy. Headers like `x-merchant` and the Bearer `token` are used for authentication and scope enforcement within the Norce API environment.
* **Tokenization:** Payment requests handle tokens and should **never** expose raw payment details (PCI data).
* **History Flattening:** The repository's Git history has been explicitly flattened and rewritten to ensure that no credential values were inadvertently committed and exposed.
