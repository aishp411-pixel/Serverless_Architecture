# Assignment 5: Auto-Tagging EC2 Instances on Launch Using AWS Lambda and Boto3
Automatically tag newly launched EC2 instances with the current date and a custom label using AWS Lambda. This ensures consistent and automated resource tagging for better management and tracking.
## 1) EC2 setup:
Create a EC2 instance Instance-2 and stop it (note the tags)

<img width="1901" height="876" alt="Screenshot 2025-08-17 205248" src="https://github.com/user-attachments/assets/b350de03-41ec-4a25-8925-9045632f79ba" />

## 2) Lambda IAM Role:
Create a role  for lambda function and add `AmazonEC2FullAccess` , `AWSLambdaBasicExecutionRole` policy to this role


<img width="1899" height="871" alt="Screenshot 2025-08-13 102400" src="https://github.com/user-attachments/assets/f400b787-57be-4778-ae24-080a53403e70" />

## 3) Lambda Function
Create a Lambda function myFunction_1 and assign the role `lambda-ec2-control-role` created previously to the function

<img width="1897" height="871" alt="Screenshot 2025-08-13 102849" src="https://github.com/user-attachments/assets/c4fb1a08-f060-4c5f-bee8-3b2924440165" />

## 4) Write Boto3 Python script 

```python 
import boto3
from datetime import datetime

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    
    # Extract instance ID from the event
    instance_id = event['detail']['instance-id']
    
    # Define tags
    current_date = datetime.utcnow().strftime('%Y-%m-%d')
    tags = [
        {'Key': 'LaunchDate', 'Value': current_date},
        {'Key': 'Custom-tag', 'Value': 'Aishwarya'}
    ]
    
    # Tag the instance
    ec2.create_tags(
        Resources=[instance_id],
        Tags=tags
    )
    
    print(f"Successfully tagged instance {instance_id} with LaunchDate and Environment tags.")

```
Click on Deploy 

<img width="1899" height="866" alt="Screenshot 2025-08-17 205100" src="https://github.com/user-attachments/assets/9c1ee3a3-68b5-4720-8f5f-289ad9da783f" />


## 5) Create a Rule in Amazon EventBridge 
Choose Event Source: `AWS Events or EventBridge` and add Event pattern :

```JSON
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running"]
  }
}
 
```

Select target type Lambda and add myFunction_1 to target


<img width="1895" height="867" alt="Screenshot 2025-08-17 205145" src="https://github.com/user-attachments/assets/da51c3f8-6322-4748-abab-b6991fe82381" />

<img width="1899" height="871" alt="Screenshot 2025-08-17 205156" src="https://github.com/user-attachments/assets/3e641837-296e-4ec9-9f3a-fb383622dd50" />


## 5) Test

Run the EC2 Instance-2  and check the tags




<img width="1899" height="876" alt="Screenshot 2025-08-17 205351" src="https://github.com/user-attachments/assets/b9104135-2586-40f7-957a-0d9a04353dba" />


As instance state changes to running 2 tags are added:
`LaunchDate`  `2025-08-17`
`Custom-tag`  `Aishwarya`
