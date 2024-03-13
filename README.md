# AWS SES Email Forwarder

This project provides a serverless solution to automatically forward emails received on AWS Simple Email Service (SES) to a specified email address, using AWS Lambda and S3 for processing and storage. It's particularly useful for handling emails on a custom domain without setting up a full email server.

Any email to any possible address on your domain will be redirected to the email address specified in Lambda in the form in which it was sent. You will receive a letter from the address to which it was sent. It is very convenient and does not require creating additional aliases and you can extend the message processing logic in Python.

It is very important not to create a bucket in advance. It will need to be created when you create a rule for processing incoming messages in SES. Otherwise, you may receive an error when adding processing for saving emails to S3

You may be wondering why you need to save emails to S3 at all? It would be good, but unfortunately the contents of the letter do not come to the Lambda event, there is only metadata.

## Prerequisites

-   An AWS account
-   A verified domain in AWS SES
-   AWS CLI installed and configured (optional, but useful for deployment)

## Setup and Deployment

### 1\. Verify Your Domain with SES

Before you begin, ensure your domain is verified in AWS SES. This process involves adding DNS records to your domain's DNS settings. Follow the AWS SES documentation for domain verification.

### 2\. Create an S3 Bucket

Create an S3 bucket where incoming emails will be stored. Note down the bucket name as it will be used in the Lambda function and SES rules.

### 3\. Deploy the Lambda Function

The lambda\_function.py contains the logic to read emails from S3 and forward them to your specified address. Deploy this Lambda function in the AWS Lambda console or use the AWS CLI.
```
aws lambda create-function --function-name EmailForwarder  \
--runtime python3.12 --role <ROLE\_ARN>  \
--handler lambda\_function.lambda\_handler  \
--zip-file fileb://path/to/your/lambda/package.zip
```

Replace <ROLE\_ARN> with your Lambda execution role ARN and adjust the file paths as necessary.

### 4\. Set Up SES Receipt Rules

In the SES console, create a receipt rule for your domain to store incoming emails to the S3 bucket and trigger the Lambda function.

For Lambda functions triggered by SES, the event object structure looks something like this:
```json
{
  "Records": [
    {
      "eventSource": "aws:ses",
      "eventVersion": "1.0",
      "ses": {
        "mail": {
          "messageId": "example-id",
          "source": "sender@example.com",
          "timestamp": "example-timestamp"
          "destination": [
              "test@example.com"
          ],
          // Other metadata fields...
        },
        "receipt": {
          // Receipt details...
        }
      }
    }
  ]
}

```

Example IAM Policy for S3 Access
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-email-bucket/*"
        }
    ]
}
```


### 5\. Update IAM Policies

Ensure the Lambda execution role has permissions for ses:SendRawEmail, and s3:GetObject. Refer to the policy examples provided in the troubleshooting section of this document.

## Configuration

Configure the lambda\_function.py script with your source and destination email addresses, and ensure the S3 bucket name is correctly set.

## Usage

Once deployed, any email sent to your domain will be automatically forwarded to the specified email address. The process is automated and requires no manual intervention.

## Troubleshooting

-   **Access Denied Errors**: Ensure your IAM roles and policies are correctly configured.
-   **Emails Not Forwarding**: Check the SES receipt rule setup and ensure the Lambda function is triggered correctly.

## Contributing

Contributions are welcome! Please feel free to submit a pull request or create an issue for any bugs or feature requests.

## License

MIT
