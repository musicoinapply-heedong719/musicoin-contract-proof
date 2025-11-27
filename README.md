# Musicoin 3.0 – Contract Proof & Document Archive

## Overview

The Musicoin 3.0 Contract Proof & Document Archive is a system for storing
and verifying important documents related to music business:

- Contracts (e.g. between artist and label)
- Agreements (e.g. revenue splits, licensing)
- Other critical records that need tamper-resistant proof

The focus is on **proof-of-existence and integrity**, not on storing full
documents on-chain. Large files are stored off-chain, while hashes and
references are stored on-chain.

## Core Goals

- Provide a trustworthy proof layer for music-related contracts
- Allow any party to verify that a given document existed at a certain time
- Integrate with other Musicoin 3.0 services (e.g. instrument history, ticketing)

## Key Features

- Document registration:
  - Document hash (e.g. SHA-256)
  - Metadata: type, title, parties involved
  - Optional linkage to artist profile, instrument, or event
- Verification:
  - Given a document file, recompute hash and compare to on-chain record
- Versioning:
  - Multiple versions of a contract with clear timestamps
- Access:
  - On-chain data is public
  - Actual document content stored off-chain (encrypted if needed)

## Architecture

- **Smart Contract**:
  - Registry of document hashes and basic metadata
- **Frontend**:
  - UI for registering and verifying documents
- **Backend**:
  - Optional storage solution (encrypted files, links, etc.)

## Typical Workflow

1. Parties finalize a contract as a PDF.
2. The file hash is computed locally.
3. A registration transaction is sent to the contract with:
   - Hash
   - Title
   - Participant addresses
4. Later, anyone can:
   - Recompute the hash of the same PDF
   - Compare with on-chain hash to verify integrity and timestamp

## Roadmap

1. Basic hash registration and lookup
2. Version control and diff metadata
3. Integration with instrument and ticketing systems
4. GUI for non-technical users

## Contribution

- UI for non-technical artists and managers
- Templates and tools for hash calculation
- Best-practice guides for legal usage

## License

TBD.

