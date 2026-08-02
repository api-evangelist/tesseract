---
name: Assign a yield strategy to a vault (EIP-712 signed)
description: >-
  List available Tesseract strategies and assign one to a deployed vault by
  submitting an EIP-712 typed-data signature signed by the vault owner. No
  on-chain transaction or gas is required.
api: openapi/tesseract-public-api-openapi.json
base_url: https://api.vault.tesseract.fi
operations:
  - StrategiesController_getStrategies
  - StrategiesController_getStrategy
  - VaultsController_setStrategy
  - VaultsController_getVaultDetail
---

# Assign a yield strategy to a vault (EIP-712 signed)

Strategy selection is off-chain metadata signed by the vault owner and submitted
to the Public API. This is the only write operation on the surface.

## Steps

1. **List strategies.** Call `StrategiesController_getStrategies`
   (`GET /strategies`) and optionally `StrategiesController_getStrategy`
   (`GET /strategies/{id}`) to pick a `strategyId` (UUID). Each `StrategyModel`
   carries `name`, `asset`, `expectedApy`, and dashboard metadata.
2. **Build the EIP-712 payload.** Sign the `SetStrategy` typed data with the
   vault owner (client) key:
   - Domain: `{ name: "Tesseract Public API", version: "1", chainId: 1 }`
   - Types: `SetStrategy(address vaultAddress, string strategyId, uint256 timestamp)`
   - Message: `{ vaultAddress, strategyId, timestamp }` where `timestamp` is
     Unix seconds and must be recent — submit within a few minutes of signing.
3. **Submit.** Call `VaultsController_setStrategy`
   (`PUT /vaults/{vault}/strategy`) with body `SetStrategyDto`
   (`clientAddress`, `strategyId`, `signature`, `timestamp`). The API validates
   the signature against the vault owner.
4. **Confirm.** Re-read `VaultsController_getVaultDetail`
   (`GET /vaults/{vault}`) and check the assigned `strategy`.

## Rules
- The signature must be produced by the vault owner; a mismatch is rejected
  (400).
- Signatures expire shortly after signing — do not cache a stale timestamp.
- Strategy changes may be restricted once a vault is actively managed; a
  rejected submission (409) requires contacting Tesseract to arrange a managed
  transition.
- No gas is required; this is off-chain metadata, not an on-chain transaction.
