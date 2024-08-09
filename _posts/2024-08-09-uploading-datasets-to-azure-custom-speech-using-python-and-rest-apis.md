---
layout: post
title: "Uploading Datasets to Azure Custom Speech using Python and REST APIs"
date: 2024-08-09 12:00:00 +0000
categories: Azure
tags: azure custom-speech ai python rest
---

[Azure Custom Speech](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/custom-speech-overview) is a 
feature of [Azure’s Speech service](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/overview) 
that allows you to enhance the accuracy of speech recognition for your specific applications and products. It allows 
models to be trained on your own data, which can be particularly useful if you have a specific domain or accent that
is not well-represented in the general models provided by Azure. It can be used for real-time transcription, batch 
transcription, and translations.

## Training Datasets

To train a custom speech model, you need to provide 
[training datasets](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/how-to-custom-speech-test-and-train). 
This data is used to train the model to recognize speech in your specific domain. The data should be representative of 
the speech that the model will encounter in the real world. This can include audio files, transcriptions, and other 
metadata that describes the data.

## Uploading Datasets

To upload a dataset to Azure Custom Speech, you have three options. You can use the Speech Studio UI, the REST API, or
the SDK. In this post, we will focus on using the REST API to upload datasets to Azure Custom Speech but, you can take a 
look [here](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/how-to-custom-speech-upload-data?pivots=speech-cli) 
for more information on the other options.

### Why use the REST API?

Using the REST API to upload datasets to Azure Custom Speech has several advantages over using the UI:
- **Automation**: You can automate the process of uploading datasets using scripts or code.
- **Integration**: You can integrate the dataset upload process into your existing workflows and systems.
- **Control**: You have more control over the upload process and can customize it to suit your needs.
- **Repeatability**: You can easily repeat the dataset upload process with different datasets or configurations.

There are two ways to upload datasets using the REST API, you can either upload the dataset to blob storage and then
import it into Azure Custom Speech, or you can upload the dataset directly to Azure Custom Speech.

The full documentation for the REST API can be found 
[here](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/rest-speech-to-text). 

### Uploading Datasets from Blob Storage

This is the simpler of the two methods and involves uploading the dataset to a blob storage account and then importing
it into Azure Custom Speech. After the dataset is imported, you can use the Python code below to create **and** upload 
the dataset to Azure Custom Speech.


<script src="https://gist.github.com/adamdougal/6b9a554c950a79786e54d9a99dc98744.js"></script>


Note the `contentUrl` field in the data dictionary.

### Uploading Datasets from your local machine using REST APIs

Uploading datasets from your local machine is more complex than uploading from blob storage, but it can be useful if you
don't have a blob storage account or if you want to avoid the extra step of importing the dataset into Azure Custom 
Speech.

The complexity arises from need to upload the dataset in blocks, rather than as a single file. This is because Azure 
Custom Speech has a limit on the size of the dataset that can be uploaded in a single request for reliability and 
scalability. The maximum size of a dataset block is 4 MiB.

To upload a dataset to Azure Custom Speech using the REST API, you will need to make a series of API calls:
1. Create a dataset
2. Upload dataset blocks
3. Commit dataset blocks

See the Python code below for an example of how to upload a dataset to Azure Custom Speech using the REST API from your 
local machine.

<script src="https://gist.github.com/adamdougal/d9705bf3a78366e5835704ce3bacb54b.js"></script>



Note that the `upload_dataset_blocks` function reads the dataset file in blocks of 4MB and uploads each block to Azure
Custom Speech. The `commit_dataset_blocks` function commits the uploaded blocks to Azure Custom Speech, triggering the
processing of the dataset. 

The code above is a simple example of how to upload a dataset to Azure Custom Speech using the REST API. When using this 
code in a production environment, you should add error handling, logging, and other features to make it more robust and 
reliable.