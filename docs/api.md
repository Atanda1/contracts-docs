---
sidebar_position: 3
---

# APIs

Here are some of the endpoints you'd require to get neccessary information to perform transactions

```
baseUrl = https://api.paycrest.io
```

---

## Fetch Token Rate

Endpoint: `{baseUrl}/v1/rates/:token/:amount/:fiat`
Method: `GET`

### Path Parameters

- **`token`**: The cryptocurrency token for which you wish to fetch the rate.
- **`amount`**: The amount of the cryptocurrency token. Larger amounts could have better rates.
- **`fiat`**: The local currency to which you want to convert the cryptocurrency token amount.

### Query Parameters

- **`provider_id`**: The ID of a provider. Use this to get the rate of a specific liquidity provider

### Response Format

A successful response from the `GET {baseUrl}/v1/rates/:token/:amount/:fiat` endpoint will contain a JSON object with the following structure:

```json
{
  "message": "OK",
  "status": "success",
  "data": "1500"
}
```

---

## Verify Account

Endpoint: `{baseUrl}/v1/verify-account`
Method: `POST`

### Payload

A JSON object containing the recipient account details.

| Field name          | Type     | Required | Comment                                                                                                                                                                                     |
| ------------------- | -------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `institution`       | `string` | Yes      | The `code` of the financial institution receving the local currency. [Fetch supported institutions](https://www.notion.so/Sender-API-Documentation-10d2482d45a2802da265f53e212f3eae?pvs=21) |
| `accountIdentifier` | `string` | Yes      | The recipient's bank or mobile money account number                                                                                                                                         |

```json
{
  "institution": "FBNINGLA",
  "accountIdentifier": "123456789"
}
```

### Response Format

A JSON object containing the receive address to accept the cryptocurrency token.

```json
{
  "message": "Account name was fetched successfully",
  "status": "success",
  "data": "John Doe" // will return "OK" if PSP doesn't support name resolution
}
```

---

## Fetch Supported Institutions

Endpoint: `{baseUrl}/v1/institutions/:currency_code`
Method: `GET`

### Response Format

A successful response from the `GET {baseUrl}/v1/institutions/:currency_code` endpoint will contain a JSON object with the following structure:

```json
{
  "message": "OK",
  "status": "success",
  "data": [
    {
      "name": "GT Bank Plc",
      "code": "GTBINGLA",
      "type": "bank" // bank or mobile_money
    },
    {
      "name": "First Bank of Nigeria",
      "code": "FBNINGLA",
      "type": "bank" // bank or mobile_money
    }
  ]
}
```

---

## Fetch Supported Currencies

Endpoint: `{baseUrl}/v1/currencies`
Method: `GET`

### Response Format

A successful response from the `GET {baseUrl}/v1/currencies` endpoint will contain a JSON object with the following structure:

```json
{
  "message": "OK",
  "status": "success",
  "data": [
    {
      "code": "XOF-BEN",
      "name": "West African CFA franc",
      "shortName": "Céfa Benin",
      "decimals": 2,
      "symbol": "CFA",
      "marketRate": "599.5"
    },
    {
      "code": "NGN",
      "name": "Nigerian Naira",
      "shortName": "Naira",
      "decimals": 2,
      "symbol": "₦",
      "marketRate": "1629.59"
    },
    {
      "code": "KES",
      "name": "Kenyan Shilling",
      "shortName": "KES",
      "decimals": 2,
      "symbol": "KSh",
      "marketRate": "129.3"
    }
  ]
}
```

---

## Fetch Payment Order by ID

Endpoint: `{baseUrl}/v1/sender/orders/:id`
Method: `GET`

### Path Parameters

- `id`: The unique identifier for the order

### Response Format

A JSON object containing detailed information about the fetched payment order.

```json
{
  "message": "The order has been successfully retrieved",
  "status": "success",
  "data": {
    "id": "uuid-string",
    "amount": "100.00",
    "amountPaid": "80.00",
    "amountReturned": "0.00",
    "token": "USDT",
    "senderFee": "0.50",
    "transactionFee": "0.10",
    "rate": "1500",
    "network": "polygon",
    "gatewayId": "gw-123",
    "reference": "unique-reference",
    "recipient": {
      "institution": "GTBINGLA",
      "accountIdentifier": "123456789",
      "accountName": "John Doe",
      "memo": "Payment from John Doe"
    },
    "fromAddress": "0x123...",
    "returnAddress": "0x123...",
    "receiveAddress": "0x123...",
    "feeAddress": "0x123...",
    "createdAt": "2024-06-26T12:34:56Z",
    "updatedAt": "2024-06-27T12:34:56Z",
    "txHash": "abc123",
    "status": "initiated",
    "transactions": [
      {
        "id": "uuid-string",
        "gatewayId": "gw-123",
        "status": "crypto_deposited",
        "txHash": "abc123",
        "createdAt": "2024-06-26T12:34:56Z"
      }
    ]
  }
}
```

---

## Fetch All Payment Orders

Endpoint: `{baseUrl}/v1/sender/orders`
Method: `GET`

### Query Parameters

- `ordering`: Sorts the results based on the specified field.
- `status`: Filters orders by their status.
- `token`: Filters orders by their token.
- `network`: Filters orders by their network.
- `page`: Specifies the page number for pagination.
- `pageSize`: Specifies the number of items per page for pagination.

### Response Format

A JSON array containing detailed information about each fetched payment order.

```json
{
    "message":"Payment orders retrieved successfully",
    "status":"success",
    "data":
        {
          "total": 100,
          "page": 1,
          "pageSize": 20,
          "orders":
                [
                    {
                      "id": "uuid-string",
                      "amount": "100.00",
                      "amountPaid": "80.00",
                      "amountReturned": "0.00",
                      "token": "unique-token-id",
                      "senderFee": "0.50",
                      "transactionFee": "0.10",
                      "rate": "1500",
                      "network": "polygon",
                      "gatewayId": "0x123...",
                      "reference": "unique-reference",
                      "recipient": {
                        "institution": "GTBINGLA",
                        "accountIdentifier": "123456789",
                        "accountName": "John Doe",
                        "memo": "Payment from John Doe"
                      },
                      "fromAddress": "0x123...",
                      "returnAddress": "0x123...",
                      "receiveAddress": "0x123...",
                      "feeAddress": "0x123...",
                      "createdAt": "2024-06-26T12:34:56Z",
                      "updatedAt": "2024-06-27T12:34:56Z",
                      "txHash": "abc123...",
                      "status": "settled",
                    }
                    ...
                ]
        }
}
```
