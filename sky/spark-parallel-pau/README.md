---
Title:    Spark Parallel Controller PAU DV
Author:   ChainSecurity
Date:     22. Sep, 2026
Client:   Sky
---

# Deployment Validation Details: Spark Parallel Controller PAU

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation). The same eight contracts were deployed on Arbitrum One and Base, from two codebases:

- Diamond PAU Contracts (`Beacon`, `PAUFactory`, `CCTPFacet`, `AccessControls`, `RateLimits`, `Controller`):
    - Repository: https://github.com/sky-ecosystem/diamond-pau
    - Commit: `cbf71b2ac840ca9288eb867d3dd354e08089e1d7` (v1.14.0)

- Administered Agent Contracts (`AdministeredAgentFactory`, `AdministeredAgent`):
    - Repository: https://github.com/sky-ecosystem/pau-administered-agent
    - Commit: `bfaaf709a8664d74d12604455f0365a0a12439cf` (v1.0.0)

Both are pinned as submodules of the deploy repository, which is where the contracts were compiled and broadcast from:

- Repository: https://github.com/sparkdotfi/spark-pau-deploy
- Commit: `8982d6441940d1c04a9d68e3e1979df88c97cc0f`

| Contract | Arbitrum One | Base |
| --- | --- | --- |
| Beacon | `0x86036ce5d2f792367c0aa43164e688d13c5a60a8` | `0x7ac96180c4d6b2a328d3a19ac059d0e7fc3c6d41` |
| PAUFactory | `0x3968a022d955bbb7927cc011a48601b65a33f346` | `0x011a115b5498b85b3d12245a3a7296f77325b5c3` |
| CCTPFacet | `0xecca0d296cb133081d41e9772b60d57f5fd2798e` | `0xb22d50c393c6e1d13e3e05b172448dd8bf8ddc32` |
| AccessControls | `0x8386f819860d54b1180539ff4852e4caecef8a1d` | `0xe593c8c6a31d88cab100244afd352efb127f9a16` |
| RateLimits | `0x4824c4336a1a11979068a544958dce5d49b42752` | `0x5e311e8bce95f4e8d4920e70985ed4ac122a838a` |
| Controller | `0x04acb9e9bbd64a425677edc535d6b30cfd74e42f` | `0xd864bf1ea2f78dc2013e3fc7e4c474383be9d456` |
| AdministeredAgentFactory | `0xcba0c0a2a0b6bb11233ec4ea85c5bffea33e724d` | `0xd711dbfd937a45e2c89ca0d4781cfa5bab32e752` |
| AdministeredAgent | `0x0745aae633e8318a063d383791bcc0d8c82f46c6` | `0x70e46baf2e3f27a119757d7b796c641c8bc087ce` |

