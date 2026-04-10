# SolShield

**Decentralized digital contract signing on Solana.**

SolShield is a blockchain-based platform that lets individuals and organizations create, sign, and verify contracts entirely on-chain. By replacing paper, fax, and physical shipping with cryptographically verifiable e-signatures, SolShield removes the friction and forgery risks of traditional contract workflows while keeping every agreement transparent, immutable, and auditable.

Live app: https://solshield.onrender.com/

## Award

SolShield received **First Prize in the U20 category** at **Solana Vietnam Coding Camp Season 2**. You can read the official write-up on the Superteam Vietnam newsletter: https://superteamvn.substack.com/p/phan-tich-10-du-an-oat-giai-solana

## Why SolShield

Traditional contract management suffers from several well-known problems:

- Physical documents can be lost, damaged, or destroyed.
- Paper contracts are susceptible to forgery and tampering.
- Printing, storing, and shipping paperwork wastes time and resources.
- Signing across different locations is slow and logistically complex.

SolShield addresses these issues by anchoring contracts to the Solana blockchain, where every signature and document hash is publicly verifiable and cannot be altered after the fact.

## How it works

1. **Upload** – A user uploads a contract document (typically a PDF) through the web app.
2. **Hash** – The client computes a cryptographic hash of the file, so the on-chain record can prove the document has not been modified.
3. **On-chain accounts** – Two Solana accounts are created for each agreement: one that stores the contract metadata and one that tracks the list of signers.
4. **Sign** – Each party connects their Solana wallet and signs the contract transaction. Signatures are recorded on-chain and tied to the signer's wallet address.
5. **Finalize** – Once every required signer has signed, the contract is finalized and becomes a tamper-proof, verifiable record that any party can audit at any time.

## Tech stack

**Frontend**

- React 18 with TypeScript
- Redux Toolkit and React Redux for state management
- React Router for navigation
- Ant Design for UI components
- AOS for scroll animations
- CRACO for build configuration

**Solana integration**

- `@project-serum/anchor` for interacting with the on-chain program
- `@solana/wallet-adapter-react` and `@solana/wallet-adapter-wallets` for multi-wallet connection
- Custom Anchor program: `sol-shield-sdk` (see [programs/sol-shield-sdk](programs/sol-shield-sdk))

**Document handling**

- `pdfjs-dist` and `@react-pdf-viewer` for rendering PDF contracts in the browser
- `base64topdf` for encoding and decoding contract payloads
- `md5` for client-side document hashing
- `localforage` for offline-friendly local storage

## On-chain program

The smart contract lives in [programs/sol-shield-sdk](programs/sol-shield-sdk) and is written in Rust using the [Anchor](https://www.anchor-lang.com/) framework (v0.25.0).

**Program ID:** `33eJhB9P4Kmf51MUpRgpJpdDab2MWP9FmpVHrASspJ3B`

### Accounts

The program defines two PDA-based account types:

**`Contract`** — represents a single agreement on-chain.

| Field          | Type         | Description                                             |
| -------------- | ------------ | ------------------------------------------------------- |
| `authority`    | `Pubkey`     | Wallet that created and owns the contract              |
| `hash`         | `[u8; 16]`   | Cryptographic hash of the underlying document           |
| `expired_at`   | `i64`        | Unix timestamp after which the contract is no longer valid |
| `total_signer` | `u8`         | Total number of signers registered for the contract    |
| `total_signed` | `u8`         | Number of signers who have already signed              |
| `state`        | `ContractState` | `Uninitialized`, `Initialized`, `Processing`, or `Approved` |

PDA seeds: `["contract", hash]`.

**`ContractSigner`** — represents one signer's participation in one contract.

| Field       | Type                  | Description                             |
| ----------- | --------------------- | --------------------------------------- |
| `authority` | `Pubkey`              | Wallet expected to sign the contract    |
| `contract`  | `Pubkey`              | Reference to the associated contract    |
| `state`     | `ContractSignerState` | `Uninitialized`, `Initialized`, or `Singed` |

PDA seeds: `["signer", contract, signer_authority]`.

### Instructions

The program exposes four instructions that together implement the full signing lifecycle:

1. **`create_contract(hash, expired_at)`** – The contract owner initializes a new `Contract` PDA derived from the document hash. The contract starts in the `Initialized` state with zero signers.
2. **`create_signer()`** – While the contract is still `Initialized`, the owner registers each required signer by creating a `ContractSigner` PDA for that wallet. Each call increments `total_signer`.
3. **`active_contract()`** – The owner transitions the contract from `Initialized` to `Processing`, freezing the signer list and opening the contract for signatures.
4. **`sign_contract()`** – Each registered signer calls this with their own wallet while the contract is in `Processing`. The matching `ContractSigner` is flipped to `Singed` and `total_signed` is incremented. When `total_signed == total_signer`, the contract automatically advances to the final `Approved` state.

### Error codes

Defined in [programs/sol-shield-sdk/src/errors.rs](programs/sol-shield-sdk/src/errors.rs):

- `Overflow` – arithmetic overflow during an update.
- `InvalidState` – the contract is not in the expected state for the requested instruction.
- `InvalidPermission` – the caller is not authorized to perform the action.

## Project layout

```
solshield/
├── app/                      React frontend
│   ├── public/
│   ├── src/
│   │   ├── components/       Reusable UI components
│   │   ├── configs/          Solana cluster and program configuration
│   │   ├── hooks/            Custom React hooks
│   │   ├── static/           Images, fonts, and other static resources
│   │   ├── store/            Redux store, slices, and selectors
│   │   ├── view/             Page-level views and routes
│   │   ├── watcher/          Side effects and on-chain event watchers
│   │   ├── index.tsx         Application entry point
│   │   └── utils.ts          Shared utilities
│   ├── craco.config.js
│   ├── package.json
│   └── tsconfig.json
├── programs/
│   └── sol-shield-sdk/       Anchor program (Rust)
│       └── src/
│           ├── lib.rs            Program entry and instruction router
│           ├── constants.rs      Byte-size constants for account layouts
│           ├── errors.rs         Custom error codes
│           ├── instructions/     create_contract, create_signer, active_contract, sign_contract
│           └── schema/           Contract and ContractSigner account definitions
├── tests/                    Anchor integration tests (TypeScript)
├── migrations/               Anchor deployment scripts
├── Anchor.toml               Anchor workspace configuration
└── Cargo.toml                Rust workspace configuration
```

## Getting started

### Prerequisites

- Node.js 16 or newer
- Yarn or npm
- Rust and the Solana CLI
- [Anchor](https://www.anchor-lang.com/) 0.25.0
- A Solana wallet browser extension (for example Phantom)

### Frontend

```bash
cd app
npm install
npm start
```

The app will be available at `http://localhost:3000`. To produce a production bundle, run `npm run build` from the `app` directory — the output is written to `app/build`.

### On-chain program

From the repository root:

```bash
anchor build      # compile the sol-shield-sdk program
anchor test       # run the TypeScript integration tests under tests/
anchor deploy     # deploy to the cluster configured in Anchor.toml
```

The target cluster and wallet path are configured in [Anchor.toml](Anchor.toml).

## License

This project was built for the Solana Vietnam Coding Camp. See the repository root for licensing details.
