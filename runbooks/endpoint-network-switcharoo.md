# Endpoint Network Switcharoo
This runbook explains how to take a new set of virtual machines (VMs) running upgraded software for one of our endpoints, deploy them to the endpoint, then remove the existing set of virtual machines running outdated software from our endpoints...ideally with zero downtime for our customers.

### Index
1. [Prerequisites](#prerequisites)
1. [Steps](#steps)
1. [Next Steps](#next-steps)
1. [See Also](#see-also)

## Prerequisites
This runbook is based on several assumptions that must be met.
1. This upgrade has been approved by all relevant stakeholders.
1. You have an Amazon Web Services (AWS) IAM user account with the necessary permissions to perform the upgrade(s) in the `evm-testnet` and/or `evm-mainnet` AWS accounts.
1. The new virtual machine(s):
    1. Are in the correct [availability zones](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/aws-region.md) (AZs),
    1. Have been initialized with upgraded software,
    1. Are prepared to pass health checks if they are functioning correctly, and;
    1. Will fail health checks if they are not functioning correctly.

## Steps
Here are the steps to take a new set of virtual machines (VMs) running upgraded software for one of our endpoints, deploy them to the endpoint, then remove the existing set of virtual machines running outdated software from our endpoints...ideally with zero downtime for our customers.
1. Login to the [AWS web console](https://console.aws.amazon.com).
1. Switch to the intended [region in AWS](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/aws-region.md).
1. Perform a smoke test on the existing endpoint to verify everything is working before making any changes.
    - To guarantee your traffic is being served from these endpoints, use a public virtual private network (VPN) to connect to this part of the world for the smoke test. After the smoke test, you can disconnect.
1. Perform a smoke test on each and every individual virtual machine to verify the new virtual machines are working as expected.
1. [EC2](https://console.aws.amazon.com/ec2/home) > Target Groups > `${TARGET_GROUP_NAME}` > Targets > Register targets
    - For example, if you are upgrading the testnet RPC API in the Asia-Pacific datacenter, `${TARGET_GROUP_NAME}` might be `evm-testnet-ap-api-tg`.
1. Under "Available instances," check the instances with the upgraded software.
    - Following the previous example, `evm-testnet-ap-api-vm-1-v0.4.1` and `evm-testnet-ap-api-vm-2-v0.4.1`.
1. Click "Include as pending below."
1. Verify the upgraded virtual machines (VMs) appear in "Review targets," then click "Register pending targets."
1. Wait for the "Health status" column of the new instances to change from "initial" to "healthy."
	- This took about one minute for me.
	- If the health status changes to "unhealthy," or anything besides "healthy," stop here and escalate the situation! We need to investigate further.
1. Under "Registered targets," check the VMs that are running the outdated software.
1. Click "Deregister."
1. The "Health status" column of the old instances will be "draining." Wait until these instances disappear.
	- This took a long time for me, maybe ten minutes.
	- This process is where AWS verifies no traffic is being routed to these instances before removing them.
1. Perform a smoke test on the public endpoint to verify everything is working.
    - To guarantee your traffic is being served from these endpoints, use a public virtual private network (VPN) to connect to this part of the world for the smoke test. After the smoke test, you can disconnect.

Repeat this process for the other regions.

## Next Steps
You may want to leave the outdated virtual machines (VMs) up for 12-24 hours in case there is an issue with the upgrade and you need to rollback. The process to rollback is the same as the upgrade process, where the VMs running the previous version are added and the VMs running the current version are removed. After a sufficient amount of time with no reported issues using the upgraded software, the VMs running outdated software should be terminated to minimize costs.

## See Also
- EOS-EVM [Cloud Infrastructure](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/README.md)
    - [Availability Zones](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/aws-region.md)
    - [Regions](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/aws-region.md)
- [eos-evm](https://github.com/eosnetworkfoundation/eos-evm) - core EOS Ethereum virtual machine source code
- [Runbooks](./README.md)
