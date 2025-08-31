# HydroChain: Blockchain-Based Hydrogen Credit Management System

**Empowering a Sustainable Future with Blockchain-Powered Green Hydrogen Credits**

A comprehensive platform for managing green hydrogen credits using Ethereum smart contracts, IoT data verification, and decentralized identity (DID).

## Overview

HydroChain is a comprehensive blockchain platform designed to be **Transparent · Verifiable · Decentralized**. Our system enables:

- **Issuing** of hydrogen credits by renewable energy plants
- **Trading** of credits between plants and industries
- **Retirement** of credits by industries after consumption
- **Auditing** of transactions by authorized regulators/auditors
- **IoT integration** for data-driven verification
- **Decentralized Identity (DID)** for secure authentication

The system combines blockchain's immutability with IoT data verification to create a trusted, transparent hydrogen credit ecosystem.

## Use Case & Market Challenges

HydroChain addresses critical market challenges in green hydrogen credit trading by:

- **Enabling secure buying and selling** of green hydrogen credits
- **Preventing greenwashing** through transparent verification
- **Eliminating duplicate selling** with blockchain-based immutability
- **Facilitating safe retirement** of credits with complete audit trails

Blockchain technology serves as the foundation for trust, ensuring every transaction is transparent, verifiable, and tamper-proof.

## User Roles & Workflow

The platform serves three primary user types:

1. **Plants (Producers)** – Generate green hydrogen and issue credits
2. **Industries (Consumers)** – Purchase, use, and retire hydrogen credits
3. **Auditors/Regulators** – Validate and approve all credit-related requests

### Core Workflow

```
┌──────────────┐        ┌──────────────────┐        ┌───────────────┐
│              │        │                  │        │               │
│   IoT Data   │───────►│  Auto-Verification│───────►│   Auditor     │
│  Collection  │        │  & VC Issuance    │        │   Review      │
│              │        │                  │        │               │
└──────────────┘        └──────────────────┘        └───────┬───────┘
                                                           │
                                                           ▼
┌───────────────┐        ┌──────────────────┐        ┌───────────────┐
│               │        │                  │        │               │
│  Blockchain   │◄───────│   Transaction    │◄───────│   Approval    │
│  Recording    │        │   Processing     │        │   Decision    │
│               │        │                  │        │               │
└───────────────┘        └──────────────────┘        └───────────────┘
```

## Technical Architecture

### System Components

1. **REST API Layer** - Built with Elysia.js, providing endpoints for user and credit operations
2. **Database Layer** - PostgreSQL with Prisma ORM for data persistence
3. **Blockchain Layer** - Ethereum smart contract for immutable record-keeping
4. **Authentication System** - Session-based authentication with DID integration
5. **IoT Integration Layer** - For collecting and validating operational data
6. **Verifiable Credentials System** - For automated verification of requests
7. **Role-Based Access Control** - Different permissions for Plant, Industry, and Auditor users

### Features Implemented

#### Backend Infrastructure

- **Issue Request Management**: Streamlined credit issuance workflow
- **Buy Request Processing**: Secure purchase transaction handling
- **Authentication System**: Role-based access control for all user types
- **Request Tracking**: PostgreSQL database for comprehensive audit trails

#### Smart Contracts

- **Solidity-based Smart Contracts**: Automated credit lifecycle management
- **Non-Duplicatable Credits**: Unique token generation preventing fraud
- **Buying & Selling Logic**: Secure transfer mechanisms
- **Governing Authority Integration**: Automated approval/rejection workflows

#### Governance

- **Authority Verification**: Governing bodies can accept or reject permissions
- **Compliance Monitoring**: Real-time oversight of all transactions
- **Regulatory Integration**: Support for external auditors and verifiers

### Frontend Experience

Built with modern web technologies for optimal user experience:

- **Landing Page**: Professional introduction to the platform
- **Plant Dashboard**: Specialized interface for hydrogen producers
- **High-Performance Stack**: React.js with Vite, ShadCN UI components

### Core Technologies

