
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/bedrockagentcore_agent_runtime

# Deploying AWS Bedrock AgentCore Using Terraform
## Overview

This project demonstrates how to design and implement Infrastructure as Code (IaC) for AWS Bedrock AgentCore using Terraform.

---

## Architecture Overview
- Build Layer   → Docker
- Infra Layer   → Terraform (ECR + IAM)
- Deploy Layer  → Terraform (Runtime + Endpoint)

## AWS Components Used

## 1.Amazon ECR (Elastic Container Registry)

Used to store the container image for the AgentCore runtime, which pulls image from here

Resource:
`aws_ecr_repository`

## 2.IAM Role for Agent Runtime
- `aws_iam_role`
- `aws_iam_role_policy`

```json
{
  "Effect": "Allow",
  "Action": "sts:AssumeRole",
  "Principal": {
    "Service": "bedrock-agentcore.amazonaws.com"
  }
}
```

## 3.IAM Permissions Required

### ECR Access

```
ecr:GetAuthorizationToken
ecr:BatchGetImage
ecr:GetDownloadUrlForLayer
```

Allows runtime to pull container image.

### CloudWatch Logs

```
logs:CreateLogGroup
logs:CreateLogStream
logs:PutLogEvents
```

Allows runtime container to write logs.

### Bedrock Model Invocation

```
bedrock:InvokeModel
bedrock:InvokeModelWithResponseStream
bedrock:Converse
bedrock:ConverseStream
```

## 4.Agent Runtime

`aws_bedrockagentcore_agent_runtime`
Defines:

- Runtime name
- Execution role
- Container image URI
- Network mode (PUBLIC)

## 5. Runtime Endpoint

Resource:

`aws_bedrockagentcore_agent_runtime_endpoint`

Purpose:

- Creates invokable endpoint
- Required before invoking runtime
- Automatically provisions `DEFAULT` qualifier


- `terraform apply -target=aws_ecr_repository.agentcore -target=aws_iam_role.runtime -target=aws_iam_role_policy.runtime`
- Push image to ECR
- `terraform apply -var="image_tag=v1"`
