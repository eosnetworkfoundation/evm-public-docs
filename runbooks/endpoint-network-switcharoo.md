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
    1. Are prepared to pass [health checks](../endpoint-health-checks.md) if they are functioning correctly, and;
    1. Will fail [health checks](../endpoint-health-checks.md) if they are not functioning correctly.

## Steps
Here are the steps to take a new set of virtual machines (VMs) running upgraded software for one of our endpoints, deploy them to the endpoint, then remove the existing set of virtual machines running outdated software from our endpoints...ideally with zero downtime for our customers.
1. Login to the [AWS web console](https://console.aws.amazon.com).
1. Switch to the intended [region in AWS](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/aws-region.md).
    ![1](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/91370d24-6668-4993-ab1e-ef127b370dd2)
1. Perform all smoke tests on the existing endpoint to verify everything is working before making any changes.
    ![2](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/cf1405c5-2183-4616-bde2-515bd17f0431)
    - To guarantee your traffic is being served from these endpoints, use a public virtual private network (VPN) to connect to this part of the world for the smoke tests. After the smoke tests, you can disconnect.
1. Perform all relevant smoke tests on each and every individual virtual machine to verify the new virtual machines are working as expected.
1. [EC2](https://console.aws.amazon.com/ec2/home) > Target Groups > `${TARGET_GROUP_NAME}` > Targets > Register targets
    ![3](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/66582579-eac3-4583-9ac1-473f179444b6)
    - For example, if you are upgrading the testnet RPC API in the Asia-Pacific datacenter, `${TARGET_GROUP_NAME}` might be `evm-testnet-ap-api-tg`.
1. Under "Available instances," check the instances with the upgraded software.
    ![4](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/fc5b5364-c5a3-4201-aa0b-6dc6a9e3f907)
    - Following the previous example, `evm-testnet-ap-api-vm-1-v0.4.1` and `evm-testnet-ap-api-vm-2-v0.4.1`.
1. Click "Include as pending below."
    ![5](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/8a2bc028-a5e2-4eae-83e9-9886d11b8e68)
1. Verify the upgraded virtual machines (VMs) appear in "Review targets," then click "Register pending targets."
    ![6](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/5d870711-107a-4bd9-af27-494b6076ac90)
1. The "Health status" column of the new instances will be "initial."
    ![7](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/c4f0b5d0-e938-46e0-a0ff-8ec41abf7c0c)
    Wait for the "Health status" column of the new instances to change from "initial" to "healthy."
    ![8](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/f5af43ed-c67b-49d8-942c-ee8695a652d0)
	- This took about one minute for me.
	- If the health status changes to "unhealthy," or anything besides "healthy," stop here and escalate the situation! We need to investigate further.
1. Under "Registered targets," check the VMs that are running the outdated software.
    ![9](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/1678635d-06c2-4ad1-a6a3-53f241261570)
1. Click "Deregister."
    ![A](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/3c44c332-f425-46d6-ba82-e112168af2f2)
1. The "Health status" column of the old instances will be "draining."
    ![B](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/a5a035d8-994a-41aa-b3e7-13ccd46e75e9)
    Wait until these old instances disappear.
    ![C](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/53e9659a-1221-4469-bc1f-9583155b6950)
	- This took a long time for me, maybe ten minutes.
	- This process is where AWS verifies no traffic is being routed to these instances before removing them.
1. Perform a smoke test on the public endpoint to verify everything is working.
    ![D](https://github.com/eosnetworkfoundation/evm-public-docs/assets/34947245/7b59981d-d91d-453c-92dd-1c8290b89461)
    - To guarantee your traffic is being served from these endpoints, use a public virtual private network (VPN) to connect to this part of the world for the smoke test. After the smoke test, you can disconnect.

Repeat this process for the other regions.

## Next Steps
You may want to leave the outdated virtual machines (VMs) up for 12-24 hours in case there is an issue with the upgrade and you need to rollback. The process to rollback is the same as the upgrade process, where the VMs running the previous version are added and the VMs running the current version are removed. After a sufficient amount of time with no reported issues using the upgraded software, the VMs running outdated software should be terminated to minimize costs.

## See Also
- EOS-EVM [Cloud Infrastructure](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/README.md)
    - [Availability Zones](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/aws-region.md)
    - [Endpoint Health Checks](../endpoint-health-checks.md)
    - [Regions](https://github.com/eosnetworkfoundation/eos-evm-internal/blob/main/cloud/aws-region.md)
- [eos-evm](https://github.com/eosnetworkfoundation/eos-evm) - core EOS Ethereum virtual machine source code
- [Runbooks](./README.md)
