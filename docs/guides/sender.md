# Sender API

# Sender API Documentation

Welcome to the comprehensive guide for the Paycrest Sender API. This documentation serves as a detailed reference for developers integrating Paycrest’s Sender API services into their applications. Whether you're looking to initiate orders, fetch order details, manage webhooks, or handle user authentication, this documentation covers all the necessary aspects to get started and ensure a smooth integration process.

If you encounter issues, find bugs, or have suggestions for improvements, just send an email to [support@paycrest.io](mailto:support@paycrest.io).

Thank you for choosing Paycrest as your off-ramp partner. We're excited to see the innovative solutions you build with our API and look forward to supporting your success.

## Getting Started

### Get API keys

Visit [your sender dashboard](https://app.paycrest.io/sender/overview) to retrieve your `Client ID` and `Client Secret`.

If you're a new user, [signup here](https://app.paycrest.io/signup) as a "sender" and complete our Know-Your-Business (KYB) process.

Your `Client Secret` should always be kept secret. If you accidentally expose it via version control or other means, you should immediately reach out to [support@paycrest.io](mailto:support@paycrest.io) to generate a new API key.

### Configure tokens

Head over to the [settings page](https://app.paycrest.io/sender/settings) of your Sender Dashboard to configure the `feePercent`, `feeAddress`, and `refundAddress` across the tokens and blockchain networks you intend to use.

### Interacting with an endpoint

Include your `Client ID` in the "API-Key" header of every request you make to Paycrest Offramp API.

e.g

```tsx
const headers = {
  "API-Key": "208a4aef-1320-4222-82b4-e3bca8781b4b",
}
```

Any request without a valid API key will fail with status code `401: Unauthorized`. All API calls must be made over HTTPS.

Body of requests and responses are formatted as JSON with Content-Type always as `application/json`

---

## Initiate Payment Order

Endpoint: `/v1/sender/orders`
Method: `POST`

### Payload

A JSON object containing the details of the payment order and the recipient information.

| Field name | Type | Required | Comment |
| --- | --- | --- | --- |
| `amount` | `number` | Yes | The amount of the token to be off-ramped. |
| `rate` | `number` | Yes | The exchange rate from the token to the local currency. [Fetch current rate](https://www.notion.so/Sender-API-Documentation-10d2482d45a2802da265f53e212f3eae?pvs=21) |
| `network` | `string` | Yes | The blockchain network on which the token operates.
**Supported networks**: `tron`, `base`, `bnb-smart-chain`, `polygon`, `arbitrum-one` |
| `token` | `string` | Yes | The symbol of the cryptocurrency token.
**Supported tokens**: `USDT`, `USDC` 
* *USDC is not supported on Tron*
* *USDT is not supported on Base* |
| `recipient` | `object` | Yes | [see more](https://www.notion.so/Sender-API-Documentation-10d2482d45a2802da265f53e212f3eae?pvs=21) |
| `returnAddress` | `string` | No | The wallet address to refund the fund to if there is a failure. |
| `reference` | `string` | No | If specified, the field should be a unique identifier for the payment order. Only **`-`**,**`_`** and alphanumeric characters allowed. |
| `feePercent` | `number` | No | The percentage of the token amount charged for creating the order. |
| `feeAddress` | `string` | No | The address to which the fees for the transaction should be sent. |

### Recipient object

Ensure you verify the account details using the [Verify Account endpoint](https://www.notion.so/Sender-API-Documentation-10d2482d45a2802da265f53e212f3eae?pvs=21).

| Field name | Type | Required | Comment |
| --- | --- | --- | --- |
| `institution` | `string` | Yes | The `code` of the financial institution receving the local currency. [Fetch supported institutions](https://www.notion.so/Sender-API-Documentation-10d2482d45a2802da265f53e212f3eae?pvs=21) |
| `accountIdentifier` | `string` | Yes | The recipient's bank or mobile money account number |
| `accountName` | `string` | Yes | The name associated with the recipient's account. |
| `memo` | `string` | Yes | An arbitrary narration for the fiat transfer, could be the name of the sender. |
| `providerId` | `string` | No | An optional identifier for the liquidity provider. Set this to off-ramp with a specific provider |

```json
{
  "amount": 100.00,
  "token": "USDC",
  "rate": 1500,
  "network": "polygon",
  "recipient": {
    "institution": "GTBINGLA",
    "accountIdentifier": "123456789",
    "accountName": "John Doe",
    "memo": "Payment from John Doe",
    "providerId": ""
  },
  "returnAddress": "0x123...",
  "reference": "unique-reference",
  "feePercent": "2",
  "feeAddress": "0x123..."
}
```

### Response Format

A JSON object containing the receive address to accept the cryptocurrency token.

```json
{
  "message": "Payment order initiated successfully",
  "status": "success",
  "data":
    {
      "id": "uuid-string",
      "amount": "100.00",
      "token": "USDT",
      "network": "polygon",
      "receiveAddress": "0x1234...",
      "validUntil": "2024-07-01T12:34:56Z",
      "senderFee": "0.50",
      "transactionFee": "0.10",
      "reference": "unique-reference"
    }
}
```

---

## Fetch Token Rate

Endpoint: `/v1/rates/:token/:amount/:fiat`
Method: `GET`

### Path Parameters

- **`token`**: The cryptocurrency token for which you wish to fetch the rate.
- **`amount`**: The amount of the cryptocurrency token. Larger amounts could have better rates.
- **`fiat`**: The local currency to which you want to convert the cryptocurrency token amount.

### Query Parameters

- **`provider_id`**: The ID of a provider. Use this to get the rate of a specific liquidity provider

### Response Format

A successful response from the `GET /v1/rates/:token/:amount/:fiat` endpoint will contain a JSON object with the following structure:

```json
{
  "message": "OK",
  "status": "success",
  "data": "1500"
}
```

---

## Verify Account

Endpoint: `/v1/verify-account`
Method: `POST`

### Payload

A JSON object containing the recipient account details.

| Field name | Type | Required | Comment |
| --- | --- | --- | --- |
| `institution` | `string` | Yes | The `code` of the financial institution receving the local currency. [Fetch supported institutions](https://www.notion.so/Sender-API-Documentation-10d2482d45a2802da265f53e212f3eae?pvs=21) |
| `accountIdentifier` | `string` | Yes | The recipient's bank or mobile money account number |

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

Endpoint: `/v1/institutions/:currency_code`
Method: `GET`

### Response Format

A successful response from the `GET /v1/institutions/:currency_code` endpoint will contain a JSON object with the following structure:

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
    },
  ]
}
```

---

## Fetch Supported Currencies

Endpoint: `/v1/currencies`
Method: `GET`

### Response Format

A successful response from the `GET /v1/currencies` endpoint will contain a JSON object with the following structure:

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
    },
  ]
}
```

---

## Fetch Payment Order by ID

Endpoint: `/v1/sender/orders/:id`
Method: `GET`

### Path Parameters

- `id`: The unique identifier for the order

### Response Format

A JSON object containing detailed information about the fetched payment order.

```json
{
  "message": "The order has been successfully retrieved",
  "status":"success",
  "data":
    {
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

Endpoint: `/v1/sender/orders`
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

---

## Webhooks

Listen to payment order events by configuring a Webhook URL in your [dashboard settings](https://app.paycrest.io/sender/settings). We trigger various events based on the status of the payment order. Our webhook events are sent exponentially until 24 hours from when the first one is sent.

### Payload Structure

The webhook payload contains the event type and the associated data.

```json
{
  "event": "payment_order.settled",
  "data": {
    "id": "uuid-string",
    "amount": "100.00",
    "amountPaid": "80.00",
    "amountReturned": "0.00",
    "percentSettled": "90.00",
    "senderFee": "0.50",
    "networkFee": "0.05",
    "rate": "1500",
    "network": "polygon",
    "gatewayId": "gw-123",
    "reference": "unique-reference",
    "senderId": "uuid-string",
    "recipient": {
      "currency": "NGN",
      "institution": "FBNINGLA",
      "accountIdentifier": "123456789",
      "accountName": "John Doe",
      "memo": "Payment from John Doe"
    },
    "fromAddress": "0x123...",
    "returnAddress": "0x123...",
    "updatedAt": "2024-06-26T12:34:56Z",
    "createdAt": "2024-06-26T12:34:56Z",
    "txHash": "abc123",
    "status": "settled"
  }
```

### Event Types

The webhook can trigger various events based on the status of the payment order.

- `payment_order.pending`
- `payment_order.expired`
- `payment_order.settled`
- `payment_order.refunded`

### Verifying events

Each event must include a signature in its header (`"X-Paycrest-Signature"`) to ensure the message originates from Paycrest. Upon receiving a request, the receiver should verify this signature to authenticate the request.

1. **Extract the Signature**: Retrieve the `"X-Paycrest-Signature"` header from the incoming request.
2. **Calculate the Signature**: Compute the HMAC signature of the request body using your `Client Secret` key.
3. **Compare Signatures**: Compare the extracted signature with the calculated signature. If they match, the request is authenticated.

### TypeScript sample implementation

```
import crypto from 'crypto';

function verifyPaycrestSignature(requestBody: string, signatureHeader: string, secretKey: string): boolean {
  const calculatedSignature = calculateHmacSignature(requestBody, secretKey);
  return signatureHeader === calculatedSignature;
}

function calculateHmacSignature(data: string, secretKey: string): string {
  const key = Buffer.from(secretKey);
  const hash = crypto.createHmac('sha256', key);
  hash.update(data);
  return hash.digest('hex');
}
```