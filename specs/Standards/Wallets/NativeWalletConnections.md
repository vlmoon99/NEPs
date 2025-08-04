# Native wallet connections

## Summary

Standard interface for connecting native wallets to native apps and web apps.

## Motivation

Currently, there is no standardized approach for connecting native wallets to native apps. Nowadays, each NEAR wallet tries to create their own solutions, which makes it harder for native apps and web apps to connect to native wallets. This new standard solves this problem. 


## Rationale and alternatives

The core functionality needed for native/web apps to connect with native wallets includes signing transactions, sending them to the blockchain, and authorizing users. While desktop web browsers already have this functionality through wallet selectors implemented by wallet providers, mobile apps and mobile web browsers lack a standardized approach.

Currently, mobile developers must implement their own backend solutions to create transactions using near-api-js, sign transactions, send them to the blockchain, check results, manage sessions, and handle all the complex infrastructure. This creates significant development overhead and fragmentation.

This standard provides the necessary interface and bindings for NEAR native wallets to support connections from native apps and web apps, returning signed transactions, messages, meta transactions, or handling sign-and-send operations directly from the wallet.

**No alternatives exist** - all NEAR operations must be performed through native wallets using a standardized protocol, which this specification provides.

## Specification

Native wallet connections use a simple stateless architecture based on request IDs and HTTP requests. The process involves a wallet backend that stores signing requests and returns signed data via simple GET requests.

### Architecture Overview

The connection flow consists of five simple steps:

1. **POST to Wallet Backend** - Submit transaction data and receive a request ID
2. **Deeplink Redirect** - Redirect to wallet app with request ID and callback URL
3. **User Approval** - User reviews and approves/rejects in wallet app
4. **Callback Redirect** - Return to app via callback URL
5. **GET from Wallet Backend** - Retrieve signed transaction data using request ID

### Core Principles

- **Stateless Design**: No persistent connections required
- **Simple HTTP Requests**: Use standard GET/POST operations
- **Request ID Based**: All operations reference a unique request ID
- **Wallet Backend Responsibility**: Wallet providers maintain their own backend infrastructure

### Methods

#### `signTransaction`

Sign a regular NEAR transaction with optional execution.

```ts
interface SignTransactionParams {
  // Encoded Transaction via transactions.Transaction.encode()
  transaction: Uint8Array;
  callbackUrl: string;
  // Optional: if true, wallet will execute the transaction, otherwise return signed transaction
  executeTransaction?: boolean;
  metadata?: {
    appName: string;
    description?: string;
    iconUrl?: string;
  };
}

interface SignTransactionResponse {
  requestId: string;
  deeplinkUrl: string;
}

interface SignTransactionResult {
  status: 'pending' | 'approved' | 'rejected';
  signedTransaction?: Uint8Array; // Encoded SignedTransaction
  transactionHash?: string; // If wallet executed the transaction
  blockHeight?: number; // If wallet executed the transaction
  error?: string;
}
```

#### `getAccounts`

Retrieve available accounts from wallet (for transaction construction).

```ts
interface GetAccountsParams {
  callbackUrl: string;
  metadata?: {
    appName: string;
    description?: string;
    iconUrl?: string;
  };
}

interface GetAccountsResponse {
  requestId: string;
  deeplinkUrl: string;
}

interface GetAccountsResult {
  accounts: Array<Account>;
}

interface Account {
  accountId: string;
  publicKey: string;
}
```

#### `signMessage`

Sign a message for a specific recipient using the user's NEAR account (NEP-0413).

```ts
interface SignMessageParams {
  message: string; // The message to be signed
  recipient: string; // The recipient to whom the message is destined (e.g. "alice.near" or "myapp.com")
  nonce: Uint8Array; // A 32-byte nonce that ensures message uniqueness
  callbackUrl?: string;
  state?: string;
  metadata?: {
    appName: string;
    description?: string;
    iconUrl?: string;
  };
}

interface SignMessageResponse {
  requestId: string;
  deeplinkUrl: string;
}

interface SignedMessage {
  accountId: string; 
  publicKey: string; 
  signature: string;
  state?: string; 
}
```

#### `signMetaTransaction`

Sign a meta transaction using DelegateAction (NEP-0366).

