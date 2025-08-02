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
```
nearwallet://bridge/v1?payload=<url-encoded-json>
```

#### 6.1.2. Session Initiation Request
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
    "websocket_endpoint_preference": "wss://preferred-endpoint.com/ws"
  },
  "client_public_key": "base64-encoded-ephemeral-public-key",
  "callback_scheme": "myapp://bridge/v1",
  "request_id": "unique-request-identifier",
  "timestamp": 1645123456789,
  "signature": "base64-encoded-signature"
}
```

#### 6.1.3. Session Establishment Response
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

Wallet providers MUST implement the following backend functions:

**Function**: `registerSession(sessionInitRequest: SessionInitRequest)`
- **Purpose**: Register a new session with the wallet provider backend
- **Input**: Session initialization request from client
- **Output**: `{ sessionId: string, websocketEndpoint: string, walletPublicKey: string }`
- **Notes**: Generate session ID, validate request, return connection details

**Function**: `validateSession(sessionId: string, messageSignature: string)`
- **Purpose**: Validate session and message authenticity
- **Input**: 
  - `sessionId`: Session identifier
  - `messageSignature`: HMAC signature of message
- **Output**: `boolean` (session valid/invalid)
- **Notes**: Check session expiration, verify signature

**Function**: `broadcastEvent(sessionId: string, event: Event)`
- **Purpose**: Broadcast event to connected client
- **Input**: 
  - `sessionId`: Target session identifier
  - `event`: Event data to broadcast
- **Output**: `boolean` (event sent successfully)
- **Notes**: Encrypt event payload before sending

### 7.2. Client API Documentation Requirements

Wallet providers MUST provide the following information to developers:

#### 7.2.1. Connection Information
```json
{
  "wallet_provider_id": "provider.near",
  "supported_platforms": ["ios", "android", "web"],
  "websocket_endpoint": "wss://wallet-backend.provider.near/ws",
  "api_documentation_url": "https://provider.near/docs/api",
  "sdk_download_url": "https://provider.near/sdk"
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

### 7.4. Security Requirements

**Function**: `validateCertificate(certificate: string)`
- **Purpose**: Validate TLS certificate for secure connections
- **Input**: Certificate data
- **Output**: `boolean` (certificate valid/invalid)
- **Notes**: Check certificate chain and revocation status

**Function**: `encryptMessage(message: string, encryptionKey: string)`
- **Purpose**: Encrypt message payload for secure transmission
- **Input**: 
  - `message`: Plaintext message
  - `encryptionKey`: Base64-encoded encryption key
- **Output**: `{ nonce: string, ciphertext: string }`
- **Notes**: Use AES-256-GCM with random nonce

**Function**: `decryptMessage(encryptedData: object, encryptionKey: string)`
- **Purpose**: Decrypt message payload
- **Input**: 
  - `encryptedData`: `{ nonce: string, ciphertext: string }`
  - `encryptionKey`: Base64-encoded encryption key
- **Output**: `string` (decrypted message)
- **Notes**: Verify authentication tag during decryption

## 8. Client SDK Requirements

### 8.1. Core Interface Functions

Client SDKs MUST implement the following functions:

**Function**: `initializeSession(config: SessionConfig)`
- **Purpose**: Establish connection with wallet provider
- **Input**: 
  - `config.appId`: Application identifier
  - `config.appName`: Application name
  - `config.walletId`: Target wallet provider (optional)
  - `config.timeout`: Connection timeout in seconds (default: 30)
- **Output**: `{ sessionId: string, accounts: Account[], websocketEndpoint: string }`
- **Notes**: Generate key pair, send deeplink, establish WebSocket connection

**Function**: `closeSession()`
- **Purpose**: Terminate wallet connection
- **Input**: None
- **Output**: `boolean` (session closed successfully)
- **Notes**: Close WebSocket connection, cleanup session data

**Function**: `signAndSendTransaction(transaction: Transaction)`
- **Purpose**: Sign and submit transaction to blockchain
- **Input**: 
  - `transaction.receiverId`: Target account ID
  - `transaction.actions`: Array of transaction actions
- **Output**: `{ transactionId: string, status: string }`
- **Notes**: Send transaction request, return immediately, listen for events

**Function**: `getAccounts()`
- **Purpose**: Retrieve connected wallet accounts
- **Input**: None
- **Output**: `Account[]` (array of account information)
- **Notes**: Return account IDs, public keys, and balances

### 8.2. Event Handling Functions

**Function**: `addEventListener(eventType: string, callback: Function)`
- **Purpose**: Register event listener for wallet events
- **Input**: 
  - `eventType`: Event type (e.g., "transaction_completed", "account_updated")
  - `callback`: Function to handle event
- **Output**: None
- **Notes**: Store callback for event type

**Function**: `removeEventListener(eventType: string, callback: Function)`
- **Purpose**: Remove event listener
- **Input**: 
  - `eventType`: Event type
  - `callback`: Function to remove
- **Output**: None
- **Notes**: Remove callback from event type

**Function**: `isConnected()`
- **Purpose**: Check if wallet connection is active
- **Input**: None
- **Output**: `boolean` (connection status)
- **Notes**: Check WebSocket connection state

### 8.3. Error Handling

**Function**: `handleError(error: Error)`
- **Purpose**: Process and handle wallet connection errors
- **Input**: Error object with code and message
- **Output**: None
- **Notes**: Log error, notify user, attempt recovery if possible

**Common Error Types**:
- `WALLET_NOT_INSTALLED`: Wallet app not found on device
- `SESSION_EXPIRED`: Connection timeout or session expired
- `NETWORK_ERROR`: WebSocket connection failed
- `INVALID_TRANSACTION`: Transaction format or validation error
- `USER_REJECTED`: User rejected transaction in wallet

## 9. Message Format Specification

### 9.1. WebSocket Message Structure

All WebSocket messages MUST follow this format:

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
   - [ ] TLS 1.2+ with certificate validation
   - [ ] AES-256-GCM encryption
   - [ ] ECDH P-256 key exchange
   - [ ] HMAC-SHA256 message authentication
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

### 10.2. Client Application Integration

**Function**: `connectToWallet(walletProviderId: string)`
- **Purpose**: Connect to specific wallet provider
- **Input**: Wallet provider identifier
- **Output**: `{ connected: boolean, sessionId: string }`
- **Notes**: Initialize session with specified wallet

**Function**: `sendTransaction(transaction: Transaction)`
- **Purpose**: Send transaction to connected wallet
- **Input**: Transaction object
- **Output**: `{ transactionId: string, status: string }`
- **Notes**: Submit transaction and listen for completion events

### 10.3. Testing and Validation

**Function**: `testWalletConnection(walletProviderId: string)`
- **Purpose**: Test connectivity with wallet provider
- **Input**: Wallet provider identifier
- **Output**: `{ success: boolean, latency: number, errors: string[] }`
- **Notes**: Verify session establishment and basic communication

**Function**: `validateTransaction(transaction: Transaction)`
- **Purpose**: Validate transaction format before sending
- **Input**: Transaction object
- **Output**: `{ valid: boolean, errors: string[] }`
- **Notes**: Check transaction structure and required fields

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
