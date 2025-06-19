# Phase 2: Detailed Approach  
**Multi-Node Local Environment Setup**

---

## Objective

To establish a robust local development environment simulating a decentralized storage and sharing system, consisting of:
- **3-4 blockchain nodes** (using Hardhat, simulating a mini Ethereum network)
- **3 IPFS nodes** (using Docker for decentralized storage)
- **1 backend orchestrator** (Node.js, for threat intelligence scanning)

This phase lays the technical groundwork for developing, testing, and demonstrating secure, decentralized file storage and sharing with on-chain permissions and threat scanning.

---

## 1. Environment Preparation

### A. System Requirements

- **Node.js (v16 or higher)**
- **npm (v7 or higher)**
- **Docker** (for IPFS nodes)
- **Git**
- Adequate disk space and RAM for running multiple containers/nodes

### B. Directory Structure

```
project-root/
  contracts/       # Smart contracts & Hardhat config
  backend/         # Node.js backend for VirusTotal relay/API orchestration
  infra/           # Docker Compose/scripts
  docs/            # Documentation & tutorials
```

---

## 2. Blockchain Node Setup (3-4 Nodes with Hardhat)

### A. Initialize Hardhat Project

```bash
mkdir contracts && cd contracts
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npx hardhat
```
- Select **“Create a JavaScript project”** and accept the defaults.

### B. Simulate Multi-Node Blockchain Network

#### **Option 1: Multiple Hardhat Nodes (Simple Simulation)**

Open **3-4 terminals** and run in each:

```bash
npx hardhat node --port 8545   # Node 1 (default)
npx hardhat node --port 8546   # Node 2
npx hardhat node --port 8547   # Node 3
npx hardhat node --port 8548   # Node 4 (optional)
```
- Each node simulates a local Ethereum node on a unique port.
- **Leave all terminals running** for your "network".

#### **Option 2: Advanced – Geth Clique Network (for consensus, optional)**

