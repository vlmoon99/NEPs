# Native Wallets Bridge

## Summary

This spec proposes a standard protocol for bridging NEAR native wallets with other native apps and web applications, enabling seamless interaction via deeplinks and session keys. The goal is to provide a secure, universal, and extensible way for apps to connect to NEAR native wallets, sign transactions, similar to WalletConnect v2, but designed specifically for NEAR's ecosystem.

## Motivation

Currently, NEAR does not support a standardized method for native wallets to interact with external apps. This limits the usability of NEAR wallets in mobile environments, and restricts dApps from leveraging native wallet features. By introducing a bridge protocol, we enable:

- Secure session establishment between wallets and apps.
- Transaction signing and sending from any app via the user's wallet.
- Access to account information for richer dApp experiences.
- Universal connectivity using deeplinks.

## Specification

### Session Establishment

- The app starts a connection by opening the wallet with a deeplink.
- The wallet asks the user to approve the connection and pick which accounts to use.
- After approval, the wallet creates a session key and sends it back to the app so they can talk

### Methods

- **signAndSendTransaction**: Sign and send a single transaction to the NEAR blockchain.
- **signAndSendTransactions**: Sign and send multiple transactions to the NEAR blockchain.
- **getAccounts**: Get account info (accountId, publicKey) for the session.
- **signIn**: Add FunctionCall access keys for dApp contracts.
- **signOut**: Remove FunctionCall access keys.

#### Deeplink URL Format and Expected Behavior

#### connect

App → Wallet

```
nearwallet://connect?dapp_url=myapp://onSessionApproved&app_name=MyApp&request_id=xyz
```

Wallet → App

```
myapp://onSessionApproved?session_key=abc123&account_id=myname.near&public_key=ed25519:...
```

#### signAndSendTransaction

App → Wallet

```
nearwallet://signAndSendTransaction?session_key=abc123&transaction=<url-encoded-json>&callback_url=myapp://onTxComplete&request_id=tx1
```

Wallet → App

```
myapp://onTxComplete?request_id=tx1&status=success&tx_hash=XYZ
```

#### signAndSendTransactions

App → Wallet

```
nearwallet://signAndSendTransactions?session_key=abc123&transactions=<url-encoded-json-array>&callback_url=myapp://onMultiTxComplete&request_id=batch1
```

Wallet → App

```
myapp://onMultiTxComplete?request_id=batch1&status=success&tx_hashes=[tx1,tx2]
```

#### getAccounts

App → Wallet

```
nearwallet://getAccounts?session_key=abc123&callback_url=myapp://onAccounts
```

Wallet → App

```
myapp://onAccounts?accounts=[{"account_id":"user.near","public_key":"ed25519:..."}]
```

#### signIn

App → Wallet

```
nearwallet://signIn?session_key=abc123&contract_id=mycontract.near&method_names=callA,callB&callback_url=myapp://onSignIn
```

Wallet → App

```
myapp://onSignIn?status=success&account_id=user.near
```

#### signOut

App → Wallet

```
nearwallet://signOut?session_key=abc123&callback_url=myapp://onSignOut
```

Wallet → App

```
myapp://onSignOut?status=success
```

### Transport

- Communication via deeplinks.
- Session keys used for authentication and message integrity.

### Example Flow

1. App opens wallet via deeplink, requesting a session.
2. Wallet displays session approval UI, user selects accounts.
3. Wallet returns session key and account info to app.
4. App requests transaction signing and sending via deeplink, referencing session key.
5. Wallet signs the transaction and sends it to the NEAR blockchain.
6. Wallet returns the transaction result

## Rationale

This standard is inspired by WalletConnect v2, but tailored for NEAR's native wallet architecture and ecosystem needs. It prioritizes security, user experience, and cross-platform compatibility.

## Call for Feedback

This is an initial draft. Feedback and suggestions from the NEAR developer community are welcome to refine and finalize the standard.