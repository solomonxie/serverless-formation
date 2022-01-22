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


## HOW TO RUN

1. Create an application repo

2. Create a Tag of the application repo

3. Setup AWS account with a profile name on local machine: `~/.aws/credentials`
```
[my-profile-name]
region = us-east-1
aws_access_key_id = abc
aws_secret_access_key = def
```

4. Specify environment variables listed in `envfile`:
    method-1) `$ export AWS_REGION=us-east-1; export xxx=abc`
    method-2) Add variables into `envfile-local`, and run `$ ./scripts/inject_envfile.sh`

5. Run deployment:
```
$ make deploy

#or
$ make deploy-lambda
$ make deploy-rest-api
$ make deploy-stepfunc
$ make deploy-schedules
```


## Naming Convention

- Lambda Function Full NAME: "${StageName}-${StageSubName}-${ApplicationName}-${FunctionName}"
- LAMBDA FUNCTION PATH: "lambda-function/${FunctionFullName}/${BUILD_NO}.zip"
- Lambda Package Layer NAME: "lambda-layer-${ManifestMd5}"
- Lambda Package Layer PATH: "lambda-layer/${ManifestMd5_LEVELED_DIR}/{ManifestMd5}.zip"
- API: "${StageName}-${StageSubName}-${ApplicationName}-${ApiName}"
- ApiStage: only one -> "latest_release", also points to each Lambda's alias "latest_release"


## Staging

- StageName=prod: (having production env variables and vpc settings)
    - StageSubName=main: production release
    - StageSubName=beta: beta release
    - StageSubName=prev: previous release for easy rollback & debug
- StageName=dev: (having dev env variables, developer has R/W permission)
    - StageSubName=feature1: independent deployment CD for each new feature development
    - StageSubName=feature2: independent deployment CD for each new feature development


