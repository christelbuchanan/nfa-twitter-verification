# NFA Twitter Verification & On-Chain Attestation Bridge

A trust bridge system that verifies Twitter social actions off-chain and attests them on-chain for ChatAndBuild to read via Boolean API or BNB Chain events.


## 🏗️ System Architecture

```mermaid
graph TB
    subgraph User Layer
        U[User Wallet]
        T[Twitter Account]
    end
    
    subgraph Verification Bridge
        W[Wallet Connect]
        TV[Twitter Verifier]
        BA[Boolean API]
    end
    
    subgraph BNB Chain
        AR[AttestationRegistry]
        RD[NFARewardDistributor]
    end
    
    subgraph ChatAndBuild
        DB[Dashboard]
        API[API Reader]
        EL[Event Listener]
    end
    
    U --> W
    T --> TV
    W --> BA
    TV --> BA
    BA --> AR
    AR --> RD
    AR --> EL
    AR --> API
    RD --> DB
    EL --> DB
    API --> DB
    DB --> U
```

## 🔄 Verification Flow

```mermaid
sequenceDiagram
    participant User
    participant Twitter
    participant Backend
    participant BooleanAPI
    participant BNBChain
    participant ChatAndBuild
    
    User->>Twitter: Post tweet with hashtags and wallet
    Twitter->>Backend: Detect tweet via API/Crawler
    Backend->>BooleanAPI: Request verification
    BooleanAPI->>BooleanAPI: Generate proofHash
    BooleanAPI->>BNBChain: Submit attestation
    BNBChain->>BNBChain: Store in AttestationRegistry
    BNBChain->>BNBChain: Trigger NFARewardDistributor
    BNBChain-->>ChatAndBuild: Emit AttestationCreated event
    ChatAndBuild->>ChatAndBuild: Update dashboard
    ChatAndBuild-->>User: Show completion status
    BNBChain-->>User: Distribute reward
```

## 📊 Data Flow

```mermaid
flowchart LR
    A[User Tweets] --> B[Backend Detection]
    B --> C{Verification Method}
    C -->|OAuth| D[Twitter API]
    C -->|Crawler| E[Hashtag Scanner]
    D --> F[Boolean API]
    E --> F
    F --> G[Generate ProofHash]
    G --> H[AttestationRegistry]
    H --> I[Store on BNB Chain]
    I --> J[Emit Event]
    J --> K[ChatAndBuild Dashboard]
    J --> L[NFARewardDistributor]
    L --> M[User Reward]
```

## 🔐 Attestation Process

```mermaid
stateDiagram-v2
    [*] --> WalletConnected: Connect Wallet
    WalletConnected --> TweetPosted: Post Tweet
    TweetPosted --> Detecting: Backend Scanning
    Detecting --> Verifying: Tweet Found
    Verifying --> Generating: Validation Success
    Generating --> Submitting: ProofHash Created
    Submitting --> OnChain: Transaction Sent
    OnChain --> Completed: Attestation Stored
    Completed --> Rewarded: Reward Distributed
    Rewarded --> [*]
    
    Detecting --> Failed: Tweet Not Found
    Verifying --> Failed: Validation Failed
    Submitting --> Failed: Transaction Failed
    Failed --> [*]
```

## 🎯 Component Interaction

```mermaid
graph LR
    subgraph Frontend
        WC[WalletConnect]
        TV[TwitterVerify]
        AL[AttestationList]
        SC[StatsCards]
    end
    
    subgraph Hooks
        TA[useTwitterAuth]
        BA[useBooleanAPI]
        AT[useAttestation]
    end
    
    subgraph Utils
        TU[twitter.ts]
        AU[attestation.ts]
        BU[blockchain.ts]
    end
    
    WC --> AT
    TV --> TA
    TV --> BA
    AL --> AT
    SC --> AT
    
    TA --> TU
    BA --> AU
    AT --> BU
```

## 🔄 Smart Contract Flow

```mermaid
flowchart TD
    A[User Action] --> B{Wallet Connected?}
    B -->|No| C[Connect Wallet]
    B -->|Yes| D[Post Tweet]
    C --> D
    D --> E[Backend Verifies]
    E --> F{Valid Tweet?}
    F -->|No| G[Show Error]
    F -->|Yes| H[Call AttestationRegistry]
    H --> I[Store Attestation]
    I --> J[Emit Event]
    J --> K[Trigger NFARewardDistributor]
    K --> L[Mint/Transfer Reward]
    L --> M[Update User Balance]
    M --> N[Show Success]
```

## 📈 Monitoring Dashboard Flow

