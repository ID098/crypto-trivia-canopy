# 🎮 CryptoTrivia — DApp on Canopy Network

> A decentralized crypto trivia quiz game built on the **Canopy Network** using **TypeScript**.  
> Answer questions. Earn CNPY tokens. Dominate the leaderboard.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Canopy Architecture](#canopy-architecture)
- [Features](#features)
- [Game Flow](#game-flow)
- [Transaction Types (FSM)](#transaction-types-fsm)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Connecting to a Real Canopy Node](#connecting-to-a-real-canopy-node)
- [Roadmap](#roadmap)
- [Resources](#resources)

---

## Overview

**CryptoTrivia** is a decentralized application (dapp) running as an **appchain** on the [Canopy Network](https://www.canopynetwork.org/). It demonstrates how to architect a game using Canopy's Finite State Machine (FSM) model in TypeScript — the same pattern used by all sovereign chains built on the Canopy Stack.

Players register a wallet, answer 5 randomly selected crypto trivia questions per round, earn **CNPY tokens** for correct answers, and their scores are recorded permanently on-chain. A live leaderboard ranks all players by total score.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Appchain Protocol | Canopy Network (Nested Chain) |
| Smart Logic Language | TypeScript |
| Frontend Framework | React (JSX) |
| Styling | CSS-in-JS (inline `<style>`) |
| Fonts | Orbitron · Share Tech Mono (Google Fonts) |
| State Model | Canopy FSM (Finite State Machine) |
| RPC Layer | Mock (swap for Canopy node RPC on mainnet) |

---

## Canopy Architecture

This project follows Canopy's canonical **appchain architecture**, where all application logic is expressed as state transitions via typed transactions processed by an FSM engine.

```
┌─────────────────────────────────────────────────────┐
│                  CANOPY ROOT CHAIN                   │
│           (Shared Security · NestBFT PoS)            │
└────────────────────┬────────────────────────────────┘
                     │ Nested Chain (Day-0 Security)
┌────────────────────▼────────────────────────────────┐
│              CryptoTrivia Appchain                   │
│               Chain ID: cnpy-trivia-1                │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌────────────────────┐ │
│  │Controller│  │   FSM    │  │   P2P · BFT · RPC  │ │
│  │  (Bus)   │◄─►│ Engine  │  │     (Networking)   │ │
│  └──────────┘  └────┬─────┘  └────────────────────┘ │
│                     │                                │
│          applyTransaction(state, tx) => newState     │
└─────────────────────────────────────────────────────┘
```

### Core Modules (mirrored from Canopy Go reference)

| Module | Role |
|---|---|
| **Controller** | Central hub coordinating all chain components |
| **FSM Engine** | Processes typed transactions and updates state |
| **BFT Consensus** | Agrees on new blocks even with faulty nodes |
| **P2P Network** | Encrypted peer-to-peer node communication |
| **RPC Layer** | External interface for submitting transactions |

---

## Features

- ✅ **Wallet Connect** — generates a unique `cnpy1...` address per player
- ✅ **Token Economy** — players start with 50 CNPY; earn tokens per correct answer
- ✅ **15 Crypto Questions** — randomized 5-question rounds with varied reward tiers
- ✅ **On-Chain Leaderboard** — sorted by cumulative score across all rounds
- ✅ **Transaction Mempool** — live view of all on-chain transactions with hashes
- ✅ **Chain State Panel** — block height, FSM module status, active sessions
- ✅ **Claim Reward Flow** — explicit `CLAIM_REWARD` transaction to finalize earnings
- ✅ **Canopy FSM Pattern** — pure `applyTransaction(state, tx) => newState` architecture

---

## Game Flow

```
[Player] → REGISTER_PLAYER tx
               │
               ▼
          START_QUIZ tx  ──► 5 random questions loaded from on-chain state
               │
               ▼
    SUBMIT_ANSWER tx (×5)
       ├─ Correct → +reward CNPY (pending)
       └─ Wrong   → +0 CNPY
               │
               ▼
         CLAIM_REWARD tx
               │
               ▼
    Balance updated · Leaderboard refreshed · Block height advances
```

---

## Transaction Types (FSM)

All state changes in the appchain go through the FSM as typed transactions:

### `REGISTER_PLAYER`
```typescript
{
  type: "REGISTER_PLAYER",
  address: "cnpy1ABC123...",
  payload: { name: "PlayerName" }
}
```
Creates a new player account with **50 CNPY** starting balance.

---

### `START_QUIZ`
```typescript
{
  type: "START_QUIZ",
  address: "cnpy1ABC123...",
  payload: {}
}
```
Opens a new session with 5 randomly selected questions. Previous unclaimed sessions are cleared.

---

### `SUBMIT_ANSWER`
```typescript
{
  type: "SUBMIT_ANSWER",
  address: "cnpy1ABC123...",
  payload: { answerIdx: 1 }  // 0–3
}
```
Records the player's answer. Correct answers add to `pendingReward`. Advances `currentIdx`.

---

### `CLAIM_REWARD`
```typescript
{
  type: "CLAIM_REWARD",
  address: "cnpy1ABC123...",
  payload: {}
}
```
Finalizes the round: transfers `pendingReward` to balance, updates `totalScore`, and refreshes the leaderboard. Only callable once the quiz session is complete.

---

## Project Structure

```
canopy-trivia-dapp/
│
├── canopy-trivia-dapp.jsx   # Main application (React + Canopy FSM logic)
│
└── README.md                # This file
```

### Key Sections Inside `canopy-trivia-dapp.jsx`

```
// ─── TYPES ────────────────────── TxType enum (4 transaction types)
// ─── QUESTIONS DB ─────────────── 15 crypto trivia questions + rewards
// ─── FSM ENGINE ───────────────── applyTransaction(state, tx) => newState
// ─── MOCK RPC ─────────────────── useCanopyChain() hook (swap for real node)
// ─── WALLET ───────────────────── useWallet() with address generation
// ─── UI COMPONENTS ────────────── React UI (connect, game, leaderboard, chain)
```

---

## Getting Started

### Prerequisites

- Node.js v18+
- npm or yarn

### Install & Run

```bash
# Clone the repository
git clone https://github.com/your-username/canopy-trivia-dapp
cd canopy-trivia-dapp

# Install dependencies
npm install react react-dom

# If using Vite:
npm create vite@latest . -- --template react
# Replace src/App.jsx with canopy-trivia-dapp.jsx

npm run dev
```

### Using with Create React App

```bash
npx create-react-app canopy-trivia --template typescript
cd canopy-trivia
# Replace src/App.tsx with the contents of canopy-trivia-dapp.jsx
npm start
```

---

## Connecting to a Real Canopy Node

The current implementation uses an **in-memory mock RPC layer**. When Canopy's TypeScript SDK ships and the testnet/mainnet is live, replace `useCanopyChain()` with real node calls:

### Step 1 — Replace the mock dispatcher

```typescript
// Current (mock)
const sendTransaction = useCallback((tx) => {
  setState(prev => applyTransaction(prev, { ...tx, hash: mockHash() }));
}, []);

// Production (Canopy RPC)
const sendTransaction = useCallback(async (tx) => {
  const signedTx = await wallet.sign(tx);          // Sign with private key
  const response = await fetch("https://your-canopy-node:26657/broadcast_tx", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(signedTx),
  });
  return response.json();
}, [wallet]);
```

### Step 2 — Subscribe to chain state

```typescript
// Poll or WebSocket for latest state
const fetchState = async () => {
  const res = await fetch("https://your-canopy-node:26657/app_state");
  return res.json();
};
```

### Step 3 — Deploy your FSM to the appchain

Follow [Canopy Templates](https://www.canopynetwork.org/) to deploy the `applyTransaction` logic as your chain's FSM, written in TypeScript, in ~200 lines.

---

## Question Reward Tiers

| Difficulty | Reward |
|---|---|
| Basic (e.g. acronyms, definitions) | 10 CNPY |
| Intermediate (e.g. consensus, wallets) | 15 CNPY |
| Advanced (e.g. smart contracts, Layer 2) | 20–25 CNPY |
| Canopy-specific | 30 CNPY |

Maximum possible reward per 5-question round: **~100 CNPY**

---

## Roadmap

- [ ] Connect to Canopy public testnet RPC
- [ ] Wallet signature with real keypair (ed25519 / secp256k1)
- [ ] Persistent leaderboard via Canopy chain query
- [ ] Time-limited rounds (30 sec per question)
- [ ] Multiplayer head-to-head mode
- [ ] NFT trophy for top-ranked players
- [ ] Admin panel to add community-submitted questions
- [ ] CNPY staking to enter high-reward tournaments

---

## Resources

| Resource | Link |
|---|---|
| Canopy Network | https://www.canopynetwork.org |
| Canopy Docs | https://canopy-network.gitbook.io/docs |
| Canopy GitHub (Go reference) | https://github.com/canopy-network/canopy |
| Canopy Testnet | Active as of February 2026 |
| Community / Discord | Via canopynetwork.org |

---

## License

MIT © 2026 — Built on [Canopy Network](https://www.canopynetwork.org/)

> *"Blockchain development has been far too complex for far too long."*  
> — Adam Liposky, CEO of Canopy Network
