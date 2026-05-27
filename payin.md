# PayIN API Specification V2 INDIA

> **Спецификация:** PayIN API V2 (С платежным виджетом)  
> **Регион:** Индия (INDIA)  

---

## 1. Authentication
In order to access the API, clients must authenticate using a token. The token should be included in the request header as an authorization bearer token.

**Header example:**
Authorization: Bearer <your_token_here>

---

## 2. Payment Initialization by Merchant
The payment initialization process involves the merchant initiating a payment request. Request parameters are being sent in a form of JSON.

### Endpoints
* **PROD (POST):** https://navenpravah.in/v2/payin/client
* **SANDBOX (POST):** https://merchant-api.pserv-iner.in/v2/payin/client

### Request Example
curl --location --request POST 'https://navenpravah.in/v2/payin/client' \
--header 'Authorization: Bearer <your_token_here>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "sender_account": "Test ID sender",
    "sender_name":  "Name sender",
    "amount": 100,
    "currency": "INR",
    "payment_method_type": "imps",
    "webhook_url": "https://example.com",
    "redirect_url": "https://google.com",
    "merchant_request_id": "34535432",
    "bank": "paytm",
    "widget_id": "p2p",
    "force_redirect": false,
    "traffic_type": "ftd",
    "campaign": "clm8wvjah774323vgcdqful7re7"
}'

### Request Parameters
* **sender_account** (string, Required): The name or unique_id of the person who will send the money.
* **sender_name** (string, Optional): The full name of the sender. Max length: 150 characters.
* **amount** (number, Required): The amount of money that will be transferred.
* **currency** (string, Required): Currently, only 'INR' is supported.
* **payment_method_type** (string, Optional): Lowercase values: "imps", "upi", "upi_intent", or "mqr".
* **webhook_url** (string, Required): The URL for receiving webhook notifications.
* **redirect_url** (string, Required): The URL where the user will be redirected after payment.
* **merchant_request_id** (string, Required): A unique identifier of the request provided by the client.
* **bank** (string, Optional): Allowed values: "paytm" / "phonepe" / "gpay" / "qr".
* **widget_id** (string, Required): A unique identifier of the widget.
* **force_redirect** (boolean, Optional, default: false): If true, bypasses standard success/failure pages.
* **traffic_type** (string, Optional): Values: "ftd" (first-time deposit) or "std" (recurrent traffic).
* **campaign** (string, Optional): Unique identifier of the marketing campaign.

---

## 3. Request PayIn Status
To retrieve the current status of a payment request, clients should make a GET request.

### Endpoints
* **By ps_request_id (PROD):** GET https://navenpravah.in/payin/client/{ps_request_id}
* **By ps_request_id (SANDBOX):** GET https://merchant-api.pserv-iner.in/payin/client/{ps_request_id}
* **By merchant_request_id (PROD):** GET https://navenpravah.in/payin/client/merchant-request/{merchant_request_id}
* **By merchant_request_id (SANDBOX):** GET https://merchant-api.pserv-iner.in/payin/client/merchant-request/{merchant_request_id}

### Response Example (200 OK)
{
  "status": "pending",
  "amount": 2000,
  "currency": "INR",
  "receivedAmount": 2000,
  "isChargeback": false
}

### Status Values Reference
* 'idle' — transaction processing has not started yet.
* 'pending' — awaiting transaction (15 min timer active).
* 'completed' — transaction operation complete.
* 'rejected' — transaction rejected due to technical reasons.
* 'successed_by_partner' — transaction complete and sent to Merchant successfully.
* 'rejected_by_partner' — transaction rejected and sent to Merchant successfully.

---

## 4. Webhook Notifications
Webhooks inform the client about status changes. Sent to the merchant's specified webhook_url.

### Webhook Data Example
{
    "merchant_request_id": "123456",
    "ps_request_id": "clcszbdt937269ouwdl27c8nq",
    "status": "completed",
    "amount": 6000,
    "receivedAmount": 6000,
    "currency": "INR",
    "isChargeback": false
}

---

## 5. Cancel PayIN Request
Allows the merchant to cancel a PayIN request that is not yet completed.

### Endpoints
* **PROD (PUT):** https://navenpravah.in/v2/payin/client/{ps_request_id}/reject
* **SANDBOX (PUT):** https://merchant-api.pserv-iner.in/v2/payin/client/{ps_request_id}/reject

---

## 6. Balance Request
Returns the merchant's current account balance.

### Endpoints
* **PROD (PUT):** https://navenpravah.in/partner/balance
* **SANDBOX (PUT):** https://merchant-api.pserv-iner.in/partner/balance

**Response (200 OK):** {"balance": 1000}

---

## 7. Server Health Check
Verify the operational status of the gateway server.

* **Method:** GET
* **URL:** https://navenpravah.in/api/status/check
* **Healthy Response:** {"status":true}
