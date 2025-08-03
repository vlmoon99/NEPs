# Native Wallets Bridge v1.0 Specification

## Abstract

This specification defines a protocol for secure communication between NEAR blockchain applications (native and web) and native wallets through a backend-driven architecture. The protocol establishes secure sessions via deeplinks, enables real-time communication through encrypted WebSocket connections, and provides standardized client SDK implementations across iOS, Android, and Web platforms.

This specification provides:
- Asynchronous transaction processing with real-time updates
- Standardized encryption and authentication protocols
- Cross-platform SDK consistency
- Enhanced security through backend-mediated communication
- Event-driven architecture for responsive user experiences

## Table of Contents

1. [Introduction](#1-introduction)
2. [Definitions](#2-definitions)
3. [Key Words](#3-key-words)
4. [Architecture Overview](#4-architecture-overview)
5. [Cryptographic Requirements](#5-cryptographic-requirements)
6. [Session Establishment](#6-session-establishment)
7. [Wallet Provider Requirements](#7-wallet-provider-requirements)
8. [Client SDK Requirements](#8-client-sdk-requirements)
9. [Message Format Specification](#9-message-format-specification)
10. [Implementation Guidelines](#10-implementation-guidelines)
11. [Conclusion](#11-conclusion)

## 1. Introduction

### 1.1. Purpose

The Native Wallets Bridge v1.0 protocol enables secure, efficient, and user-friendly interaction between NEAR blockchain applications and native wallets through a sophisticated backend-driven architecture. This specification addresses the limitations of direct deeplink communication by introducing persistent WebSocket connections managed by wallet provider backends.

### 1.2. Scope

This specification covers:
- Session establishment via deeplinks
- Cryptographic session key management
- WebSocket-based real-time communication
- Message format standardization
- Backend architecture requirements
- Client SDK specifications
- Security and compliance requirements


## 2. Definitions

### 2.1. Core Entities

- **Native App**: A mobile or desktop application requesting wallet services
- **Web App**: A browser-based application requesting wallet services
- **Native Wallet**: A mobile wallet application capable of managing NEAR accounts
- **Wallet Provider**: The organization operating the wallet and its backend infrastructure
- **Wallet Provider Backend (WPB)**: The server infrastructure managing wallet sessions and communications
- **Bridge Session**: A secure communication session between a client application and wallet
- **Session Key**: Cryptographic key used for end-to-end encryption within a bridge session
- **Client Connector**: SDK implementation enabling applications to interact with wallet providers

### 2.2. Protocol Components

- **Session Establishment Protocol (SEP)**: Deeplink-based session initialization
- **Secure Communication Channel (SCC)**: WebSocket-based encrypted communication
- **Event Broadcasting System (EBS)**: Real-time event delivery mechanism
- **Cryptographic Service Provider (CSP)**: Encryption/decryption service interface

## 3. Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

The key word "CONDITIONAL" is to be interpreted as follows:
**CONDITIONAL**: The usage of an item is dependent on the usage of other items and is further qualified under which conditions the item is REQUIRED or RECOMMENDED.



## 4. Architecture Overview

### 4.1. System Components

```
┌─────────────────┐    ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈
│                 │    ┊                                    Wallet Provider Backend (WPB)                                    ┊
│  Native/Web App │    ┊  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐                ┊
│                 │    ┊  │   Session       │  │   WebSocket     │  │   Event         │  │   Crypto        │                ┊
│  ┌─────────────┐│    ┊  │   Manager       │  │   Gateway       │  │   Broadcasting  │  │   Service       │                ┊
│  │   Client    ││    ┊  │                 │  │                 │  │   System        │  │   Provider      │                ┊
│  │  Connector  ││◄───┊──┤   - Session     │  │   - Connection  │  │                 │  │                 │                ┊
│  │             ││    ┊  │     Validation  │  │     Management  │  │   - Event       │  │   - Key         │                ┊
│  │   - Session ││    ┊  │   - Key         │  │   - Message     │  │     Routing     │  │     Generation  │                ┊
│  │     Manager ││    ┊  │     Management  │  │     Routing     │  │   - Delivery    │  │   - Encryption  │                ┊
│  │   - WS      ││    ┊  │   - Auth        │  │   - Event       │  │     Tracking    │  │   - Decryption  │                ┊
│  │     Client  ││    ┊  │     Validation  │  │     Dispatch    │  │                 │  │                 │                ┊
│  │   - Crypto  ││    ┊  └─────────────────┘  └─────────────────┘  └─────────────────┘  └─────────────────┘                ┊
│  └─────────────┘│    ┊                                                                                                      ┊
└─────────────────┘    ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈
         │                                                            │
         │                                                            │
         │                        ┌─────────────────┐                 │
         └────────deeplink────────►│                 │◄────websocket───┘
                                  │  Native Wallet  │
                                  │                 │
                                  │  - Session      │
                                  │    Handler      │
                                  │  - Transaction  │
                                  │    Processor    │
                                  │  - Event        │
                                  │    Publisher    │
                                  └─────────────────┘
```

### 4.2. Communication Flows

#### 4.2.1. Session Establishment Flow
1. **Deeplink Initiation**: Client app triggers wallet via deeplink
2. **Session Registration**: Wallet registers session with WPB
3. **Key Exchange**: Cryptographic session keys are generated and exchanged
4. **WebSocket Connection**: Persistent connection established between client and WPB
5. **Session Confirmation**: Both parties confirm successful session establishment

#### 4.2.2. Transaction Processing Flow
1. **Transaction Request**: Client sends transaction request via WebSocket
2. **Wallet Notification**: WPB notifies wallet of pending transaction
3. **User Interaction**: Wallet presents transaction to user for approval
4. **Immediate Response**: Wallet returns to client app immediately after user action
5. **Asynchronous Processing**: Transaction is processed on blockchain
6. **Event Broadcasting**: WPB broadcasts transaction updates via WebSocket

## 5. Cryptographic Requirements

### 5.1. Supported Encryption Methods

#### 5.1.1. AES-256-GCM (RECOMMENDED)
- **Key Length**: 256 bits
- **IV Length**: 96 bits (12 bytes)
- **Tag Length**: 128 bits (16 bytes)
- **Usage**: Default encryption for all wallet providers

#### 5.1.2. ChaCha20-Poly1305 (RECOMMENDED)
- **Key Length**: 256 bits
- **Nonce Length**: 96 bits (12 bytes)
- **Tag Length**: 128 bits (16 bytes)
- **Usage**: Alternative high-performance encryption

#### 5.1.3. Custom Encryption (OPTIONAL)
- Wallet providers MAY implement custom encryption methods
- MUST provide WASM bindings for cross-platform compatibility
- MUST achieve equivalent security level (≥128-bit security)
- MUST be documented and auditable

### 5.2. Key Exchange Requirements

#### 5.2.1. Elliptic Curve Diffie-Hellman (RECOMMENDED)
- **Curves**: P-256 (REQUIRED), P-384 (RECOMMENDED), X25519 (OPTIONAL)
- **Key Derivation**: HKDF-SHA256
- **Ephemeral Keys**: New key pair for each session

#### 5.2.2. Random Number Generation
- MUST use cryptographically secure random number generators
- Entropy MUST be sufficient for 128-bit security level
- Platform-specific secure random implementations REQUIRED



## 6. Session Establishment

### 6.1. Session Initialization Protocol

#### 6.1.1. Deeplink Structure

Wallet providers are RECOMMENDED to use this deeplink structure :

```
nearwallet://bridge/v1?payload=<url-encoded-json>
```

#### 6.1.2. Session Initiation Request

Wallet providers MAY use this session init req structure :

```json
{
  "version": "1.0",
  "type": "session_init",
  "app_id": "com.example.myapp",
  "app_name": "MyApp",
  "platform": "ios|android|web",
  "session_config": {
    "encryption_method": "aes256_gcm|chacha20_poly1305|custom",
    "key_exchange_method": "ecdh_p256|ecdh_p384|x25519",
    "websocket_endpoint": "wss://preferred-endpoint.com/ws"
  },
  "client_public_key": "base64-encoded-ephemeral-public-key",
  "callback_scheme": "myapp://bridge/v1",
  "request_id": "unique-request-identifier",
  "timestamp": 1645123456789,
  "signature": "base64-encoded-signature"
}
```

#### 6.1.3. Session Establishment Response

Wallet providers MAY use this session init resp structure :

```json
{
  "version": "1.0",
  "type": "session_established",
  "request_id": "unique-request-identifier",
  "session_id": "unique-session-identifier",
  "wallet_provider_id": "provider.near",
  "websocket_endpoint": "wss://wallet-backend.provider.near/ws",
  "session_config": {
    "encryption_method": "aes256_gcm",
    "key_exchange_method": "ecdh_p256",
    "session_duration": 3600,
    "heartbeat_interval": 30
  },
  "wallet_public_key": "base64-encoded-wallet-public-key",
  "shared_secret": "base64-encoded-derived-shared-secret",
  "accounts": [
    {
      "account_id": "user.near",
      "public_key": "ed25519:...",
      "permissions": ["view_account", "sign_transactions"]
    }
  ],
  "timestamp": 1645123456790,
  "signature": "base64-encoded-signature"
}
```

## 7. Wallet Provider Requirements

### 7.1. Backend Infrastructure

Wallet providers MAY implement the following backend functions: 

**Function**: `registerSession(sessionInitRequest: SessionInitRequest)`
- **Purpose**: Register a new session with the wallet provider backend
- **Input**: 
  ```typescript
  {
    "version": "1.0",
    "type": "session_init",
    "app_id": "com.example.myapp",
    "app_name": "MyApp",
    "platform": "ios|android|web",
    "session_config": {
      "encryption_method": "aes256_gcm|chacha20_poly1305|custom",
      "key_exchange_method": "ecdh_p256|ecdh_p384|x25519",
      "websocket_endpoint": "wss://preferred-endpoint.com/ws"
    },
    "client_public_key": "base64-encoded-ephemeral-public-key",
    "callback_scheme": "myapp://bridge/v1",
    "request_id": "unique-request-identifier",
    "timestamp": 1645123456789,
    "signature": "base64-encoded-signature"
  }
  ```
- **Output**: 
  ```typescript
  {
    "version": "1.0",
    "type": "session_established",
    "request_id": "unique-request-identifier",
    "session_id": "unique-session-identifier",
    "wallet_provider_id": "provider.near",
    "websocket_endpoint": "wss://wallet-backend.provider.near/ws",
    "session_config": {
      "encryption_method": "aes256_gcm",
      "key_exchange_method": "ecdh_p256",
      "session_duration": 3600,
      "heartbeat_interval": 30
    },
    "wallet_public_key": "base64-encoded-wallet-public-key",
    "shared_secret": "base64-encoded-derived-shared-secret",
    "accounts": [
      {
        "account_id": "user.near",
        "public_key": "ed25519:...",
        "permissions": ["view_account", "sign_transactions"]
      }
    ],
    "timestamp": 1645123456790,
    "signature": "base64-encoded-signature"
  }
  ```
- **Logic**: Validate request signature, generate unique session ID, create WebSocket endpoint, store session data with expiration, return connection details

**Function**: `validateSession(sessionId: string, messageSignature: string)`
- **Purpose**: Validate session and message authenticity
- **Input**: 
  ```typescript
  {
    sessionId: "sess_def456",
    messageSignature: "hmac_sha256_signature_base64"
  }
  ```
- **Output**: 
  ```typescript
  {
    valid: true,
    accountId: "user.near",
    expiresAt: 1691238167890,
    permissions: ["sign_transactions", "view_account"]
  }
  ```
- **Logic**: Check session exists and not expired, verify HMAC signature using session key, return validation result with session details

**Function**: `broadcastEvent(sessionId: string, event: Event)`
- **Purpose**: Broadcast event to connected client
- **Input**: 
  ```typescript
  {
    sessionId: "sess_def456",
    event: {
      type: "transaction_completed",
      data: {
        transactionId: "tx_ghi789",
        status: "success",
        blockHeight: 150000001
      },
      timestamp: 1691234567890
    }
  }
  ```
- **Output**: 
  ```typescript
  {
    sent: true,
    messageId: "msg_jkl012",
    queuedAt: 1691234567895
  }
  ```
- **Logic**: Encrypt event payload using session key, send via WebSocket to client, return delivery confirmation and message ID

**Function**: `getSessionInfo(sessionId: string)`
- **Purpose**: Retrieve session information and status
- **Input**: 
  ```typescript
  "sess_def456"
  ```
- **Output**: 
  ```typescript
  {
    sessionId: "sess_def456",
    appId: "myapp.com",
    accountId: "user.near",
    status: "active",
    createdAt: 1691234567890,
    expiresAt: 1691238167890,
    lastActivity: 1691234600000
  }
  ```
- **Logic**: Lookup session in storage, return session metadata including status and timing information

**Function**: `terminateSession(sessionId: string, reason?: string)`
- **Purpose**: Terminate active session and cleanup resources
- **Input**: 
  ```typescript
  {
    sessionId: "sess_def456",
    reason: "user_logout" // optional: "timeout", "error", "user_logout"
  }
  ```
- **Output**: 
  ```typescript
  {
    terminated: true,
    finalEvent: {
      type: "session_terminated",
      reason: "user_logout",
      timestamp: 1691234567890
    }
  }
  ```
- **Logic**: Close WebSocket connection, clear session data from storage, send final termination event to client


### 7.2. Client API Documentation Requirements

Wallet providers MUST provide the following information to developers:

#### 7.2.1. Connection Information
```json
{
  "wallet_provider_id": "provider.near",
  "supported_platforms": ["ios", "android", "web"],
  "websocket_endpoint": "wss://wallet-backend.provider.near/ws",
  "api_documentation_url": "https://provider.near/docs/api"
}
```

#### 7.2.2. Supported Features
```json
{
  "encryption_methods": ["aes256_gcm", "chacha20_poly1305"],
  "key_exchange_methods": ["ecdh_p256", "x25519"],
  "supported_transactions": ["transfer", "function_call", "stake", "unstake"],
  "max_session_duration": 3600,
  "heartbeat_interval": 30
}
```

#### 7.2.3. Error Codes and Messages
```json
{
  "error_codes": {
    "SESSION_EXPIRED": "Session has expired, please reconnect",
    "INVALID_SIGNATURE": "Message signature verification failed",
    "INSUFFICIENT_BALANCE": "Account balance insufficient for transaction",
    "NETWORK_ERROR": "Network connection error, please retry",
    "WALLET_NOT_AVAILABLE": "Wallet application not installed"
  }
}
```

### 7.3. SDK Distribution

Wallet providers MUST provide:

1. **Cross-platform SDKs**: iOS, Android, and Web implementations
2. **Documentation**: Complete API reference and integration guides
3. **Examples**: Sample applications demonstrating integration
4. **Testing Tools**: Utilities for testing wallet connectivity
5. **Support**: Developer support channels and resources


## 8. Client SDK Requirements

### 8.1. Core Interface Functions

Wallet providers MUST implement the following client functions. They may use their own data structures, but the function signatures MUST remain the same. These functions SHOULD optionally support two types of usage: via WebSocket and via raw deeplinks (except for event listeners). When using deeplinks, the URL MUST be less than or equal to 2000 characters. If this limit is exceeded, only the WebSocket method MUST be used :

**Function**: `initializeSession(config: SessionConfig)`
- **Purpose**: Establish connection with wallet provider
- **Input**: 
  ```typescript
  {
    appId: "myapp.com",
    appName: "My DApp",
    walletId: "mynearwallet", 
  }
  ```
- **Output**: 
  ```typescript
  {
    sessionId: "sess_abc123",
    accounts: [
      { accountId: "user.near", publicKey: "ed25519:..." }
    ],
    websocketEndpoint: "wss://wallet.near.org/ws"
  }
  ```
- **Logic**: Generate key pair, send deeplink to wallet, establish WebSocket connection, return session details

**Function**: `closeSession()`
- **Purpose**: Terminate wallet connection
- **Input**: None
- **Output**: 
  ```typescript
  true // boolean: session closed successfully
  ```
- **Logic**: Close WebSocket connection, cleanup session data, notify wallet of disconnection

**Function**: `signAndSendTransaction(transaction: Transaction)`
- **Purpose**: Sign and submit transaction to blockchain
- **Input**: 
  ```typescript
  {
    receiverId: "contract.near",
    actions: [
      {
        type: "FunctionCall",
        params: {
          methodName: "transfer",
          args: { amount: "1000000000000000000000000" },
          gas: "30000000000000",
          deposit: "0"
        }
      }
    ]
  }
  ```
- **Output**: 
  ```typescript
  {
    transactionId: "tx_def456",
    status: "pending"
  }
  ```
- **Logic**: Send transaction request to wallet, wallet signs and broadcasts to network, return transaction ID immediately

**Function**: `sendTransaction(transaction: Transaction)`
- **Purpose**: Send pre-signed transaction to blockchain
- **Input**: 
  ```typescript
  {
    signedTransaction: "encoded_signed_transaction",
    receiverId: "contract.near"
  }
  ```
- **Output**: 
  ```typescript
  {
    transactionId: "tx_ghi789",
    status: "pending"
  }
  ```
- **Logic**: Submit already signed transaction to network, return transaction ID and listen for completion events

**Function**: `signTransaction(transaction: Transaction)`
- **Purpose**: Sign transaction without sending it to the network
- **Input**: 
  ```typescript
  {
    receiverId: "contract.near",
    actions: [
      {
        type: "Transfer",
        params: { deposit: "1000000000000000000000000" }
      }
    ]
  }
  ```
- **Output**: 
  ```typescript
  {
    signedTransaction: "encoded_signed_transaction",
    signature: "ed25519:signature",
    publicKey: "ed25519:public_key"
  }
  ```
- **Logic**: Request wallet to sign transaction, return signed transaction data without broadcasting to network

**Function**: `signDelegateAction(delegateAction: DelegateAction)`
- **Purpose**: Sign meta-transaction (NEP-366) without relaying
- **Input**: 
  ```typescript
  {
    sender_id: "user.near",
    receiver_id: "contract.near",
    actions: [
      {
        type: "FunctionCall",
        params: {
          methodName: "mint_nft",
          args: { token_id: "123" },
          gas: "30000000000000",
          deposit: "0"
        }
      }
    ],
    nonce: 12345,
    max_block_height: 150000000,
    public_key: "ed25519:user_public_key"
  }
  ```
- **Output**: 
  ```typescript
  {
    signedDelegateAction: {
      delegate_action: { /* input data */ },
      signature: "ed25519:delegate_signature"
    },
    signature: "ed25519:delegate_signature"
  }
  ```
- **Logic**: Wallet signs DelegateAction structure, returns signed meta-transaction that application can submit via chosen relayer

**Function**: `signMessage(params: SignMessageParams)`
- **Purpose**: Sign arbitrary message for authentication (NEP-413)
- **Input**: 
  ```typescript
  {
    message: "Login to MyApp",
    recipient: "myapp.com",
    nonce: new Uint8Array(32),
    callbackUrl: "https://myapp.com/auth",
    state: "auth_state_token"
  }
  ```
- **Output**: 
  ```typescript
  {
    accountId: "user.near",
    publicKey: "ed25519:user_public_key",
    signature: "encoded_signature",
    state: "auth_state_token"
  }
  ```
- **Logic**: Wallet signs message with NEP-413 format (prepends tag, hashes with SHA256), returns signature for off-chain authentication

**Function**: `verifySignature(signature: string, message: string, publicKey: string)`
- **Purpose**: Verify signature locally without network calls
- **Input**: 
  ```typescript
  {
    signature: "encoded_signature",
    message: "original_message_or_hash",
    publicKey: "ed25519:public_key"
  }
  ```
- **Output**: 
  ```typescript
  {
    valid: true,
    error: null
  }
  ```
- **Logic**: Perform local cryptographic verification of signature against message and public key

**Function**: `getAccountInfo(accountId?: string)`
- **Purpose**: Get account information from connected wallet
- **Input**: 
  ```typescript
  "user.near"
  ```
- **Output**: 
  ```typescript
  {
    accountId: "user.near",
    publicKeys: ["ed25519:key1", "ed25519:key2"],
    balance: "1500000000000000000000000"
  }
  ```
- **Logic**: Query wallet for account details, return account information needed for transaction preparation

**Function**: `getNetworkInfo()`
- **Purpose**: Get current network configuration
- **Input**: None
- **Output**: 
  ```typescript
  {
    networkId: "mainnet",
    nodeUrl: "https://rpc.mainnet.near.org",
    blockHeight: 150000000
  }
  ```
- **Logic**: Retrieve current network configuration from wallet, used for setting max_block_height in DelegateActions

**Function**: `addEventListener(eventType: string, callback: Function)`
- **Purpose**: Register event listener for wallet events
- **Input**: 
  ```typescript
  {
    eventType: "transaction_completed",
    callback: (event) => console.log("Transaction done:", event.transactionId)
  }
  ```
- **Output**: None
- **Logic**: Store callback function for specified event type, invoke when events are received via WebSocket

**Function**: `removeEventListener(eventType: string, callback: Function)`
- **Purpose**: Remove event listener
- **Input**: 
  ```typescript
  {
    eventType: "transaction_completed",
    callback: previouslyRegisteredFunction
  }
  ```
- **Output**: None
- **Logic**: Remove specific callback function from event type listeners

**Function**: `isConnected()`
- **Purpose**: Check if wallet connection is active
- **Input**: None
- **Output**: 
  ```typescript
  true // boolean: connection status
  ```
- **Logic**: Check WebSocket connection state and session validity

**Function**: `handleError(error: Error)`
- **Purpose**: Process and handle wallet connection errors
- **Input**: 
  ```typescript
  {
    code: "WALLET_NOT_FOUND",
    message: "Wallet application not installed",
    details: { walletId: "mynearwallet" }
  }
  ```
- **Output**: None
- **Logic**: Log error details, notify user with appropriate message, attempt automatic recovery if possible (e.g., reconnection)

**Common Error Types**:
- `WALLET_NOT_INSTALLED`: Wallet app not found on device
- `SESSION_EXPIRED`: Connection timeout or session expired
- `NETWORK_ERROR`: WebSocket connection failed
- `INVALID_TRANSACTION`: Transaction format or validation error
- `USER_REJECTED`: User rejected transaction in wallet

### 8.2. Cryptographic Interface Requirements

Wallet providers are RECOMMENDED to implement the following cryptographic functions. While the specific cryptographic algorithms and implementations are left to each provider’s discretion, they MUST implement a function that securely encrypts the session and ensures that the session is valid and the user is verified. The interface contract for this function MUST be maintained :

**Function**: `encryptMessage(message: string, encryptionKey: string)`
- **Purpose**: Encrypt message payload for secure transmission between client and wallet
- **Input**: 
  ```typescript
  {
    message: string, // Plain text message to encrypt
    encryptionKey: string // Encryption key in provider-specific format
  }
  ```
- **Output**: 
  ```typescript
  {
    encryptedPayload: string, // Encrypted message in provider-specific format
    metadata: object // Any additional data needed for decryption (e.g., nonce, IV)
  }
  ```
- **Logic**: Encrypt message using wallet provider's chosen encryption method, return encrypted payload with necessary metadata for decryption

**Function**: `decryptMessage(encryptedData: object, encryptionKey: string)`
- **Purpose**: Decrypt message payload from client
- **Input**: 
  ```typescript
  {
    encryptedData: {
      encryptedPayload: string, // Encrypted message
      metadata: object // Decryption metadata from encryptMessage
    },
    encryptionKey: string // Decryption key in provider-specific format
  }
  ```
- **Output**: 
  ```typescript
  string // Decrypted plain text message
  ```
- **Logic**: Extract encrypted payload and metadata, decrypt using wallet provider's method, verify message integrity, return plain text

**Function**: `generateSessionKeys()`
- **Purpose**: Generate cryptographic key material for secure session
- **Input**: None
- **Output**: 
  ```typescript
  {
    publicKey: string, // Public key in provider-specific format
    privateKey: string, // Private key in provider-specific format  
    keyMetadata: object // Additional key information (algorithm, curve, etc.)
  }
  ```
- **Logic**: Generate cryptographic key pair using wallet provider's preferred algorithm and parameters

**Function**: `establishSharedSecret(privateKey: string, peerPublicKey: string)`
- **Purpose**: Establish shared encryption key with communication peer
- **Input**: 
  ```typescript
  {
    privateKey: string, // Own private key
    peerPublicKey: string // Peer's public key
  }
  ```
- **Output**: 
  ```typescript
  {
    sharedSecret: string, // Derived shared secret
    encryptionKey: string, // Key for message encryption/decryption
    keyDerivationInfo: object // Metadata about key derivation method
  }
  ```
- **Logic**: Perform key exchange using wallet provider's chosen method, derive encryption keys from shared secret

**Function**: `signMessage(message: string, privateKey: string)`
- **Purpose**: Create cryptographic signature for message authentication
- **Input**: 
  ```typescript
  {
    message: string, // Message to sign
    privateKey: string // Signing key
  }
  ```
- **Output**: 
  ```typescript
  {
    signature: string, // Digital signature
    algorithm: string, // Signature algorithm used
    publicKey: string // Corresponding public key for verification
  }
  ```
- **Logic**: Create digital signature using wallet provider's signature algorithm, return signature with verification information

**Function**: `verifySignature(message: string, signature: string, publicKey: string)`
- **Purpose**: Verify message signature authenticity
- **Input**: 
  ```typescript
  {
    message: string, // Original message
    signature: string, // Signature to verify
    publicKey: string // Signer's public key
  }
  ```
- **Output**: 
  ```typescript
  {
    valid: boolean, // Signature verification result
    algorithm: string, // Algorithm used for verification
    details: object // Additional verification information
  }
  ```
- **Logic**: Verify signature using wallet provider's verification method, return validation result

### 8.3. Cryptographic Implementation Guidelines

#### Security Requirements
- All cryptographic operations MUST use industry-standard, well-vetted algorithms
- Key generation MUST use cryptographically secure random number generators
- Encryption MUST provide both confidentiality and authentication
- Key exchange MUST provide forward secrecy where possible

#### Algorithm Flexibility
- Wallet providers MAY choose their preferred cryptographic algorithms
- Common acceptable choices include:
  - **Symmetric Encryption**: AES-256-GCM, ChaCha20-Poly1305, XSalsa20-Poly1305
  - **Key Exchange**: ECDH (P-256, P-384), X25519, X448
  - **Signatures**: Ed25519, ECDSA (P-256), RSA-PSS
  - **Key Derivation**: HKDF, PBKDF2, Argon2

#### Interoperability
- Wallet providers MUST document their chosen cryptographic methods
- Client applications MUST support the cryptographic methods of target wallets
- Cross-wallet compatibility is achieved through standardized interface contracts, not algorithm standardization

#### Migration and Upgrades
- Wallet providers SHOULD support algorithm negotiation for future upgrades
- Cryptographic parameters SHOULD be versioned for backward compatibility
- Migration paths SHOULD be planned for deprecated algorithms

### 8.4. Security Considerations

#### Key Management
- Private keys MUST be stored securely and never transmitted
- Session keys SHOULD be ephemeral and rotated regularly
- Key derivation MUST be deterministic and reproducible

#### Message Protection
- All sensitive messages MUST be encrypted in transit
- Message integrity MUST be verified before processing
- Replay protection SHOULD be implemented using nonces or timestamps

#### Implementation Security
- Cryptographic implementations SHOULD be reviewed by security experts
- Side-channel attacks MUST be considered in implementation
- Constant-time operations SHOULD be used where applicable

## 9. Message Format Specification

### 9.1. WebSocket Message Structure

All WebSocket messages are RECOMMENDED to follow this format:

```json
{
  "message_id": "unique-message-id",
  "session_id": "session-identifier",
  "type": "message_type",
  "timestamp": 1645123456789,
  "payload": {
    "nonce": "base64-encoded-nonce",
    "ciphertext": "base64-encoded-encrypted-data"
  },
  "signature": "base64-encoded-hmac-signature"
}
```


## 10. Implementation Guidelines

### 10.1. Wallet Provider Implementation Checklist

1. **Backend Services**:
   - [ ] Session management service
   - [ ] WebSocket gateway
   - [ ] Event broadcasting system
   - [ ] Cryptographic service provider

2. **Security Implementation**:
   - [ ] Certificate validation
   - [ ] Messages encryption
   - [ ] Key exchange
   - [ ] Message authentication
   - [ ] Session timeout and cleanup

3. **Client SDK Development**:
   - [ ] Cross-platform SDKs (iOS, Android, Web)
   - [ ] Complete API documentation
   - [ ] Sample applications
   - [ ] Error handling and recovery
   - [ ] Testing utilities

4. **Documentation Requirements**:
   - [ ] API reference documentation
   - [ ] Integration guides
   - [ ] Security best practices
   - [ ] Troubleshooting guides

## 11. Conclusion

This specification defines the Native Wallets Bridge v1.0 protocol for secure communication between NEAR applications and native wallets. The protocol provides:

- **Secure Communication**: End-to-end encryption with standardized cryptographic protocols
- **Cross-platform Support**: Unified SDK implementations across iOS, Android, and Web
- **Real-time Updates**: Asynchronous transaction processing with event-driven architecture
- **Developer-friendly**: Simple function interfaces with clear input/output specifications
- **Extensible Design**: Support for custom encryption methods and wallet-specific features

Wallet providers implementing this specification MUST provide complete SDKs, documentation, and developer support to enable seamless integration with NEAR applications.

---

**Document Version**: 1.0.0  
**Last Updated**: 2025-08-02  
**Status**: Draft Specification  