```ts
interface SignMetaTransactionParams {
  delegateAction: {
    senderId: string;
    receiverId: string; 
    actions: Array<Action>;
    nonce: number;
    maxBlockHeight: number;
    publicKey: string;
  };
  signature: string;
  callbackUrl: string;
  // Optional: if true, wallet will execute the transaction, otherwise return signed transaction
  executeTransaction?: boolean;
  metadata?: {
    appName: string;
    description?: string;
    iconUrl?: string;
  };
}

interface SignMetaTransactionResponse {
  requestId: string;
  deeplinkUrl: string;
}

interface SignMetaTransactionResult {
  status: 'pending' | 'approved' | 'rejected';
  signedDelegateAction?: Uint8Array;
  transactionHash?: string; // If wallet executed the transaction
  blockHeight?: number; // If wallet executed the transaction
  error?: string;
}
```

### Wallet Backend Requirements

Wallet providers MUST implement a simple backend with these endpoints:

#### `POST /api/v1/sign-transaction`

**Request Body:**
```json
{
  "transaction": "base64-encoded-transaction",
  "callbackUrl": "myapp://wallet-callback",
  "executeTransaction": true,
  "metadata": {
    "appName": "MyApp",
    "description": "Transfer 1 NEAR",
    "iconUrl": "https://myapp.com/icon.png"
  }
}
```

**Response:**
```json
{
  "requestId": "unique-request-id",
  "deeplinkUrl": "nearwallet://native-bridge?requestId=unique-request-id&callback=myapp://wallet-callback"
}
```

#### `GET /api/v1/sign-transaction/{requestId}`

**Response:**
```json
{
  "status": "approved",
  "signedTransaction": "base64-encoded-signed-transaction",
  "transactionHash": "blockchain-transaction-hash",
  "blockHeight": 12345678
}
```

#### `POST /api/v1/accounts`

**Request Body:**
```json
{
  "callbackUrl": "myapp://wallet-callback",
  "metadata": {
    "appName": "MyApp",
    "description": "Get available accounts",
    "iconUrl": "https://myapp.com/icon.png"
  }
}
```

**Response:**
```json
{
  "requestId": "unique-request-id",
  "deeplinkUrl": "nearwallet://native-bridge?requestId=unique-request-id&callback=myapp://wallet-callback"
}
```

#### `GET /api/v1/accounts/{requestId}`

**Response:**
```json
{
  "accounts": [
    {
      "accountId": "user.near",
      "publicKey": "ed25519:..."
    }
  ]
}
```

#### `POST /api/v1/sign-message`

**Request Body:**
```json
{
  "message": "Hello, this is a test message",
  "recipient": "myapp.com",
  "nonce": "base64-encoded-32-byte-nonce",
  "callbackUrl": "myapp://wallet-callback",
  "state": "optional-state-for-auth",
  "metadata": {
    "appName": "MyApp",
    "description": "Sign message for authentication",
    "iconUrl": "https://myapp.com/icon.png"
  }
}
```

**Response:**
```json
{
  "requestId": "unique-request-id",
  "deeplinkUrl": "nearwallet://native-bridge?requestId=unique-request-id&callback=myapp://wallet-callback"
}
```

#### `GET /api/v1/sign-message/{requestId}`

**Response:**
```json
{
  "status": "approved",
  "accountId": "user.near",
  "publicKey": "ed25519:6TupyNrcHGTt5XRLmHTc2KGaiSbjhQi1KHtCXTgbcr4Y",
  "signature": "base64-encoded-signature",
  "state": "optional-state"
}
```

#### `POST /api/v1/sign-meta-transaction`

**Request Body:**
```json
{
  "delegateAction": {
    "senderId": "user.near",
    "receiverId": "relayer.near",
    "actions": [
      {
        "type": "FunctionCall",
        "params": {
          "methodName": "transfer",
          "args": "base64-encoded-args",
          "gas": "300000000000000",
          "deposit": "0"
        }
      }
    ],
    "nonce": 123456,
    "maxBlockHeight": 12345678,
    "publicKey": "ed25519:6TupyNrcHGTt5XRLmHTc2KGaiSbjhQi1KHtCXTgbcr4Y"
  },
  "signature": "base64-encoded-signature",
  "callbackUrl": "myapp://wallet-callback",
  "executeTransaction": true,
  "metadata": {
    "appName": "MyApp",
    "description": "Meta transaction for gasless transfer",
    "iconUrl": "https://myapp.com/icon.png"
  }
}
```

**Response:**
```json
{
  "requestId": "unique-request-id",
  "deeplinkUrl": "nearwallet://native-bridge?requestId=unique-request-id&callback=myapp://wallet-callback"
}
```

#### `GET /api/v1/sign-meta-transaction/{requestId}`

**Response:**
```json
{
  "status": "approved",
  "signedDelegateAction": "base64-encoded-signed-delegate-action",
  "transactionHash": "blockchain-transaction-hash",
  "blockHeight": 12345678
}
```

