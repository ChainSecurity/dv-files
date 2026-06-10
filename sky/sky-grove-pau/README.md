---
Title:    Grove PAU DV
Author:   ChainSecurity  
Date:     9. Jun, 2026
Client:   Sky
---

# Deployment Validation Details: Grove PAU

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation). The contracts come from two codebases:

- Diamond PAU Contracts (`AccessControls`, `Controller`):
    - Repository: https://github.com/sky-ecosystem/diamond-pau
    - Commit: `5c5ad6ae174bf467081ca82342ced2bd42a5c732`
    - Contracts:
        - AccessControls: `0x10d1ade77f1b81ef95057bb2face292313f66277`
        - Controller: `0x0dd65461610fe5b65ce50a870b10ed0f3d24d8c2`

- Administered Agent Contracts (`AdministeredAgent`):
    - Repository: https://github.com/sky-ecosystem/pau-administered-agent
    - Commit: `bfaaf709a8664d74d12604455f0365a0a12439cf`
    - Contracts:
        - AdministeredAgent: `0x0f7ca6616cc38132530dc4695778a54de42c21f4`

The shared `ALMProxy` (`0x491edfb0b8b608044e227225c715981a30f3a44e`) and `RateLimits` (`0x5f5cfcb8a463868e37ab27b5eff3ba02112df19a`) referenced by the `Controller` are validated under `sky/bloom-alm-controller` and are not in scope here.

## Details

In summary, the deployed bytecode matches the source repositories at the given commits, and the on-chain configuration matches the deployment scripts in the [`grove-pau-deploy`](https://github.com/sky-ecosystem/grove-pau-deploy/tree/3d6c4e761c0df94a831567065c7084e997f04d35) repository. The `Controller`'s integration wiring matches the `Beacon` wiring in [`diamond-pau-deploy`](https://github.com/sky-ecosystem/diamond-pau-deploy/tree/90df5687155df6ba8ca9b9bcfdf947ff69895405).

Addresses have been manually and/or automatically validated against a list of references, see [References](#references).

- *Controller*: ERC-7201 namespaced. Holds the integration registry (`configs`/`dispatches`) for the four enabled facets (`BASIN`, `ERC4626`, `MAPLE`, `UNISWAP_V3`), copied from the `Beacon`; operational calls `delegatecall` the wired facet, gated on `ALLOCATOR_ROLE`. Per-facet config (UniswapV3 pool params/slippage, ERC4626 max exchange rate) is stored under the facets' own namespaces and was copied from the previous controller (`ALM_CONTROLLER`).
- *AccessControls*: `AccessControlEnumerable`. `DEFAULT_ADMIN_ROLE` → `GROVE_PROXY`, `ALLOCATOR_ROLE` → the `AdministeredAgent`. The deployer's admin role was revoked after configuration.
- *AdministeredAgent*: actors are `ALM_RELAYER` and the Grove primary/secondary relayer operators, admin is `GROVE_PROXY`, revoker is `ALM_FREEZER`. Deployed by the `AdministeredAgentFactory` (validated separately); the deployer was removed as admin after configuration.

Further, note the following adjustments have been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *Storage*: The role-membership, integration-registry and facet-config slots are reported as `unknown` (nested/ERC-7201 mapping/struct/array entries reached through computed slots). These were programmatically labeled (`var_name`, `var_type`, `offset`, `value_hint`) from the deploy scripts and facet interfaces, splitting packed slots into their fields.
- *Events*: All events are access-control / configuration events and were retained. Six `Controller` events reported as `Unknown Signature` were resolved — they are facet config-setter events (`ERC4626MaxExchangeRateSet` and the UniswapV3 setters) emitted via `delegatecall`, hence absent from the `Controller` ABI.

## Considerations

The `Controller`'s integration registry and per-facet configuration are admin/allocator-configurable; re-wiring or reconfiguring will change storage and invalidate the DV file.

The per-facet config values were copied from the previous controller (`ALM_CONTROLLER`) rather than set to literals, so they were validated structurally (correct keys/slots).

The files can be validated retroactively with the `--validationblock` option (see the respective `README`).

## References

Note that addresses are validated manually and/or automatically.
Below is the list of references to the expected values used for comparing the address values.

- [Grove Address Registry](https://github.com/grove-labs/grove-address-registry/blob/main/src/Ethereum.sol) for `GROVE_PROXY`, `ALM_RELAYER`, `ALM_FREEZER`, the relayer operators, `MAPLE_SYRUP_USDC` and `UNISWAP_V3_AUSD_USDC`.
- [Sky Chainlog](https://chainlog.skyeco.com/) for the shared `ALMProxy` and `RateLimits`.
