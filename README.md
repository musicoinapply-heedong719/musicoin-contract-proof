# Musicoin 3.0 – Contract Proof & Document Archive
**Repository:** `musicoin-contract-proof`

## Overview

The **Musicoin Contract Proof & Document Archive** provides a tamper-resistant, time-stamped proof layer  
for agreements and important documents in the Musicoin 3.0 ecosystem.

Instead of storing entire documents on-chain, this project focuses on:

- Storing **cryptographic hashes** of documents (proof-of-existence & integrity)
- Recording **timestamps, involved parties, and linkage to Artist IDs / instruments / tickets**
- Providing an easy way to **verify** that a given file matches the original registered version

This system is designed to support:

- Artist–label agreements
- Revenue-sharing contracts
- Licensing agreements (samples, stems, remixes, etc.)
- Ticketing and event-related documents
- Instrument provenance certificates
- Any other document that needs long-term, verifiable proof

---

## Core Goals

- Provide a **neutral, verifiable registry** for contracts and important documents
- Ensure that **documents cannot be secretly modified** after registration
- Allow any party to verify:
  - “This file existed at this time”
  - “These addresses were involved”
  - “This hash has not changed”
- Integrate seamlessly with other Musicoin 3.0 services

---

## Key Features

### 1. Document Hash Registration
- Store only the **hash** of a file (e.g., SHA-256)
- Record:
  - Hash value
  - Owner / submitter address
  - Related Artist ID(s) (via `musicoin-artist-id`)
  - Optional: instrument ID, event ID, etc.
  - Document type (contract, license, certificate, etc.)
  - Human-readable title

### 2. Proof-of-Existence & Integrity
- Any party can:
  1. Compute the hash of a document locally
  2. Query the contract using that hash
  3. Confirm whether it is registered, by whom, and when

### 3. Versioning
- Multiple versions of the same contract can be registered:
  - `baseContractId` or `parentHash` linkage
  - Clear chronological history of amendments / revisions

### 4. Linkage to Musicoin Identity & Assets
- Link to:
  - **Artist ID** (from `musicoin-artist-id`)
  - Instrument IDs (`instrument-history`)
  - Event IDs (`ticketing platform`)
  - Track IDs (Musicoin content registry)
- Makes it easy to audit: “This contract belongs to this artist / event / instrument.”

### 5. Off-Chain Storage Flexibility
- Documents themselves are **not** stored on-chain
- They may be kept:
  - In private storage (law firm, label, or artist)
  - In encrypted object storage (S3, etc.)
  - In IPFS/Arweave (public or private gateway)
- On-chain record stays valid regardless of storage mechanism, as long as the file hash remains unchanged

---

## Design Principles

- Minimal on-chain footprint (hash + metadata)
- Clear, simple verification process
- Strong links to existing Musicoin identity & content systems
- Legal/forensic usability:
  - Clear timestamps
  - Clear participant addresses
  - Immutable history

---

## Architecture

### 1. Smart Contract Layer

Core responsibilities:
- Register document hashes
- Store minimal metadata
- Track versions and relationships
- Provide read-only view functions

Example Solidity-style interface (simplified):

```solidity
struct DocumentRecord {
    bytes32 hash;           // SHA-256 or Keccak-256 hash
    address submitter;      // Address that registered the document
    uint256 artistId;       // Optional Artist ID (0 if none)
    uint256 relatedId;      // Optional linked entity (instrument, event, track, etc.)
    uint8   relatedType;    // 0 = None, 1 = Instrument, 2 = Event, 3 = Track, ...
    string  docType;        // e.g., "contract", "license", "certificate"
    string  title;          // Short human-readable title
    uint256 createdAt;      // Block timestamp
    bytes32 parentHash;     // Parent document hash for versioning (0x0 if none)
}

function registerDocument(
    bytes32 hash,
    uint256 artistId,
    uint256 relatedId,
    uint8 relatedType,
    string calldata docType,
    string calldata title,
    bytes32 parentHash
) external returns (uint256 docId);

function getDocumentById(uint256 docId) external view returns (DocumentRecord memory);

function getDocumentsByHash(bytes32 hash) external view returns (uint256[] memory docIds);

function getDocumentsByArtist(uint256 artistId) external view returns (uint256[] memory docIds);
```

> 실제 구현 시에는 가스 최적화, 이벤트 설계, 접근 제어 등을 추가해야 함.

---

### 2. Indexing & Backend API

Responsibilities:
- Listen to contract events (`DocumentRegistered`, `DocumentUpdated`, etc.)
- Store normalized data in a database
- Provide search/filter APIs for frontends and external tools

Recommended stack:
- Node.js (NestJS / Express) or Python (FastAPI)
- PostgreSQL / MySQL for relational data
- Redis for cache (option)
- REST or GraphQL API

Example API endpoints:
- `GET /documents/:docId`
- `GET /documents/hash/:hash`
- `GET /documents?artistId=123`
- `GET /documents?relatedType=event&relatedId=858`
- `POST /verify` (upload file or hash, returns matching registry info)

---

### 3. Frontend (Web UI)

Functional goals:
- Let users easily:
  - Register a new document hash
  - View existing documents
  - Verify a document by uploading a file
- Provide a clear, non-technical explanation of:
  - What a document hash is
  - How proof-of-existence works

Recommended stack:
- React / Next.js
- Wallet integration: MetaMask, WalletConnect, etc.
- Simple drag-and-drop file hashing tool (computed **locally** in browser):
  - Compute SHA-256
  - Show hash
  - Use that hash for registration / verification

