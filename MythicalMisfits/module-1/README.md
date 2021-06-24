# Module 1: IDE Setup and Static Website Hosting

![Architecture](https://github.com/aws-samples/aws-modern-application-workshop/blob/java/images/module-1/architecture-module-1.png)

**Services used:**
* [AWS Cloud9](https://aws.amazon.com/cloud9/)
* [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/)

In this module, follow the instructions to create your cloud-based IDE on [AWS Cloud9](https://aws.amazon.com/cloud9/) and deploy the first version of the static Mythical Misfits website.  [Amazon S3](https://aws.amazon.com/s3/) is a highly durable, highly available, and inexpensive object storage service that can serve stored objects directly via HTTP. This makes it adequate for serving static web content (html, js, css, media, etc.) directly to web browsers.  

Region for this project:

- us-east-2 (Ohio)


### Setting up the AWS CLI Environment

Use the [AWS Cloud9](https://aws.amazon.com/cloud9/) to carry out the commands listed in this module.

### Cloning the Mythical Misfits Repository


```
git clone https://github.com/Sanjiv-Madhavan/AWS-Projects.git
```

```
cd MythicalMisfits
```

## Creating a Static Website in Amazon S3

### Create an S3 Bucket

```
aws s3 mb s3://REPLACE_ME_BUCKET_NAME
```

Check if bucket is created

```
aws s3 ls s3://REPLACE_ME_BUCKET_NAME
```

### Upload the Website Content to your S3 Bucket

Copy the initial page of the Mystical Misfits website (index.html) to your S3 bucket using the [aws s3 cp] command:

```
aws s3 cp ~/environment/MythicalMisfits/module-1/web/index.html s3://REPLACE_ME_BUCKET_NAME/index.html
```

### Update the S3 Bucket Policy

The JSON document for the necessary bucket policy is located at: `~/environment/aws-modern-application-workshop/module-1/aws-cli/website-bucket-policy.json`.  This file contains a string that needs to be replaced with the bucket name you've chosen (indicated with `REPLACE_ME_BUCKET_NAME`). 


**! Before you do that !**

When you are done with the project, you can prevent public access by deleting the bucket policy with the following command:
```
aws s3api delete-bucket-policy --bucket REPLACE_ME_BUCKET_NAME 
```

Execute the following CLI command to add a public bucket policy to your website:

```
aws s3api put-bucket-policy --bucket REPLACE_ME_BUCKET_NAME --policy file://~/environment/MythicalMisfits/module-1/aws-cli/website-bucket-policy.json
```

Your S3 Object should now be accessible:

```
curl -I "https://REPLACE_ME_BUCKET_NAME.s3-website-$(aws configure get region).amazonaws.com/index.html"
HTTP/1.1 200 OK
```

### Configure website Hosting

Configuring [static website hosting](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html) gives you a different endpoint that serves the specified index and error documents, and can evaluate redirection rules. Create a website configuration with the [aws s3 website](https://docs.aws.amazon.com/cli/latest/reference/s3/website.html) command:

```
aws s3 website s3://REPLACE_ME_BUCKET_NAME --index-document index.html
```

Now you can use the website endpoint to serve static content directly from your S3 Bucket. The string to replace **REPLACE_ME_YOUR_REGION** should match whichever region you  created the S3 bucket within (us-east-2):

```
http://REPLACE_ME_BUCKET_NAME.s3-website-REPLACE_ME_YOUR_REGION.amazonaws.com
```

Verify that the static website is being served correctly:

```
curl -I http://REPLACE_ME_BUCKET_NAME.s3-website-REPLACE_ME_YOUR_REGION.amazonaws.com"
HTTP/1.1 200 OK
```

## Visit the initial Mythical Misfits website

![mysfits-welcome](/MythicalMisfits/images/module-1/mysfits-welcome.png)

Congratulations, you have created the basic static Mythical Misfits Website!

That concludes Module 1.

[Proceed to Module 2](/MythicalMisfits/main/module-2)

