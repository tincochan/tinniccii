---
layout: post
title: "Deploy StableLM models on AWS Sagemaker Endpoints"
description: "As of 30 April 2023, the process of deploying the model on Sagemaker Endpoints is not as straightforward as some of the other models on HuggingFace, due to the need to package custom inference code with the model. "
date: 2023-07-07
tags: [算法]
---

I'll be deploying the StableLM-Tuned-Alpha 7b variant on the model on an ml.g5.4xlarge instance. StableLM-Tuned-Alpha is a suite of 3B and 7B parameter decoder-only language models built on top of the StableLM-Base-Alpha models and further fine-tuned on various chat and instruction-following datasets.

<!--more-->

### Steps:

1. Download the model from Huggingface.
2. Create a custom inference script.
3. Package the model and inference script by creating the model.tar.gz archive.
4. Upload the model to S3.
5. Create a Sagemaker Endpoint.

You can follow the steps on your local machine, or on an AWS Sagemaker Studio notebook / terminal. You'll need to make sure that your local machine or the Sagemaker instance has enough disk space to download the model & create the archive file. This blog post will also not go into details about how to set up your AWS account permissions, so please make sure to follow the blog post provided in the references on how to do this for a similar model.

### 1. Download the model from Huggingface.

Make sure you have *git* and *git-lfs* installed on your system. Simply run the following commands in your terminal to download the model:

```bash
Copycopy code to clipboard
1# Make sure you have git-lfs installed (https://git-lfs.com)
2git lfs install
3git clone https://huggingface.co/stabilityai/stablelm-tuned-alpha-7b
```

The model will then be available inside the directory *stablelm-tuned-alpha-7b*.

### 2. Create a custom inference script.

Change the directory to the model directory and create a directory called *code*.

```bash
Copycopy code to clipboard
1cd stablelm-tuned-alpha-7b
2mkdir code
```

Create a file called *inference*.py inside the *code* directory and copy the following code into it:

```python
Copycopy code to clipboard
1from transformers import AutoModelForCausalLM, AutoTokenizer, StoppingCriteria, StoppingCriteriaList
2import torch
3

4SYSTEM_PROMPT = """<|SYSTEM|># StableLM Tuned (Alpha version)
5- StableLM is a helpful and harmless open-source AI language model developed by StabilityAI.
6- StableLM is excited to be able to help the user, but will refuse to do anything that could be considered harmful to the user.
7- StableLM is more than just an information source, StableLM is also able to write poetry, short stories, and make jokes.
8- StableLM will refuse to participate in anything that could harm a human.
9"""
10

11class StopOnTokens(StoppingCriteria):
12    def __call__(self, input_ids: torch.LongTensor, scores: torch.FloatTensor, **kwargs) -> bool:
13        stop_ids = [50278, 50279, 50277, 1, 0]
14        for stop_id in stop_ids:
15            if input_ids[0][-1] == stop_id:
16                return True
17        return False
18

19def model_fn(model_dir):
20    # Load model from S3
21    tokenizer = AutoTokenizer.from_pretrained(model_dir)
22    model = AutoModelForCausalLM.from_pretrained(model_dir)
23    model.half().cuda()
24    return model, tokenizer
25    
26def predict_fn(data, model_and_tokenizer):
27    
28    model, tokenizer = model_and_tokenizer
29    
30    input = data.pop("input", None)
31    
32    prompt = f"{SYSTEM_PROMPT}<|USER|>{input}<|ASSISTANT|>"
33    inputs = tokenizer(prompt, return_tensors="pt").to("cuda")
34    input_ids = inputs['input_ids']
35    tokens = model.generate(
36      **inputs,
37      max_new_tokens=128,
38      temperature=0.5,
39      do_sample=True,
40      stopping_criteria=StoppingCriteriaList([StopOnTokens()])
41    )
42    # the code has been changed to only return the generated text, and not the original text
43    # simply remove the slicing below to return the input text in addition to
44    # the generated text
45    output = tokenizer.decode(tokens[:, input_ids.shape[1]:][0], skip_special_tokens=True)
46    return output
```

This script now includes the custom code needed for reading the model correctly and making predictions. I've included some slicing on the tokens to only return the generated text, and not the original text. You can remove this if you want to return the original text as well.

You can change the SYSTEM_PROMPT to whatever you like, but keep in mind that including the system prompt inside the inference code will bundle it with your model. If you want to try out the model with a different system prompt, you can pass it as an input when invoking the endpoint instead of having it hardcoded inside the inference code.

### 3. Package the model and inference script by creating the model.tar.gz archive.

Now that we have the model and the inference code, we need to package it into a single archive file. We'll be using the *tar* command to do this. Make sure you have *tar* installed on your system.

Assuming you are inside the *stablelm-tuned-alpha-7b* directory, run the following command to create the archive file:

```bash
Copycopy code to clipboard
1tar zcvf model.tar.gz *
```

The command includes all the files within the above directory in the *model.tar.gz*. It takes around 30mins to run on my M1 Mac.

### 4. Upload the model to S3.

After creating a machine learning model, the next step is to make it accessible for deployment. One way to do this is to upload the model to an S3 bucket. This process involves compressing the model into a .tar.gz file and then uploading it to an S3 bucket.

To accomplish this, you can refer to the [official AWS guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html) on how to upload objects to S3. This guide provides step-by-step instructions on how to create an S3 bucket, configure the appropriate permissions, and upload files to the bucket. While we won't go into the specifics of this process, following the AWS guide will ensure that your model is properly uploaded and ready for deployment.

### 5. Create a Sagemaker Endpoint.

Now that we have the model uploaded to S3, we can create a Sagemaker Endpoint using the Sagemaker python SDK. It includes a class called *HuggingFaceModel* that we can use to create the endpoint.

Make sure to install the Sagemaker SDK first in your Python environment:

```bash
Copycopy code to clipboard
1pip install sagemaker
```

Make sure that you have your S3 object URL ready. It starts with `s3://`. Please make sure that you have your AWS credentials set up in your environment as well. You can follow the [official AWS guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) on how to do this.

Now, you can run the following Python code to create the endpoint.

```python
Copycopy code to clipboard
1import sagemaker
2from sagemaker.huggingface.model import HuggingFaceModel
3

4MODEL_S3_LOCATION = "" # fill in with your S3 object URL for the model
5

6huggingface_model = HuggingFaceModel(
7    model_data=MODEL_S3_LOCATION, 
8    role= sagemaker.get_execution_role(), # IAM role with permissions to create an Endpoint
9    transformers_version="4.26",
10    pytorch_version="1.13",
11    py_version="py39"
12)
13

14predictor = huggingface_model.deploy(initial_instance_count=1, instance_type="ml.g5.4xlarge")
```

The code above creates a HuggingFaceModel object and deploys it to a Sagemaker Endpoint. It uses the *ml.g5.4xlarge* instance type, but you can experiment with other instance types if you are using a smaller model (like the 3b variant).

You can invoke the endpoint using the following code:

```python
Copycopy code to clipboard
1predictor.predict({ "input": "Write me a poem about AWS."})
```

## References

1. [Deploy FLAN-UL2 20B on Amazon SageMaker](https://www.philschmid.de/deploy-flan-ul2-sagemaker)
2. [StabileLM Tuned Alpha 7b on HuggingFace](https://huggingface.co/stabilityai/stablelm-tuned-alpha-7b)
3. [Uploading objects to an S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html)



