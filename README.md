# Serverless Standard

A simple yet powerful framework to build serverless applications with a well curated standard of best practices.


## What Can / Cannot Be Automated

The framework automates "direct-connection" infrastructure — the wiring AWS supports natively between its own services. Anything that reaches outside that boundary is intentionally left to a manual ticket, so that access to sensitive resources always has a human decision behind it.

Direct connections (automated):
- API Gateway -> Lambda
- Step Function -> Lambda
- EventBridge -> Lambda
- EventBridge -> Step Function

Indirect connections (not automated, requires an Ops/DBA ticket):
- Anything involving S3
- Anything involving a database
- Lambda -> API Gateway
- Lambda -> Step Function
- Lambda -> other services


## Getting Started

1. Create an application repository containing your Lambda source code.
2. Tag the application repository at the commit you want to deploy.
3. Configure an AWS CLI profile on the deploying machine:
   ```
   # ~/.aws/credentials
   [my-profile-name]
   region = us-east-1
   aws_access_key_id = abc
   aws_secret_access_key = def
   ```
4. Set the environment variables listed in `envfile`, either:
   - `export AWS_REGION=us-east-1 APPLICATION_NAME=abc ...`, or
   - populate `envfile-local` and run `./scripts/inject_envfile.sh`
5. Deploy:
   ```
   make deploy-all

   # or deploy individual resource types
   make deploy-lambda
   make deploy-rest-api
   make deploy-stepfunc
   make deploy-event
   ```

See `examples/` for complete `template.yaml` definitions covering REST APIs, HTTP APIs, Step Functions, EventBridge schedules, and combinations of all of them.


## Naming Convention

- Lambda function full name: `${StageName}-${StageSubName}-${ApplicationName}-${FunctionName}`
- Lambda function package path: `lambda-function/${FunctionFullName}/${BUILD_NO}.zip`
- Lambda layer name: `lambda-layer-${ManifestMd5}`
- Lambda layer package path: `lambda-layer/${ManifestMd5_LEVELED_DIR}/${ManifestMd5}.zip`
- API name: `${StageName}-${StageSubName}-${ApplicationName}-${ApiName}`
- API stage: a single stage, `latest_release`, which also points to each Lambda alias `latest_release`

Layer packages are content-addressed by the hash of their dependency manifest, so identical dependency sets are built and uploaded once and reused across functions and applications.


## Staging

- `StageName=prod` — production environment variables and VPC settings
  - `StageSubName=main` — production release
  - `StageSubName=beta` — beta release
  - `StageSubName=prev` — previous release, kept for fast rollback
- `StageName=dev` — development environment variables, developer read/write access
  - `StageSubName=<feature>` — one independent deployment per feature branch


