
# Bitcoiner Test API Documentation

This API allows developers to integrate Bitcoiner Test questions and test submissions into their applications. The API requires authentication using API keys.

### Authentication

- **API Key**: All requests require an API key, passed in the request headers.
- **API Key Format**:
  - **Standard Package**: `P1<unique_key>` or `P1<unique_key>TEST` (for test mode).
  - **Premium Package**: `P2<unique_key>` or `P2<unique_key>TEST` (for test mode).

Ensure you're using the appropriate API key for live or test mode, depending on your package.

---

## 1. Questions Endpoint

**URL**: `/wp-json/bitcoiner-api/v1/questions`  
**Method**: `GET`  
**Description**: Fetches a set of randomized test questions based on the specified category.

### Request Headers

- `api-key`: Your unique API key (required).

### Query Parameters

| Parameter  | Type   | Required | Description                            |
|------------|--------|----------|----------------------------------------|
| `category` | string | Yes      | The category of questions to retrieve. Acceptable values: `technical`, `non-technical`. |

### Python Example Request

```python
import requests

url = "https://bitcoinertest.org/wp-json/bitcoiner-api/v1/questions"
headers = {
    "api-key": "P1yourapikey"
}
params = {
    "category": "technical"
}

response = requests.get(url, headers=headers, params=params)

if response.status_code == 200:
    print("Questions received:", response.json())
else:
    print(f"Error: {response.status_code}, {response.json()}")
```

### Success Response (200 OK)

```json
{
  "test_session_token": "session_abc12345",
  "questions": [
    {
      "id": "test-1",
      "question": "What is a Bitcoin block?",
      "options": ["a. A unit of data", "b. A wallet", "c. A mining device"]
    },
    {
      "id": "test-2",
      "question": "What does POW stand for?",
      "options": ["a. Proof of Work", "b. Power of Wallet", "c. Protocol of Wealth"]
    }
  ]
}
```

### Error Responses

| Status Code | Message                                              |
|-------------|------------------------------------------------------|
| 400         | Invalid category. Use "technical" or "non-technical".|
| 403         | Invalid or Inactive API key.                         |
| 429         | Rate limit exceeded. Please try again later.         |
| 500         | Question bank not found or server error.             |

---

## 2. Submit Endpoint

**URL**: `/wp-json/bitcoiner-api/v1/submit`  
**Method**: `POST`  
**Description**: Submits answers to the test questions and returns the result.

### Request Headers

- `api-key`: Your unique API key (required).

### Request Body Parameters

| Parameter           | Type    | Required | Description                                              |
|---------------------|---------|----------|----------------------------------------------------------|
| `test_session_token`| string  | Yes      | Unique token received from the Questions endpoint.        |
| `name`              | string  | Yes      | Full name of the test taker.                              |
| `email`             | string  | Yes      | Email address of the test taker.                          |
| `country`           | string  | Yes      | Country of the test taker.                                |
| `answers`           | dict    | Yes      | An object mapping question IDs to selected option labels. |

### Python Example Request

```python
import requests

url = "https://bitcoinertest.org/wp-json/bitcoiner-api/v1/submit"
headers = {
    "api-key": "P1yourapikey",
    "Content-Type": "application/json"
}
data = {
    "test_session_token": "session_abc12345",
    "name": "John Doe",
    "email": "john@example.com",
    "country": "Nigeria",
    "answers": {
        "test-1": "a",
        "test-2": "b"
    }
}

response = requests.post(url, json=data, headers=headers)

if response.status_code == 200:
    print("Submission successful:", response.json())
else:
    print(f"Error: {response.status_code}, {response.json()}")
```

### Success Response (200 OK)

```json
{
  "total_score": 80,
  "percentage": 80,
  "grade": "MERIT",
  "certificate_number": "BT000123",
  "certificate_url": "https://bitcoinertest.org/certificates/BT000123"
}
```

### Error Responses

| Status Code | Message                                      |
|-------------|----------------------------------------------|
| 400         | Missing required fields or invalid answers.  |
| 403         | Invalid or Inactive API key.                 |
| 500         | Server error or invalid test session token.  |

---

## API Key Management

API keys are generated during user registration and payment processing. The API keys have specific limits based on the subscription package (Standard or Premium).

### Key Attributes

| Attribute        | Description                                  |
|------------------|----------------------------------------------|
| `paid`           | Payment status (1 = Paid, 0 = Not Paid).     |
| `active`         | Key status (1 = Active, 0 = Inactive).       |
| `usage_count`    | Number of API requests made.                 |
| `success_requests`| Number of successful API requests.          |
| `error_requests` | Number of failed API requests.               |
| `last_used`      | Timestamp of the last API request.           |
| `last_reset`     | Timestamp when usage counts were last reset. |
| `package`        | User’s subscription package (P1 or P2).      |

---

## Rate Limiting and Quota

The API enforces rate limits to prevent overuse and abuse. Each API key has a maximum number of allowed requests per month, based on the user’s subscription plan.

### Quota

- **Standard Package (P1)**: 100 requests per month.
- **Premium Package (P2)**: 70 requests per month.

### Rate Limit

Requests are limited to 3 per minute. If the limit is exceeded, a 429 status code is returned.

### Example Error Response

```json
{
  "status": 429,
  "message": "Rate limit exceeded. Please try again later."
}
```

---

## Example API Key Usage

### Live API Key

```plaintext
P1abcd1234efgh5678ijkl
```

### Test API Key

```plaintext
P1abcd1234efgh5678ijklTEST
```

Test keys have the suffix `TEST` and can be used to perform tests without affecting your live quota.

---

This documentation provides an overview of the available API endpoints, including Python sample requests. Ensure to replace placeholder values with actual API keys and data before making any requests.
