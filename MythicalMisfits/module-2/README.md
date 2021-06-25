# Module 2: Creating a Service with AWS Fargate

![Architecture](/MythicalMisfits/images/module-2/architecture-module-2.png)


**Services used:**
* [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
* [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/)
* [Amazon Virtual Private Cloud (VPC)](https://aws.amazon.com/vpc/)
* [Amazon Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/)
* [Amazon Elastic Container Service (ECS)](https://aws.amazon.com/ecs/)
* [AWS Fargate](https://aws.amazon.com/fargate/)
* [AWS Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)
* [AWS CodeCommit](https://aws.amazon.com/codecommit/)
* [AWS CodePipeline](https://aws.amazon.com/codepipeline/)
* [AWS CodeDeploy](https://aws.amazon.com/codedeploy/)
* [AWS CodeBuild](https://aws.amazon.com/codebuild/)


## Overview

In Module 2 I have replaced the static list of misfits by a computed list obtained from a new microservice API hosted using [AWS Fargate](https://aws.amazon.com/fargate/) on [Amazon Elastic Container Service](https://aws.amazon.com/ecs/). 

## Creating the Core Infrastructure using AWS CloudFormation

### VPC setup used in this stack :

![VPC](/MythicalMisfits/images/module-2/vpc-module-2.png)

```
aws cloudformation create-stack --stack-name MythicalMisfitsCoreStack --capabilities CAPABILITY_NAMED_IAM --template-body file://~/environment/${YOUR_PATH}/module-2/cfn/core.yml
  
```

You can check on the status of your stack creation either via the AWS Console or by running the command:


```
aws cloudformation wait stack-create-complete --stack-name MythicalMisfitsCoreStack && echo "stack created"
```

Save the names and ids of created resources by CloudFormation to use in the next steps of this workshop. 

```
aws cloudformation describe-stacks --stack-name MythicalMisfitsCoreStack > ~/environment/cloudformation-core-output.json
```

In this workshops architecture this core infrastructure resources are stable and shared across application deployments. 

## Deploying a Service with AWS Fargate

### Building A Docker Image

* Update Java to OpenJDK 1.8
```

sudo yum -y install java-1.8.0-openjdk-devel
sudo alternatives --set java /usr/lib/jvm/java-1.8.0-openjdk.x86_64/bin/java
sudo alternatives --set javac /usr/lib/jvm/java-1.8.0-openjdk.x86_64/bin/javac
```

* Install [Apache Maven](https://maven.apache.org)
```
sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo

sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo

sudo yum install -y apache-maven
```

* Build the application at `~/environment/MythicalMisfits//module-2/app/service`

```
cd ~/environment/MythicalMisfits/aws-modern-application-workshop/module-2/app/service
mvn clean install
```
* Build docker Image

```
cd ..
docker build . -t REPLACE_ME_ACCOUNT_ID.dkr.ecr.REPLACE_ME_REGION.amazonaws.com/mythicalmisfits/service:latest
```

```
docker run -p 8080:8080 REPLACE_ME_WITH_DOCKER_IMAGE_TAG
```

As a result you will see docker reporting that your container is up and running locally:

```
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
```

### Pushing the Docker Image to Amazon ECR

```
aws ecr create-repository --repository-name mythicalmisfits/service
```

To obtain authentication credentials for our Docker client to the repository.

```
$(aws ecr get-login --no-include-email)
```

```
docker push REPLACE_ME_WITH_DOCKER_IMAGE_TAG
```

## Configuring the Service Prerequisites in Amazon ECS

### Create an ECS Cluster

```
aws ecs create-cluster --cluster-name MythicalMisfits-Cluster
```

#### Create an AWS CloudWatch Logs Group

To create the new log group in CloudWatch logs, run the following command:

```
aws logs create-log-group --log-group-name mythicalmisfits-logs
```

### Register an ECS Task Definition

Replace the indicated values in the ``` environment/aws-modern-application-workshop/module-2/aws-cli/task-definition.json ```with the appropriate ones from your created resources.

```
aws ecs register-task-definition --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/task-definition.json
```


## Enabling a Load Balanced Fargate Service

```
aws elbv2 create-load-balancer --name misfits-nlb --scheme internet-facing --type network --subnets REPLACE_ME_PUBLIC_SUBNET_ONE REPLACE_ME_PUBLIC_SUBNET_TWO > ~/environment/nlb-output.json
```

```
aws elbv2 create-target-group --name MythicalMisfits-TargetGroup --port 8080 --protocol TCP --target-type ip --vpc-id REPLACE_ME_VPC_ID --health-check-interval-seconds 10 --health-check-path / --health-check-protocol HTTP --healthy-threshold-count 3 --unhealthy-threshold-count 3 > ~/environment/target-group-output.json
```

```
aws elbv2 create-listener --default-actions TargetGroupArn=REPLACE_ME_NLB_TARGET_GROUP_ARN,Type=forward --load-balancer-arn REPLACE_ME_NLB_ARN --port 80 --protocol TCP
```

## Creating a Service with Fargate

### Creating a Service Linked Role for ECS

The ECS service would not be granted permissions to perform the actions required.  To create the role, execute the following command in the terminal:

```
aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com
```

### Create the Service

Open ```~/environment/aws-modern-application-workshop/module-2/aws-cli/service-definition.json``` in the IDE and replace the indicated values of `REPLACE_ME`. 

```
aws ecs create-service --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/service-definition.json
```

### Check

```
#Replace with your NLB DNS name
http://misfits-nlb-123456789-abc123456.elb.us-east-1.amazonaws.com/misfits
```


## Update Mythical Misfits to Call the NLB

You'll need to update the following file to use the same NLB URL for API calls (do not inlcude the /misfits path): `/module-2/web/index.html`

Open the file in Cloud9 and replace the highlighted area below between the quotes with the NLB URL:

![before replace](/MythicalMisfits/images/module-2/before-replace.png)

After pasting, the line should look similar to below:

![after replace](/MythicalMisfits/images/module-2/after-replace.png)

### Upload to S3

```
aws s3 cp ~/environment/aws-modern-application-workshop/module-2/web/index.html s3://INSERT-YOUR-BUCKET-NAME/index.html
```

http://REPLACE_ME_BUCKET_NAME.s3-website-REPLACE_ME_YOUR_REGION.amazonaws.com"

## Automating Deployments using AWS Code Services

![Architecture](/MythicalMisfits/images/module-2/architecture-module-2b.png)

### Create a S3 Bucket for Pipeline Artifacts

Create bucket for PipeLine Artifacts 

```
aws s3 mb s3://REPLACE_ME_ARTIFACTS_BUCKET_NAME
```

Once you've modified and saved this file, execute the following command to grant access to this bucket to your CI/CD pipeline:

```
aws s3api put-bucket-policy --bucket REPLACE_ME_ARTIFACTS_BUCKET_NAME --policy file://~/environment/aws-modern-application-workshop/module-2/aws-cli/artifacts-bucket-policy.json
```

### Create a CodeCommit Repository

```
aws codecommit create-repository --repository-name MythicalMisfitsService-Repository
```

### Create a CodeBuild Project


To create the CodeBuild project, Replace values in  `~/environment/aws-modern-application-workshop/module-2/aws-cli/code-build-project.json`.  

```
aws codebuild create-project --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/code-build-project.json
```

### Create a CodePipeline Pipeline

open `~/environment/aws-modern-application-workshop/module-2/aws-cli/code-pipeline.json` and replace the required attributes within, and save the file.

```
aws codepipeline create-pipeline --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/code-pipeline.json
```

### Enable Automated Access to ECR Image Repository


Update `~/environment/aws-modern-application-workshop/module-2/aws-cli/ecr-policy.json`.  and save this file and then run the following command to create the policy:

```
aws ecr set-repository-policy --repository-name mythicalmisfits/service --policy-text file://~/environment/aws-modern-application-workshop/module-2/aws-cli/ecr-policy.json
```

## Test the CI/CD Pipeline

### Using Git with AWS CodeCommit

```
git config --global user.name "REPLACE_ME_WITH_YOUR_NAME"
```

```
git config --global user.email REPLACE_ME_WITH_YOUR_EMAIL@example.com
```

```
git config --global credential.helper '!aws codecommit credential-helper $@'
```

```
git config --global credential.UseHttpPath true
```

Next change directories in your IDE to the environment directory using the terminal:

```
cd ~/environment/
```

Clone tur repository :

```
git clone https://git-codecommit.REPLACE_REGION.amazonaws.com/v1/repos/MythicalMisfitsService-Repository
```

```
cp -r ~/environment/aws-modern-application-workshop/module-2/app/* ~/environment/MythicalMisfitsService-Repository/
```

### Pushing a Code Change

Make a change to the service to demonstrate that the CI/CD pipeline we've created is working. In Cloud9, open the file stored at `~/environment/MythicalMisfitsService-Repository/service/src/main/resources/json/misfits-response.json` and change the age of one of the misfits to another value and save the file.


```
cd ~/environment/MythicalMisfitsService-Repository/
```

Push: 

```
git add .
git commit -m "I changed the age of one of the misfits."
git push
```

![Pipeline](/MythicalMisfits/images/module-2/Pipeline-module-2.PNG)

This concludes The project.



