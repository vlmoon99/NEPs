# Native Wallets Bridge

## Summary

This specification proposes a protocol for bridging NEAR wallets (native and web) with external applications, enabling seamless interaction via deeplinks and event-based payloads. The protocol is designed to be extensible, secure, and compatible with the Wallet Selector infrastructure, allowing future evolution and integration with existing NEAR wallet standards.

## Motivation

Currently, NEAR lacks a unified method for wallets to interact with external apps, limiting usability and feature access for dApps. By introducing a bridge protocol based on a single endpoint and event payloads, we enable:

- Secure session establishment between wallets and apps.
- Transaction signing and sending from any app via the user's wallet.
- Access to account information for richer dApp experiences.
- Universal connectivity using deeplinks or web messaging.
- Extensibility for future wallet features and standards.

## Specification

### Session Establishment

- The app initiates a connection by opening the wallet using a deeplink or web messaging, passing a payload with `type: "connect"` and a `callback_url`.
- The wallet prompts the user to approve the connection and select accounts.
- Upon approval, the wallet returns a payload with session details to the provided `callback_url`.

### Methods

All requests use the same endpoint:

```
nearwallet://bridge?payload=<url-encoded-json>
```

The payload must include a `type` field to specify the method, a `callback_url` for the response, and any required parameters.

#### Payload Types

- `connect`
- `signAndSendTransaction`
- `signAndSendTransactions`
- `getAccounts`
- `signIn`
- `signOut`

#### Example Payloads

**Connect**

Request:
```json
{
  "type": "connect",
  "dapp_url": "myapp://onSessionApproved",
  "app_name": "MyApp",
  "request_id": "xyz",
  "callback_url": "myapp://onSessionApproved"
}
```

Response (to `callback_url`):
```json
{
  "type": "connectResult",
  "session_key": "abc123",
  "accounts": [
    {
      "account_id": "myname.near",
      "public_key": "ed25519:..."
    }
  ]
}
```

**signAndSendTransaction**

Request:
```json
{
  "type": "signAndSendTransaction",
  "session_key": "abc123",
  "transaction": { /* transaction object */ },
  "request_id": "tx1",
  "callback_url": "myapp://onTxComplete"
}
```

Response (to `callback_url`):
```json
{
  "type": "transactionResult",
  "request_id": "tx1",
  "status": "success",
  "tx_hash": "XYZ"
}
```

**signAndSendTransactions**

Request:
```json
{
  "type": "signAndSendTransactions",
  "session_key": "abc123",
  "transactions": [ /* array of transaction objects */ ],
  "request_id": "batch1",
  "callback_url": "myapp://onMultiTxComplete"
}
```

Response (to `callback_url`):
```json
{
  "type": "multiTransactionResult",
  "request_id": "batch1",
  "status": "success",
  "tx_hashes": ["tx1", "tx2"]
}
```

**getAccounts**

Request:
```json
{
  "type": "getAccounts",
  "session_key": "abc123",
  "callback_url": "myapp://onAccounts"
}
```

Response (to `callback_url`):
```json
{
  "type": "accountsResult",
  "accounts": [
    {
      "account_id": "user.near",
      "public_key": "ed25519:..."
    }
  ]
}
```

**signIn**

Request:
```json
{
  "type": "signIn",
  "session_key": "abc123",
  "contract_id": "mycontract.near",
  "method_names": ["callA", "callB"],
  "callback_url": "myapp://onSignIn"
}
```

Response (to `callback_url`):
```json
{
  "type": "signInResult",
  "status": "success",
  "account_id": "user.near"
}
```

**signOut**

Request:
```json
{
  "type": "signOut",
  "session_key": "abc123",
  "callback_url": "myapp://onSignOut"
}
```

Response (to `callback_url`):
```json
{
  "type": "signOutResult",
  "status": "success"
}
```

### Transport

- Communication via deeplinks for native apps.
- Communication via web messaging (e.g., postMessage) for web wallets.
- All requests and responses use the same endpoint and event-based payloads.
- Session keys are used for authentication and message integrity.
- The `callback_url` parameter must be provided in every request and is used for delivering results to the originating application.

### Example Flow

1. App opens wallet via deeplink or web messaging, sending a `connect` payload with a `callback_url`.
2. Wallet displays session approval UI, user selects accounts.
3. Wallet returns session key and account info to the provided `callback_url`.
4. App requests transaction signing via a `signAndSendTransaction` payload with a `callback_url`.
5. Wallet signs the transaction and sends it to the NEAR blockchain.
6. Wallet returns the transaction result to the provided `callback_url`.

## Rationale

This protocol is designed to be flexible and extensible, using a single endpoint and event-based payloads. It supports both native and web wallets, and is compatible with the Wallet Selector layer. The use of a `callback_url` in every request ensures the native app receives the result of each operation, supporting robust communication and future extensibility.

## Call for Feedback

This is an initial draft. Feedback and suggestions from the NEAR developer community and infrastructure groups are welcome to refine and finalize the standard.