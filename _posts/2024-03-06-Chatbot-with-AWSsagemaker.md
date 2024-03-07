---
layout: post
title: Deploy a Chatbot with AWSsagemaker using JumpStart
date: 2024-03-06 13:26 +0800
last_modified_at: 2024-03-06 13:45:25 +0800
tags: [AWS, tutorial]
toc:  true
---

## Overview
Amazon Sagemaker provides a convinient servise to deploy the SOTA open source model in minutes, which is perticulary useful when you want to check if a new published model is a good solution to a problem in your special domain.  

Generally, the deplyment following a special invocation flow.

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/invocation_flow.png){: .align-center}

- **Model:** Deploy the pretrained model to Sagemaker and create an endpoint and wait for serving. This step can be done by few click in JumpStart.
- **Lambda:** Create a Lambda function to invoke the endpoint. It will passes the intput from the user request to the endpoint and return the model output. It will only invoke when a request is recieved. Therefore, it will save resource and cost by the on demand service.
- **API Gateway:** Add a Amazon API Gateway for receiving request and calling the lambda function as needed. 

## Create Domain

To begin with, we need to create a Domain in AWS to run all the service.  

- Log in your AWS Console and search `Amazon SageMaker`

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_sagemaker.png){: .align-center}

- On the left tool bar, look for the `Admin configurations` and hit `Domains`, then click `Create domain` on the page. 

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_domain.png){: .align-center}

- Wite for few minutes, we will see a new domain created on the list.

## Deploy AWSsagemaker Endpoint in JumpStart
We will use the SageMaker Studio to deploy a pretained model with JumpStart.

- Click the new created domain and open the drop down menu on the right side. Open the Studio.

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_studio1.png){: .align-center}

- We will deploy a Llama2 model. Hit `JumpStart`, go to the `meta` list and find the `Llama 2 7B Chat` model and click `Deploy` on the top right.

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_jumpstart.png){: .align-center}

- Then use the default endpoint name and choose `ml.g5.2xlarge` instance to run the model, click `Deploy`. After few minute, the endpoint deployment will be completed. **Coutious: This deployment will cause payment. The instance we selected will cause around 1.7$/hr. Please makesure you clean up all the service following the cleaning up guide to avoide extra cost** [Clean up](https://docs.aws.amazon.com/sagemaker/latest/dg/ex1-cleanup.html)

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_deploy.png){: .align-center}


## Test The Endpoint

- Wait until the endpoint and model are both `In service`.

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_inservice.png){: .align-center}

- Go to `Test inference` and hit `Test the sample request`, you will see a sample request in JSON format. Click `Send Request` for a model prediction. If the endpoint work well, you will get an response also in JSON format.

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_test.png){: .align-center}

## Create a Lambda serverless framework

Now that we have a endpoint ready for service. We need a trigger and a handler to use this service. We will use `Lambda` function as a handler and `API Gateway` as trigger.

- Back to AWS Console, search for `Lambda`, and click `creat function`. Then we give the function a name and running environment.

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_lambda.png){: .align-center}

- Add a `API Gateway` to trigger the lambda function. Click `Add trigger` and search for `API Gateway`

![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_addt.png){: .align-center}
![image](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_api_2.png){: .align-center}

- Config permissions to role. Under the lambada function page, goto `configuration`, `Permissions` and click the `Role name`. Find the `Permissions policies`, click the drop down menu `Add permissions`, and click `Attach policies`. Then searh and add the `AmazonSageMakerFullAccess` to the list.

![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_role_permission.png){: .align-center}
![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_attach_policies.png){: .align-center}
![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_asfullaccess.png){: .align-center}

- Finally, write the lambda function to receive data, process by endpoint and return output.(Noticed that the lambda runtime environment may not have a up-to-date boto3 package, we need to add a `lambda layer` for setting up the dependencies, we will discuss it later on the Debugging section.)

``` python
import json
import boto3

ENDPOINT_NAME = "<your-endpoint-name>" 
client = boto3.client("sagemaker-runtime")

def lambda_handler(event, context):
    print(boto3.__version__)
    response = client.invoke_endpoint(
        EndpointName=ENDPOINT_NAME, 
        InferenceComponentName='<your-model-name>',
        ContentType="application/json",
        Body=json.dumps(event['body']),
        CustomAttributes="accept_eula: true"
    )
    response = response["Body"].read().decode("utf8")
    response = json.loads(response)

    return {
        'statusCode': 200,
        'body': json.dumps(response)
    }

```

Specifically, you can copy your endpoint name and model name from the `SageMaker Studio`.:

![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_epnane.png){: .align-center}

## Test by Postman
- In order to test the service, we need a api endpoint to send a post request, We can get it by here:

![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_api_link.png){: .align-center}
![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_api_link2.png){: .align-center}

- Then We can use `Postman` to send a post request to this api. Here is the body of the request:

```json
{
    "inputs": "<s>[INST] <<SYS>>\nAlways answer with emojis\n<</SYS>>\n\nHow to go from Beijing to NY? [/INST]", 
    "parameters": {"max_new_tokens": 256, "top_p": 0.9, "temperature": 0.6}
}
```

![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_postman.png){: .align-center}

The response only contains emojis, which is exactly what we need.

By now, the Llama2 7B model has perfectly in service. We can design a website to send post request to the api and start chatting! 

## Debug Experiences
It is worth knowing that the default running environment of the lambda function in AWS is not working in this project because of the version of boto3 package. We have to deploy a layer and attach it to the lambda function. The layer package and the lambda function will be uploaded to the github repository. Fell free to download them and play around. [Github_repo](https://github.com/yu-jinh/AWS_chatbot)

Now let's deploy a lambda layer given those files.

- Open the `layers` page in the right tool bar, and click `create layer`.

![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_layer.png){: .align-center}

- Upload the `python.zip` file from the git repository. Set up the configuration as what I am showing below. Click `Create`.

![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_layer2.png)

- Click the `Functions` on the left tool bar, go back to the lambda function page and attach the layer to the function.Scroll down and click `Add layer`.The configuration should be the same as following:

![img](/assets/post_img/2024-03-06-Chatbot-with-AWSsagemaker/aws_layer3.png)

- The `lambda_function.py` file in the repo is the lambda function you need on this experiment. Fell free to copy it and don't forget to replace the endpoint name and model name.

That's it! The lambda function should be run under the compatible version of boto3 library! You are all set! You can also create your own layer package with the binary files you want by following this [Tutorial](https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html)

