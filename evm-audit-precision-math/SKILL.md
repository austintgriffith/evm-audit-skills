---
name: evm-audit-precision-math
description: "Audit Solidity contracts for precision loss, rounding errors, division ordering, fixed-point math, and mathematical edge cases. Covers division-before-multiplication, rounding direction, downcast overflow, decimal mismatches, and accumulator math — the #1 source of DeFi exploits."
---
# EVM Audit — Precision & Math Vulnerabilities

Use when auditing any Solidity contract that performs arithmetic — especially DeFi protocols with token pricing, fee calculations, share conversions, or interest accrual. Load this for **every** audit alongside `evm-audit-general`.

## Workflow

1. Read the contract source and identify all arithmetic operations (division, multiplication, casting, scaling)
2. Load `references/checklist.md` for the full 23+ item checklist organized by category
3. Walk each checklist item against the contract's math, flagging findings in the standard format
4. Pay special attention to division ordering, rounding direction, and decimal handling
5. Cross-check each finding against the contract's test suite or invariants to reduce false positives — trace values through edge cases (zero, 1 wei, max uint) to confirm exploitability

## Key Vulnerability Categories

The full checklist in `references/checklist.md` covers these categories (23+ items total):

- **Division before multiplication** — precision loss from incorrect operation ordering
- **Rounding direction** — deposits must round down, withdrawals must round up (protocol-favoring)
- **Integer overflow/underflow** — even with Solidity ≥0.8 (`unchecked` blocks, downcasts, signed/unsigned mixing)
- **Decimal handling** — oracle decimal mismatches, non-18-decimal tokens, scaling errors
- **Accumulator & interest math** — reward-per-token precision loss, compounding errors, missing state updates

## Example: Division Before Multiplication

```solidity
// VULNERABLE — division before multiplication loses precision
uint256 rate = (utilRate / optimalUsageRate) * slope1;

// FIXED — multiply first, then divide
uint256 rate = (utilRate * slope1) / optimalUsageRate;
```

This is the single most common precision bug in DeFi. Always check that no division appears to the left of a multiplication in the same expression chain.

## Example: Downcast Overflow (Solidity ≥0.8)

```solidity
// VULNERABLE — uint256 silently truncated to uint128
uint128 shares = uint128(totalShares); // if totalShares > type(uint128).max, wraps silently

// FIXED — use SafeCast
import {SafeCast} from "@openzeppelin/contracts/utils/math/SafeCast.sol";
uint128 shares = SafeCast.toUint128(totalShares); // reverts on overflow
```

Look for: any explicit downcast (`uint128(x)`, `uint64(x)`, `uint32(x)`) especially with user-influenced values.

## Reference Files
- `references/checklist.md` — Full precision/math checklist (23+ items with "Look for" patterns and source citations)
