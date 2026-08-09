# VeilPay

Confidential payroll prototype on Arc Network, designed to be forward-compatible with Circle's upcoming APS (Arc Privacy Sector) precompile.

**Live demo:** https://veilpay-beta.vercel.app

## What this is

Arc Network's official privacy roadmap (Arc Privacy / Arc Privacy Sector, announced June 2026) centers on a TEE-based confidential smart contract engine. Circle's own materials consistently highlight **payroll** as the primary use case: paying employees across jurisdictions without publicly exposing compensation details, while keeping the data auditable by authorized parties.

That precompile is not yet live on Arc Testnet (see `circlefin/arc-node`, where it is listed as "Coming soon"). VeilPay is a working prototype of the same interaction model, built so that once the real precompile ships, only the underlying address needs to change, not the application logic.

## Architecture

**`MockAPS.sol`** mimics the interface described in Circle's Arc Privacy Sector whitepaper:
- `commitPrivateState` / `readPrivateState` — store and retrieve encrypted data per account
- `grantViewAccess` / `hasViewAccess` — selective disclosure ("view key" pattern)
- `readPrivateStateFor` / `grantViewAccessFor` — variants that let an intermediary contract (VeilPay) pass through the real caller's identity

This contract does not provide real TEE/enclave security. It stores whatever encrypted bytes it's given, nothing more. It exists purely to mirror the shape of the real precompile's interface.

**`VeilPay.sol`** is the payroll logic on top of `MockAPS`:
- Salary **amount** stays confidential (only the employee and anyone granted view access can decrypt it)
- Employee **address** (who got paid) stays public
- **Total payroll spend** (`totalPaidOut`) stays public, for transparency/audit purposes

This split mirrors Arc's own framing: not "hide everything" or "show everything," but selective disclosure calibrated to what actually needs to be private.

## Encryption

The frontend (`index.html`) uses the Web Crypto API: a shared passphrase is run through PBKDF2 to derive an AES-256-GCM key, which encrypts the salary amount client-side before it's sent on-chain. Only ciphertext ever touches the contract. The employer and employee must agree on the passphrase off-chain — this is a simplification; the real APS precompile will handle key exchange via X-Wing KEM instead of a shared passphrase.

## Deployed contracts (Arc Testnet)

- MockAPS: `0x9C2DC9230d1A2E8b345cbeEB84c90F625A4A598B`
- VeilPay: `0xbFD6e8525F91595916E9a2DCb4Ff9084A19ada11`

## Migration notes

When Arc's real APS precompile becomes available on testnet:
1. Replace the `MockAPS` deployment address referenced in `VeilPay.sol`'s constructor with the real APS precompile address
2. Confirm the real precompile's function signatures match (or add a thin adapter contract if they differ)
3. No changes should be required to `VeilPay.sol`'s business logic, since it was written against the interface described in the whitepaper, not against `MockAPS`'s implementation details

## Status

Prototype / testnet only. Not audited. Not affiliated with Circle or the Arc team.
