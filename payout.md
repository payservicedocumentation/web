# PayOUT API Specification V2 INDIA

> **Спецификация:** PayOUT API V2 (Массовые выплаты)  
> **Регион:** Индия (INDIA)  

---

## 1. Authentication
In order to access the API, clients must authenticate using a token. The token should be included in the request header as an authorization bearer token.

**Header example:**
Authorization: Bearer <your_token_here>

---

## 2. Create Payment Request by Merchant
The payment initialization process involves the merchant initiating a payout request. Request parameters are sent in JSON format.

### Endpoints
* **PROD (POST):** https://navenpravah.in/v2/payout/client
* **SANDBOX (POST):** https://merchant-api.pserv-iner.in/v2/payout/client

### Request Example
curl --location --request POST 'https://navenpravah.in/v2/payout/client' \
--header 'Authorization: Bearer <your_token_here>' \
--header 'Content-Type: application/json' \
--data-raw '{ 
    "merchant_request_id": "0000",
    "request_date": "2020-06-29 21:10:36",
    "payout_amount": 6000,
    "payout_currency": "INR",
    "recipient_bank": "Ziraat Bank",
    "recipient_card": "jerom123@kkotak",
    "recipient_name": "Test client",
    "webhook_url": "https://example.com",
    "campaign": "clm8wvjah774323vgcdqful7re7",
    "ifsc": "FFFFF0000000"
}'

### Request Parameters
* **merchant_request_id** (string, Required): A unique identifier provided by the merchant to track the specific payout.
* **request_date** (string, Required): Date and time in format `yyyy-mm-dd hh:mm:ss`.
* **payout_amount** (number, Required): The total sum that the recipient will receive.
* **payout_currency** (string, Required): Currently, only "INR" (Indian Rupees) is supported.
* **recipient_bank** (string, Required): Name of the recipient's bank.
* **recipient_card** (string, Required): Contains either the UPI ID or the bank account number:
  * *For UPI:* The UPI ID string (e.g., `"jerom123@kkotak"`).
  * *For IMPS:* The bank account number string (e.g., `"123456789012"`).
* **recipient_name** (string, Required): Full name of the recipient for identity verification.
* **webhook_url** (string, Required): The URL where final success or failure notifications are delivered.
* **campaign** (string, Optional): Unique tracking identifier for marketing or analytics campaigns.
* **ifsc** (string): Required **only for IMPS**. Alphanumeric code matching format `"FFFFF0000000"`. Not required for UPI.

### Response Example (201 Created)
{
  "ps_request_id": "clcszbdt937269ouwdl27c8nq"
}

---

## 3. Get Payout Status
Retrieves the status of a specific payout request using either the internal `ps_request_id` or the merchant's original `merchant_request_id`.

### Endpoints
* **By ps_request_id (PROD):** GET https://navenpravah.in/v2/payout/client/{ps_request_id}
* **By ps_request_id (SANDBOX):** GET https://merchant-api.pserv-iner.in/v2/payout/client/{ps_request_id}
* **By merchant_request_id (PROD):** GET https://navenpravah.in/v2/payout/client/merchant-request/{merchant_request_id}
* **By merchant_request_id (SANDBOX):** GET https://merchant-api.pserv-iner.in/v2/payout/client/merchant-request/{merchant_request_id}

### Request Example
curl --location --request GET 'https://navenpravah.in/v2/payout/client/clcszbdt937269ouwdl27c8nq' \
--header 'Authorization: Bearer <your_token_here>'

### Response Example (200 OK)
{
  "ps_request_id": "clcszbdt937269ouwdl27c8nq",
  "status": "completed",
  "payout_amount": 6000,
  "payout_currency": "INR",
  "request_date": "2020-06-29 21:10:36",
  "merchant_request_id": "123456",
  "fee": 100,
  "payout_receipts": [
    "https://navenpravah.in/images/example.jpg"
  ]
}

### Status Values Reference
* 'pending' — processing of transaction has not started yet.
* 'processing' — transaction is currently being processed.
* 'completed' — payment operation complete.
* 'rejected' — transaction rejected due to technical reasons.
* 'successed_by_partner' — transaction complete and sent to Merchant server successfully.
* 'rejected_by_partner' — transaction rejected and sent to Merchant server successfully.

---

## 4. Webhook Notifications
Sent automatically to the merchant's specified `webhook_url` when the payout transaction status changes.

### Webhook Data Example
{
    "ps_request_id": "clcszbdt937269ouwdl27c8nq",
    "status": "completed",
    "payout_amount": 6000,
    "payout_currency": "INR",
    "merchant_request_id": "0000",
    "fee": 100,
    "payout_receipts": [
        "https://navenpravah.in/files/example.pdf",
        "https://navenpravah.in/images/example.png"
    ]
}

### Webhook Signature Verification (signatureV2)
Verify authenticity using the `signatureV2` header and your account salt.
Node.js example:
const salt = '<your-salt>';
const signature = request.headers.signatureV2;
const bodyString = JSON.stringify(request.body);
const isReal = bcrypt.compareSync(bodyString, salt + signature);

---

## 5. Balance Request
Returns the current account balance for the merchant.

* **PROD (GET):** https://navenpravah.in/partner/balance
* **SANDBOX (GET):** https://merchant-api.pserv-iner.in/partner/balance

**Response:** {"balance": 1000}

---

## 6. Server Health Check
Verify gateway operational status.

* **Method:** GET
* **URL:** https://navenpravah.in/api/status/check
* **Response:** {"status": true}
