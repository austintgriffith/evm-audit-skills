---
name: evm-audit-defi-amm
description: "Audit AMM and DEX contracts for slippage attacks, concentrated liquidity manager vulnerabilities, Uniswap V3/V4 hook exploits, swap routing issues, TWAP manipulation, fee-on-transfer token mishandling, and DEX integration pitfalls."
---

# EVM Audit — AMM & DEX Vulnerabilities

Use when auditing AMMs, DEXes, swap routers, Uniswap V3/V4 hooks, concentrated liquidity managers (CLMs), or any contract that integrates with decentralized exchange protocols.

## Workflow

1. Identify all swap, liquidity, and routing logic in the contract
2. Load `references/checklist.md` for the full 30+ item checklist organized by category
3. Check slippage protection, hook permissions, and fee handling against checklist items
4. Verify CLM-specific patterns if concentrated liquidity is involved

## Key Vulnerability Categories

The full checklist in `references/checklist.md` covers these categories (30+ items total):

- **General AMM** — cross-contract view reentrancy on reserve updates, flash loan callback ordering, fee-on-transfer and rebasing token mishandling, arbitrary call from user input
- **Slippage protection** — hardcoded `minAmountOut = 0`, on-chain slippage calculation is manipulable, missing deadline, missing refunds after partial swaps
- **Uniswap V4 hooks** — permission bits encoded in address, incorrect return types, BeforeSwapDelta sign confusion, unsettled deltas revert `unlock()`, async hooks stealing custody, missing access control
- **Concentrated liquidity managers** — TWAP bypass, sandwich attacks via owner rebalance functions, stuck tokens from tick range changes, stale approvals, retrospective fee extraction

## Example: Hardcoded Zero Slippage

```solidity
// VULNERABLE — zero slippage = free sandwich attack
router.exactInputSingle(ISwapRouter.ExactInputSingleParams({
    amountIn: amount,
    amountOutMinimum: 0,  // attacker sandwiches for free
    sqrtPriceLimitX96: 0,
    deadline: type(uint256).max  // no expiration either
}));

// FIXED — user-specified slippage and deadline
router.exactInputSingle(ISwapRouter.ExactInputSingleParams({
    amountIn: amount,
    amountOutMinimum: minOut,  // calculated off-chain
    sqrtPriceLimitX96: priceLimit,
    deadline: block.timestamp + 300
}));
```

Look for: `amountOutMinimum: 0`, `sqrtPriceLimitX96: 0`, or `deadline: type(uint256).max`.

## Reference Files
- `references/checklist.md` — Full AMM/DEX checklist (30+ items with "Look for" patterns and source citations)
