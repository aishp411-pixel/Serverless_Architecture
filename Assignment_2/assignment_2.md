# Automated S3 Bucket Cleanup Using AWS Lambda and Boto3
Create an AWS Lambda function to automatically delete files older than 30 days from an S3 bucket using Boto3. This helps keep the bucket clean and manage storage efficiently.

## 1) Lambda IAM Role:
Create a role for lambda function and add `AmazonS3FullAccess` , `CloudWatchLogsFullAccess` policy to this role

<img width="1897" height="876" alt="Screenshot 2025-08-14 161209" src="https://github.com/user-attachments/assets/55510578-9422-4d95-b991-a6a72a5052e6" />

## 2) Lambda Function
Create a Lambda function myFunction_1 and assign the role `lambda-s3-role` created previously to the function

<img width="1899" height="871" alt="Screenshot 2025-08-14 161301" src="https://github.com/user-attachments/assets/f998e4f7-6fd6-4da9-8572-a82bb2d73a68" />

## 3) Write Boto3 Python script 

```python


import boto3
import json
from datetime import datetime, timezone, timedelta


s3 = boto3.client('s3')


BUCKET_NAME = 'aishwaryabucket-411'  
LOG_PREFIX = ''                
CUTOFF_DAYS = 2


def lambda_handler(event, context):
   
    cutoff_date = datetime.now(timezone.utc) - timedelta(days=CUTOFF_DAYS)


    # List objects in the bucket
    response = s3.list_objects_v2(Bucket=BUCKET_NAME, Prefix=LOG_PREFIX)


    # Print full raw response (optional - useful for debugging)
    print("Raw Response from list_objects_v2:")
    print(json.dumps(response, default=str, indent=2))  # default=str to handle datetime


    # Check if there are any files
    if 'Contents' not in response:
        print("No log files found.")
        return


    deleted = 0


    for obj in response['Contents']:
        key = obj['Key']
        last_modified = obj['LastModified']
        size = obj['Size']


        # Log each object detail
        print(f"\nFound file:")
        print(f" - Key: {key}")
        print(f" - Last Modified: {last_modified}")
        print(f" - Size: {size} bytes")


        # Delete if older than cutoff
        if last_modified < cutoff_date:
            s3.delete_object(Bucket=BUCKET_NAME, Key=key)
            deleted += 1
            print(f" --> Deleted: {key}")


    print(f"\nDeleted {deleted} old log file(s).")
```
Create a TestEvent and Deploy

## 4) Test

Create or select an S3 bucket that contains a mix of log files — some older than 30 days and some newer. This will allow you to test that the Lambda function correctly deletes only the old logs while retaining the recent ones

**Note:-** *In my case, I didn’t have bucket with objects older than 30 daysso i used a bucket named aishwarya-bucket411 containing a files added 2 days prior and 0 days prior for testing. 
This helped verify that only files older than 2 days were deleted, while newer ones remained untouched.* <br>



S3 bucket `aishwarya-bucket411` has 4 files older than 2 days and 2 file recent before trigerring the lambda

<img width="1547" height="752" alt="Screenshot 2025-08-17 174115" src="https://github.com/user-attachments/assets/7ca00d26-d808-4b23-a424-c680fbd4f5cc" />

<br>
Trigerring the Lambda with Bucket `aishwarya-bucket411` , log_prefix `` and cutoff_days 2 (considering it as 30 )

<img width="1822" height="1003" alt="Screenshot 2025-08-17 174542" src="https://github.com/user-attachments/assets/ed035566-0265-4e44-8280-6ed018b308f2" />

Lambda Deleted 4 files 

<img width="1589" height="733" alt="Screenshot 2025-08-17 174621" src="https://github.com/user-attachments/assets/72ea92df-9ad9-45d8-97ac-f17058933722" />

S3 bucket after trigerring the Lambda 


<img width="1550" height="751" alt="Screenshot 2025-08-17 174728" src="https://github.com/user-attachments/assets/19084b1d-f067-45cb-86de-0d96bd42899d" />