- Initialize a Clique PoA network with 3-4 real nodes using [Geth](https://geth.ethereum.org/docs/interface/private-network).
- More complex, but closer to a real-world decentralized network.

### C. Compile and Deploy Contracts to Each Node

- In `contracts/scripts/deploy.js`, write a deploy script for your contracts (e.g., FileRegistry).
- Deploy to each node:

```bash
npx hardhat compile

# For each node (change URL for each)
npx hardhat run --network localhost --url http://127.0.0.1:8545 scripts/deploy.js
npx hardhat run --network localhost --url http://127.0.0.1:8546 scripts/deploy.js
npx hardhat run --network localhost --url http://127.0.0.1:8547 scripts/deploy.js
npx hardhat run --network localhost --url http://127.0.0.1:8548 scripts/deploy.js
```

- Keep track of deployed contract addresses for each node.

---

## 3. IPFS Node Setup (3 Nodes with Docker)

### A. Launch IPFS Nodes

In a new terminal, run:

```bash
docker run -d --name ipfs0 -p 5001:5001 -p 8080:8080 ipfs/go-ipfs
docker run -d --name ipfs1 -p 5002:5001 -p 8081:8080 ipfs/go-ipfs
docker run -d --name ipfs2 -p 5003:5001 -p 8082:8080 ipfs/go-ipfs
```

### B. Verify and Peer IPFS Nodes

1. Get node IDs and multiaddresses:

```bash
docker exec -it ipfs0 ipfs id
docker exec -it ipfs1 ipfs id
docker exec -it ipfs2 ipfs id
```

2. Peer the nodes (connect each to the others):

```bash
docker exec -it ipfs0 ipfs swarm connect <ipfs1_multiaddress>
docker exec -it ipfs0 ipfs swarm connect <ipfs2_multiaddress>
docker exec -it ipfs1 ipfs swarm connect <ipfs0_multiaddress>
docker exec -it ipfs1 ipfs swarm connect <ipfs2_multiaddress>
docker exec -it ipfs2 ipfs swarm connect <ipfs0_multiaddress>
docker exec -it ipfs2 ipfs swarm connect <ipfs1_multiaddress>
```

3. Test file replication:

```bash
docker exec -it ipfs0 ipfs add /etc/hosts
# Get CID
docker exec -it ipfs1 ipfs cat <CID>
docker exec -it ipfs2 ipfs cat <CID>
```

---

## 4. Backend Orchestrator Setup (Node.js)

### A. Initialize Backend

```bash
mkdir ../backend && cd ../backend
npm init -y
npm install express axios dotenv cors
```

### B. Implement API Server

1. **Create `index.js`:**

```javascript name=backend/index.js
const express = require('express');
const cors = require('cors');
const { scanFileWithVirusTotal } = require('./virusTotal');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json({ limit: '50mb' }));

app.post('/api/scan', async (req, res) => {
  try {
    const { fileBase64 } = req.body;
    const result = await scanFileWithVirusTotal(fileBase64);
    res.json(result);
  } catch (e) {
    res.status(500).json({ error: e.toString() });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => console.log(`Backend running on port ${PORT}`));
```

2. **Create `virusTotal.js`:**

```javascript name=backend/virusTotal.js
const axios = require('axios');
const FormData = require('form-data');

const scanFileWithVirusTotal = async (fileBase64) => {
  const apiKey = process.env.VIRUSTOTAL_API_KEY;
  const url = 'https://www.virustotal.com/api/v3/files';
  const buffer = Buffer.from(fileBase64, 'base64');
  const formData = new FormData();
  formData.append('file', buffer, 'file.bin');

  const response = await axios.post(url, formData, {
    headers: {
      'x-apikey': apiKey,
      ...formData.getHeaders(),
    },
    maxContentLength: Infinity,
    maxBodyLength: Infinity,
  });
  return response.data;
};

module.exports = { scanFileWithVirusTotal };
```

3. **Add `.env`:**

```
VIRUSTOTAL_API_KEY=your_virustotal_api_key_here
```

### C. Start Backend

```bash
node index.js
```

---

## 5. Integration Workflows

### A. File Upload & IPFS Storage

- Write a Node.js script (or use Postman) to upload a file to an IPFS node using its HTTP API.
- Example:

```javascript name=infra/ipfs-upload.js
const axios = require('axios');
const fs = require('fs');
const FormData = require('form-data');

async function uploadToIpfs(filePath, ipfsApiUrl) {
  const file = fs.createReadStream(filePath);
  const formData = new FormData();
  formData.append('file', file);

  const res = await axios.post(`${ipfsApiUrl}/api/v0/add`, formData, {
    headers: formData.getHeaders(),
  });
  return res.data.Hash; // The CID
}

// Usage
uploadToIpfs('my-encrypted-file.bin', 'http://localhost:5001')
  .then(cid => console.log('File uploaded to IPFS with CID:', cid));
```

### B. Register CID on Blockchain

- Use `ethers.js` in a script to interact with your deployed contract on the specific node/port.

### C. Malware Scanning

- Send the file (base64) to the backend’s `/api/scan` endpoint and store the scan result/status in your contract.

---

## 6. Node Map/Network Topology

| Node Type     | Name    | Ports                | Purpose                                  |
|---------------|---------|----------------------|------------------------------------------|
| Blockchain    | node1   | 8545                 | Smart contracts, file registry           |
| Blockchain    | node2   | 8546                 | Smart contracts, file registry           |
| Blockchain    | node3   | 8547                 | Smart contracts, file registry           |
| Blockchain    | node4   | 8548 (optional)      | Smart contracts, file registry           |
| IPFS 1        | ipfs0   | 5001, 8080           | Decentralized file storage               |
| IPFS 2        | ipfs1   | 5002, 8081           | Decentralized file storage               |
| IPFS 3        | ipfs2   | 5003, 8082           | Decentralized file storage               |
| Backend       | backend | 3001                 | VirusTotal integration, API orchestrator |

---

## 7. Testing & Verification

- **Hardhat nodes:** Confirm each node is running and contracts are deployed.
- **IPFS nodes:** Confirm all nodes are peered and files replicate.
- **Backend:** Confirm `/api/scan` endpoint works with a test file.
- **Integration:**  
  - Upload a file to IPFS, get CID.
  - Register CID and scan status in contract (on each node).
  - Retrieve and verify file and scan status via script.

---

## 8. Troubleshooting

- **Docker:** Use `docker ps`/`docker logs` to check containers.
- **IPFS:** Use `ipfs swarm peers` to verify peering.
- **Hardhat:** Make sure each node uses a unique port.
- **Backend:** Ensure VIRUSTOTAL_API_KEY is set and valid.

---

## 9. Documentation

- Update `README.md` in each directory with setup and usage.
- Document API endpoints, contract addresses, peer addresses.
- Add diagrams of the network topology for clarity.

---

## 10. Deliverables for Phase 2

- [ ] All blockchain and IPFS nodes running locally
- [ ] Backend orchestrator operational
- [ ] End-to-end test: file upload, scan, registration, and audit
- [ ] Documentation for setup, scripts, and integration

---

**With Phase 2 complete, your team is ready to implement core business logic, permissions, and full end-to-end flows in Phase 3.**
