---
name: Read a client's vault portfolio and performance
description: >-
  Discover the vaults held by a client wallet on the Tesseract Public API and
  read their allocation, TVL, and APY/APR performance. All read operations are
  unauthenticated — the vault and client addresses act as the key.
api: openapi/tesseract-public-api-openapi.json
base_url: https://api.vault.tesseract.fi
operations:
  - ClientsController_getClientVaults
  - ClientsController_getClientSummary
  - VaultsController_getVaultDetail
  - InsightsController_getSummary
  - InsightsController_getPerformance
  - InsightsController_getAllocationsLatest
  - VaultsController_getVaultTransactions
---

# Read a client's vault portfolio and performance

Tesseract Dedicated Client Vaults are per-client on-chain vaults on Ethereum
mainnet. The Public API (`https://api.vault.tesseract.fi`) exposes read-only
vault discovery, state, and performance. No authentication is required — the
client/vault addresses are the access key. Numeric and monetary values are
returned as strings; parse leniently.

## Steps

1. **List the client's vaults.** Call `ClientsController_getClientVaults`
   (`GET /clients/{client}/vaults`) with the client wallet address. Each
   `PublicVaultModel` includes the vault `address`, `asset`/`assetSymbol`,
   share token, and assigned `strategy`.
2. **Get the portfolio headline.** Call `ClientsController_getClientSummary`
   (`GET /clients/{client}/summary`) for the weighted-average APY and vault count.
3. **Inspect one vault.** For a vault address, call `VaultsController_getVaultDetail`
   (`GET /vaults/{vault}`) for asset/shares/fees/status.
4. **Read KPIs.** Call `InsightsController_getSummary`
   (`GET /insights/{vaultAddress}/summary`) for current TVL, current APY, 24h
   change, and 7d/30d average APY.
5. **Read time-series.** Use `InsightsController_getPerformance`
   (`GET /insights/{vaultAddress}/performance`) and
   `InsightsController_getAllocationsLatest`
   (`GET /insights/{vaultAddress}/allocations/latest`). Optional `from`/`to`
   ISO-8601 query params bound the range (defaults to last 30 days).
6. **Audit activity.** Call `VaultsController_getVaultTransactions`
   (`GET /vaults/{vault}/transactions`) — paginate with `offset`/`limit`, filter
   with `type` (deposit, withdraw, execute, …).

## Rules
- Read endpoints need no token; do not send credentials.
- Performance snapshots are refreshed hourly (simulated against on-chain state),
  so APY/TVL reflect yield accrued between vault transactions.
- Treat unknown response fields as additive (lenient parsing); the API adds
  backwards-compatible fields without notice.
