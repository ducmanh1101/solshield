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
- Custom Anchor program: `sol-shield-sdk` (see [programs/](../programs/))

**Document handling**

- `pdfjs-dist` and `@react-pdf-viewer` for rendering PDF contracts in the browser
- `base64topdf` for encoding and decoding contract payloads
- `md5` for client-side document hashing
- `localforage` for offline-friendly local storage

## Project layout

```
app/
├── public/           Static assets and HTML shell
├── src/
│   ├── components/   Reusable UI components
│   ├── configs/      Solana cluster and program configuration
│   ├── hooks/        Custom React hooks
│   ├── static/       Images, fonts, and other static resources
│   ├── store/        Redux store, slices, and selectors
│   ├── view/         Page-level views and routes
│   ├── watcher/      Side effects and on-chain event watchers
│   ├── index.tsx     Application entry point
│   └── utils.ts      Shared utilities
├── craco.config.js
├── package.json
└── tsconfig.json
```

## Getting started

### Prerequisites

- Node.js 16 or newer
- Yarn or npm
- A Solana wallet browser extension (for example Phantom)

### Install

```bash
cd app
npm install
```

### Run the development server

```bash
npm start
```

The app will be available at `http://localhost:3000`.

### Build for production

```bash
npm run build
```

The production bundle will be emitted to `app/build`.

### Run tests

```bash
npm test
```

## Related

- **On-chain program:** [programs/sol-shield-sdk](../programs/sol-shield-sdk) – the Anchor program that powers contract and signer accounts.
- **Program tests:** [tests/](../tests/)

## License

This project was built for the Solana Vietnam Coding Camp. See the repository root for licensing details.
