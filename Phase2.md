# Phase 2 Tutorial: Multi-Node Local Environment Setup

**Phase:** 2  
**Objective:** Set up a multi-node development environment with 3-4 local blockchain nodes (Hardhat), 3 local IPFS nodes (Docker), and a backend orchestrator for threat intelligence scanning.

---

## 1. Prerequisites

- **Node.js** (v16+)
- **npm** (v7+)
- **Docker** (for IPFS nodes)
- **Git**
- **Basic terminal knowledge**

---

## 2. Directory Structure

```
project-root/
  contracts/       # Solidity smart contracts & Hardhat config
  backend/         # Node.js backend orchestrator
  infra/           # (Optional) Docker Compose/scripts
  docs/            # Documentation, tutorials
```

---

## 3. Setup: 3-4 Local Blockchain Nodes (Hardhat)

### A. Install and Initialize Hardhat

```bash
mkdir contracts && cd contracts
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npx hardhat
```
- When prompted, select: **Create a JavaScript project**.  
- Accept defaults for other prompts.

### B. Simulate Multiple Blockchain Nodes

Hardhat’s built-in node simulates multiple accounts, but for true multi-node simulation, use either:
- **(Recommended for dev/test):** Run multiple Hardhat nodes on different ports, each with its own data directory, or
- **Advanced:** Setup a local Ethereum network, e.g., [Geth Clique](https://geth.ethereum.org/docs/interface/private-network) (optional for advanced decentralization).

#### For most users, this approach is sufficient:

Open **3-4 terminals** and run in each:

```bash
npx hardhat node --port 8545   # Terminal 1
npx hardhat node --port 8546   # Terminal 2
npx hardhat node --port 8547   # Terminal 3
# (Optional) npx hardhat node --port 8548   # Terminal 4
```

Each node will simulate a blockchain node on a separate port.  
**Leave all terminals open and running.**

### C. Deploy Smart Contracts to Each Node

In a new terminal, for each node (change port as needed):

```bash
cd contracts
npx hardhat compile
npx hardhat run --network localhost --url http://127.0.0.1:8545 scripts/deploy.js
npx hardhat run --network localhost --url http://127.0.0.1:8546 scripts/deploy.js
npx hardhat run --network localhost --url http://127.0.0.1:8547 scripts/deploy.js
# etc.
```
*(Edit `scripts/deploy.js` to deploy your contract, e.g., `FileRegistry.sol`.)*

---

## 4. Setup: 3 Local IPFS Nodes with Docker

In a terminal, run:

```bash
docker run -d --name ipfs0 -p 5001:5001 -p 8080:8080 ipfs/go-ipfs
docker run -d --name ipfs1 -p 5002:5001 -p 8081:8080 ipfs/go-ipfs
docker run -d --name ipfs2 -p 5003:5001 -p 8082:8080 ipfs/go-ipfs
```

### A. Confirm IPFS Nodes Are Running

```bash
docker ps
```
You should see `ipfs0`, `ipfs1`, and `ipfs2`.

### B. Get Peer Addresses

```bash
docker exec -it ipfs0 ipfs id
docker exec -it ipfs1 ipfs id
docker exec -it ipfs2 ipfs id
```
Save the `ID` and `Addresses` for each.

### C. Peer the Nodes

Connect all nodes to each other (replace `<multiaddress>` as appropriate):

```bash
docker exec -it ipfs0 ipfs swarm connect <ipfs1_multiaddress>
docker exec -it ipfs0 ipfs swarm connect <ipfs2_multiaddress>
docker exec -it ipfs1 ipfs swarm connect <ipfs0_multiaddress>
docker exec -it ipfs1 ipfs swarm connect <ipfs2_multiaddress>
docker exec -it ipfs2 ipfs swarm connect <ipfs0_multiaddress>
docker exec -it ipfs2 ipfs swarm connect <ipfs1_multiaddress>
```

### D. Test File Replication

Add a file with:

```bash
docker exec -it ipfs0 ipfs add /etc/hosts
```
Copy the CID and check on other nodes:

```bash
docker exec -it ipfs1 ipfs cat <CID>
docker exec -it ipfs2 ipfs cat <CID>
```
You should see the file content, confirming replication.

---

## 5. Setup: Backend Orchestrator (Node.js)

### A. Initialize Backend

```bash
mkdir ../backend && cd ../backend
npm init -y
npm install express axios dotenv cors
```

### B. Create API Server

**Create `index.js`:**

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

**Create `virusTotal.js`:**

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

**Create `.env`:**

```
VIRUSTOTAL_API_KEY=your_virustotal_api_key_here
```

### C. Start Backend

```bash
node index.js
```

---

## 6. Integration and Testing

### A. Upload a File to IPFS

**Example Node.js script:**

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

// Usage:
uploadToIpfs('my-encrypted-file.bin', 'http://localhost:5001')
  .then(cid => console.log('File uploaded to IPFS with CID:', cid));
```

### B. Register CID in Smart Contract

- Use `ethers.js` to call your contract (connect to each node’s endpoint as needed).

### C. Scan File with VirusTotal via Backend

- Send the encrypted file as base64 to backend `/api/scan` endpoint.

---

## 7. Node Topology

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

## 8. Troubleshooting

- **Docker:** Use `docker ps` to check containers, `docker logs <container>` for errors.
- **IPFS:** Double-check peer addresses and firewall settings if nodes don’t connect/replicate.
- **Hardhat:** Ensure each node has a unique port. If you want advanced consensus between nodes, consider Geth/Parity with Clique PoA.
- **Backend:** Verify `.env` and API key.

---

## 9. Checklist

- [ ] 3-4 Hardhat blockchain nodes running on separate ports.
- [ ] 3 IPFS nodes running, peered, and replicating files.
- [ ] Backend orchestrator running and responding to scan requests.
- [ ] Can upload to IPFS, register CID in contract, and get scan result from backend.

---

**Phase 2 complete!  
You are now ready for contract development, integration, and full workflow in Phase 3.**