- **Bun** - JavaScript runtime and package manager
- **Elysia.js** - TypeScript web framework optimized for Bun
- **Prisma** - ORM for database access and migrations
- **Ethers.js** - Ethereum blockchain interaction library
- **PostgreSQL** - Primary database for storing user data and transaction records
- **React** - Frontend user interface with Vite
- **DID Framework** - For decentralized identity management
- **Verifiable Credentials** - For cryptographic proof of verification
- **Tailwind CSS** - For responsive, modern styling
- **ShadCN UI** - For consistent, accessible components

### System Architecture Diagram

```
┌─────────────┐      ┌──────────────┐      ┌───────────────┐
│             │      │              │      │               │
│   React     │──────►   REST API   │──────►   Database    │
│  Frontend   │◄──────│  (Elysia.js) │◄──────│   (PostgreSQL) │
│             │      │              │      │               │
└─────────────┘      └──────┬───────┘      └───────────────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
┌──────▼──────┐     ┌──────▼──────┐     ┌──────▼──────┐
│              │     │              │     │              │
│  Ethereum    │     │  IoT Data    │     │     DID      │
│Smart Contract│     │  Integration │     │   System     │
│              │     │              │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
```

## Smart Contract Implementation

Our core smart contract handles the complete lifecycle of green hydrogen credits:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HydrogenCredit {
    struct Credit {
        string id;
        address issuer;
        address holder;
        uint256 amount;
        uint256 timestamp;
        bool retired;
    }

    Credit[] public credits;
    mapping(string => uint256) public creditIdToIndex; // Map string ID to array index
    mapping(string => bool) public creditExists; // Check if ID already exists

    event CreditIssued(
        string indexed id,
        address indexed issuer,
        uint256 amount
    );
    event CreditTransferred(
        string indexed id,
        address indexed from,
        address indexed to
    );
    event CreditRetired(string indexed id, address indexed holder);

    function issueCredit(string memory id, address to, uint256 amount) public {
        require(!creditExists[id], 'Credit ID already exists');
        credits.push(
            Credit({
                id: id,
                issuer: msg.sender,
                holder: to,
                amount: amount,
                timestamp: block.timestamp,
                retired: false
            })
        );
        // Map the string ID to array index
        creditIdToIndex[id] = credits.length - 1;
        creditExists[id] = true;
        emit CreditIssued(id, msg.sender, amount);
    }

    function transferCredit(string memory creditId, address to) public {
        require(creditExists[creditId], "Credit doesn't exist");
        uint256 index = creditIdToIndex[creditId];
        require(credits[index].holder == msg.sender, 'Not your credit');
        require(!credits[index].retired, 'Credit already retired');
        credits[index].holder = to;
        emit CreditTransferred(creditId, msg.sender, to);
    }

    function retireCredit(string memory creditId) public {
        require(creditExists[creditId], "Credit doesn't exist");
        uint256 index = creditIdToIndex[creditId];
        require(credits[index].holder == msg.sender, 'Not your credit');
        require(!credits[index].retired, 'Already retired');
        credits[index].retired = true;
        emit CreditRetired(creditId, msg.sender);
    }

    function getCredit(
        string memory creditId
    ) public view returns (Credit memory) {
        require(creditExists[creditId], "Credit doesn't exist");
        uint256 index = creditIdToIndex[creditId];
        return credits[index];
    }

    function getAllCredits() public view returns (Credit[] memory) {
        return credits;
    }
}
```

### Contract Features

- **Unique Credit IDs**: Prevents duplicate credit creation
- **Full Lifecycle Management**: Issue, transfer, and retire credits
- **Ownership Verification**: Only credit holders can transfer or retire
- **Immutable Records**: All transactions permanently recorded on blockchain
- **Event Logging**: Complete audit trail through blockchain events

## Core System Flows

### 1. Digital Identity (DID) for Secure Access

- Plants and Industries must have IoT-based data verification systems installed
- Auditors physically verify the IoT system and plant/industry location
- Upon verification, a Decentralized Identity (DID) is issued
- DID serves as the secure login credential for platform access

### 2. IoT Data Integration

Real-time IoT data is displayed on user dashboards, including:

- Hydrogen **production** metrics for Plants
- Hydrogen **transfer** records between Plants and Industries
- Hydrogen **consumption** metrics for Industries

This IoT data provides the foundational evidence for all credit-related activities.

### 3. Credit Issuance Flow

1. Plant initiates credit issuance request with amount and production data
2. System auto-verifies request against IoT data
3. If valid, a Verifiable Credential (VC) is attached to the request
4. Request is forwarded to assigned Auditor with VC status
5. Auditor reviews and approves/rejects
6. If approved:
   - Smart contract issues credit tokens
   - Transaction hash is recorded
   - Request status changes to ISSUED
   - Plant's lifetime generated credits are updated

### 4. Credit Transfer Flow

1. Industry submits buy request for specific credit ID
2. System verifies transfer data against IoT records
3. If valid, a VC is attached to the request
4. Request is forwarded to assigned Auditor with VC status
5. Auditor reviews and approves/rejects
6. If approved:
   - Smart contract transfers credit ownership
   - Transaction hash is recorded
   - Request status changes to TRANSFERRED
   - Plant and industry lifetime statistics are updated

### 5. Credit Retirement Flow

1. Industry submits retirement request for specific credit ID
2. System verifies consumption data against IoT records
3. If valid, a VC is attached to the request
4. Request is forwarded to assigned Auditor with VC status
5. Auditor reviews and approves/rejects
6. If approved:
   - Smart contract marks credit as retired
   - Transaction hash is recorded
   - Request status changes to RETIRED
   - Industry's lifetime retired credits are updated

## Database Schema Details

The application uses Prisma with PostgreSQL and includes the following core models:

### User Model

```prisma
model User {
  id String @id @default(uuid())
  role UserRole
  username String @unique
  password String
  companyName String?
  governmentLicenseId String?
  did String?
  walletAddress String? @unique
  walletPrivateKey String?
  assignedAuditor User? @relation("AuditorAssignment", fields: [assignedAuditorId], references: [id])
  assignedAuditorId String?
  assignedUsers User[] @relation("AuditorAssignment")
  lifeTimeGeneratedCredits Float @default(0)
  lifeTimeTransferredCredits Float @default(0)
  lifeTimeRetiredCredits Float @default(0)
  lifeTimeBoughtCredits Float @default(0)
  // Relations
  sessions Session[]
  creditIssueRequests CreditIssueRequest[] @relation(name: "issuedPlant")
  creditIssueRequestsActions CreditIssueRequest[] @relation(name: "issueRequestAuditor")
  creditBuyRequests CreditBuyRequest[] @relation(name: "creditBuyerPlant")
  CreditBuyRequest CreditBuyRequest[] @relation(name: "creditSellingPlant")
  creditRetireRequests CreditRetireRequest[] @relation(name: "retiringIndustry")
  creditRetireRequestsActions CreditRetireRequest[] @relation(name: "retireRequestAuditor")
}
```

### Credit Request Models

The system uses three types of request models:

1. **CreditIssueRequest** - For plants to request new credit issuance
2. **CreditBuyRequest** - For industries to purchase credits from plants
3. **CreditRetireRequest** - For industries to retire credits after use

## Prerequisites

- [Bun](https://bun.sh/) >= 1.0.0
- [PostgreSQL](https://www.postgresql.org/) >= 15.0
- [Solc](https://docs.soliditylang.org/en/v0.8.0/installing-solidity.html) (Solidity Compiler)
- [Node.js](https://nodejs.org/) >= 16.0.0 (for frontend)
- Ethereum provider (like [Infura](https://www.infura.io/), [Alchemy](https://www.alchemy.com/), or a local node)
- MetaMask browser extension

## Environment Setup

### Backend Environment

Create a `.env` file in the `backend` directory with the following variables:

```env
# Server Configuration
PORT=3000
TRAFFIC_LOG=true
LOG_LEVEL=1
ORIGIN=http://localhost:3000
FRONTEND_ORIGINS=http://localhost:3000,http://localhost:5173

