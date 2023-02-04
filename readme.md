
# Codepipeline technical challenge

## Description
The challenge requires to create codepipeline to deploy simple index.html file.

The file will be stored in challenge-deployment repository and will be deployed to S3 bucket once changes occur in main branch. CodePipeline is configured only with source and deployment steps for this challenge.<br/> 
Our codepipeline and its creation will be stored in another repository called challenge-configurations in order to prevent unnecessary deployments when we change or update configurations.</br>

## Prerequisites
I chose to use GitHub version 2 for connection.<br/>
In order to AWS communicate with GitHub successfully AWS Connector for GitHub application needs to be installed and configured.

## 1. Сreate buckets for artifacts and deployment
```
aws s3api create-bucket \
    --bucket bucket-for-artifacts.v1 \
    --region eu-central-1 \
    --create-bucket-configuration LocationConstraint=eu-central-1 \
    --acl private

aws s3api create-bucket \
    --bucket bucket-for-deployments.v1 \
    --region eu-central-1 \
    --create-bucket-configuration LocationConstraint=eu-central-1 \
    --acl private
```
## 2. Block public access for our buckets
```
aws s3api put-public-access-block \
    --bucket bucket-for-artifacts.v1 \
    --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

aws s3api put-public-access-block \
    --bucket bucket-for-deployments.v1 \
    --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```
## 3. Create connection for GitHub
```
aws codestar-connections create-connection  \
--provider-type GitHub   \
--connection-name connect-with-github
```
At this stage we want to save arn value provided in output to pass it to our pipeline. 
Also we need to change status to "Active" in AWS UI Console

## 4. Create policy and role for pipeline
```
aws iam create-policy   \
    --policy-name policy-for-codepipeline  \
    --policy-document file://policy-pipeline.json

aws iam create-role   \
    --role-name role-for-codepipeline   \
    --assume-role-policy-document file://role-pipeline.json 

aws iam attach-role-policy   \
    --policy-arn <arn-policy-value>   \
    --role-name role-for-codepipeline
```
## 5. Create pipeline
It's necessary to populate the field "ConnectionArn" in source step with value provided in output when we created codestar connection.<br/>
```
aws codepipeline create-pipeline --cli-input-json file://pipeline.json
```


