# Serverless Standard

An infrastructure-as-code framework for building and deploying AWS serverless applications under a single, enforced company standard. It is not a generic wrapper around CloudFormation or SAM — it exists to make naming, tagging, IAM, and code packaging consistent and predictable across every application, every team, and every environment.

The code in this repo is functional and used for real deployments.


## How to Use

Every application is described by a single `template.yaml`. You declare its services and resources; the framework derives the naming, IAM roles/policies, and packaging on its own.

```yaml
serverless-framework-version: "0.3"

info:
  title: My Application
  description: Description in Markdown.
  version: 1.0.0
  team: myteam

services:
  rest-api:
    type: AWS::ApiGateway::RestApi
    name: "demo-rest-api"
    swagger-path: ./definitions/swagger.yaml
  lambda:
    handler: application.services.service1.lambda_handlers.handler
    layers:
      - type: python-requirements
        manifest: ./application/services/service1/requirements.txt

resources:
  lambda:
    - name: "func-get-status"
      handler: application.services.service1.lambda_handlers.status_handler
    - name: "func-get-user"
      handler: application.services.service2.lambda_handlers.user_handler
```

- `services` sets the defaults shared by everything in the application (API type, Lambda handler base config, dependency layers).
- `resources` lists the actual functions, APIs, state machines, and schedules to deploy; each one inherits the `services` defaults and can override them.
- Run `make deploy-all` (or a resource-specific target, see [Getting Started](#getting-started)) and the framework applies the naming convention, builds and uploads the Lambda package, creates/attaches the least-privilege IAM role, and deploys the resource.

More complete examples — REST API, HTTP API, Step Functions, EventBridge schedules, and combinations of all of them — are in `examples/`.


## Why This Instead of Another Serverless Framework

Most serverless frameworks let each team decide how to name resources, structure IAM roles, and package code. That flexibility is exactly what makes large organizations hard to manage: inconsistent naming breaks tooling, over-permissioned roles become a security liability, and every app ends up with its own packaging convention.

This framework removes that flexibility on purpose:

- **Naming and tagging are enforced, not suggested.** Every Lambda function, API, role, and policy is derived from a single naming formula (stage, sub-stage, application, resource name). There is no path to deploy a resource that doesn't follow it.
- **Least-privilege IAM by default.** No admin or wildcard roles. Each role gets only the specific policy statements its resources actually need — a Lambda that never touches Step Functions never receives `states:*`. See [IAM Policy Model](#iam-policy-model) below.
- **Code packaging is standardized.** Lambda code and dependency layers are built, hashed, and uploaded to S3 using a fixed path/naming scheme, so packages are content-addressed and deployments are reproducible.
- **Application definitions are declarative.** A single `template.yaml` per application describes its APIs, functions, schedules, and state machines; the framework derives everything else (roles, policies, packaging, deployment order).


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


## IAM Policy Model

Every role is created per application/stage and starts with zero permissions. Policy documents live under `iam/` as templates (`iam-policy-*.json`) and are rendered per deployment with the specific resource ARNs involved — nothing is granted by wildcard unless the action itself has no resource-level scoping in AWS (e.g. `logs:CreateLogGroup`).

- A Lambda only gets `logs:*` for its own log group, plus X-Ray permissions if tracing is enabled.
- A Lambda only gets `states:StartExecution` on a specific state machine if the application actually invokes one.
- A Step Function execution role only gets `lambda:InvokeFunction` on the functions it actually calls.
- VPC access, SQS access, and API Gateway execution permissions are all attached conditionally, only when the application declares that dependency in `template.yaml`.
- The deployment identity itself (the CI/CD role that runs `make deploy`) is scoped by `iam/serverless-admin-least-privilege-policy.json` — it can create and manage the resources this framework owns, and nothing else.

The result: reading an application's IAM role tells you exactly what it touches, and no resource in the account is one stolen credential away from full access.


## Repository Layout

- `deploy/aws/` — deployment logic per resource type (Lambda, REST API, HTTP API, Step Functions, EventBridge)
- `deploy/gcp/` — equivalent groundwork for GCP Cloud Functions
- `iam/` — IAM policy templates, rendered per deployment with resource-specific ARNs
- `utils/` — shared helpers (IAM, Lambda packaging, API Gateway, Step Functions, SQS, S3, CloudWatch)
- `scripts/` — operational scripts (env injection, cleanup, diagram generation, manual API invocation)
- `examples/` — reference `template.yaml` definitions for common application shapes
- `tests/` — unit tests

See [`FEATURES.md`](FEATURES.md) for the current feature checklist and what's still on the roadmap.


## License

GPL-3.0 — see [`LICENSE`](LICENSE).
