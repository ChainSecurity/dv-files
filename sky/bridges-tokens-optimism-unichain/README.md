---
Title:    Optimism and Unichain Native Token Bridge DV
Author:   ChainSecurity  
Date:     16. May, 2025
Client:   Sky
---

# Deployment Validation Details: Optimism and Unichain Native Token Bridges

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation).

- OP Token Bridge Contracts:
    - Repository: https://github.com/makerdao/op-token-bridge
    - Commit: `82918f4853d50c6520dac53fdb70a42fd4ce671b`
    - Note: The relevant audit report can be found [here](https://www.chainsecurity.com/security-audit/makerdao-op-token-bridge) 
    - Contracts:
        - Unichain:
            - escrow: `0x1196F688C585D3E5C895Ef8954FFB0dCDAfc566A`
            - l1Bridge: `0xDF0535a4C96c9Cd8921d8FeC92A7680b281681d2`
            - l1BridgeImp: `0x8A925ccFd5F7f46332E2D719A916f8b4a643599F`
            - l1GovRelay: `0xb383070Cf9F4f01C3a2cfD0ef6da4BC057b429b7`
            - l2Bridge: `0xa13152006D0216Fe4627a0D3B006087A6a55D752`
            - l2BridgeImp: `0xd78292C12707CF28E8EB7bf06fA774D1044C2dF5`
            - l2BridgeSpell: `0x32760698c87834c02ED9AFF2d4FC3e16c029B936`
            - l2GovRelay: `0x3510a7F16F549EcD0Ef018DE0B3c2ad7c742990f`
        - Optimism:
            - l1Bridge: `0x3d25B7d486caE1810374d37A48BCf0963c9B8057`
            - l1BridgeImp: `0xA50adBad34c1e9786979bD44220F8fd46e43A6B0`
            - l2Bridge: `0x8F41DBF6b8498561Ce1d73AF16CD9C0d8eE20ba6`
            - l2BridgeImp: `0xc2702C859016db756149716cc4d2B7D7A436CF04`
            - l2BridgeSpell: `0x99892216eD34e8FD924A1dBC758ceE61a9109409`

- Usds
    - Repository: https://github.com/makerdao/usds
    - Commit: `e39e3fa4befdf08f1798528cf3024db29b6de34e`
    - Contracts:
        - Unichain:
            - l2Usds: `0x7E10036Acc4B56d4dFCa3b77810356CE52313F9C`
            - l2UsdsImp: `0x8fb6EF2dA06a6957fC84C2C55bb195837fB7DBa9`
        - Optimism:
            - l2Usds: `0x4F13a96EC5C4Cf34e442b46Bbd98a0791F20edC3`
            - l2UsdsImp": `0x2A3541003B34f34833a82F194e4dC69a7a39B057`

- SUsds
    - Repository: https://github.com/makerdao/sdai
    - Commit: `77eeb2cafd260789d6308d739b849c07dfeb55ed`
    - Contracts:
        - Unichain:
            - sUsds: `0xA06b10Db9F390990364A3984C04FaDf1c13691b5`
            - sUsdsImp: `0x15c2A564b987470FAFCaB0B036029532bd168E10`
        - Optimism:
            - sUsds: `0xb5B2dc7fd34C249F4be7fB1fCea07950784229e0`
            - "sUsdsImp": `0x6f0888DDA6a5E35451D5bE0fABb20171715788B3`

## Details

We can confirm that the bytecode matches the given repositories at the given commits.

Addresses have been manually validated against a list of references, see [References](#references).

No Escrow and GovRelay contracts have been validated for OP Mainnet.

All wardens for contracts deployed on L1 have been set to `0xbe8e3e3618f7474f8cb1d074a26affef007e98fb` the `MCD_PAUSE_PROXY`.

Note the following about the deployment:

- The Escrow for Unichain has not approved the L1 bridge. Note that this is expected as the approval should be part of a governance spell. Note that the one for Optimism was not in scope and that the property has not been validated.
- The L1 bridges for Optimism and Unichain do not have their escrow set. This is expected as the governance spell should ensure the escrows are set accordingly. 
- All bridges are open. Note that this is as expected. However, tokens are not configured (part of a governance spell) and thus the bridges are not functional given the current deployment.
- Further, note that the L2 token supply and balances have not been modified maliciously. Given the state at deployment the values will only be able change according to the governance's future configuration.
- Further, note that upgradeable contracts have been initialized accordingly (for implementation contracts: the initializer has been blocked).

Note that prior to setting up the escrow contracts in the L1TokenBridge contracts, the bridge is not operational. DV must be updated after the escrow has been set (expect to be done using the governance spell) since a critical storage variable (`escrow`) will have changed.

Further, note the following adjustments have been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

Events and storage variables related to regular operations have been removed from the DV files. Below, more details are listed:

- *L1TokenBridgeProxy*: Events related to normal operation, `ERC20BrdigeFinalized` and `ERC20BridgeInitiated` have been removed.
- *L2TokenBridgeProxy*: Events related to normal operation, `ERC20BrdigeFinalized` and `ERC20BridgeInitiated` have been removed. Furthermore, the event `MaxWithdrawSet` has been removed since this is a parameter governance may change during normal operation without requiring a new DV.
- *L2SUSDSProxy* and *L2USDSProxy*: Events related to normal ERC-20 operation, `Approval` and `Transfer` have been removed. The critical storage variable ``totalSupply`` has been removed, as it is expected to change during normal operation.


## Considerations

Note that the DV files will become invalid on initialization due to new roles being assigned.

However, the files can be validated retroactively with the `--validationblock` option (see the respective `README`).
The published DV files might be updated in the future to reflect the finalized initialization.


## References

Note that addresses are partially validated manually.
Below is the list of references to the expected values used for comparing the address values.

- [Sky Chainlog (May 15th 2025)](https://chainlog.sky.money/) for addresses related to Sky
- [Unichain Contract Addresses (May 15th 2025)](https://docs.unichain.org/docs/technical-information/contract-addresses)
- [Optimism Contract Addresses (May 15th 2025)](https://docs.optimism.io/superchain/addresses)
- [Unichain l2 messenger (May 16th 2025)](https://unichain.blockscout.com/address/0x4200000000000000000000000000000000000007). Note that the address also matches the one of Optimism which is expected given that Unichain is built on the OP stack.
