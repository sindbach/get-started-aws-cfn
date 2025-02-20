# Get-Started AWS CloudFormation

Repository to help getting started with using MongoDB Atlas with AWS CloudFormation.

## Information


This Get-Started project will deploy the MongoDB Atlas AWS Quick Start which provisions complete MongoDB Atlas deployments through CloudFormation using official MongoDB Atlas AWS CloudFormation Resource Types. The project provides a quick and simple way to use the [MongoDB Atlas AWS Quick Start](https://github.com/aws-quickstart/quickstart-mongodb-atlas) and the [MongoDB Atlas CloudFormation Resources](https://github.com/aws-quickstart/quickstart-mongodb-atlas-resources).

After you `get-started.sh` with this Get-Started project you will have a complete MongoDB Atlas deployment managed through AWS CloudFormation. This includes:

* 1 Project
* 1 M10 MongoDB Atlas Cluster
* 1 AWS IAM Role
* 1 MongoDB Atlas Database User (Type/AWS IAM Role)
* 1 Project Ip Access List entry
* VPC Peering (optional) [TODO/ need add this option to get-started]

The outputs include a lambda ready IAM Role and connection string to your new MongoDB Atlas Cluster.

*NOTE* This project integrates with the AWS and MongoDB cloud which can incur cost when deploying the project! (TODO - make nicer)

## Pre-requisites 

### AWS Tooling

In order to use this Get-Started project you need to install the AWS cli on your machine.
You can download and install from: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

An AWS Account with appropriate CloudFormation and IAM permissions. 
Please see the section [AWS IAM Permissions](#aws-iam-permissions) for details.

### Docker 

Have Docker running on your machine. You can download and install from: https://docs.docker.com/install/

### `mongocli`

The best way to manage your MongoDB Cloud apikeys today is `mongocli`(https://github.com/mongodb/mongocli). This project can leverage your `mongocli` configuration.

### MongoDB Atlas

In order to execute the code example, you need to have: 

* Create an organizational-level [MongoDB Atlas Programmatic API](https://docs.atlas.mongodb.com/configure-api-access#programmatic-api-keys). The key needs `Project Creator` permissions.

Once created, run `mongocli configure` and enter the API Key just created.

##  Execution Steps 

### Deploy MongoDB Atlas CFN Resource into your AWS Region

#### `get-setup.sh`

1. Execute the helper shell setup script to complete this step. This will package and deploy the MongoDB Atlas CloudFormation resources into your current default AWS region: 

  ```
  ./get-setup.sh 
  ```

  You can optionally pass in the region or change your local `aws` cli configuration. 

  ```
  ./get-setup.sh us-west-2
  ```
  or
  ```
  aws configure set region eu-west-3
  ./get-setup.sh
  ```
  Note this step can take up to 45 minutes to run.
  Run this step once in each region you wish to use.

  Once complete, you will find a set of CFN Stacks for the MongoDB Atlas Resources.

#### `get-started.sh`

2. Execute the helper shell starter script, optionally providing a project name. The output from `get-setup.sh` helper script will inform you of the details for your new MongoDB Atlas deployment, including AWS AIM Role and Cluster connection string information for you apps. Note this step takes 7-10 minutes. 

If you have installed `mongocli` then run:

  ```
  ./get-started.sh <GETSTARTED_NAME> 
  ```

Or you can explicitly set the apikey or get prompted:

  ```
  ./get-started.sh <PUBLIC_KEY> <PRIVATE_KEY> <ORG_ID> <GETSTARTED_NAME> 
  ```

  Once successful, you should be able to access your new deployment through the AWS console, the Atlas console or even the clis.

## Connecting to your cluster

You can see the connection information in the AWS CloudFormation stack output.

```bash
GETSTARTED_NAME="get-started-aws-quickstart"
MDB=$(aws cloudformation list-exports | \
 jq -r --arg stackname "${GETSTARTED_NAME}" \
 '.Exports[] | select(.Name==$stackname+"-ClusterSrvAddress") | .Value')
echo "Found stack:${GETSTARTED_NAME} with ClusterSrvAddress: ${MDB}"
```

_Note_ This example requires the `jq` tool. See: https://stedolan.github.io/jq/download/

### Testing AWS IAM connection

There is a helper script [test-iam-mongo-connection.sh](./test-iam-mongo-connection.sh) available to script passing AWS IAM session credentials into the mongo shell. To use this, pass the STACK_NAME as a parameter:

```bash
./test-iam-mongo-connection.sh NewRoleBased-1
Found stack:NewRoleBased-1 with ClusterSrvAddress: mongodb+srv://cluster-1.5hmwe.mongodb.net
STACK_ROLE={
    "StackResources": [
        {
            "StackName": "NewRoleBased-1",
...
MongoDB shell version v4.4.1
connecting to: mongodb://cluster-1-shard-00-00.5hmwe.mongodb.net:27017,cluster-1-shard-00-01.5hmwe.mongodb.net:27017,cluster-1-shard-00-02.5hmwe.mongodb.net:27017/admin?authMechanism=MONGODB-AWS&authmechanismproperties=AWS_SESSION_TOKEN%3AIQoJb3JpZ2luX2VjEMv%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJIMEYCIQDQ4mV7RtfYCLrQwBg
...
PRIMARY>
 ```


## Tear Down 

How to remove the environment setup (deleting traces of this get-started project):

* Delete the Quick Start stack using the helper script or you can do this in the AWS Web Console or cli yourself.
  ```
  ./teardown.sh <GETSTARTED_NAME>
  ```

* Terminate the `get-started.sh` process if it's running. This is to stop the web service on `localhost:3000`.
* Delete the CloudFormation stack created, by default this will have the <Quick Start-Name>:
   ```
   aws cloudformation delete-stack --stack-name <Quick Start-Name>
   ```
* Remove the Docker volumes
* Remove the Docker image

## Tutorials

TODO - add links to repos, example stacks, using this with lambda

## About 

This project is part of the MongoDB Get-Started code examples. Please see [get-started-readme](https://github.com/mongodb-developer/get-started-readme) for more information. 


## Developer notes Notes

Not required unless needing to refesh with latest resource source code.
This will build a fresh image of the resources for stable distribution.

   Build Docker image with a tag name. Within the top level directory execute: 
  ```
  docker build . -t mongodb-developer/get-started-aws-cfn
  ```
   This will build a docker image with a tag name `get-started-aws-cfn`. 

   *NOTE* Currently the source repositories are private which will prevent a clean build without proper Github ssh access. A pre-build image has been upload for convience until these repos become public: `mongodb-developer/get-started-aws-cfn`. 

To build the container - need this:
```
export DOCKER_BUILDKIT=1
docker build --ssh github=$HOME/.ssh/id_rsa -t atlas-aws .
```

This docker image is currently built internally and published to:


public.ecr.aws/u1r4t8v5/mongodb-developer/get-started-aws-cfn:latest

## Troubleshoot

### Check access to docker image

Try this command to check if you can access the Docker image required for this project.

```bash
docker run -it public.ecr.aws/u1r4t8v5/mongodb-developer/get-started-aws-cfn "head -1 /quickstart-mongodb-atlas-resources/README.md"
# MongoDB Atlas AWS CloudFormation Resources & Quickstart
```

## AWS IAM Permissions 

In order to run this project you will need a certain set of AWS permissions.
We've included a sample minimal-example [policy.yaml](./policy.yaml) which you can assume safely and use with this project.

First, create a new role with the supplied policy. Here's how to do that via CloudFormation:


```
aws cloudformation update-stack --capabilities CAPABILITY_NAMED_IAM --template-body file://./policy.yaml --stack-name get-started-aws-cfn-role
```

You can then assume the role with `aws sts assume-role`. We recommend this for exporting your AWS environment into the docker environment to run this project, like this (note you need to change the --role-arn to the arn create in the step above).

```
source <(aws sts assume-role --role-arn arn:aws:iam::<YOUR_AWS_ACCOUNT>:role/MongoDB-Atlas-CloudFormation-Get-Started --role-session-name "get-started"  | jq -r  '.Credentials | @sh "export AWS_SESSION_TOKEN=\(.SessionToken)\nexport AWS_ACCESS_KEY_ID=\(.AccessKeyId)\nexport AWS_SECRET_ACCESS_KEY=\(.SecretAccessKey) "')
```

Remember to UNSET your `AWS_*` environment variables to "un-assume" the role.

```
source <(env | awk -F= '/AWS/ {print "unset ", $1}'
```

(Reference: [get-token.md](https://gist.github.com/brianredbeard/035ee1419bc38a0e2d854fb828d585d7))
