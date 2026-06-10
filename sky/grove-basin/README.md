---
Title:    Grove Basin DV
Author:   ChainSecurity  
Date:     10. Jun, 2026
Client:   Grove
---

# Deployment Validation Details: Grove Basin

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation).

- Grove Basin Contracts:
    - Repository: https://github.com/grove-labs/grove-basin
    - Commit: `6fdcbd5173dffc54d172e23fca083e441560f737`
    - Contracts:
        - FixedRateProvider: `0x7928a185b8137d1cd2a0996a810a04db2837419d`
        - ChronicleRateProvider (BUIDL): `0x69a171853575ffd41574ea80abfc6337acbc4d43`
        - ChronicleRateProvider (JTRSY): `0x29209cecfefa6f675e6f1f829320d67ce2b025e5`
        - GroveBasinFactory: `0x78dc98d689fe9a1b0056ac1cdfc14722bda6d49a`
        - UsdsUsdcPocket (BUIDL): `0x621727a05db6aeb33118b3f9de3eaf2d8fc86ada`
        - UsdsUsdcPocket (JTRSY): `0xa15b8c07fa32a4f8bea3882600a673dc9cc1d6b9`
        - BUIDLTokenRedeemer: `0x99e5e7c533c7319f855b940561df285be022c82d`
        - JTRSYTokenRedeemer: `0x212697f0a9fc218210d98cd1a159dc8d8a87b8a8`
        - GroveBasin (BUIDL): `0x10b3d3a96646720f8b3a29229cf96d513f3c84f1`
        - GroveBasin (JTRSY): `0x1fa4db8d545cbd22b7bba2084348a2e6ef36e363`
        - TimelockController (BUIDL): `0xdb8c7c814e9780659b23478ef4bda9032cc9ff34`
        - TimelockController (JTRSY): `0xa52dc9876ab4a9db6dafbb83410554086054d140`

## Details

