---
name: evm-audit-general
description: "Audit Solidity contracts for cross-cutting EVM security footguns that apply to every smart contract. Covers external calls, force-feeding, pause mechanisms, read-only reentrancy, merkle trees, code asymmetry, multicall hazards, storage pointers, struct deletion, and general EVM quirks."
---

# EVM Audit — General Solidity/EVM Footguns

Use when auditing any EVM smart contract — these are non-obvious, cross-cutting issues that apply universally. Load this for **every** audit alongside `evm-audit-precision-math`.

## Workflow

1. Read the contract source and map all external calls, state modifications, and token interactions
2. Load `references/checklist.md` for the full 46+ item checklist organized by category
3. Walk each checklist item against the contract, flagging findings in the standard format
4. Cross-reference findings with domain-specific skills (AMM, lending, etc.) for interaction bugs

## Key Vulnerability Categories

The full checklist in `references/checklist.md` covers these categories (46+ items total):

- **External calls & low-level interactions** — returndata bombing, fixed gas, unchecked return values, non-existent address returns true
- **Force-feeding attacks** — `selfdestruct`, CREATE2 pre-funding, coinbase, direct token transfers bypassing accounting
- **Pause mechanism pitfalls** — paused liquidations causing bad debt, front-running pause, missing unpause path
- **Non-obvious reentrancy** — read-only reentrancy via view functions, ERC777/ERC721 callbacks, cross-contract state staleness
- **Merkle tree issues** — second preimage attacks, leaf vs node confusion, front-running proofs
- **Multicall hazards** — `msg.value` persistence across delegatecall loops, double-spending ETH in batch calls
- **Storage & deletion** — storage pointer aliasing, incomplete struct deletion, mapping cleanup

## Example: msg.value Reuse in Multicall

```solidity
// VULNERABLE — msg.value is reused in every iteration
function multicall(bytes[] calldata data) external payable {
    for (uint i = 0; i < data.length; i++) {
        (bool ok, ) = address(this).delegatecall(data[i]);
        // msg.value is the SAME in every delegatecall — attacker sends 1 ETH, "spends" it N times
    }
}
```

Look for: `msg.value` used inside any loop or batch execution pattern.

## Reference Files
- `references/checklist.md` — Full general security checklist (46+ items with "Look for" patterns and source citations)
