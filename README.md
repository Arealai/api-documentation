# Areal.ai API Documentation

Welcome to Areal.ai.  Areal.ai provides automated real-time image classification and data extraction services. Areal.ai’s REST API takes an image or PDF file and returns a document type ID and extracted data in a structured format.

Areal.ai can be used in various use-cases including operations that require real time document processing such as underwriting, customer onboarding, document ingestion and more. Areal.ai document classification enables customers to give instant feedback to their users about the validity of their documents and to accelerate document ingestion. 

In this document, we will review how to get started using Areal.ai APIs

## Getting Started

Document processing is accomplished with a single API call. Sample Python code is as follows:

'''
import base64
import json
import requests

headers = {
    'Authorization': 'ApiKey Customer_account_id:Customer_key',  # TODO: Update here
    'Content-Type': 'application/json',
}
file_name = 'drivers_license.png'  # TODO: Update here
file_type = ‘pdf’
image_type = 'data:application/pdf;base64,' if file_type.lower() == "pdf" else 'data:image/png;base64,'
image_base64 = base64.b64encode(open(file_name, 'rb').read()).decode('ascii')
data = {
    'source': "web",
    'name': file_name,
    'image': image_type + str(image_base64),
    "template_uuid": template_uuid # TODO: Optional
}

response = requests.post('https://areal.ai/api/v1/ocr/', headers=headers, json=data)
'''

Areal.ai’s API supports concurrent requests/calls. API connections must be over HTTPS secured channel and must be authenticated in the headers as shown in the sample code above.


## Authentication



## Document Classification



## Data Extraction



## Pulling Documents



## Sessions, What are they and how to pull them?



## Giving Feedback to Platform