The deployed bytecode matches the source repository at the given commit (see [Scope](#scope)), and addresses have been manually and/or automatically validated against a list of references, see [References](#references). The on-chain configuration matches the `grove-basin` deployment scripts, except for the two `GroveBasin`s and their `TimelockController`s, which were subject to post-deployment actions not present in the scripts (see below). These actions are meaningful and deliberate — they finalize the role holders and fully de-privilege the deployer — rather than misconfigurations.

The two `GroveBasin`s were initialized according to the deploy scripts. Their access control, however, was subsequently modified by actions not present in the scripts:

- On the **BUIDL** basin only, the redeemers were rotated as part of a Securitize issuer rotation: the old `BUIDLTokenRedeemer` (`0x0d46f8a832b76a79ac3b5f29fffc35aceebad885`, `REDEEMER_CONTRACT_ROLE`) and old Securitize redeemer (`0xdfc603076ea75895dd4d59c6e2ee5038f881cb74`, `REDEEMER_ROLE`) were revoked and the roles re-granted to the current `BUIDLTokenRedeemer` and Securitize redeemer. The JTRSY basin registered its redeemers directly and was not changed.
- On **both** basins, the deployer's `MANAGER_ADMIN_ROLE` was revoked, which the deploy scripts leave granted (reported in the audit report).

The resulting access control is held as follows:

| Role | BUIDL basin | JTRSY basin |
|---|---|---|
| `OWNER_ROLE` | BUIDL admin `TimelockController` | JTRSY admin `TimelockController` |
| `MANAGER_ADMIN_ROLE` | `GROVE_SUBPROXY` | `GROVE_SUBPROXY` |
| `MANAGER_ROLE` | `ALM_RELAYER` | `ALM_RELAYER` |
| `PAUSER_ROLE` | `ALM_FREEZER` | `ALM_FREEZER` |
| `REDEEMER_ROLE` | Securitize redeemer | JTRSY redeemer |
| `REDEEMER_CONTRACT_ROLE` | `BUIDLTokenRedeemer` | `JTRSYTokenRedeemer` |

The two `TimelockController`s (the basins' admins) were deployed by the `DeployTimelockController` script, which sets `minDelay` = 1 week and grants (constructor + explicit `CANCELLER`) `DEFAULT_ADMIN_ROLE` → self and the deployer, `EXECUTOR_ROLE` → `GROVE_SUBPROXY`, `CANCELLER_ROLE` → `ALM_FREEZER`, and `PROPOSER_ROLE`/`CANCELLER_ROLE` → the constructor proposer (the deployer for BUIDL, the JTRSY issuer multisig `0x9184ddbcc4824b76ce2aefa72534a1a87aa5037c` for JTRSY). The following actions were then performed post-deployment, none present in the script. On the BUIDL side they form a Securitize issuer rotation — the old issuer multisig and redeemers were rotated out for the current ones:

**BUIDL timelock:**

1. Granted `PROPOSER_ROLE` and `CANCELLER_ROLE` to the old Securitize issuer multisig (`0x551e841e6fb54431a0664c8776784f6d7e611428`).
2. Revoked `PROPOSER_ROLE` and `CANCELLER_ROLE` from the deployer.
3. Granted `PROPOSER_ROLE` and `CANCELLER_ROLE` to the current Securitize issuer multisig (`0x453a28b31fdc31858c35b02bc3a42bcd8bfbad3a`).
4. Revoked `PROPOSER_ROLE` and `CANCELLER_ROLE` from the old Securitize issuer multisig.
5. Revoked `DEFAULT_ADMIN_ROLE` from the deployer.
6. Scheduled one operation (`CallScheduled`): a no-op call to the Securitize issuer multisig — empty calldata, zero value, 7-day delay — still pending (no `CallExecuted`/`Cancelled`).

**JTRSY timelock:**

1. Revoked `DEFAULT_ADMIN_ROLE` from the deployer.
2. Scheduled one operation (`CallScheduled`): a no-op call to the JTRSY issuer multisig — empty calldata, zero value, 7-day delay — still pending (no `CallExecuted`/`Cancelled`).

The pending operation's ETA is recorded in the `_timestamps[operationId]` slot; an empty no-op is consistent with a liveness/test operation rather than a state-changing action. The rotated-out Securitize addresses (old issuer multisig, redeemer and token-redeemer) hold no roles in the final state, consistent with the address registry flagging them as "must not hold any roles". As on the basins, the deployer's `DEFAULT_ADMIN_ROLE` revoke was reported in the audit report. The resulting access control is held as follows:

| Role | BUIDL timelock | JTRSY timelock |
|---|---|---|
| `DEFAULT_ADMIN_ROLE` | self (timelock) | self (timelock) |
| `PROPOSER_ROLE` | Securitize issuer multisig | JTRSY issuer multisig |
| `EXECUTOR_ROLE` | `GROVE_SUBPROXY` | `GROVE_SUBPROXY` |
| `CANCELLER_ROLE` | `ALM_FREEZER`, Securitize issuer multisig | `ALM_FREEZER`, JTRSY issuer multisig |

Across the DV files, address fields (constructor arguments, immutables and address storage) were annotated with `value_hint`s identifying the referenced tokens and contracts. Further, note the following adjustments have been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *GroveBasinFactory*: its only event, `GroveBasinDeployed` (a deployment record), was removed.
- *GroveBasin* and *TimelockController*: the role-membership, pause, seed-share and scheduled-operation slots (nested/computed `keccak256` slots reported as `unknown`) were programmatically labeled (`var_name`, `var_type`, `offset`, `value_hint`) from the deploy scripts and source. All storage was retained.

No further adjustments were made — the rate providers, pockets and token redeemers were validated as deployed.

## Considerations

The `GroveBasin`s' configuration (fees, swap-size and staleness bounds, rate providers, pocket and pause state) and the access control on the basins and `TimelockController`s are governance-tunable; changing any of these alters storage and will invalidate the corresponding DV file. The files reflect the post-deployment state described above, including each timelock's pending queued operation.

The files can be validated retroactively with the `--validationblock` option (see the respective `README`).

## References

Note that addresses are validated manually and/or automatically.
Below is the list of references to the expected values used for comparing the address values.

- [Grove Address Registry](https://github.com/grove-labs/grove-address-registry/blob/master/src/Ethereum.sol) for `ALM_PROXY`, `ALM_RELAYER`, `ALM_FREEZER`, `BUIDLI_REDEEM` and `BUIDL` (also [Etherscan](https://etherscan.io/token/0x7712c34205737192402172409a8f7ccef8aa2aec)).
- [Sky Chainlog](https://chainlog.skyeco.com/) for `USDS`, `WRAPPER_USDS_LITE_PSM_USDC_A` and `GROVE_SUBPROXY`; [USDC Documentation](https://developers.circle.com/stablecoins/usdc-contract-addresses#mainnet) for `USDC`.
- [Chronicle Proof of Asset](https://chroniclelabs.org/dashboard/proofofasset/) for the `ChronicleRateProvider` oracles — [BUIDL](https://chroniclelabs.org/dashboard/proofofasset/blackrock-buidl) and [JTRSY](https://chroniclelabs.org/dashboard/proofofasset/janus-henderson-anemoy-treasury-fund); [Centrifuge Deployments](https://docs.centrifuge.io/developer/protocol/deployments/) for the `JTRSY` token and its vault.
- Issuer-provided (`grove-basin` deploy address list): `SECURITIZE_ISSUER_MULTISIG` (`0x453a28b31fdc31858c35b02bc3a42bcd8bfbad3a`) and `JTRSY_ISSUER_MULTISIG` (`0x9184ddbcc4824b76ce2aefa72534a1a87aa5037c`) hold `PROPOSER_ROLE`/`CANCELLER_ROLE`; `SECURITIZE_REDEEMER` (`0xcbeee2c39601e1ee5502f2593f6758e6598c47a6`) and `JTRSY_REDEEMER` (`0xb6e8d3e47c4fc5606e6c24d097dd1791885ce05a`) hold `REDEEMER_ROLE`.
