# Assignment 1: Automated Instance Management Using AWS Lambda and Boto3
Create an AWS Lambda function using Python and Boto3 to automate the management of EC2 instances based on custom tags. The function should start or stop instances by detecting tags like Auto-Start and Auto-Stop, enabling hands-free resource control.
## 1) EC2 setup:
Create two EC2 instance Instance-1 and Instance-2 and add tag `Action = Auto-stop` , `Action = Auto-start` respectively 

<img width="1918" height="1079" alt="Screenshot 2025-08-14 104416" src="https://github.com/user-attachments/assets/a72ae2a6-f06e-4125-bc42-8807b77dd5c2" />

<img width="1918" height="1043" alt="Screenshot 2025-08-14 104532" src="https://github.com/user-attachments/assets/838037bd-3039-4bdf-b5bd-e14c2e7e1c99" />

## 2) Lambda IAM Role:
Create a role  for lambda function and add `AmazonEC2FullAccess` , `AWSLambdaBasicExecutionRole` policy to this role


<img width="1899" height="871" alt="Screenshot 2025-08-13 102400" src="https://github.com/user-attachments/assets/f400b787-57be-4778-ae24-080a53403e70" />

## 3) Lambda Function
Create a Lambda function myFunction_1 and assign the role `lambda-ec2-control-role` created previously to the function

<img width="1897" height="871" alt="Screenshot 2025-08-13 102849" src="https://github.com/user-attachments/assets/c4fb1a08-f060-4c5f-bee8-3b2924440165" />

## 4) Write Boto3 Python script 

```python 
import json
import boto3


def lambda_handler(event, context):
    # 1. Initialize boto3 EC2 client
    ec2_client = boto3.client('ec2' ,  region_name='ca-central-1')


    # 2. Describe instances with Auto-Stop tag
    auto_stop_instances = ec2_client.describe_instances(
        Filters=[
            {'Name': 'tag:Action', 'Values': ['Auto-Stop']},
            {'Name': 'instance-state-name', 'Values': ['running']}  # Only running instances
        ]
    )


    # extract instance IDs
    stop_instance_ids = [
        instance['InstanceId']
        for reservation in auto_stop_instances['Reservations']
        for instance in reservation['Instances']
    ]

    # Describe instances with Auto-Start tag
    auto_start_instances = ec2_client.describe_instances(
        Filters=[
            {'Name': 'tag:Action', 'Values': ['Auto-Start']},
            {'Name': 'instance-state-name', 'Values': ['stopped']}  # Only stopped instances
        ]
    )
    start_instance_ids = [
        instance['InstanceId']
        for reservation in auto_start_instances['Reservations']
        for instance in reservation['Instances']
    ]

    # 3. Stop Auto-Stop instances
    if stop_instance_ids:
        ec2_client.stop_instances(InstanceIds=stop_instance_ids)
        print(f"Stopped instances: {stop_instance_ids}")
    else:
        print("No running instances found with tag Auto-Stop.")


    # 4. Start Auto-Start instances
    if start_instance_ids:
        ec2_client.start_instances(InstanceIds=start_instance_ids)
        print(f"Started instances: {start_instance_ids}")
    else:
        print("No stopped instances found with tag Auto-Start.")
    return {
        "stopped": stop_instance_ids,
        "started": start_instance_ids
    }
```
## 5) Test
Click on Deploy and then on Test

**note:** *Increase Lambda timeout for about 30 secs*

<img width="1919" height="913" alt="Screenshot 2025-08-14 105311" src="https://github.com/user-attachments/assets/cc330a21-5da0-4491-a4bd-5e865fc8c652" /><br/>


Check the instances affected by Lambda 

<img width="1402" height="765" alt="Screenshot 2025-08-14 105522" src="https://github.com/user-attachments/assets/f1de518e-d954-4dec-802f-e66737c2428c" /><br/>


Initial Status of EC2 before triggering Lambda 

<img width="1586" height="309" alt="Screenshot 2025-08-14 105415" src="https://github.com/user-attachments/assets/9b406e0e-82c6-4029-b3d0-ad047413e309" /><br/>


After Triggering the Lambda

<img width="1583" height="314" alt="Screenshot 2025-08-14 105436" src="https://github.com/user-attachments/assets/67f5c11f-9e97-4370-ae42-7a4a61348c16" /><br/>


Logs

<img width="1577" height="650" alt="Screenshot 2025-08-14 105636" src="https://github.com/user-attachments/assets/12a91cba-c7c7-488e-845f-fc96857f1b6e" />

Instance-1 with tag `Auto-stop` was stopped and Instance-2 with tag `Auto-start` was started successfully 
