---
slug: '2025-06-10-Building-AWS-Lambda-with-Nova-Lite'
title: 'Building a serverless GenAI app with Nova foundation models and AWS Lambda'
date: 2025-06-10 10:00 +1100
categories: 'blog'
---

# Nova foundation models

Amazon recently introduced [Nova Foundation Models](https://www.aboutamazon.com.au/news/aws/introducing-amazon-nova-our-new-generation-of-foundation-models) as a part of Amazon Bedrock (a managed solution to build applications with foundational models). The great thing about these models is that they balance cost-effectiveness vs accuracy/performance really quite well - text, multimodality (models that can work with text, photos or videos including photo and video generation) as well as dedicated image and video generation models.

In this particular example, we will just write a simple text based application and use their base multimodal model - Nova Lite. Its worth noting that should you require a different usecase (ie. an image based app), the API delta wouldn't be much different.

# Model access in Amazon Bedrock

It would be remiss of me to not note here that Amazon Bedrock democratises Gen AI model usage and provides a way to build Gen AI applications across a range of model providers including Anthropic, Deepseek and Mistral and the first step is enable models within the AWS console. Also worth noting in this example is the fact that the model I am going to be enabling (Nova Lite) is available in the `ap-southeast-2` (Sydney) region. Nova models also use `cross region inference` ie. instead of provisioning throughput in any serveless app prior to calling it, you can just call the API directly (we'll unpack this in the lambda configuration section) and it will route the inference call to the region with the closest promixity of the request.

To get started with enabling this model:

1) Go to `AWS console` > `Amazon Bedrock` > Scroll towards the bottom (at least at the time of writing this post in June 2025) > `Bedrock Configurations` > `Model Access`

![Bedrock Configuration](/images/nova-lite/bedrock_configurations.png)

2) Tap on the `Available to request` link in the Nova Lite row.

![Available to request](/images/nova-lite/available_to_request.png)


3) This will open up the `Edit Model Access` window > Tap on next > then submit.

3) In my case it took about 5 minutes for the model to be available in Bedrock.

Curiously, I noticed that when you request access to Anthropic models, a use-case justification needs to be provided 🤔

Before we move onto the next section, its also worth noting that we need an `arn` for our lovely model we've just got access to.

Now we're cooking with gas.

# IAM role for Lambda

We'll go through our lambda configuration in a tick, however we need to give Lambda permissions to list models in Bedrock as well as to invoke them. This can be done with a new policy like so:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:ListFoundationModels"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/amazon.nova-lite-v1:0",
                "arn:aws:bedrock:*:<your account id>:inference-profile/apac.amazon.nova-lite-v1:0"
            ]
        }
    ]
}
```

Remember when we unpacked "cross-region inference" above? Well, that's really why we need two ARNs in our lambda service role. One is to call the foundation model across multiple regions ("the cross region") part, and the other to make the call through to the inference profile. Getting my head around the nuances of why we need both was probably the most challenging part of building this app.

# Lambda configuration

Before we get started here, let's talk about multimodality aspect of this model again. The `Nova lite` model supports Gen AI transforms on text, images and videos. What I found super perplexing is that the official doco on the BedRock page (screenshot below) completely missed the image and video part of the [request body](https://ap-southeast-2.console.aws.amazon.com/bedrock/home?region=ap-southeast-2#/model-catalog/serverless/amazon.nova-lite-v1:0) (see this link, its missing the `photo` and `video` data objects completely as of writing this post in June 2025).

No matter as AWS doco is really quite impeccable, the trick is to keep looking. The [complete schema](https://docs.aws.amazon.com/nova/latest/userguide/complete-request-schema.html) takes care of this issue. The interesting part here is that you can provide a `base-64` encoded `byte array` of the photo or video or you can even upload the video separately to an `s3` bucket and give lambda bucket policy permissions to its service role. There is nothing else extra needed on top.

In our example we'll use a python based lambda and leverage the `boto3` library to do and work with text though.

```python
import json
import boto3

# grab a hold of the boto3 client to call the bedrock service.
bedrock_runtime = boto3.client(
    service_name='bedrock-runtime',
    region_name='ap-southeast-2'
)

# extract the body and customer feedback objects.
# define the system prompt.
def lambda_handler(event, context):
    body = json.loads(event['body'])
    customer_feedback = body.get('text')

    store_name = "Ma petite boulangerie"

    prompt = "The text that follows is feedback from a customer of my store " + store_name + " a boutique french bakery in the heart of Preston in Melbourne, Australia. Please detect the tone and sentiment of this customer and do not hallucinate, remain completely factual and double check your answers. Please extract any constructive feedback concisely "

    body = {
        "system": [
            {
                "text": prompt
            }
        ],
        "messages": [
            {
                "role": "user",
                "content": [
                    {
                        "text": customer_feedback
                    }
                ]
            },
            {
                "role": "assistant",
                "content": [
                    {
                        "text": "string"
                    }
                ]
            }
        ]
    }

    # invoke our lovely model
    response = bedrock_runtime.invoke_model(
        body=json.dumps(body),
        modelId="arn:aws:bedrock:ap-southeast-2:<your account id>:inference-profile/apac.amazon.nova-lite-v1:0",
        accept='application/json',
        contentType='application/json'
    )

    response_body = json.loads(response.get('body').read())
    message_content_text = response_body["output"]["message"]["content"][0]["text"]

    # return the response.
    return {
        'statusCode': 200,
        'body': json.dumps(message_content_text)
    }
```

While the doco does a really good job of unpacking exactly what's required in the `requestBody`:

1) Once we have our `boto3` client calling the `bedrock runtime` service, we define our system prompt.
2) As you can guess, this app detects customer sentiment and extracts any actionable feedback for my (hypothetical) boutique french bakery in Melbourne. The initial `prompt` object is in fact the system prompt.
3) the `text` content is the actual feedback that the lambda receives from its calling API. 
4) Once invoked, the model returns a `response_body` with a `message` content ie. answer.


That's pretty much it! To test this, we can just use lambda console test functionality by defining the following test event:

```json
{
    "body": "{\"text\": \"The croissants were amazing, but the coffee could be better. 4 out of 5 stars though!\"}"
}
```

And here's what we get back from `Nova Lite`

```json
{
  "statusCode": 200,
  "body": "\"Positive with room for improvement\\\"\\n\\nThe sentiment of the customer's feedback is generally positive, as indicated by the 4 out of 5 stars rating. The customer expressed satisfaction with the croissants, describing them as \\\"amazing.\\\" However, there is a suggestion for improvement regarding the coffee, which the customer feels could be better. \\n\\nConstructive feedback:\\n- The croissants are highly appreciated.\\n- There is an opportunity to improve the quality of the coffee.\""
}
```

Classic Melbourne people - very particular about their coffee really. I'd give this app 4/5 stars 👨‍🍳🤌 