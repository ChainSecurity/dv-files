---
Title:    Diamond PAU DV
Author:   ChainSecurity  
Date:     5. Jun, 2026
Client:   Sky
---

# Deployment Validation Details: Diamond PAU

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation).

- Diamond PAU Contracts:
    - Repository: https://github.com/sky-ecosystem/diamond-pau
    - Commit: `a84fe9616412e3b6d10ea64301a3481f7dcf9a05` (`PAUFactory` from `5c5ad6ae174bf467081ca82342ced2bd42a5c732`)
    - Contracts:
        - Beacon: `0x829dc2b7e94b1954f0764e573f2e0d45afa28199`
        - PAUFactory: `0x69a5d548830ac2a4ba90a44a2c75bda71f97fc66`
        - AaveFacet: `0x8ce890a96a193ff2dd4b2ea3c682326f655f6b62`
        - BasinFacet: `0xc84825bcd13aeddc372400239499380376a44a39`
        - CCTPFacet: `0xadf62692340e46ef90336f2e75ce3b37f1148873`
        - CentrifugeFacet: `0xa0a10ba97be1412730d694b8de1afe7eff20ec31`
        - CurveFacet: `0x139d81d7d6040faef7cf0ef5a2636ca8a97a30d8`
        - DAIUSDSFacet: `0x3817f734cae6ad2bdb79f9ff23091f2ad478da5f`
        - ERC4626Facet: `0x1dca18608c89174181153e786778705b4a0e1a06`
        - ERC7540Facet: `0x4f7e0e3612b0e1e156a2b6570a51d4bd709f1315`
        - EthenaFacet: `0xec48d773ceef1c6b07cda1afa2716c478b55187b`
        - FarmFacet: `0xf24e91f5d8529436c9fb92dd94f80d4a6c25d0f0`
        - LayerZeroFacet: `0xa0c323a0acb20f259ea4ff343319d450be6472e5`
        - MapleFacet: `0x691b5c26ad2b74d2376f4ed87904e9d3e47bd630`
        - MerklFacet: `0x321138db5e056e9d0080d4c278e10a1edc091eb0`
        - OTCFacet: `0x46b24ba00b65cb4f603447590e539b08097fb7ac`
        - PendleFacet: `0xcc9dd4c9b2a9c08f2692e7060f43d29a03e87348`
        - PSMFacet: `0xe4a5dac768a310cc2316f258901b32e499653064`
        - SparkVaultFacet: `0xff0d19920e207e3a17eb5a2e5ba3afa44836362b`
        - SuperstateFacet: `0xee197475607e9a27ccaa4786e740d2f0d0e706a7`
        - TransferAssetFacet: `0x4da7608c331b8f135df5b985018933780ecd089d`
        - UniswapV3Facet: `0x445d9dc752f269be48250f1a180cac4c61ce4bab`
        - UniswapV4Facet: `0x75d35ffb8e6b871e12eb549ccf6afd324c46e47d`
        - USDSFacet: `0x1221cc4b85ab260660ad21c2829e0eb516dffbc7`
        - WEETHFacet: `0x1d8d089eb7d558f5dc6aa0cf98dde13b77b3f641`
        - WrapProxyETHFacet: `0x081506de21c695af5e61a81ad288c8a96b6b59b9`
        - WSTETHFacet: `0x3a82d11cd37fb0098363262dc69425d07fa05516`

## Details

