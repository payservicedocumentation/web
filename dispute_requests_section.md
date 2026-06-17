## Dispute Requests via API

This section describes how merchants can submit dispute requests for PayIN and PayOUT transactions via API. This API automates dispute processing and integrates it into the Support dashboard, reducing manual effort and processing time.

### Endpoint

**POST `/partner/appeal`**

This endpoint allows merchants to submit a dispute request with the option to attach up to three files (e.g., images and documents).

### Request Format

**Content-Type:** multipart/form-data

### Parameters

- **id** (Required)
  - **Type:** string
  - **Description:** The unique identifier for the transaction (PayIN or PayOUT).
  - **Example:** `"cm2bec4u10001338d6lzayn78"`

- **attachments** (Required)
  - **Type:** file (1-3 files)
  - **Description:** Files attached to the dispute request. Supported formats include JPEG, PNG, PDF, etc.
  - **Example:** `"file1.jpeg"`, `"file2.pdf"`, `"file3.png"`

- **comment** (Optional)
  - **Type:** string
  - **Description:** A comment provided by the merchant to clarify the request.
  - **Example:** `"Additional details about the dispute."`

### Request

| Instance | Method | URL |
| --- | --- | --- |
| PROD | POST | https://navenpravah.in/v2/ |
| SANDBOX | POST | https://merchant-api.stageswiss.net/ |

### Example Request

```bash
curl -X POST --location 'https://merchant-api.pserv-iner.in' \
--header 'Authorization: Bearer <your_token_here>' \
--form 'id="cm2bec4u10001338d6lzayn78"' \
--form 'attachments=@"file1.jpeg"' \
--form 'attachments=@"file2.pdf"' \
--form 'attachments=@"file3.png"' \
--form 'comment="Additional information about the transaction."'
```

### Example Response

```json
{
    "status": "ok"
}
```

### Workflow

1. **Submit Request** — The merchant sends a POST request to the `/partner/appeal` endpoint with the required and optional parameters.
2. **Status Update** — Upon submission, the transaction status is automatically updated to `Review`. The request is displayed in a new column in the Support dashboard: **API Request for Review**.
3. **Review Process** — The Support team processes the request and updates its status accordingly.

### Notes

- Ensure that the Authorization token is valid to authenticate API requests.
- The `id` parameter must correspond to an existing transaction in the system.
- The attachment file size and format must comply with system limits (5 MB).
