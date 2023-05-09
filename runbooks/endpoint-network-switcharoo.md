# Endpoint Network Switcharoo
This runbook explains how to take a new set of virtual machines running upgraded software for one of our endpoints, deploy them to the endpoint, then remove the existing set of virtual machines running outdated software from our endpoints...ideally with zero downtime for our customers.

### Index
1. [Prerequisites](#prerequisites)
1. [See Also](#see-also)

## Prerequisites
This runbook is based on several assumptions that must be met.
1. This upgrade has been approved by all relevant stakeholders.
1. You have an Amazon Web Services (AWS) IAM user account with the necessary permissions to perform the upgrade(s) in the `evm-testnet` and/or `evm-mainnet` AWS accounts.
1. The new virtual machine(s):
    1. Are in the correct availability zones (AZs),
    1. Have been initialized with upgraded software,
    1. Are prepared to pass health checks if they are functioning correctly, and;
    1. Will fail health checks if they are not functioning correctly.

## See Also
- [Runbooks](./README.md)
    - [Upgrade Runbooks](./README.md#upgrades)
