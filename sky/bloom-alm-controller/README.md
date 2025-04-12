# Deployment Validation Details: Bloom ALM Controller

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation).

- Bloom ALM Controller Contracts:
    - Repository: https://github.com/bloom-foundation/bloom-alm-controller
    - Commit: `53e4c1ad02756d2ca45f7386078a6cc5b899864a`
    - Note: Note that the deployed contracts match the [Spark ALM repository](https://github.com/sparkdotfi/spark-alm-controller) at tag `v1.3.0`. The relevant audit report can be found [here](https://www.chainsecurity.com/security-audit/spark-alm-controller) (`v1.3.0` corresponds to report version 12).
    - Contracts:
        - ALMProxy: `0x491EDFB0B8b608044e227225C715981a30F3A44E`
        - MainnetController: `0x3048386E09c72C20FB268a37d2B630D7f2Ee9138`
        - RateLimits: `0x5F5cfCB8a463868E37Ab27B5eFF3ba02112dF19a`

- Safe Proxy:
    - Repository: https://github.com/safe-global/safe-smart-account
    - Tag: `v1.4.1`
    - Note: Note that the contracts at the given tag have been used for validating the deployment of the SafeProxy contracts.
    - Contracts:
        - ALM Freezer: `0xB0113804960345fd0a245788b3423319c86940e5`
        - ALM Relayer: `0x0eEC86649E756a23CBc68d9EFEd756f16aD5F85f`

## Details

We can confirm that the bytecode matches the given repositories at the given commits.

Addresses have been manually validated against a list of references, see [References](#references).

The threshold of the Safe contracts ALM Freezer and ALM Relayer were confirmed by the team and is 1. Both contracts have the same owners which were confirmed by the team and are:

- `0xa2bdfaa0329c2ebdb645f29c3971587e97e55e21`
- `0xc3e9458cefcf298ae11702959c15723e56b2dc0a`

Futher, note the following adjustments have been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *Bloom Contracts*: Note that given the initial setup, no regular operations should be possible. Thus, events and storage variables related to regular operations have been removed accordingly from the DV files. Below, more details are listed:
    - *ALMProxy*: No changes were performed compared to the initialization.
    - *RateLimits*: Events unrelated to access control were removed from `critical_events` (`RateLimit*`). Note that the initialization of DV files showed no occurance of such events.
    - *MainnetController*
        - The `CENTRIFUGE_REQUEST_ID` has been removed from `critical_storage_variables`. Note that the initialization of the DV files showed the value to be 0 as expected.
    - The `CCTPTransferInitiated` has been removed from `critical_events`. Note that the initialization of DV files showed no occurance of such events.
- *Safe Contracts: ALM Freezer and ALM Relayer*: Storage variables and events unrelated to the configuration of a Safe have been removed. Below, more details listed:
    - `nonce` has been removed from `critical_storage_variables`. Note that the value at initialization was 0.
    - `_deprecatedDomainSeparator` has been removed from `critical_storage_variables`. Note that the value at initialization was 0.
    - Events `ApproveHash`, `ExecutionFailure*`, `SafeReceived` and `SignMsg` have been removed accordingly.

## Considerations

Note that the DV files will become invalid in on initialization due to new roles being assigned.

However, the files can be validated retroactively with the `--validationblock` option (see the respective `README`).
The published DV files might be updated in the future to reflect the finalized initialization.

## References

Note that addresses are partially validated manually.
Below is the list of references to the expected values used for comparing the address values.

- [Bloom Address Registry](https://github.com/bloom-foundation/bloom-address-registry/blob/2b4371d552b7d1743cfb20da7f4ca8b94c8b2c32/src/Ethereum.sol) for addresses related to Bloom that were not validated as part of the DV. Note that these addresses correspond accordingly to `bloom-foundation/bloom-alm-controller/` repository's [mainnet-production.json](https://github.com/bloom-foundation/bloom-alm-controller/blob/7f3341a98fc279eb145c36bd57a436a956850813/script/input/1/mainnet-production.json) file.
- [Sky Chainlog (April 11th 2025)](https://chainlog.sky.money/) for addresses related to Sky
- [CCTP Documentation](https://developers.circle.com/stablecoins/evm-smart-contracts#tokenmessenger-mainnet) for CCTP. Note that the V1 TokenMessenger is used (expected as the controller requires function `burnLImitsPerMessage()` which is only present in the V1 CCTP token messenger).
- [USDC Documentation](https://developers.circle.com/stablecoins/usdc-on-main-networks#usdc-mainnet-addresses) for USDC.
- [Ethena Documentation](https://docs.ethena.fi/solution-design/key-addresses) for addresses related to Ethena (Ethena Minter, USDE and sUSDE). Note that the Ethena Minter corresponds to the V2 contract given that this is the set minter in the USDE contract.
- [Superstate Documentation](https://docs.superstate.co/introduction-to-superstate/smart-contracts) for addresses related to Superstate (redemptions and USTB).
- [Safe Global Deployments](https://github.com/safe-global/safe-deployments?tab=readme-ov-file#deployments-overview) for addresses related to the Safe contracts deployed.
- BUIDL: The address of the BUIDL redemption contract is listed in the following [governance forum discussion](https://forum.sky.money/t/april-3-2025-proposed-changes-to-spark-for-upcoming-spell/26155).
