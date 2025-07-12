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

#### Standardized Request and Response Examples

All interactions between the native app and native wallet use a single deeplink endpoint:

```
nearwallet://bridge?payload=<url-encoded-json>
```

Every request from the native app MUST include a `callback_url` parameter, which is the app’s own scheme, e.g.:

```
myapp://bridge
```

The wallet parses the payload, executes the requested method, and returns the result to the app using the same callback URL and a URL-encoded JSON payload.

---

**Connect**

- Request (Native App → Native Wallet):

  ```
  nearwallet://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "connect",
    "app_name": "MyApp",
    "request_id": "xyz",
    "callback_url": "myapp://bridge"
  }
  ```

- Response (Native Wallet → Native App):

  ```
  myapp://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "connectResult",
    "request_id": "xyz",
    "session_key": "abc123",
    "accounts": [
      {
        "account_id": "myname.near",
        "public_key": "ed25519:..."
      }
    ]
  }
  ```

---

**signAndSendTransaction**

- Request:

  ```
  nearwallet://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "signAndSendTransaction",
    "session_key": "abc123",
    "transaction": { /* transaction object */ },
    "request_id": "tx1",
    "callback_url": "myapp://bridge"
  }
  ```

- Response:

  ```
  myapp://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "transactionResult",
    "request_id": "tx1",
    "status": "success",
    "tx_hash": "XYZ"
  }
  ```

---

**signAndSendTransactions**

- Request:

  ```
  nearwallet://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "signAndSendTransactions",
    "session_key": "abc123",
    "transactions": [ /* array of transaction objects */ ],
    "request_id": "batch1",
    "callback_url": "myapp://bridge"
  }
  ```

- Response:

  ```
  myapp://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "multiTransactionResult",
    "request_id": "batch1",
    "status": "success",
    "tx_hashes": ["tx1", "tx2"]
  }
  ```

---

**getAccounts**

- Request:

  ```
  nearwallet://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "getAccounts",
    "session_key": "abc123",
    "request_id": "acc1",
    "callback_url": "myapp://bridge"
  }
  ```

- Response:

  ```
  myapp://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "accountsResult",
    "request_id": "acc1",
    "accounts": [
      {
        "account_id": "user.near",
        "public_key": "ed25519:..."
      }
    ]
  }
  ```

---

**signIn**

- Request:

  ```
  nearwallet://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "signIn",
    "session_key": "abc123",
    "contract_id": "mycontract.near",
    "method_names": ["callA", "callB"],
    "request_id": "signin1",
    "callback_url": "myapp://bridge"
  }
  ```

- Response:

  ```
  myapp://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "signInResult",
    "request_id": "signin1",
    "status": "success",
    "account_id": "user.near"
  }
  ```

---

**signOut**

- Request:

  ```
  nearwallet://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "signOut",
    "session_key": "abc123",
    "request_id": "signout1",
    "callback_url": "myapp://bridge"
  }
  ```

- Response:

  ```
  myapp://bridge?payload=<url-encoded-json>
  ```

  Payload:
  ```json
  {
    "type": "signOutResult",
    "request_id": "signout1",
    "status": "success"
  }
  ```
