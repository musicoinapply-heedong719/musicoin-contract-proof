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