```mermaid
graph TB
    subgraph Data Sources
        BC[BNB Chain Events]
        BA[Boolean API]
        TA[Twitter API]
    end
    
    subgraph Processing
        EL[Event Listener]
        AP[API Poller]
        DT[Data Transformer]
    end
    
    subgraph Dashboard
        TV[Total Verifications]
        AU[Active Users]
        SR[Success Rate]
        HS[Health Status]
    end
    
    BC --> EL
    BA --> AP
    TA --> AP
    EL --> DT
    AP --> DT
    DT --> TV
    DT --> AU
    DT --> SR
    DT --> HS
```

### Flow Breakdown

### 1. Wallet Binding
- User connects via **BNB Chain Wallet** 
- Wallet address stored for attestation linkage

### 2. Off-Chain Twitter Verification

**Option A: Twitter OAuth API** (Recommended)
- Real-time verification
- Direct API access
- Instant feedback

**Option B: Hashtag Crawler**
- 1-3 minute delay
- Fallback option
- No OAuth required

### 3. Boolean API Attestation

**Attestation Payload Structure:**
```typescript
{
  walletAddress: string;    // User's BNB Chain address
  taskId: string;           // Unique task identifier
  tweetId: string;          // Twitter post ID
  proofHash: string;        // SHA-256 hash of verification data
  timestamp: number;        // Unix timestamp
  signature: string;        // Cryptographic signature
}
```

**Smart Contract Integration:**
- `AttestationRegistry`: Stores verified attestations on BNB Chain
- `NFARewardDistributor`: Distributes rewards based on verified tasks

### 4. ChatAndBuild Dashboard Integration

ChatAndBuild can read verified tasks via:
- **Boolean API Query**: REST endpoint for attestation lookup
- **BNB Chain Events**: Listen to `AttestationCreated` events
- **Real-time Dashboard**: Live feed of verified social actions

## 🚀 Getting Started

### Prerequisites
- Node.js 18+ 
- npm or yarn
- BNB Chain wallet (for testing)
- Twitter Developer Account (for OAuth option)


## 🛠️ Tech Stack

- **Frontend**: React 18, TypeScript, Vite
- **Styling**: Tailwind CSS (Neo-Brutalism theme)
- **Charts**: Recharts
- **Icons**: Lucide React
- **Blockchain**: BNB Chain, Thirdweb SDK
- **APIs**: Twitter API, Boolean API

## 📦 Project Structure

```
nfa-twitter-verification-bridge/
├── src/
│   ├── components/
│   │   ├── Dashboard.tsx          # Main dashboard
│   │   ├── WalletConnect.tsx      # Thirdweb wallet integration
│   │   ├── TwitterVerify.tsx      # Twitter verification UI
│   │   ├── AttestationList.tsx    # Attestation history
│   │   └── StatsCards.tsx         # Statistics display
│   ├── hooks/
│   │   ├── useTwitterAuth.ts      # Twitter OAuth logic
│   │   ├── useBooleanAPI.ts       # Boolean API integration
│   │   └── useAttestation.ts      # Smart contract interaction
│   ├── utils/
│   │   ├── twitter.ts             # Twitter API helpers
│   │   ├── attestation.ts         # Attestation generation
│   │   └── blockchain.ts          # BNB Chain utilities
│   ├── types/
│   │   └── index.ts               # TypeScript definitions
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── public/
├── package.json
├── vite.config.ts
├── tailwind.config.js
├── tsconfig.json
└── README.md
```

## 🔐 Security Considerations

- **Rate Limiting**: Twitter API has rate limits (consider caching)
- **Proof Verification**: All attestations are cryptographically signed
- **Smart Contract Audits**: Ensure AttestationRegistry is audited before mainnet

## 🧪 Testing

```bash
# Run linter
npm run lint

# Build for production
npm run build

# Preview production build
npm run preview
```

## 📊 Monitoring & Analytics

The dashboard provides real-time metrics:
- Total verified tasks
- Active users
- Attestation success rate
- Twitter API health status
- BNB Chain transaction status

## 🤝 Integration Guide for ChatAndBuild

### Reading Attestations via Boolean API

```typescript
// Query attestation by wallet address
const response = await fetch(
  `${BOOLEAN_API_ENDPOINT}/attestations/${walletAddress}`,
  {
    headers: {
      'Authorization': `Bearer ${BOOLEAN_API_KEY}`
    }
  }
);

const attestations = await response.json();
```

### Listening to BNB Chain Events

```typescript
// Listen for AttestationCreated events
const contract = new ethers.Contract(
  ATTESTATION_REGISTRY_ADDRESS,
  ABI,
  provider
);

contract.on('AttestationCreated', (walletAddress, taskId, proofHash) => {
  console.log('New attestation:', { walletAddress, taskId, proofHash });
  // Trigger reward distribution
});
```

## 🐛 Known Issues

- Twitter API rate limits may cause delays during high traffic
- Hashtag crawler option has 1-3 minute latency
- BNB Chain gas fees may fluctuate

## 📝 License

MIT License - See LICENSE file for details

