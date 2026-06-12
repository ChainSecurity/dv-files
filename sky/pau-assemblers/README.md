---
Title:    PAU Assemblers DV
Author:   ChainSecurity  
Date:     12. Jun, 2026
Client:   Sky
---

# Deployment Validation Details: PAU Assemblers

This document outlines the details regarding the deployment validation performed.

## Scope

The deployment validation (DV) has been performed with the [Deployment Validation Tool](https://github.com/ChainSecurity/deployment_validation).

- PAU Assemblers Contracts:
    - Repository: https://github.com/sky-ecosystem/pau-assemblers
    - Commit: `d7d6f084604d4f7e45879a3b15d03ee870231467`
    - Contracts:
        - DefaultPAUAssembler: `0xc812aad3fae2d3511c664374b601a9bebfecca2e`

## Details

The deployed bytecode matches the source repository at the given commit (see [Scope](#scope)), and addresses have been manually and/or automatically validated against a list of references, see [References](#references).

The `DefaultPAUAssembler` is stateless (no `critical_storage_variables`); its two immutables — the `AdministeredAgentFactory` and the `PAUFactory` — match the constructor arguments.

Further, note the following adjustment has been made to the initial DV files (see [DV tool's README](https://github.com/ChainSecurity/deployment_validation?tab=readme-ov-file#step-2---validate-data-and-select-constraints)):

- *DefaultPAUAssembler*: its only event, `Deployment` (a deployment record), was removed.

## References

Note that addresses are validated manually and/or automatically.
Below is the list of references to the expected values used for comparing the address values.

- The `AdministeredAgentFactory` (`0x2968c3b5478cf93b70ab1e24255d4edbbd27a089`) and `PAUFactory` (`0x69a5d548830ac2a4ba90a44a2c75bda71f97fc66`) are validated against their own DV files (`sky/pau-administered-agent` and `sky/diamond-pau`).
