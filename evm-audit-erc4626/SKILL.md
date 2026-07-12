---
name: evm-audit-erc4626
description: "Audit ERC4626 vault implementations for inflation attacks, rounding direction errors, share price manipulation, first depositor exploits, EIP-4626 compliance violations, cross-chain vault issues, and 85+ vulnerability patterns from the Dacian ERC4626 primer."
---

# EVM Audit — ERC4626 Vault Vulnerabilities

Use when auditing any ERC4626 vault implementation, vault-like protocol with share/asset conversion, or contract that integrates with ERC4626 vaults as a depositor or strategy.

## Workflow

1. Identify the vault's share/asset conversion logic, `totalAssets` implementation, and fee model
2. Load `references/checklist.md` for the full 42+ item checklist organized by category
3. Check rounding direction for every preview/convert function against EIP-4626 requirements
4. Verify first-depositor protection and share price manipulation resistance

## Key Vulnerability Categories

The full checklist in `references/checklist.md` covers these categories (42+ items total):

- **First depositor / inflation attack** — 1-wei deposit + donation to steal subsequent deposits, inconsistent deposit/mint paths, virtual shares must equal virtual assets
- **Rounding direction (EIP-4626 compliance)** — deposits round down, withdrawals round up, all rounding must favor the vault, specific requirements for each preview/convert function
- **Compliance requirements** — `totalAssets` must include yield and never revert, `maxDeposit`/`maxMint` must not rely on `balanceOf`, preview functions must include fees
- **Share price manipulation** — direct token transfer inflates price via `balanceOf`, pessimistic `totalAssets` accounting, external protocol dependency in price calculation
- **Cross-chain vault issues** — bridge delay causing stale share prices, L2 sequencer downtime

## Example: First Depositor Inflation Attack

```solidity
// ATTACK SEQUENCE:
// 1. Attacker deposits 1 wei → gets 1 share
// 2. Attacker donates 1000 USDC directly to vault (transfer, not deposit)
// 3. Victim deposits 999 USDC → gets 0 shares (999 * 1 / 1001 = 0)
// 4. Attacker redeems 1 share → gets ~1999 USDC

// MITIGATION — virtual shares/assets (OpenZeppelin pattern)
function _decimalsOffset() internal pure override returns (uint8) {
    return 3; // virtual shares prevent rounding to zero
}
```

Look for: empty vaults where first deposit can be 1 wei without virtual shares or minimum deposit enforcement.

## Reference Files
- `references/checklist.md` — Full ERC4626 checklist (42+ items with "Look for" patterns and source citations)
