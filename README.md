# AI-OPs


https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/bedrockagentcore_agent_runtime

# Task 1: Explore the AWS Components Required for Bedrock AgentCore

## Overview

This document outlines the AWS components required to deploy and run a Bedrock AgentCore agent, based on a successful manual deployment using the `agentcore` CLI.

Deployment was performed using: `agentcore configure` and `agentcore launch`


The following AWS resources were created and analyzed.

---

## 1. Amazon ECR (Elastic Container Registry)

**Purpose:**  
Stores the Docker container image of the AgentCore application.


**Why Required:**
- AgentCore Runtime pulls the container image from ECR.
- Containerized execution is mandatory for AgentCore.

---

## 2. IAM Roles

Two types of IAM roles were created during deployment.

### A. Runtime Execution Role

**Example:**
AmazonBedrockAgentCoreSDKRuntime-us-east-1-653715a7b6

**Trust Relationship:**

**Purpose:**
Allows the AgentCore runtime to:
- Pull container images from ECR
- Write logs to CloudWatch
- Interact with Bedrock services

**Status:**  
Required for all deployments.

---

### B. CodeBuild Execution Role (Cloud Build Mode Only)

**Example:**

**Policy Includes Permissions For:**
- ECR image push (`PutImage`, `UploadLayerPart`, etc.)
- S3 source download
- CodeBuild CloudWatch logging

**Purpose:**
Used only when deploying via cloud-based build (`agentcore launch` without local Docker).

**Status:**  
Optional. Not required if container is built and pushed manually.

---

## 3. AWS CodeBuild

**Purpose:**
- Builds the Docker image in AWS
- Pushes the image to ECR
- Used when no local container engine is installed

**Status:**  
Optional. Used only for CLI-managed cloud builds.

---

## 4. Amazon S3 (Temporary Source Storage)

**Purpose:**
- Stores uploaded source code for CodeBuild

**Status:**  
Indirect dependency. Required only in cloud build mode.

---

## 5. Bedrock AgentCore Runtime

**Observed ARN:**
**Purpose:**
- Managed container execution environment
- Hosts and runs the AI agent
- Exposes runtime endpoint for invocation


**Terraform Equivalent Resource:**
aws_bedrockagentcore_agent_runtime

**Status:**  
Core required component.


---

## 6. Amazon CloudWatch Logs

**Purpose:**
- Stores runtime logs
- Stores telemetry logs
- Enables monitoring and debugging

**Status:**  
Auto-created but essential for observability.

---

# Summary of Required Components

## Mandatory for AgentCore Runtime

- Amazon ECR Repository
- IAM Runtime Execution Role
- Bedrock AgentCore Runtime
- CloudWatch Logs
- Network Configuration (PUBLIC or VPC)

## Optional (CLI Convenience Components)

- CodeBuild Project
- CodeBuild IAM Role
- Temporary S3 Source Bucket

---

# Conclusion

The `agentcore launch` CLI abstracts multiple AWS components including container builds, IAM role creation, and runtime provisioning.

For Terraform-based deployments, the minimum required infrastructure includes:

- ECR for container storage
- IAM execution role for runtime
- Bedrock AgentCore runtime resource
- Logging configuration

Build-related services such as CodeBuild and S3 can be excluded if container images are built and pushed externally (e.g., via CI/CD pipeline).

---
