# Features

## Done

- Rollback mechanism
- Lambda deploys a new version, then switches the alias over
- Auto-create and attach IAM role/policy per application
- Safe (versioned, non-destructive) deployment
- VPC support for Lambda
- REST API creation
- API throttling
- IAM authorizer for Lambda (HTTP / REST)
- Pull application definition/code from another repo
- Provisioned Concurrency support
- Auto-removal of old Lambda versions (Lambda has a version/storage quota)
- Decoupled Lambda deployments from application deployments
- Unified definitions across all application types
- AppType: Script (scheduled event)
- AppType: StateMachine (Step Function)
- Scheduled Lambda
- Scheduled State Machine
- Detect and deploy IAM dependencies automatically
- Deploy a specific Lambda function on demand
- Ephemeral storage support
- Stage info injected into Lambda environment variables
- X-Ray support for Lambda / API Gateway
- EventBridge event filters
- Full API Gateway feature set, including direct API-to-S3 upload

## Planned

- Support for Lambda Authorizer
- Auto-removal of old Lambda layers (needs "shared-layers" redesigned to "in-app-shared-layers")
- Configurable Schedule settings (DLQ, logs, roles, retry policy)
- REST API: remove unused routes on deploy
- Pre-deployment spec validation (VPC, IAM, Swagger)
- Resource tagging support
- Lambda Function URL support
- Lambda reserved/provisioned concurrency limits
- SNS support
- SQS support
- Dead-letter queue (DLQ) support
- REST API triggering a Step Function directly
- Attach policy for Lambda to call Step Functions
- Render IAM policy documents with full resource specs

## Explicitly Not Planned

- Enforce an IAM Authorizer on every API — not needed when using a REST Private API instead