The addresses correspond to [sparkdotfi/spark-address-registry#115](https://github.com/sparkdotfi/spark-address-registry/pull/115).

## Details

In summary, the deployed bytecode matches the source repositories at the given commits, and addresses have been manually and/or automatically validated against a list of references, see [References](#references).

The two chains' deployments differ only in their chain-specific inputs: the `Beacon` immutable of the `PAUFactory` and `Controller`, the `USDC` immutable of the `CCTPFacet`, and the `SPARK_EXECUTOR` and `ALM_PROXY` addresses in storage. Reproducing the bytecode requires compiling from the `spark-pau-deploy` root, as solc's metadata hash covers the source-unit paths and the remapping list, which differ when the libraries are built standalone.

Both deployments were performed by the same deployer EOA (`0xc758519ace14e884fdba9cce25f2dbe81b7e136f`) with the `0-DeploySparkPAUParallel` and `1-ConfigureSparkPAUParallel` scripts, and match them with no deviation. The deployer revoked itself from all four role-bearing contracts; the DV files pin the revoked entries (`hasRole[deployer] = 0`) rather than omitting them.

Per contract:

- *Beacon*: exactly one integration is registered, `CCTP_FACET` pointing at the chain's `CCTPFacet`. Its ten wires match `BeaconConfig.setCCTPIntegration` selector for selector, and are the same ten call to delegate pairs as on the Sky PAU `Beacon` on Ethereum mainnet, which stores them in a different array order; dispatch is keyed by call selector, so the order has no effect.
- *Controller*: has synced that one integration and nothing else, and is bound to the chain's `Beacon`, `AccessControls`, the existing `ALM_PROXY` and the `RateLimits`.
- *CCTPFacet*: immutables are `CCTP_TOKEN_MESSENGER` (the same address on both chains) and the chain's native `USDC`; it holds no storage of its own.
- *PAUFactory* / *AdministeredAgentFactory*: `PAUFactory` is bound to the chain's `Beacon` and deployed the `AccessControls`, `RateLimits` and `Controller`; the `AdministeredAgentFactory` deployed the `AdministeredAgent`.

The resulting access control, identical on both chains and with a single holder per role:

| Contract | Role | Holder |
| --- | --- | --- |
| Beacon | `DEFAULT_ADMIN_ROLE` | `SPARK_EXECUTOR` |
| AccessControls | `DEFAULT_ADMIN_ROLE` | `SPARK_EXECUTOR` |
| AccessControls | `ALLOCATOR_ROLE` | `AdministeredAgent` |
| RateLimits | `DEFAULT_ADMIN_ROLE` | `SPARK_EXECUTOR` |
| RateLimits | `CONTROLLER` | `Controller` |
| AdministeredAgent | admin | `SPARK_EXECUTOR` |
| AdministeredAgent | actor / grantor / revoker | `ALM_RELAYER_MULTISIG` / `PAU_GRANTOR_MULTISIG` / `ALM_FREEZER_MULTISIG` |

Further, note the following adjustments have been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *Storage*: The `Controller`'s integration registry is reported as `unknown` (ERC-7201 namespaced mapping entries reached through computed slots). These were programmatically labeled (`var_name`, `var_type`, `value_hint`) from the deploy scripts and facet interfaces, as were the role-membership keys on the other contracts. No storage variables were removed.
- *Events*: All events were retained; none removed. This includes the factory deployment events and the `CCTPFacet` events, which sibling DV files drop. Here they assert that each factory deployed exactly the contracts in scope, and that the facet was never called directly.

## Findings

No findings of Low severity or above were identified. On both chains the deployed state is consistent with the pinned sources, with the specification set out in the registry PR, and with our understanding of the system from the Diamond PAU and PAU Administered Agent audits. The configuration and role state were checked against those two references; the deploy scripts agree with the result but were read as supporting material, not taken as the statement of intent.

The shared `ALMProxy` topology that this deployment creates once the spell runs is the subject of CS-SKYDPAU-044 (Design, Low, risk accepted) in our Diamond PAU v1.13 report, and is documented in `diamond-pau/docs/ARCHITECTURE.md`.

One observation, informational: the PR states that the `Beacon`'s `CCTP_FACET` wiring matches the Sky PAU `Beacon` on Ethereum mainnet. The set of ten call to delegate pairs is identical, but mainnet stores them in a different array order. Dispatch is keyed by the call selector, so there is no functional effect; it is noted because a byte-level comparison of the two integration configs will differ.

## Considerations

The integration registry and access control are admin-configurable; registering a facet, re-syncing the `Controller` or granting a role invalidates the DV file. Role grants are not detected uniformly: the `Beacon`, `AccessControls` and `AdministeredAgent` pin their membership set lengths at 1, while the `RateLimits` has no member enumeration, so a grant there is caught by the retained `RoleGranted` event instead.

At the validated state the `Controller` holds `CONTROLLER` on the `RateLimits` only, not on the shared `ALM_PROXY`, which the legacy `ForeignController` still controls alone. The spell granting that role is out of scope and will not invalidate these files, as it changes the `ALM_PROXY` rather than the contracts in scope.

The files can be validated retroactively with the `--validationblock` option (see the respective `README`); the Base files are initialized at block `51607000`, just after the configure run, the Arbitrum files at the block current at initialization.

## Final statement

ChainSecurity has reviewed the Arbitrum One and Base parallel controller deployments in [sparkdotfi/spark-address-registry#115](https://github.com/sparkdotfi/spark-address-registry/pull/115). The bytecode, configuration and role state on both chains match the pinned sources, the specification set out in the PR, and our understanding of the system as audited. The deploy scripts agree with that state and were not relied upon as the statement of intent.

We approve the deployment on Arbitrum One.

We approve the deployment on Base.

The approval covers the deployed state described above, with `CCTP_FACET` as the sole integration on the `Controller`. It does not cover the governance spell, the planned follow-ups, or the parallel controller architecture itself.

## References

Note that addresses are validated manually and/or automatically.
Below is the list of references to the expected values used for comparing the address values.

- [Spark Address Registry](https://github.com/sparkdotfi/spark-address-registry/pull/115) (`src/Arbitrum.sol`, `src/Base.sol`) for every contract in scope, `SPARK_EXECUTOR`, `ALM_PROXY`, `USDC`, `CCTP_TOKEN_MESSENGER` and the relayer, freezer and grantor multisigs.
- [`spark-pau-deploy`](https://github.com/sparkdotfi/spark-pau-deploy/tree/8982d6441940d1c04a9d68e3e1979df88c97cc0f) for the deploy and configure scripts ([`0-DeploySparkPAUParallel`](https://github.com/sparkdotfi/spark-pau-deploy/blob/8982d6441940d1c04a9d68e3e1979df88c97cc0f/script/parallel-controller/0-DeploySparkPAUParallel.s.sol), [`1-ConfigureSparkPAUParallel`](https://github.com/sparkdotfi/spark-pau-deploy/blob/8982d6441940d1c04a9d68e3e1979df88c97cc0f/script/parallel-controller/1-ConfigureSparkPAUParallel.s.sol)), the integration wiring ([`BeaconConfig`](https://github.com/sparkdotfi/spark-pau-deploy/blob/8982d6441940d1c04a9d68e3e1979df88c97cc0f/src/BeaconConfig.sol), [`InitParallelPAU`](https://github.com/sparkdotfi/spark-pau-deploy/blob/8982d6441940d1c04a9d68e3e1979df88c97cc0f/src/InitParallelPAU.sol)), the per-chain inputs and the deployment records.
- [Circle USDC contract addresses](https://developers.circle.com/stablecoins/usdc-contract-addresses) and [Circle CCTP V2 contract addresses](https://developers.circle.com/cctp/evm-smart-contracts) for `USDC` and `CCTP_TOKEN_MESSENGER`.
- ChainSecurity's Diamond PAU reports in [`diamond-pau/audits/`](https://github.com/sky-ecosystem/diamond-pau/tree/cbf71b2ac840ca9288eb867d3dd354e08089e1d7/audits) and the "Multi-Controller Topology" section of [`docs/ARCHITECTURE.md`](https://github.com/sky-ecosystem/diamond-pau/blob/cbf71b2ac840ca9288eb867d3dd354e08089e1d7/docs/ARCHITECTURE.md), for CS-SKYDPAU-044 and the shared-`ALMProxy` topology.