---

## Data Model (Off-Chain)

Example normalized table:

### `documents`
| Column          | Type        | Description                                |
|----------------|------------|--------------------------------------------|
| id             | bigint     | Internal database ID                       |
| onchain_id     | bigint     | Document ID from the smart contract        |
| hash           | varchar    | Hex string of document hash                |
| submitter      | varchar    | Wallet address                             |
| artist_id      | bigint     | Linked Artist ID (optional)                |
| related_type   | smallint   | 0=None,1=Instrument,2=Event,3=Track,...    |
| related_id     | bigint     | ID of related entity                       |
| doc_type       | varchar    | "contract", "license", "certificate", ...  |
| title          | varchar    | Human-readable title                       |
| parent_hash    | varchar    | Parent document hash (for versioning)      |
| created_at     | timestamp  | First registration time                    |
| tx_hash        | varchar    | Blockchain tx hash                         |

---

## Typical Workflows

### A. Registering a New Contract

1. Prepare final contract as a PDF (or other file format).
2. Locally compute **SHA-256 hash** of the file:
   - Via the web UI (client-side hashing)
   - Or via command-line tool:
     ```bash
     sha256sum contract.pdf
     ```
3. Connect wallet to the Contract Proof dApp.
4. Enter:
   - Document type (e.g., `"contract"`)
   - Title (e.g., `"Artist A – Label B Master Recording Agreement"`)
   - Optional: Artist ID, related instrument/event/track IDs.
5. Submit `registerDocument(...)` transaction.
6. Receive:
   - On-chain document ID
   - Transaction hash
   - Timestamp

### B. Verifying a Document Later

1. Load the saved contract file.
2. Compute SHA-256 hash again.
3. In the UI:
   - Paste hash **or** upload file to compute hash in browser.
4. The system searches:
   - If found:
     - Show on-chain registration info (who, when, linked IDs, title)
   - If not found:
     - Indicate that this exact hash has **no registered record**.

### C. Registering an Amendment / New Version

1. Modify the contract and save as new file.
2. Compute new hash.
3. Call `registerDocument(...)` again with:
   - `parentHash` = previous document hash
4. The system will show:
   - Version chain:
     - v1 → v2 → v3 …

---

## Integration With Other Musicoin Projects

| Project                       | How It Uses Contract Proof                            |
|------------------------------|--------------------------------------------------------|
| `musicoin-artist-id`         | Link artist profiles to contracts & legal documents    |
| `instrument-history`         | Store provenance certificates & sales agreements       |
| `ticketing/event platform`   | Register event contracts / venue agreements            |
| `musicoin-player`            | (Optional) Link rights/license docs to tracks          |
| `sampling-studio`            | Sample license agreements & usage rights proof         |
| `musicoin-analytics-dashboard` | Show counts & categories of registered documents   |

---

## Security & Legal Considerations (High-Level)

- Only **hashes** are stored on-chain, not the document content.
- The system proves:
  - A file with this hash existed at/after a specific time.
  - A specific address registered it.
- It does **not** judge:
  - Whether the content of the contract is fair/valid/legal.
  - Whether all parties agreed (this may require signatures outside chain or off-chain evidence).
- For legal use:
  - Parties should keep the original signed document.
  - Courts or arbitrators can:
    - Compare the original document’s hash with the on-chain hash.
    - Use the timestamp and submitter address as supporting evidence.

(법률적 해석은 각 국가/관할에 따라 달라질 수 있으므로, 실제 소송/계약에서는 추가적인 법률 자문이 필요함.)

---

## Installation

### 1. Clone Repository

```bash
git clone https://github.com/musicoin/musicoin-contract-proof.git
cd musicoin-contract-proof
```

### 2. Smart Contract

```bash
cd contracts
npm install
```

Compile & deploy (example with Hardhat):

```bash
npx hardhat compile
npx hardhat run scripts/deploy.js --network polygon
```

### 3. Backend API

```bash
cd backend
cp .env.example .env
# Fill RPC URL, DB connection, contract address, etc.

npm install
npm run migrate   # if using migrations
npm run dev
```

### 4. Frontend

```bash
cd frontend
cp .env.example .env
# Set BACKEND_API_URL, CHAIN_ID, etc.

npm install
npm run dev
```

---

## Roadmap

### Phase 1 – Core MVP
- Document registration & lookup
- Basic web UI (hash input, file upload verification)
- Backend indexer & REST API

### Phase 2 – Ecosystem Links
- Deep integration with:
  - `musicoin-artist-id`
  - `instrument-history`
  - Ticketing/events
- Document type presets (contract/license/certificate…)

### Phase 3 – Advanced Features
- Batch registration tools (for labels/law firms)
- Printable proof reports (PDF export)
- Role-based views (artist/label/lawyer)

### Phase 4 – Long-Term Enhancements
- Optional zero-knowledge features (prove some properties of a contract without revealing full content)
- Multi-chain deployments & aggregation (if Musicoin contracts distributed across chains)

---

## Contribution

Contributions are welcome in the following areas:

- Solidity contract design and optimization
- Hashing UX (front-end file handling)
- API design and documentation
- Legal/forensic best practices documentation
- Integration examples with other Musicoin repositories

Please open an issue or submit a pull request with clear details.

---

## License

TBD  
(MIT or Apache-2.0 recommended to maximize ecosystem adoption.)
