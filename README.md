# Areal.ai API Documentation

Welcome to Areal.ai.  Areal.ai provides automated real-time image classification and data extraction services. Areal.ai’s REST API takes an image or PDF file and returns a document type ID and extracted data in a structured format.

Areal.ai can be used in various use-cases including operations that require real time document processing such as underwriting, customer onboarding, document ingestion and more. Areal.ai document classification enables customers to give instant feedback to their users about the validity of their documents and to accelerate document ingestion. 

In this document, we will review how to get started using Areal.ai APIs

## Getting Started

Document processing is accomplished with a single API call. To make a successful API call to process a document, provide the following information:

- Authorization headers
- Name of the file
- Content type
- PDF or image in the base64 format

Sample Python code is as follows:

```
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
```

Areal.ai’s API supports concurrent requests/calls. To process multiple documents at a time, you can make multiple separate API call per each document.

*API connections must be over HTTPS secured channel.* 


## Authentication

API calls must be authenticated in the headers. To be able to authorize, construct the following phase and add it to the headers with Authorization key.

```
'Authorization': 'ApiKey Customer_account_id:Customer_key',  # TODO: Update here
```

Customer Account ID is the username procvided and the Customer Key is the API Key provided. If you do not have a key yet, please contact team@areal.ai.


## Document Classification API

**Document Classification API: https://areal.ai/api/v1/ocr/**

This is a POST API call. The following parameters must be provided:

Headers:

```
headers = {
  'Authorization': 'ApiKey Customer_account_id:Customer_key',
  'Content-Type' : 'application/json'
}
```
Body:
```
post_data = {
   'source': "web",
   'name': file_name,
   'image': image_type + str(image_base64),
   'template_uuid': template_uuid
}
```

Other optional fields are **template_uuid** and **upload_session_uuid**. Template UUID basically asks platform to skip Document Classification and use provided template to extract data from the document. This could be useful for speed optimization if you alredy know the type of the document you are uploading.  

Upload Sessions are used to group documents under a specific folder. Sessions are extremely useful to group all files of a mortgage application or a title document package. We will cover this more in the Sessions section below. 

```
post_data = {
   'source': "web",
   'name': file_name,
   'image': image_type + str(image_base64),
   'template_uuid': template_uuid,
   'upload_session_uuid':session_uuid
}
```

Image type could be either 'data:application/pdf;base64,' for PDF files or 'data:image/png;base64,' which covers all image types.

A sample body data is as follows:
```
{ "source": "api",
  "name":"postman.png",
  "image":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUAAASUVORK5CYII=...",
  "template_uuid": template_uuid,
  "upload_session_uuid":'6d1cd890-85f9-11eb-9ba8-614f814eb3d6'
}
```

The full sample response to this API call can be seen here. 

For the matter of 





## Data Extraction API



## Pulling Documents



## Sessions, What are they and how to pull them?



## Giving Feedback to Platform



## Sample Response

```
{
    "results": [
        {
            "document_uuid": "d0d2e07a-1da4-4aec-97ff-5edeb2018ded",
            "extracted_data": [
                {
                    "component": {
                        "category": "text",
                        "name": "1 Wages",
                        "uuid": "9b1153e0-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "50,000.00",
                    "uuid": "8ebfaca6-ddca-476a-ba66-ab51eb2fc667"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "12a Box",
                        "uuid": "cd08decf-e179-47da-b6bb-0bc45c8efa85"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "",
                    "uuid": "c3ae929e-9d2b-42ea-93ac-3203775eec02"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "15 State",
                        "uuid": "ffcfe2b0-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "PA",
                    "uuid": "c8e69635-6b2e-4b0a-adca-d18707a2a96c"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "16 State Wages",
                        "uuid": "0c0075e0-e301-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "50,000.00",
                    "uuid": "04ecde10-41c9-4c73-b326-247bcf68f9ed"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "17 State Tax",
                        "uuid": "16f024a0-e301-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "1,535.00",
                    "uuid": "d6c3c82a-1b99-40d0-a47b-0a03ae2e2de2"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "2 Federal",
                        "uuid": "a4e77c50-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "6,835.00",
                    "uuid": "7811ca28-1ced-4555-8403-293ef851f96f"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "3 Ss Wages",
                        "uuid": "abcc2a20-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "50,000.00",
                    "uuid": "c765bf79-840c-4883-bc3a-dd399ef8d495"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "4 Ss Tax",
                        "uuid": "b0ca8bc0-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "3,100.00",
                    "uuid": "c531b860-7f9b-4c59-b260-5e24b423cd3f"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "5 Medicare Wages",
                        "uuid": "ca43fff0-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "50,000.00",
                    "uuid": "8feb5dd0-d7f9-4c92-a9c7-e2468affa03e"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "6 Medicare Tax",
                        "uuid": "d0fcbbc0-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "725.00",
                    "uuid": "c63ca21b-6354-42f4-a4ca-1a895ce91da0"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "Employee Contact",
                        "uuid": "861132d0-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "Jane A. Doe 123 Elm Street Anywhere Else, PA 17111",
                    "uuid": "b4c198b1-d8a4-4ffc-9e02-58b3f3f7abe5"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "Employee Ssn",
                        "uuid": "76d9a130-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "123-45-6789",
                    "uuid": "b49f3356-972c-4404-8248-18110a4bf142"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "Employer Contact",
                        "uuid": "94b81d30-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "The Big Company 12 Main Street Anywhere, NC 28111",
                    "uuid": "8c8be14b-8e26-425f-898e-1a1ac848ef44"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "Employer Ein",
                        "uuid": "7c16b980-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "55-5765489",
                    "uuid": "54a0d310-8fed-4084-8a7a-815036748868"
                },
                {
                    "component": {
                        "category": "text",
                        "name": "Omb",
                        "uuid": "6eab0e40-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "1545-0008",
                    "uuid": "6fb43074-d56c-4186-9ae0-da2f9f1bf73f"
                },
                {
                    "component": {
                        "category": "integer",
                        "name": "Year",
                        "uuid": "bb21fca0-f74d-11e9-8b38-e5e69c6c59af"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "2017",
                    "uuid": "e53a693b-2c41-4da5-8776-e86af6620d14"
                }
            ],
            "processing_time_ms": 2721,
            "template_uuid": "4e0550c2-ecbf-40d4-bdf0-2f9de24b525f"
        }
    ],
    "upload_session_uuid": "6d1cd890-85f9-11eb-9ba8-614f814eb3d6"
}
```
