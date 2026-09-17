# Infrastructure

Terraform definitions for the FitFlow AWS environment.

| Resource | Purpose |
|---|---|
| VPC, subnets, security groups | Network isolation for services and data stores |
| ECS Fargate cluster + services | Runs the core API and the AI microservice |
| RDS PostgreSQL (Multi-AZ) | System of record, encrypted with KMS |
| ElastiCache Redis | Cache and Socket.IO pub/sub backplane |
| S3 + CloudFront | Media storage and delivery |
| SQS + EventBridge | Asynchronous event processing |
| WAF + API Gateway/ALB | Edge protection and routing |
| Secrets Manager + KMS | Secret storage and rotation |

```bash
terraform init
terraform plan -var-file=envs/dev.tfvars
terraform apply -var-file=envs/dev.tfvars
```
