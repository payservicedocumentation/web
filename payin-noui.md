# PayIN NoUI API Specification V2 INDIA

> **Спецификация:** PayIN NoUI API V2 (Прямая интеграция без виджета)  
> **Регион:** Индия (INDIA)  

---

## 1. Authentication
In order to access the API, clients must authenticate using a token. The token should be included in the request header as an authorization bearer token.

**Header example:**
Authorization: Bearer <your_token_here>

---

## 2. Payment Initialization by Merchant
The payment initialization process involves the merchant initiating a payment request. Request parameters are sent in JSON format.

### Endpoints
* **PROD (POST):** https://navenpravah.in/v2/payin/client/noui
* **SANDBOX (POST):** https://merchant-api.pserv-iner.in/v2/payin/client/noui

### Request Example
curl --location --request POST 'https://navenpravah.in/v2/payin/client/noui' \
--header 'Authorization: Bearer <your_token_here>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "sender_account": "Test ID sender",
    "sender_name": "Test name sender",
    "amount": 100,
    "currency": "INR",
    "payment_method_type": "imps",
    "webhook_url": "https://example.com",
    "merchant_request_id": "34535432",
    "traffic_type": "ftd",
    "campaign": "clm8wvjah774323vgcdqful7re7"
}'

### Request Parameters
* **sender_account** (string, Required): The name or unique_id of the person who will send the money.
* **sender_name** (string, Optional): The full name of the sender. Max length: 150 characters.
* **amount** (number, Required): The amount of money to be transferred.
* **currency** (string, Required): Only 'INR' is supported.
* **payment_method_type** (string, Optional): Lowercase values: "imps", "upi", "upi_intent", or "mqr". Essential to specify for correct routing.
* **webhook_url** (string, Required): The URL for receiving updates on success or failure.
* **merchant_request_id** (string, Required): A unique identifier of the request provided by the client.
* **traffic_type** (string, Optional): Values: "ftd" (first-time deposit) or "std" (recurrent traffic).
* **campaign** (string, Optional): Unique identifier of the marketing campaign for tracking.

### Response Example (201 Created)
{
  "valid_till": 1683128777989,
  "ps_request_id": "clgggiss10003dvm2qnd01n5e",
  "card_number": "phone_number@bank_name",
  "upi": {
    "id": "phone_number@bank_name",
    "intentUrl": "string (optional)"
  },
  "imps": {
    "accountNumber": "922010033318770",
    "beneficiaryName": "Manju Devi",
    "IFSC": "HDFC0000620"
  }
}

---

## 3. Upload Payment Confirmation (UTR/RECEIPT)
Allows the merchant to upload a confirmation document or UTR for a previously initiated transaction. Max file size: 1MB.

### Endpoints
* **PROD (POST):** https://navenpravah.in/v2/payin/client/noui/{ps_request_id}/upload
* **SANDBOX (POST):** https://merchant-api.pserv-iner.in/v2/payin/client/noui/{ps_request_id}/upload

### Request Example (UTR)
curl -X POST 'https://navenpravah.in/v2/payin/client/noui/cm66grnyt0002d8pj6rj485cb/upload-utr' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {{AUTH_PARTNER_TOKEN}}' \
--data '{
  "utr": "9876543210"
}'

### Request Example (Receipt File)
curl --location --request POST 'https://navenpravah.in/v2/payin/client/noui/clikqi25s0001trm20q0qa5dr/upload' \
--header 'Authorization: Bearer <your_token_here>' \
--form 'file=@"/D:/files/payment_confirmation.pdf"'

---

## 4. Request Status
Retrieve the current status of a NoUI payment request using the unique internal transaction ID.

### Endpoints
* **PROD (GET):** https://navenpravah.in/payin/client/noui/{ps_request_id}
* **SANDBOX (GET):** https://merchant-api.pserv-iner.in/payin/client/noui/{ps_request_id}

### Response Example (200 OK)
{
  "id": "clgggiss10003dvm2qnd01n5e",
  "status": "pending",
  "amount": 2000,
  "currency": "INR",
  "receivedAmount": 2000,
  "isChargeback": false,
  "merchantRequestId": "34535432"
}

### Status Values Reference
* 'pending' — processing of transaction has not started yet.
* 'completed' — payment operation complete.
* 'rejected' — transaction rejected due to technical reasons.
* 'successed_by_partner' — transaction complete and sent to Merchant successfully.
* 'rejected_by_partner' — transaction rejected and sent to Merchant successfully.

---

## 5. Webhook Notifications
Webhooks inform the client about status changes. Sent automatically to the merchant's specified webhook_url.

### Webhook Data Example
{
    "merchant_request_id": "123456",
    "ps_request_id": "clcszbdt937269ouwdl27c8nq",
    "status": "completed",
    "amount": 6000,
    "receivedAmount": 6000,
    "isChargeback": false,
    "currency": "INR"
}

### Webhook Signature Verification (signatureV2)
To verify webhooks, check the signatureV2 header using your salt.
Node.js example:
const salt = '<your-salt>';
const signature = request.headers.signatureV2;
const bodyString = JSON.stringify(request.body);
const isReal = bcrypt.compareSync(bodyString, salt + signature);

---

## 6. Cancel PayIN Request
Allows the merchant to cancel an active or uncompleted PayIN request.

### Endpoints
* **PROD (PUT):** https://navenpravah.in/v2/payin/client/noui/{ps_request_id}/reject
* **SANDBOX (PUT):** https://merchant-api.pserv-iner.in/v2/payin/client/noui/{ps_request_id}/reject

---

## 7. Set Customer Completed
Manually marks the customer payment step as completed within the system.

### Endpoints
* **PROD (POST):** https://navenpravah.in/payin/client/noui/customer-completed
* **SANDBOX (POST):** https://merchant-api.pserv-iner.in/payin/client/noui/customer-completed

### Request Body
{
    "id": "clgggiss10003dvm2qnd01n5e"
}

---

## 8. Balance Request
Returns the current merchant account balance.

* **PROD (GET):** https://navenpravah.in/partner/balance
* **SANDBOX (GET):** https://merchant-api.pserv-iner.in/partner/balance

**Response:** {"balance": 1000}

---

## 9. Server Health Check
Verify gateway operational status.

* **Method:** GET
* **URL:** https://mahiapi.in/api/status/check
* **Response:** {"status": true}
