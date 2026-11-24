# Simulation Cost Pre-Checking

The SDK introduces `CostPreChecker` to enforce transaction fee budgets before committing funds to network fees.

## Why In-Process Simulation Pricing?

Other tools profile local compiled WASM files via CLI invocations. In contrast, an AI agent runtime interacts with contracts already deployed on-chain.

`CostPreChecker` extracts `minResourceFee` directly from the enforced simulation:
- **Resource Fee**: Ledger read/write footprint, CPU instructions, memory allocation.
- **Inclusion Fee**: Base transaction inclusion cost.
- **Total Fee**: Total stroops required for execution.

Priced decisions may also expose a `breakdown` containing the simulation's actual
`instructions`, `diskReadBytes`, and `writeBytes` resource limits plus derived
read-only/read-write/total storage-footprint entry counts. The breakdown is
omitted when the simulation does not provide a complete resource block; it is
never replaced with fabricated zeroes.

If an operator specifies `maxFeeStroops`, calls exceeding the limit return `over_budget`. The ceiling accepts a non-negative `bigint` or a base-10 integer string through the Stellar u64 fee limit; malformed strings and out-of-range values are rejected with `InvalidInputError` before any simulation. `0n` and `"0"` are real ceilings rather than the same as omitting the option. Blocked calls are reported as `blocked` with an explicitly zero fee.

Priced decisions also carry `provenance: { ledger } | null`, copied from the same enforced simulation response that produced the fee. It identifies the ledger snapshot a quote describes; it is informational and not a promise of where the transaction will land. When the simulation response has no usable ledger marker, provenance is `null` instead of a fabricated number.
