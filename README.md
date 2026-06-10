# AlphaHoldem System Architecture Design Document (V1.0)

> **Core Positioning**: A hybrid Web3 architecture based on "On-chain Settlement + Off-chain TEE Confidential Execution."
> **Pain Points Solved**: Eliminates the high latency and exorbitant Gas fees of pure on-chain poker, while shattering the trust black box of centralized platforms (cryptographically preventing fund misappropriation and hole-card peeking).
> **Technical Vision**: To build a decentralized mind-sport network supporting everything from micro-stakes Wasm logic battles to high-roller LLM real-time AI gaming.

---

## I. Core Architecture Matrix Summary

| Module Name | Core Tech Stack | Primary Responsibilities | Data Settlement Layer |
| --- | --- | --- | --- |
| **Settlement Protocol** | Rust / Solana Anchor | Fund custody, buy-in/cash-out, forward seed commitment | On-chain (Solana) |
| **TEE Engine** | .NET 10 / Wasmtime | Fair dealing, state machine, compute sandbox isolation | Off-chain (TEE Blackbox) |
| **Data Availability (DA)** | IPFS / Pinata API | Hand history archiving, AI training dataset generation | Off-chain (IPFS Network) |
| **Lobby Tracker** | Go / Node.js / WSS | Event listening, table state discovery and broadcasting | Off-chain (Centralized) |
| **Visual Client** | Unity 3D / WebGL | 3D rendering, WSS battle comms, wallet signature UX | User Local Terminal |
| **Strategy CLI** | Rust CLI | Bot template scaffolding, local sandbox debugging | Developer Local Env |

---

## II. Detailed Module Breakdown

### 1. On-chain Settlement Protocol (`ahc-protocol`)

The **Absolute Trust Layer** and financial vault of the system. All value transfers are locked by smart contracts, bypassing human intervention.

* **Vault Management**: Manages global or per-table AHC token liquidity. Funds cannot be moved without cryptographic signatures.
* **Buy-in / Cash-out**: Handles the locking of player funds upon joining a table, and the release of funds based on TEE signatures upon leaving.
* **Rule Anchoring (TableState)**: Records the parameters of individual tables, including blind levels, max capacity, and permitted compute types (Wasm/LLM).
* **Forward Commitment (Commit-Reveal)**: Receives and anchors the hash of the TEE's random seed *before* cards are dealt. The plaintext seed is revealed post-hand, allowing network-wide verification of deck integrity.

### 2. Confidential Computing Engine (`enclave-dealer`)

The **Physical Black Box** of AlphaHoldem, ensuring absolute fairness in dealing and arbitration.

* **Provably Fair Dealing**: Utilizes CSPRNG to generate strong cryptographic random seeds for shuffling.
* **High-Frequency State Machine**: Maintains seat rotation, chip stacks, and enforces a strict 3-second "Shot Clock" timeout fold logic.
* **Compute Sandbox Isolation**:
* Securely mounts and executes `.wasm` strategy binaries.
* Constructs game context JSONs to call external LLMs, processing incremental state updates (JSON Patch).


* **Arbitration & Signature**: Compares hand strengths at showdown, generates a payload containing the winner's data, and signs it using the internal TEE private key for on-chain withdrawal.

### 3. Data Availability & Archive Layer (DA Layer)

The **Self-Proving Mechanism** and the ultimate data fountain for the AI ecosystem.

* **Hand History Snapshots**: Packages player actions, hole cards, and the plaintext Seed into a standard JSON file after every hand.
* **Async Pinning**: Asynchronously pushes the JSON payload to the IPFS network to retrieve a unique CID (Content Identifier).
* **On-chain Fingerprints**: Anchors the CID to the Solana smart contract, achieving permanent audit trails at near-zero cost.
* **Open-Source Datasets**: Provides a permissionless, massive real-world poker dataset for Wasm and LLM bot training.

### 4. Tracker & Visual Client

Handles information distribution and user experience, strictly separating heavy computation from the presentation layer.

* **Lobby Tracker (`ahc-tracker`)**: A lightweight middleware that listens to on-chain `TableCreated` events, maintains an in-memory list of active tables, and broadcasts routing information to front-end clients via WebSockets.
* **Visual Client (`showdown-client`)**: Integrates Phantom/Solflare wallets for transaction signing. Renders a cyberpunk-style 3D table, visualizes player avatars (e.g., gears for Wasm, neural networks for LLM), and establishes a direct WebSocket connection with the TEE engine for ultra-low latency gameplay.

### 5. Strategy CLI Tool (`cargo-ahc`)

A developer-focused toolchain designed to lower the barrier to entry for quants and AI developers.

* **One-Click Scaffolding**: Run `cargo ahc init my_bot` to instantly generate a standard Wasm strategy directory structure.
* **Local Sandbox Debugging**: Features a built-in, lightweight state machine, allowing developers to test their strategies locally via CLI without connecting to the mainnet.
* **Build & Publish**: Compiles the strategy into a standard binary, signs it, and uploads it to the platform.

---

## III. Core Data Workflow (Lifecycle of a Hand)

1. **Table Discovery**: `ahc-tracker` scans the Solana blockchain, detects a new table hosted by `enclave-dealer`, and pushes the lobby data to the Unity client.
2. **Buy-in & Staking**: The player authorizes the transaction via their wallet in Unity; `ahc-protocol` securely locks the player's AHC into the smart contract Vault.
3. **Engine Takeover**: `enclave-dealer` verifies the successful deposit and seats the player. Once the minimum player count is reached, the TEE generates a seed, shuffles the deck, and submits the Hash to the blockchain (Forward Commitment).
4. **High-Frequency Gaming**: The TEE pushes hole cards to the players. Wasm or LLM strategies take turns executing decisions (Check/Fold/Raise). All actions occur off-chain via WebSockets in milliseconds.
5. **Clearance & Archiving**: The hand concludes. The TEE determines the winner, packages the complete action JSON and plaintext Seed, and pushes it to IPFS to get the CID. The TEE then signs the winner's payload + IPFS CID with its private key and sends it to the `ahc-protocol` to execute the actual AHC payout and permanent on-chain record keeping.
