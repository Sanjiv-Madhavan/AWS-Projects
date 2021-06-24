# Build a Modern Application on AWS (Java)

The initial Mythical Mysfits architecture features:
* A static website served static directly from [Amazon S3](https://aws.amazon.com/s3)
* Application architecture microservices, deployed as a container to [AWS Fargate](https://aws.amazon.com/fargate/)

We are going to use the [AWS Command Line Interface](https://aws.amazon.com/cli/) to create resources, instead of the [AWS Console](https://aws.amazon.com/console/) that you may be more familiar. This allows all changes to be automatically pushed to production and continuously delivering value to customers. This project includes a fully managed build and deployment pipeline utilizing [AWS CodeCommit](https://aws.amazon.com/codecommit/), [AWS CodeBuild](https://aws.amazon.com/codebuild/), and [AWS CodePipeline](https://aws.amazon.com/codepipeline/).  There is no specific software requirements, any modern browser should work, leveraging the cloud-based IDE, [AWS Cloud9](https://aws.amazon.com/cloud9/).


Go to [Module 1: IDE Setup and Static Website Hosting](/MythicalMisfits/module-1)
