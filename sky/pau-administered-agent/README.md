---
Title:    PAU Administered Agent DV
Author:   ChainSecurity  
Date:     8. Jun, 2026
Client:   Sky
---

# Deployment Validation Details: PAU Administered Agent

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation).

- PAU Administered Agent Contracts:
    - Repository: https://github.com/sky-ecosystem/pau-administered-agent
    - Commit: `bfaaf709a8664d74d12604455f0365a0a12439cf`
    - Contracts:
        - AdministeredAgentFactory: `0x2968c3b5478cf93b70ab1e24255d4edbbd27a089`

## Details

We can confirm that the bytecode matches the given repository at the given commit.

The `AdministeredAgentFactory` is a stateless, permissionless factory: its only function, `deploy(address admin)`, creates a new `AdministeredAgent(admin)` and emits `AdministeredAgentDeployed`. It has no immutables, no constructor arguments, and no storage — so its integrity rests entirely on the codehash.

Only the factory is in scope. The `AdministeredAgent` instances it deploys (access-controlled agents with actor/admin/grantor/revoker role sets) are deployed per-call and are not part of this DV.

Further, note the following adjustment has been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *AdministeredAgentFactory*: All `critical_events` were removed. The sole event, `AdministeredAgentDeployed`, is a deployment-record event emitted on every (permissionless) `deploy` call; it signals no security-relevant state change on the factory, which holds no storage.

## Considerations

The factory is stateless and immutable; there is no configurable state that could invalidate the DV.

## References

The contract has no immutables or constructor arguments, so there are no external addresses to validate.
