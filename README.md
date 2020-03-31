

### Introduction
It seems "JIRA are not (officially) supporting the option of creating automatic backups for their cloud instance".
We need to automate the Jira backup process and add some value by using existing AWS features to make it efficient and reliable.

A complete solution was designed in Cloudformation that could be deployed in the current AWS production account.

This solution involved the utilization of AWS features such as Lambda, SQS, Cloudwatch, IAM, Route53 and S3.

It is a solution meant to be deployed once, keep it free of manual intervention and monitored all the time.

#### The problem
The backup file size was big enough (~3.2GB) and the backup process was taking close to 15 minutes to complete.

AWS Lambda has a 15 minutes execution threshold, for this reason the stream of the backup to a S3 bucket was failing.

#### How to fix it?
Use AWS SQS.

From the documentation:

"Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications."

With help of AWS SQS we can split the Jira backup stage from the file streaming process to S3.

We also created a couple of DNS records in Route53 (public and private in our case) using the bucket name to redirect to Jira (It is not necessary). 


### Installation:


Cloudformation templates:

https://github.com/tonocastillo/jira-backup-lambdas

You need to replace some entries within the the Cloudformation templates according to what you have/want in your environment.

These are the ones:

Within the jira-backup.yml template

- JiraBackupInitiate:
```
          config = {
              "JIRA_HOST": "<ACCOUNT>.atlassian.net",
              "JIRA_EMAIL": "<ACCOUNT-EMAIL>",
              "API_TOKEN": "<ACCOUNT TOKEN>",
              "INCLUDE_ATTACHMENTS": "true"
          }
```
- JiraBackupDownload
```
          config = {
              "JIRA_HOST": "<ACCOUNT>.atlassian.net",
              "JIRA_EMAIL": "<ACCOUNT-EMAIL>",
              "API_TOKEN": "<ACCOUNT TOKEN>",
              "UPLOAD_TO_S3": {
                  "S3_BUCKET": "<BUCKET_NAME>"
              }
          }
```
You need to modify *<LAYERS-BUCKET>* within the lambda-layers.yml template to use the bucket you want to use for layers.
  


### Resources:

The original idea for this backup came from:

https://github.com/datreeio/jira-backup-py

Thank you.