# Database
DATABASE_URL=postgresql://username:password@localhost:5432/hydrochain

# Authentication
SESSION_COOKIE_NAME=sid

# Blockchain
PRIMARY_ETH_ACCOUNT_PRIVATE_KEY=your_ethereum_private_key
ETHER_PROVIDER_URL=https://eth-sepolia.g.alchemy.com/v2/your_api_key
CONTRACT_ADDRESS=0x0000000000000000000000000000000000000000  # Will be updated after deployment
```

### Frontend Environment

Create a `.env` file in the `frontend` directory:

```env
VITE_API_URL=http://localhost:3000
VITE_WS_URL=ws://localhost:3000
```

## Installation

### Backend Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/henilmalaviya/hydrochain.git
   cd hydrochain/backend
   ```

2. Install dependencies:

   ```bash
   bun install
   ```

3. Setup the database:

   ```bash
   bunx prisma migrate deploy
   ```

4. Compile and deploy the smart contract:

   ```bash
   cd contracts
   bunx solc --bin --abi HydrogenCredit.sol
   cd ..
   bun run scripts/deployContracts.ts
   ```

5. Update the `CONTRACT_ADDRESS` in your `.env` file with the deployed contract address.

### Frontend Installation

1. Navigate to the frontend directory:

   ```bash
   cd ../frontend
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

## Running the Application

### Backend

```bash
# Development mode
bun run dev

