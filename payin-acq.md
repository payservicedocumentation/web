# PayIN-Acq API Specification V2 INDIA

> **Спецификация:** PayIN-Acquiring API V2 (Интернет-эквайринг)  
> **Регион:** Индия (INDIA)  

---

## 1. Authentication
In order to access the API, clients must authenticate using a token. The token should be included in the request header as an authorization bearer token.

**Header example:**
Authorization: Bearer <your_token_here>

---

## 2. Payment Initialization by Merchant (H2H)
The payment initialization process involves the merchant initiating a payment request with card details included. Request parameters are sent in JSON format.

### Endpoints
* **PROD (POST):** https://mahiapi.in/v2/payin-acq2/page
* **SANDBOX (POST):** https://merchant-api.pserv-test1.in/v2/payin-acq2/page

### Request Example
curl --location --request POST 'https://mahiapi.in/v2/payin-acq2/page' \
--header 'Authorization: Bearer <your_token_here>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "sender_account": "Test name sender",
    "amount": 2000,
    "currency": "INR",
    "webhook_url": "https://example.com",
    "redirect_url": "https://google.com",
    "merchant_request_id": "34535432",
    "card_number": "jerom123@kkotak",
    "widget_id": "acq.v2.s4.5",
    "force_redirect": false,
    "traffic_type": "ftd",
    "campaign": "clm8wvjah774323vgcdqful7re7"
}'

### Request Parameters
* **sender_account** (string, Required): The name of the person who will send the money.
* **amount** (float, Required): The amount of money to be transferred.
* **currency** (string, Required): Only 'INR' is supported.
* **webhook_url** (string, Required): URL for receiving webhook notifications.
* **redirect_url** (string, Required): Where user is redirected after completing the procedure.
* **merchant_request_id** (string, Required): Unique identifier of the request from the client.
* **card_number** (string, Required): The specified card / account data details.
* **widget_id** (string, Required): Unique identifier of the widget to be displayed.
* **force_redirect** (boolean, Optional, default: false): Avoid displaying success/failure pages, redirect immediately.
* **traffic_type** (string, Optional): Values: "ftd" (first-time deposit) or "std" (recurrent traffic).
* **campaign** (string, Optional): Unique tracking identifier for the campaign.

### Response Example (200 OK)
{
  "ps_request_id": "clgggiss10003dvm2qnd01n5e",
  "url": "https://acqr.3dscoolpaid.com/clgggiss10003dvm2qnd01n5e/transfer"
}

---

## 3. Request Status (By PS Request ID)
Retrieve the current transaction status using the unique internal payment system request ID.

### Endpoints
* **PROD (GET):** https://mahiapi.in/v2/payin-acq2/client/{ps_request_id}
* **SANDBOX (GET):** https://merchant-api.pserv-test1.in/v2/payin-acq2/client/{ps_request_id}

### Response Example (200 OK)
{
  "status": "pending",
  "amount": 2000,
  "currency": "INR",
  "receivedAmount": 2000
}

---

## 4. Request Status (By Merchant Request ID)
Retrieve the current transaction status using your original merchant request ID.

### Endpoints
* **PROD (GET):** https://mahiapi.in/v2/payin-acq2/client/merchant-request/{merchant_request_id}
* **SANDBOX (GET):** https://merchant-api.pserv-test1.in/v2/payin-acq2/client/merchant-request/{merchant_request_id}

### Response Example (200 OK)
{
  "status": "pending",
  "amount": 2000,
  "currency": "INR",
  "receivedAmount": 2000
}

### Status Values Reference
* 'pending' — processing of transaction has not started yet.
* 'completed' — payment operation complete.
* 'rejected' — transaction rejected due to technical reasons.
* 'successed_by_partner' — transaction complete and sent to merchant server successfully.
* 'rejected_by_partner' — transaction rejected and sent to merchant server successfully.

---

## 5. Webhook Notifications
Webhooks inform the client about changes in status. Sent to the specified webhook_url.

### Webhook Data Example
{
    "merchant_request_id": "123456",
    "ps_request_id": "clcszbdt937269ouwdl27c8nq",
    "status": "completed",
    "amount": 6000,
    "receivedAmount": 6000,
    "currency": "INR",
    "error3ds": "A technical error occurred during the payment process"
}

### Webhook Signature Verification (signatureV2)
Verify authenticity using the signatureV2 header.
Node.js example:
const salt = '<your-salt>';
const signature = request.headers.signatureV2;
const bodyString = JSON.stringify(request.body);
const isReal = bcrypt.compareSync(bodyString, salt + signature);

---

## 6. Balance Request
Returns the current account balance for the merchant.

### Endpoints
* **PROD (PUT):** https://mahiapi.in/partner/balance
* **SANDBOX (PUT):** https://merchant-api.pserv-test1.in/partner/balance

**Response:** {"balance": 1000}

---

## 7. Creation of ACQ PayIN without Card Data (Hosted Page)
This request enables the client to trigger a payment form interface to input card data via šecure gateway web view, rather than transmitting card details via H2H API.

### Endpoints
* **PROD (POST):** https://mahiapi.in/v2/payin-acq2/idle
* **SANDBOX (POST):** https://merchant-api.pserv-test1.in/v2/payin-acq2/idle

### Request Example
curl --location 'https://mahiapi.in/v2/payin-acq2/idle' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {{token}}' \
--data '{
    "merchant_request_id": "111112121",
    "sender_account": "Test Testovich",
    "amount": 2000,
    "currency": "INR",
    "widget_id": "acq.v2.s4.5",
    "redirect_url": "https://example.com",
    "webhook_url": "https://example.com",
    "campaign": "cls01x88a3833uguwbyrz0gh1"
}'

### Response Example (200 OK)
{
  "ps_request_id": "clswzc0yf0002xsuwhk0ih4os",
  "url": "https://example.com/clswzc0yf0002xsuwhk0ih4os/transfer"
}
