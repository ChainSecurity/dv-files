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

- Diamond PAU Contracts (`AccessControls`, `Controller`, `ALMProxy`, `RateLimits`):
    - Repository: https://github.com/sky-ecosystem/diamond-pau
    - Commit: `5c5ad6ae174bf467081ca82342ced2bd42a5c732`
    - Contracts:
        - AccessControls: `0x4f6d1704700cd494dd4cd9bf59c0c39da1bc9164`
        - Controller: `0xbf83f5974b932c7d842254042717d6a2706ce5ee`
        - ALMProxy: `0x0dcd9298e163dfd3c0b5b00f0d9093c36e40a153`
        - RateLimits: `0xe016ae733a77ba77e7907aaa749394fc5e75c0e1`

- Administered Agent Contracts (`AdministeredAgent`):
    - Repository: https://github.com/sky-ecosystem/pau-administered-agent
    - Commit: `bfaaf709a8664d74d12604455f0365a0a12439cf`
    - Contracts:
        - AdministeredAgent: `0xdbd17832df0e57b1732ce1c84c652e820e549baa`

All five contracts were deployed and configured by the `DefaultPAUAssembler` (`0xc812aad3fae2d3511c664374b601a9bebfecca2e`, validated under `sky/pau-assemblers`).

## Details

In summary, the deployed bytecode matches the source repositories at the given commits. The contracts were assembled by the `DefaultPAUAssembler`, which held `DEFAULT_ADMIN_ROLE` on each during configuration and was revoked afterwards, leaving `GROVE_PROXY` as the sole admin. The `Controller`'s integration wiring matches the `Beacon` wiring in [`diamond-pau-deploy`](https://github.com/sky-ecosystem/diamond-pau-deploy/tree/90df5687155df6ba8ca9b9bcfdf947ff69895405).

Addresses have been manually and/or automatically validated against a list of references, see [References](#references).

- *Controller*: ERC-7201 namespaced. Holds the integration registry (`configs`/`dispatches`) for the three enabled facets (`USDS_FACET`, `PSM_FACET`, `BASIN_FACET`), copied from the `Beacon`; operational calls `delegatecall` the wired facet, gated on `ALLOCATOR_ROLE`. Its `SharedControllerStorage` references the `AccessControls`, `ALMProxy` and `RateLimits`.
- *AccessControls*: `AccessControlEnumerable`. `DEFAULT_ADMIN_ROLE` → `GROVE_PROXY`, `ALLOCATOR_ROLE` → the `AdministeredAgent`.
- *AdministeredAgent*: actors are `ALM_RELAYER` and the Grove primary/secondary relayer operators, admin is `GROVE_PROXY`, revoker is `ALM_FREEZER`. Deployed by the `AdministeredAgentFactory` (validated separately).
- *ALMProxy* / *RateLimits*: `AccessControl`. `DEFAULT_ADMIN_ROLE` → `GROVE_PROXY`; the `CONTROLLER` role → the `Controller`.

Further, note the following adjustments have been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *Storage*: The role-membership, integration-registry and shared-storage slots are reported as `unknown` (nested/ERC-7201 mapping/struct/array entries reached through computed slots). These were programmatically labeled (`var_name`, `var_type`, `value_hint`) from the deploy scripts and facet interfaces.
- *Events*: All events (access-control, integration-config and rate-limit) were retained; none removed.

## Considerations

The `Controller`'s integration registry and the contracts' access control are admin/allocator-configurable; re-wiring or reconfiguring will change storage and invalidate the DV file.

The files can be validated retroactively with the `--validationblock` option (see the respective `README`).

## References

Note that addresses are validated manually and/or automatically.
Below is the list of references to the expected values used for comparing the address values.

- [Grove Address Registry](https://github.com/grove-labs/grove-address-registry/blob/main/src/Ethereum.sol) for `GROVE_PROXY`, `ALM_RELAYER`, `ALM_FREEZER` and the relayer operators.
- The `USDSFacet`, `PSMFacet`, `BasinFacet` and `Beacon` are validated under `sky/diamond-pau`; the `DefaultPAUAssembler` under `sky/pau-assemblers`.
