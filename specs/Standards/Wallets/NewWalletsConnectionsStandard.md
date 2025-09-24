## Summary

Enhanced wallet connection standards that extend the existing NEAR Injected Wallet and Bridge Wallet specifications to support universal access patterns, including native mobile integration and simplified authentication mechanisms for mass adoption.

## Motivation

Current wallet connection patterns limit user accessibility across different platforms and devices. Users often face friction when:
- Switching between web and mobile contexts
- Accessing dApps from devices without their primary wallet
- Onboarding new users who find current wallet setup processes complex

This proposal introduces two complementary approaches to make wallet connections more universal and user-friendly while maintaining security and compatibility with existing NEAR wallet standards.

Universal Cross-Platform Connection

### Overview
This approach extends the existing wallet connection flow to provide seamless integration between web dApps, mobile dApps, and native wallet applications, regardless of the user's current platform.

### Core Concept
Wallet providers must offer dual connection options:
1. **Direct Connection**: Traditional web-based or injected wallet connection
2. **Cross-Platform Connection**: Mobile-native wallet/Remote Web approval for web/native dApps

### Flow Architecture

#### Enhanced signIn Method
The standard `signIn` method is extended to include platform preference:

```typescript
interface EnhancedSignInParams extends SignInParams {
  connectionType: 'direct' | 'cross-platform';
  platformPreference?: 'web' | 'native';
}
```

#### Cross-Platform Connection Flow
1. **Connection Initiation**
   - User visits web dApp without wallet extension installed
   - dApp detects available wallet providers and their platform capabilities
   - User selects preferred wallet and chooses "native connection" option or "web"

2. **Seamless Relay Establishment** 
   - Wallet provider generates secure session identifier
   - Connection process becomes invisible to user - no QR codes or manual steps required
   - **Mobile Connection Scenario** (two variants):
     - **Web App → Native Wallet**: 
       - Push notification automatically sent to user's registered mobile device/tablet
       - Web dApp displays informational modal: "Please check your smartphone and approve the sign-in request"
       - User receives native wallet app notification on mobile device
     - **Native App → Native Wallet**: 
       - Simple deeplink with session ID for easy forwarding to wallet app
       - Direct app-to-app communication on same device
       - Cross-device scenario: notification sent to primary wallet device when using secondary device for dApp
   - **Web Connection Scenario** (two variants):
     - **Web dApp → Direct Browser Connection**: 
       - Modal opens directing user to web wallet platform in same browser
       - Standard web-based wallet connection flow proceeds
       - User completes authentication in web wallet interface
     - **Web dApp → External Wallet Authentication**:
       - dApp detects no web wallet extension available
       - Modal instructs user to open primary wallet (mobile app or different browser)
       - User approves sign-in through external wallet interface
       - Session established once external wallet confirms approval

3. **Native Wallet Authentication**
   - Native wallet app receives connection request with session context and pre-generated public key
   - User reviews dApp permissions, account selection, and the public key to be added in native interface
   - User approves connection, authorizing the addition of the client-generated FunctionCall access key
   - Wallet creates AddKey transaction using the provided public key (private key remains secure on client side)

4. **Session Bridging**
   - Native wallet sends approval and public key to wallet provider's relay service
   - Web dApp polls relay service or receives WebSocket notification
   - Connection established with access to approved accounts

5. **Transaction Flow**
   - Web dApp constructs transactions using standard NEAR APIs
   - Transactions requiring FullAccess keys trigger notifications to mobile wallet
   - User approves transactions on mobile device
   - FunctionCall access keys used directly by dApp for gas-only transactions

### Technical Requirements

#### For Wallet Providers
- Implement relay infrastructure (WebSocket, HTTPS, or hybrid)
- Support cross-platform session management
- Provide consistent UI/UX across web and mobile interfaces
- Maintain encrypted communication channels

#### For dApps
- Detect wallet platform capabilities
- Implement connection option UI
- Handle both direct and relay-based connection patterns
- Maintain compatibility with existing wallet standards
