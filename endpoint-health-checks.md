# Endpoint Health Checks
This document describes the health checks we are using for the EOS-EVM endpoint infrastructure.

### Index
1. [Health Check Definition](#health-check-definition)
1. [Bridge](#bridge)
1. [Explorer](#explorer)
1. [Faucet](#faucet)
1. [RPC API](#rpc-api)

## Health Check Definition
What is a health check?
> In the context of cloud infrastructure, a health check refers to a monitoring mechanism that assesses the status and availability of various components within a cloud-based system. It is commonly used to ensure that the cloud resources, such as virtual machines, load balancers, databases, or applications, are functioning properly and able to handle incoming requests.
>
> Health checks are typically performed at regular intervals by an automated process or a dedicated monitoring service. The checks are designed to verify the responsiveness and performance of the infrastructure components. The specific criteria and tests involved in a health check can vary depending on the nature of the resource being monitored.
>
> During a health check, the system typically sends requests or probes to the resources being monitored and analyzes the responses received. The health check process may examine factors such as response times, error rates, network connectivity, and overall availability. By evaluating these parameters, the system can determine whether the resource is functioning correctly or if there are any issues that need attention.
>
> The results of health checks are used to make informed decisions about managing the cloud infrastructure. For example, if a health check identifies a component that is not responding or is experiencing performance issues, it can trigger an automatic response, such as restarting the resource or diverting traffic to alternative resources to maintain the overall availability and reliability of the system.
>
> Health checks are crucial for maintaining the operational efficiency and resilience of cloud infrastructure, as they help identify and address potential problems proactively. By continuously monitoring the health of the resources, cloud operators can ensure optimal performance and minimize downtime, leading to improved service quality for users or customers.

## Bridge
The EOS-EVM bridge is currently determined to be healthy if it returns a 200-299 status code at HTTP:80/. We are using the default Amazon Web Services (AWS) health check parameters which, at the time of this writing, are:
- 30 second interval
- 5 second timeout
- Two consecutive failure threshold to transition to "unhealthy"
- Five consecutive success threshold to transition to "healthy"

## Explorer
The EOS-EVM explorer is currently determined to be healthy if it returns a 200-299 status code at HTTP:80/. We are using the default Amazon Web Services (AWS) health check parameters which, at the time of this writing, are:
- 30 second interval
- 5 second timeout
- Two consecutive failure threshold to transition to "unhealthy"
- Five consecutive success threshold to transition to "healthy"

## Faucet
The EOS-EVM faucet is currently only available on the testnet, and is hosted by EOS-Nation. We are using the default Amazon Web Services (AWS) health check parameters which, at the time of this writing, are:
- 30 second interval
- 5 second timeout
- Two consecutive failure threshold to transition to "unhealthy"
- Five consecutive success threshold to transition to "healthy"

## RPC API
The EOS-EVM RPC API is considered healthy if it returns a 200-299 status code at HTTP:8000/. We are using the default Amazon Web Services (AWS) health check parameters which, at the time of this writing, are:
- 30 second interval
- 5 second timeout
- Two consecutive failure threshold to transition to "unhealthy"
- Five consecutive success threshold to transition to "healthy"

***
**_Legal notice_**  
This document was generated in collaboration with ChatGPT from OpenAI, a machine learning algorithm or weak artificial intelligence (AI). At the time of this writing, the [OpenAI terms of service agreement](https://openai.com/terms) §3.a states:
> Your Content. You may provide input to the Services (“Input”), and receive output generated and returned by the Services based on the Input (“Output”). Input and Output are collectively “Content.” As between the parties and to the extent permitted by applicable law, you own all Input, and subject to your compliance with these Terms, OpenAI hereby assigns to you all its right, title and interest in and to Output.

This notice is required in some countries.
