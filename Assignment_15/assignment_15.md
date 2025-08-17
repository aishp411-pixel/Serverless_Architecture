# Implement a Log Cleaner for S3
Develop an AWS Lambda function using Boto3 to automatically clean up old log files in an S3 bucket. The function should identify and delete logs older than 90 days, and run weekly using EventBridge.

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


BUCKET_NAME = 'findmyfaculty'  
LOG_PREFIX = 'resumes/'                
CUTOFF_DAYS = 90


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

Create or select an S3 bucket that contains a mix of log files — some older than 90 days and some newer. This will allow you to test that the Lambda function correctly deletes only the old logs while retaining the recent ones

**Note:-** *In my case, I didn’t have actual log files, so I used an existing S3 bucket named findmyfaculty containing a resumes/ folder with random files for testing. 
This helped verify that only files older than 90 days were deleted, while newer ones remained untouched.* <br>



Initially S3 bucket `findmyfaculty` has 7 files older than 90 days and 1 file recent

<img width="1892" height="970" alt="Screenshot 2025-08-14 202157" src="https://github.com/user-attachments/assets/24b091fe-09ec-4b6f-af0a-0ad1d9c939a5" />

<br>
Trigerring the Lambda with Bucket `findmyfaculty`  and log_prefix `resumes/`
<img width="1892" height="869" alt="Screenshot 2025-08-14 202509" src="https://github.com/user-attachments/assets/b31aee61-ee84-4757-ae68-a6caed368155" />


Lambda Deleted 7 files 
<img width="1894" height="782" alt="Screenshot 2025-08-14 202741" src="https://github.com/user-attachments/assets/f5000957-0d31-4bbb-9230-d6763db13a5a" />


After trigerring the Lambda 
<img width="1550" height="561" alt="Screenshot 2025-08-14 202548" src="https://github.com/user-attachments/assets/ac51a302-0560-4a26-a931-32837855e174" />



## 5) Scheduled this function to run weekly using AWS EventBridge
Create a Rule `weekly-log-cleaner` , choose rate 7 days and Target `myFunction_1`



<img width="1897" height="874" alt="Screenshot 2025-08-14 203837" src="https://github.com/user-attachments/assets/406b411a-5fad-4f7b-9c0e-a78654f2a208" />



