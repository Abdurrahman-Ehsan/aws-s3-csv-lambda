AWS S3 CSV Billing Data Parser & Validation

A serverless Python application that automatically processes CSV files uploaded to Amazon S3 using AWS Lambda and Boto3. When an S3 ObjectCreated (PUT) event is configured for the bucket, the Lambda function is automatically triggered, retrieves and validates the uploaded CSV file, and moves invalid files to a dedicated error bucket before deleting them from the original bucket.

Overview

This project demonstrates an event-driven serverless workflow for automatically processing and validating billing CSV files stored in Amazon S3.

Once the Lambda function is connected to an S3 PUT/ObjectCreated event, the workflow runs automatically whenever a CSV file is uploaded.

The Lambda function:

Receives the S3 object-created event.
Extracts the source bucket and uploaded CSV file name.
Retrieves the CSV file from Amazon S3 using Boto3.
Parses the CSV contents using Python's csv module.
Validates billing data such as product, currency, bill amount, and date.
Detects invalid billing records.
If an error is found, copies/moves the CSV file to a dedicated error bucket.
Deletes the invalid file from the original bucket.
Allows valid files to remain in the source bucket.
Architecture
                CSV File Upload
                       |
                       v
              Amazon S3 Bucket
                       |
                ObjectCreated
                  (PUT Event)
                       |
                       v
                  AWS Lambda
                       |
                    Boto3
                       |
                       v
              Retrieve CSV File
                       |
                       v
              Parse & Validate
                       |
                 +-----+-----+
                 |           |
               Valid       Invalid
                 |           |
                 v           v
          Keep in Source   Error Bucket
             Bucket            |
                               v
                       Delete from Source
                            Bucket
Technologies Used
Python
Boto3
AWS Lambda
Amazon S3
AWS S3 Event Notifications
Python CSV module
AWS SAM / CloudFormation YAML
Project Structure
Billing_Bucket_Parser/
│
├── S3_Bucket_Object_Parser.py
├── event.json
├── template.yaml
└── README.md
Automatic S3 Event-Driven Processing

The key feature of this project is its integration with an Amazon S3 ObjectCreated (PUT) event.

After configuring the Lambda function as a destination for the S3 bucket's PUT event, no manual Lambda invocation is required.

The workflow becomes:

Upload CSV to S3
       ↓
S3 PUT/ObjectCreated Event
       ↓
Lambda automatically triggered
       ↓
CSV retrieved with Boto3
       ↓
CSV data validated
       ↓
   ┌───┴────┐
   ↓        ↓
 Valid    Invalid
   ↓        ↓
Remain   Move to
in S3    Error Bucket
            ↓
        Delete original

This makes the application suitable for automated serverless data processing.

CSV Processing

The Lambda function extracts the bucket and object information directly from the S3 event:

billing_bucket = event['Records'][0]['s3']['bucket']['name']
csv_file = event['Records'][0]['s3']['object']['key']

It then uses Boto3 to retrieve the uploaded object:

obj = s3.Object(billing_bucket, csv_file)

data = obj.get()['Body'].read().decode('utf-8').splitlines()

The CSV header is skipped and the remaining records are parsed:

for row in csv.reader(data[1:], delimiter=','):
Data Validation

The application validates billing records against predefined requirements, including:

Product
Currency
Bill amount
Date

For example:

valid_product_lines = ['Bakery', 'Meat', 'Dairy']

valid_currencies = ['USD', 'MXN', 'CAD']

Invalid records can be identified during processing.

Error File Handling

When the uploaded CSV contains invalid data, the application can automatically handle the file by:

Moving/copying the invalid CSV to the designated error bucket.
Deleting the original CSV from the source bucket.

This prevents invalid files from remaining in the normal processing bucket while preserving them in a separate location for investigation or correction.

Source Bucket
     |
     | Invalid CSV
     v
Error Bucket
     |
     v
Original deleted
from Source Bucket
AWS Permissions

The Lambda execution role requires appropriate permissions to access and manage the S3 objects involved in the workflow.

Depending on the implementation, this includes permissions such as:

s3:GetObject
s3:PutObject
s3:DeleteObject

The Lambda role should also have access to the relevant source and error buckets.

For production environments, permissions should follow the principle of least privilege.

Testing

A sample S3 event is provided in:

event.json

The event represents the information Lambda receives when an object is created in an S3 bucket.

For a complete end-to-end test, upload an actual CSV file to the configured S3 bucket and allow the S3 PUT/ObjectCreated event to trigger Lambda automatically.

Security
Do not hard-code AWS access keys or secret keys.
Use IAM roles for Lambda execution.
Restrict S3 permissions according to the principle of least privilege.
Do not commit credentials or sensitive information to GitHub.
Restrict access to the error bucket appropriately.
Learning Objectives

This project provided hands-on experience with:

Python AWS automation
Boto3
AWS Lambda
Amazon S3
S3 event notifications
Event-driven serverless architecture
CSV parsing
Data validation
S3 object management
Error-file handling
AWS IAM permissions
AWS SAM / CloudFormation configuration
Future Improvements

Possible improvements include:

More comprehensive CSV validation.
Detailed validation reports.
CloudWatch monitoring and alerts.
Automated deployment using Terraform.
Processing and validation of larger CSV files.
Adding a separate successful-processing bucket.
Adding automated notifications when invalid files are detected