In summary, the deployed bytecode matches the source repository at the given commit (see [Scope](#scope)), and the on-chain configuration matches the deployment scripts in the [`diamond-pau-deploy`](https://github.com/sky-ecosystem/diamond-pau-deploy/tree/90df5687155df6ba8ca9b9bcfdf947ff69895405) repository. The `PAUFactory` is not covered by those scripts — it was deployed standalone.

Addresses have been manually and/or automatically validated against a list of references, see [References](#references).

The `Beacon` is the access-controlled registry of the system. At the validation block, the sole holder of `DEFAULT_ADMIN_ROLE` on the `Beacon` is `0xbe8e3e3618f7474f8cb1d074a26affef007e98fb`, the `MCD_PAUSE_PROXY`. The `Beacon` was deployed with the deployer `0x1ca4ecaf0e13ca833c80da835deea15e1684361d` as the initial admin (this is the value of the `admin` constructor argument); the deployer wired all mainnet-suitable facets via `setIntegration` and then transferred `DEFAULT_ADMIN_ROLE` to the `MCD_PAUSE_PROXY`, revoking its own.

The `Beacon`'s storage encodes the integration registry: for each integration, the facet address and its per-selector dispatch table (`_configs` and `_dispatches`). These entries, the integration set, and the `DEFAULT_ADMIN_ROLE` membership have been programmatically verified against the expected configuration.

Further, note the following adjustments have been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *Facets*: The facets are stateless logic contracts invoked via `delegatecall`, so both their events and storage live at the proxy, not at the facet address. All `critical_events` (including `Initialized`) were removed, and no `critical_storage_variables` are tracked.
- *Beacon*:
    - `critical_events`: All of the `Beacon`'s events relate to access control or integration configuration (`IntegrationSet`, `IntegrationRemoved`, `RoleAdminChanged`, `RoleGranted`, `RoleRevoked`) and were retained; none were removed.
    - `critical_storage_variables`: The tool reports the integration registry and role-membership slots as `unknown`, since they are nested mapping/struct/array entries reached through computed (`keccak256`) slots. These have been programmatically labeled with `var_name`, `var_type`, `offset` and `value_hint` — derived from the deploy script and the facet interfaces — splitting packed slots (e.g. `Dispatch` and `Wire`) into their individual fields. The reentrancy guard (`_status`) was retained at its initialized value.
- *PAUFactory*: A stateless, permissionless deployer whose only state is the `beacon` immutable (the `Beacon`). It has no `critical_storage_variables`, and all `critical_events` (the `*Deployed` factory-deployment records) were removed.

## Considerations

The `critical_storage_variables` of the `Beacon` encode the complete integration set. Adding, removing or re-wiring any integration through `setIntegration`/`removeIntegration` (an admin-only action) changes the `_integrationIds`, `_configs` and `_dispatches` slots and will invalidate the corresponding DV file. The DV files reflect the state after deployment, wiring and the admin hand-off to the `MCD_PAUSE_PROXY`.

The files can be validated retroactively with the `--validationblock` option (see the respective `README`).

## References

Note that addresses are validated manually and/or automatically.
Below is the list of references to the expected values used for comparing the address values.

- [Sky Chainlog](https://chainlog.skyeco.com/) for addresses related to Sky (`DAI`, `USDS`, `DaiUSDS`, `PsmLite`) and the `MCD_PAUSE_PROXY` admin.
- [CCTP Documentation](https://developers.circle.com/cctp/references/contract-addresses) for the CCTP `TokenMessengerV2`.
- [USDC Documentation](https://developers.circle.com/stablecoins/usdc-contract-addresses#mainnet) for `USDC`.
- [Uniswap v3 Deployments](https://developers.uniswap.org/docs/protocols/v3/deployments/v3-ethereum-deployments) for `NonfungiblePositionManager` and `SwapRouter02`.
- [Uniswap v4 Deployments](https://developers.uniswap.org/docs/protocols/v4/deployments) for `Permit2`, the `PositionManager` and the `UniversalRouter`.
- [WETH](https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2) for `WETH`.
- [ether.fi Deployed Contracts](https://etherfi.gitbook.io/etherfi/contracts-and-integrations/deployed-contracts) for `weETH`.
- [Lido Deployed Contracts](https://docs.lido.fi/deployed-contracts/) for `wstETH` and the `Withdrawal Queue ERC721`.
- [Ethena Documentation](https://docs.ethena.fi/solution-design/key-addresses) for addresses related to Ethena (`Ethena Minter`, `USDe` and `sUSDe`). Note that the Ethena Minter corresponds to the V2 contract given that this is the set minter in the `USDe` contract.
- [Pendle Documentation](https://docs.pendle.finance/pendle-v2-dev/Contracts/PendleRouter/PendleRouterOverview#overview) for the Pendle `Router`.
- [Superstate Documentation](https://docs.superstate.com/investors/smart-contracts) for `USTB`.
