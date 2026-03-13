# VAULTLESS

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![React](https://img.shields.io/badge/react-18.2.0-61DAFB?logo=react)](https://reactjs.org/)
[![Vite](https://img.shields.io/badge/vite-5.1.0-646CFF?logo=vite)](https://vitejs.dev/)
[![Solidity](https://img.shields.io/badge/solidity-0.8.x-363636?logo=solidity)](https://docs.soliditylang.org/)
[![Sepolia](https://img.shields.io/badge/network-Sepolia-4a5efd)](https://sepolia.org)

---

## 🧠 What is VAULTLESS?

**VAULTLESS** is a passwordless authentication system powered by **behavioral biometrics** (keystroke dynamics + mouse motion) and anchored on **Ethereum Sepolia** via a lightweight smart contract.

Users enroll by typing a phrase multiple times. Their typing/mouse behavior is transformed into a normalized vector and stored on-chain. Subsequent authentication attempts are scored using cosine similarity to detect legitimate access, duress, or rejection.

---

## 🚀 Features

- ✅ Passwordless enrollment (phrase: **“Secure my account”**, typed **3×**)
- ✅ Fast biometric score calculation using cosine similarity
- ✅ Duress detection (score **0.55–0.85**) triggers a **silent EmailJS alert** + serves a **Ghost session** (fake dashboard)
- ✅ On-chain vector hash storage: network-verified, tamper-resistant
- ✅ Demo mode toggles between real on-chain calls and local stubs
- ✅ Animated UI with Framer Motion & interactive charts via Recharts

---

## 🔍 How It Works

### 1) Enrollment

1. User types the enrollment phrase **“Secure my account”** exactly **3 times**.
2. The system records:
   - **Hold times** (key down → key up)
   - **Flight times** (key up → next key down)
   - Mouse movement statistics while typing
3. The data is transformed into a **Float32Array[64] behavioral vector** and normalized.
4. A hash of the vector is stored on-chain via `VaultlessCore.sol`.

### 2) Authentication

1. User types the same phrase once.
2. The engine computes a similarity score against the enrolled vector.
3. Score thresholds:
   - **> 0.85** → **Authenticated**
   - **0.55 – 0.85** → **Duress** (silent alert + Ghost session)
   - **< 0.55** → **Rejected**

### 3) Duress Handling

- Duress triggers a **silent EmailJS alert** (see `src/lib/duressAlert.js`).
- The UI shows a **Ghost session** (fake dashboard) to avoid alerting an attacker.

---

## 📦 Behavioral Vector Breakdown

| Vector Region | Length | Weight | Description |
|---|---|---|---|
| Hold time (key press duration) | 32 | **×3** | Captures timing between keydown and keyup per key |
| Flight time (inter-key latency) | 16 | **×2** | Captures timing between successive keys |
| Typing stats | 8 | 1× | Mean, variance, error rate, etc. |
| Mouse dynamics | 8 | 1× | Speed, acceleration, jitter, path length |

> The vector is stored as `Float32Array[64]` and normalized before cosine similarity.

---

## 🧩 Tech Stack

| Layer | Tools |
|---|---|
| Frontend | React 18, Vite, Tailwind CSS, Framer Motion, Recharts |
| Blockchain | Solidity (`VaultlessCore.sol`), Ethers.js v6, Sepolia testnet |
| Wallet | MetaMask |
| RPC | Alchemy (Sepolia) |
| Alerts | EmailJS |
| Fonts | Space Grotesk, JetBrains Mono, Inter |

---

## 🗂️ Key Files

- `src/hooks/behaviouralEngine.js` — cosine similarity, stress detection, vector normalization
- `src/lib/ethereum.js` — Ethers.js contract calls + demo stubs (demo mode)
- `src/lib/VaultlessContext.jsx` — global state, wallet/chain management
- `src/lib/duressAlert.js` — EmailJS alert logic for duress
- `src/pages/Enroll.jsx` — enrollment flow
- `src/pages/Auth.jsx` — authentication flow
- `src/pages/Dashboard.jsx` — real dashboard after success
- `src/pages/Ghost.jsx` — ghost session after duress
- `src/contracts/VaultlessCore.sol` — smart contract to deploy on Sepolia

---

## ✅ Getting Started

### 1) Install dependencies

```bash
npm install
```

### 2) Configure environment variables

Copy the example and fill in your values:

```bash
cp .env.example .env
```

Update `.env` with:
- `VITE_CONTRACT_ADDRESS` (post-deployment contract address)
- `VITE_ALCHEMY_KEY` (Sepolia RPC)
- `VITE_EMAILJS_*` values (EmailJS service/template/public key)
- `VITE_ALERT_EMAIL` (email for duress alerts)
- `VITE_DEMO_MODE` (`true` for mocked backend, `false` for on-chain)

### 3) Start dev server

```bash
npm run dev
```

Then open the URL shown in the terminal (usually `http://localhost:5173`).

---

## 🛠️ Smart Contract Deployment (Sepolia)

> This repo does **not** include a deployment pipeline. Deploy `src/contracts/VaultlessCore.sol` yourself (e.g., Remix, Hardhat, Foundry).

### A) Deploy via Remix (quickest)

1. Open [Remix](https://remix.ethereum.org/).
2. Create a new file and paste in `src/contracts/VaultlessCore.sol`.
3. Select `Injected Web3` and connect MetaMask (set to Sepolia).
4. Compile and deploy.
5. Copy the deployed contract address into `.env` as `VITE_CONTRACT_ADDRESS`.

### B) (Optional) Deploy with Hardhat / Foundry

1. Initialize a Hardhat/Foundry project alongside this repo.
2. Configure Sepolia RPC URL using `VITE_ALCHEMY_KEY`.
3. Deploy and update `VITE_CONTRACT_ADDRESS`.

---

## 🌐 Network Configuration

- **Network:** Sepolia
- **RPC endpoint:** `https://eth-sepolia.g.alchemy.com/v2/<YOUR_ALCHEMY_KEY>`
- **Chain ID:** `11155111`

In MetaMask, add/enable Sepolia and ensure you have some Sepolia ETH (faucet).

---

## 🧪 Tuning Guide (Thresholds & Stability)

### Score thresholds
- **Authenticated:** `score > 0.85`
- **Duress:** `0.55 <= score <= 0.85`
- **Rejected:** `score < 0.55`

### Improve enrollment quality
- Ensure users type the phrase **3x** in a row in a quiet environment.
- Use the same keyboard layout (e.g., QWERTY) between enrollment and auth.

### Adjusting sensitivity
- Fine-tune thresholds in `src/hooks/behaviouralEngine.js`.
- Consider adding adaptive thresholds (e.g., based on historical variance).

### Demo mode
- Set `VITE_DEMO_MODE=true` to run without an on-chain wallet.
- Demo mode uses stubs in `src/lib/ethereum.js`.

---

## 🧪 Testing & Debugging

- Use the browser console to inspect the computed vector and score.
- The `VaultlessContext` exposes debug state (wallet, chain, score, duress events).

---

## 📦 Folder Structure (High Level)

```
src/
  components/        # UI widgets
  contracts/         # Solidity sources
  hooks/             # biometric engine + hooks
  lib/               # Ethereum + email helpers
  pages/             # Enroll / Auth / Dashboard / Ghost
  styles/            # Tailwind + custom styles (if present)
  App.jsx
  main.jsx
```

---

## 🔖 License

This project is provided as-is under the MIT License. Update as needed.
