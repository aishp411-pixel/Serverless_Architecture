# Analyze Sentiment of User Reviews Using AWS Lambda, Boto3, and Amazon Comprehend
Create an AWS Lambda function using Boto3 to analyze the sentiment of user reviews with Amazon Comprehend. The function should process input text and log whether the sentiment is Positive, Negative, Neutral, or Mixed.

## 1) Lambda IAM Role:
Create a role for lambda function and add `ComprehendFullAccess` , `CloudWatchLogsFullAccess` policy to this role
<img width="1901" height="871" alt="Screenshot 2025-08-14 130128" src="https://github.com/user-attachments/assets/f2ae54ad-bcd4-40e9-a48d-1f1b3ac452a9" />

## 2) Lambda Function
Create a Lambda function myFunction_1 and assign the role `lambda-comprehend-role` created previously to the function

<img width="1901" height="867" alt="Screenshot 2025-08-14 160052" src="https://github.com/user-attachments/assets/a03e1492-3003-4ee5-b8f5-2318c83ff3a6" />


## 3) Write Boto3 Python script 

```python


import json
import boto3
import logging

# Set up logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Initialize Comprehend client
comprehend = boto3.client('comprehend')

def lambda_handler(event, context):
    try:
        # Extract review text from event
        review_text = event.get('review')
       
        if not review_text:
            raise ValueError("No 'review' field found in event.")

        logger.info(f"Review Text: {review_text}")

        # Call Comprehend to detect sentiment
        response = comprehend.detect_sentiment(
            Text=review_text,
            LanguageCode='hi'
            # LanguageCode='en'

        )


        # Extract sentiment result
        sentiment = response['Sentiment']
        sentiment_score = response['SentimentScore']

        # Log the result
        logger.info(f"Detected Sentiment: {sentiment}")
        logger.info(f"Sentiment Scores: {sentiment_score}")

        # Return result as response
        return {
            'statusCode': 200,
            'body': json.dumps({
                'review': review_text,
                'sentiment': sentiment,
                'sentiment_score': sentiment_score
            })
        }

    except Exception as e:
        logger.error(f"Error analyzing sentiment: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }


```
## 4) Test

Create a TestEvent and Add Review 
<img width="1882" height="663" alt="Screenshot 2025-08-14 155808" src="https://github.com/user-attachments/assets/4432177e-0c24-4dfb-9c40-a32aa54ee242" />


Click on Deploy and then on Test

<img width="1901" height="908" alt="Screenshot 2025-08-14 155729" src="https://github.com/user-attachments/assets/8fd35860-a742-40b5-a28b-b1b9e8e397ca" />

Note the Sentiment Score and Sentiment detected 
<img width="1901" height="870" alt="Screenshot 2025-08-14 160005" src="https://github.com/user-attachments/assets/2e4b045f-9d02-4a72-8164-df3f1cebafec" />


Review given was 
 यह फ़ोन बहुत ख़राब है (This phone is so bad)
Sentiment detected is Negative

