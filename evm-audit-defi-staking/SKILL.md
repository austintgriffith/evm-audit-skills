---
name: evm-audit-defi-staking
description: "Audit staking protocols for liquid staking derivative vulnerabilities (stETH, rETH, cbETH, sfrxETH), LRT/restaking issues, reward calculation errors, cooldown exploitation, and yield farming attacks. Covers Lido, Rocket Pool, Coinbase, Frax, and EigenLayer integration pitfalls."
---

# EVM Audit — Staking & LSD Vulnerabilities

Use when auditing staking protocols, liquid staking derivative integrations, restaking/LRT protocols, yield aggregators, or any contract that holds or prices stETH, rETH, cbETH, or sfrxETH.

## Workflow

1. Identify all staking-related tokens and integrations in the contract
2. Load `references/checklist.md` for the full 30+ item checklist organized by category
3. Check each LSD-specific pattern (rebasing, withdrawal queues, rate assumptions)
4. Verify reward distribution math and lock mechanism logic against checklist items

## Key Vulnerability Categories

The full checklist in `references/checklist.md` covers these categories (30+ items total):

- **LSD integration pitfalls** — stETH rebasing breaks accounting, rETH burn reverts when pool empty, cbETH blacklisting freezes vaults, sfrxETH rate detachment during multisig operations
- **LSD protocol design** — WithdrawCredentials front-running, slashing not reflected in derivative, gas limits on batch deposits, inflation attacks on empty pools
- **Staking rewards** — reward rate dilution via `notifyRewardAmount(0)`, expired tokens still earning, missing totalSupply sync before claims
- **Lock mechanisms** — cooldown bypass via transfer-to-new-address, insufficient lock duration, lock extension without consent

## Example: stETH Rebasing Breaks DeFi Accounting

```solidity
// VULNERABLE — stETH balance changes on every Lido oracle report
uint256 deposited = stETH.balanceOf(address(this));
// ... time passes, oracle reports ...
uint256 current = stETH.balanceOf(address(this)); // different from deposited!

// FIXED — use wstETH (non-rebasing wrapper) for internal accounting
uint256 shares = wstETH.wrap(stETHAmount);
```

Look for: `stETH` in contract imports/addresses without wstETH wrapping logic.

## Reference Files
- `references/checklist.md` — Full staking/LSD checklist (30+ items with "Look for" patterns and source citations)
