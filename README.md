# Jun19-Hackathon

# Project Timeline & Frameworks

**Project:** Decentralized Blockchain File Storage & Sharing Platform with Threat Intelligence

---

## Overview

This project aims to build a decentralized platform for secure file storage, sharing, access control, and automated malware/threat scanning.  
Below is a phase-based timeline, with all frameworks and major tools/technologies decided.

---

## Phase 1: Planning & Architecture

- Define user stories, acceptance criteria, and MVP scope
- Research and select frameworks/technologies
- Design high-level architecture and data flow
- Document smart contract data model and repository structure
- Set up collaboration tools and project management

**Frameworks/Tools Decided:**
- **Smart Contract Framework:** Hardhat (JavaScript project, Solidity, free & open-source)
- **Blockchain Network:** Ethereum testnet (Goerli or Polygon Mumbai), **3-4 blockchain nodes** for decentralization
- **Frontend:** React.js (for UI), ethers.js (for blockchain interaction), IPFS HTTP client (file storage)
- **Decentralized Storage:** IPFS (local nodes via Docker, can extend to Filecoin or Infura later)
- **Backend:** Node.js (Express for API orchestration, Axios, dotenv, cors)
- **Threat Intelligence:** VirusTotal API (scanning via backend relay)
- **DevOps:** Docker Compose (multi-node, local simulation)
- **Testing:** Hardhat/Chai (contracts), Jest (JS/TS backend/frontend)
- **Collaboration:** GitHub Projects, standard .gitignore, README.md

---

## Phase 2: Environment & Node Setup

- Set up local Ethereum testnet using Hardhat with **3-4 blockchain nodes**
- Deploy 3 local IPFS nodes using Docker for decentralized storage
- Scaffold smart contract project (Hardhat, JavaScript)
- Initialize basic smart contract for file registration, permissions, scan status
- Connect Hardhat contracts to IPFS workflow (basic integration script)
- Set up Node.js backend orchestrator for VirusTotal scanning
- Ensure communication between all nodes (blockchain, IPFS, backend)
- Document setup and APIs

**Deliverables:**
- Running local network: **3-4 blockchain nodes**, 3 IPFS nodes, 1 backend node
- Initial contract, tests, and integration scripts
- Backend API for VirusTotal
- Documentation for setup and usage

---

## Phase 3: Core Logic & Integration

- Implement and test full smart contract logic:
  - File registration
  - Ownership and permission management
  - Scan status tracking
- Complete end-to-end workflow:
  - Encrypt file, upload to IPFS, scan with VirusTotal, register on blockchain
  - File sharing and permission checks
  - Download and decrypt flow
- Integrate backend orchestrator with smart contracts and storage
- Begin frontend implementation with React.js:
  - Connect UI to contracts (ethers.js)
  - File upload/download interface
  - Display scan results and permissions
- Expand testing (unit, integration, e2e)

---

## Phase 4: Frontend Completion & Advanced Features

- Complete frontend features:
  - User dashboard (list files, view scan status, manage permissions)
  - File sharing UI and workflow
  - On-chain permission management (grant/revoke)
  - Ownership transfer
  - Admin views (storage/node health)
- Optimize integration between frontend, backend, smart contracts, and IPFS
- Improve UX (error handling, notifications, loading states)

---

## Phase 5: Hardening, Scaling & Documentation


- Security review (access control, input validation, encryption)
- Expand to more nodes (multi-node Docker Compose or cloud simulation)
- Optimize gas usage and storage structure in smart contracts
- Add advanced features (optional): multi-file support, batch uploads, analytics
- Finalize documentation (setup, usage, APIs, architecture)
- Prepare for demo or deployment

---

## Summary Table

| Phase | Focus                                | Duration | Major Frameworks/Tech                |
|-------|--------------------------------------|----------|--------------------------------------|
|  1    | Planning, architecture, setup        | N/A  | Hardhat, React, IPFS, Node.js        |
|  2    | Env & node setup, integration base   | N/A   | Hardhat (3-4 nodes), Docker, IPFS    |
|  3    | Core contract logic, end-to-end flow | N/A   | Hardhat, React, IPFS, Node.js        |
|  4    | Frontend & advanced features         | N/A   | React, ethers.js, IPFS               |
|  5    | Hardening, scaling, docs             | N/A   | All above, Docker Compose            |

---

## Notes

- All frameworks chosen are free and open source.
- Security and privacy are priorities: all scanning is done via a backend relay to keep API keys secure.
- Documentation will be updated phase by phase for onboarding and reproducibility.

---
