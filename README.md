

###### Introduction
It seems "JIRA are not (officially) supporting the option of creating automatic backups for their cloud instance"
We need to automate the Jira backup process and add some value by using existing AWS features to make it efficient and reliable.

### What we wanted?
We wanted a complete solution that we could deploy in the current production AWS account using Cloudformation.
This solution involved the use of AWS features such as Lambda, SQS, Cloudwatch, IAM, Route53 and S3.
We wanted a solution that we could deploy once, keep it free of manual intervention, monitored all the time and we wanted the logging to take care of itself with a 7 days retention period.

### The problem
The backup size was ~3.2GB and it was taking almost 15 minutes to complete.
AWS Lambda has a 15 minutes execution threshold, for this reason the stream of the backup to a S3 bucket was failing.

### How to fix it?
Use AWS SQS
From the documentation:
"Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications."

With help of AWS SQS we can split the Jira backup stage from the file streaming process to S3.
We did it using 2 lambda functions: "JiraBackupInitiate" and "JiraBackupDownload"
We also created a couple of DNS records (public and private in our case)using the bucket name to redirect to Jira.

#### Installation:

Cloudformation template:
https://github.com/tonocastillo/jira-backup-lambdas

You need to replace some entries within the the Cloudformation template according to what you have in your environment.
These are the ones:

<INSERT_S3_BUCKET_HERE>
<PRIVATE_HOSTED_ZONE_ID>
<PUBLIC_HOSTED_ZONE_ID>
<SNS_TOPIC_ARN>
<ACCOUNT_ID>
<AWS_REGION>
              "JIRA_HOST": "<ACCOUNT>.atlassian.net",
              "JIRA_EMAIL": "<ACCOUNT_EMAIL",
              "API_TOKEN": "<INSERT_YOUR_API_TOKEN_HERE",
              "INCLUDE_ATTACHMENTS": "true",
              "UPLOAD_TO_S3": {
                  "S3_BUCKET": "<INSERT_S3_BUCKET_HERE>"

Once replaced, go to Cloudformation in the AWS console and create a stack with this template.

Some resources were hardcoded in the template since having this solution in one region is enough but you can add some parameters to the template and make changes when creating a new stack in Cloudformation.


Resources:
The original idea for this backup came from here:
https://github.com/datreeio/jira-backup-py
Thank you.
