# Gen AI Blog Generation using AWS

This project demonstrates how to create a Generative AI-based blog generation system using various AWS services. The system involves generating blog content using AWS Bedrock, processing it with AWS Lambda, and storing the generated content in an S3 bucket.

![AWS Lamda (1)](https://github.com/puneetgani/GenAIusingAWS/assets/106878112/92f65733-dd35-4849-ada2-3262f5993a98)

## Tech Stack

1.**AWS S3:** Storage service for output data.

2.**AWS Lambda:** Serverless compute service for processing data.

3.**AWS Bedrock:** Service for AI model integration.

4.**API Gateway:** Service for creating and managing APIs.

## Architecture

1. **Blog Topic Input:** A blog topic is sent to the API Gateway.
2. **Lambda Processing:** The Lambda function processes the blog topic and sends it to AWS Bedrock.
3. **AI Model Integration:** AWS Bedrock generates the blog content based on the provided topic.
4. **Output Data Storage:** The generated blog content is stored in an S3 bucket.

## Setup

### AWS S3

1. Create an S3 bucket to store generated blog content.

### AWS Lambda

1. Create a Lambda function to process blog topics and generate content.
2. Add the necessary permissions for the Lambda function to interact with S3 and AWS Bedrock.
3. Write the Lambda code to process the blog topic, generate the blog content, and store it in S3.

### AWS Bedrock

1. Set up AWS Bedrock and integrate the desired AI model.
2. Ensure the model can receive input from the Lambda function and return generated content.

### API Gateway

1. Create a new API.
2. Define a POST method.
3. Integrate the Lambda function with the POST method.
4. Deploy the API and get the endpoint URL.

## Lambda Function Example

```python
import boto3
import botocore.config
import json
from datetime import datetime


def blog_generate_using_bedrock(blogtopic: str) -> str:
    prompt = f"""<s>[INST]Human: write a 200 words blog on the topic {blogtopic}
    Assistant:[INST]
    """

    body = {
        "prompt": prompt,
        "max_gen_len": 512,
        "temperature": 0.5,
        "top_p": 0.9
    }

    try:
        bedrock = boto3.client("bedrock-runtime", region_name="us-east-1",
                               config=botocore.config.Config(read_timeout=300, retries={'max_attempts': 3}))

        response = bedrock.invoke_model(body=json.dumps(body), modelId='meta.llama3-70b-instruct-v1:0')

        response_content = response.get('body').read()
        response_data = json.loads(response_content)
        print(response_data)
        blog_details = response_data['generation']
        return blog_details

    except Exception as e:
        print(f"Error generating the blog: {e}")
        return ""


def save_blog_details_s3(s3_key, s3_bucket, generate_blog):
    s3 = boto3.client('s3')

    try:
        s3.put_object(Bucket=s3_bucket, Key=s3_key, Body=generate_blog)
        print("Blog saved to s3")

    except Exception as e:
        print(f"Error when saving the blog to s3: {e}")


def lambda_handler(event, context):
    event = json.loads(event['body'])
    blogtopic = event['blog_topic']

    generate_blog = blog_generate_using_bedrock(blogtopic=blogtopic)

    if generate_blog:
        current_time = datetime.now().strftime('%H%M%S')
        s3_key = f"blog-output/{current_time}.txt"
        s3_bucket = 'awsbedrockcourse12344'
        save_blog_details_s3(s3_key, s3_bucket, generate_blog)
    else:
        print("No blog was generated")

    return {
        'statusCode': 200,
        'body': json.dumps(f'Blog generation completed: {generate_blog}')
    }
```

### API Gateway Setup

1. Create a new API.
2. Define a POST method.
3. Integrate the Lambda function with the POST method.
4. Deploy the API and get the endpoint URL.

## 🔗 Links

[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/puneetgani)