# Production mode
bun run start
```

### Frontend

```bash
# Development mode
npm run dev

# Production build
npm run build
```

## API Routes Reference

### Authentication & User Management

- `POST /users` - Register a new user
- `POST /users/:username/login` - User login
- `GET /users/:username` - Get user transaction history
- `GET /me` - Get current user info

### Credit Issuance (Plant & Auditor)

- `POST /request/issues` - Create credit issue request
- `GET /request/issues` - Get issue requests (for auditor)
- `POST /request/issues/:creditId/accept` - Approve issue request
- `POST /request/issues/:creditId/reject` - Reject issue request

### Credit Trading (Industry & Auditor)

- `POST /request/buy` - Create credit buy request
- `GET /request/buy` - Get buy requests (for auditor)
- `POST /request/buy/:reqId/accept` - Approve buy request
- `POST /request/buy/:reqId/reject` - Reject buy request

### Credit Retirement (Industry & Auditor)

- `POST /request/retire` - Create credit retirement request
- `GET /request/retire` - Get retirement requests (for auditor)
- `POST /request/retire/:reqId/accept` - Approve retirement request
- `POST /request/retire/:reqId/reject` - Reject retirement request

## Usage

### For Producers (Plants)

1. Connect MetaMask wallet
2. Request credit issuance through plant dashboard
3. Wait for governing authority approval
4. Receive credits in blockchain wallet

### For Consumers (Industries)

1. Browse available credits in marketplace
2. Initiate purchase through smart contract
3. Complete transaction using Ethereum
4. Track and retire credits after hydrogen consumption

### For Authorities (Auditors)

1. Review credit issuance requests
2. Verify compliance documentation and IoT data
3. Approve/reject through governance interface
4. Monitor all transactions on blockchain

## Future Roadmap

### Phase 1 - Enhanced Security

- **Credit Retirement System**: Complete lifecycle management
- **DID Verification**: Decentralized Identity integration
- **Verifiable Credentials (VCs)**: Cryptographic proof of authenticity

### Phase 2 - IoT Integration

- **Real-time IoT Monitoring**: Sensor data integration for verification
- **Automated Validation**: IoT-triggered credit issuance
- **Enhanced Compliance**: Real-world data validation for selling and retirement

## Security Features

1. **Decentralized Identity (DID)** - Self-sovereign identity for secure authentication
2. **Verifiable Credentials** - Cryptographic proof of verification
3. **Session-based authentication** - Secure session management with cookies
4. **Role-based access control** - Endpoints protected by user roles
5. **Private key management** - Secure handling of Ethereum private keys
6. **Request validation** - Comprehensive validation against IoT data
7. **Anomaly detection** - Flagging of suspicious transactions

## Key Benefits

- **Verifiable Production**: All hydrogen production is verified via IoT data
- **Double-Counting Prevention**: Blockchain ensures credits cannot be used twice
- **Transparent Tracking**: Complete audit trail from issuance to retirement
- **Automated Verification**: System pre-verifies requests before human review
- **Regulatory Compliance**: Built-in oversight by authorized auditors
- **Fraud Resistance**: Multiple verification layers (IoT, VC, Auditor, Blockchain)

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a pull request

## License

[MIT License](LICENSE)

---

**Built for HackOut'25 at Dhirubhai Ambani University Gandhinagar**

_Addressing the first problem statement: Blockchain-Based Green Hydrogen Credit System_
