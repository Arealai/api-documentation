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
  "image":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUAAASUVORK5CYII...",
  "template_uuid": template_uuid,
  "upload_session_uuid":'8a67ce10-8670-11eb-a0e4-af71473a31ce'
}
```

A sample response to this Document Classification API call will be as follows. 

*results* key will include a list of documents detected within the original document uploaded to the platform. Each document will have a *template_uuid* assigned to it. This template_uuid represents the unique ID of the document type detected. 

```
{
    "results": [
        {
            "document_uuid": "0583b9a6-f36b-4e78-83dd-6490099e3b41",
            "extracted_data": [],
            "processing_time_ms": 2968,
            "template_uuid": "4e43f618-c09c-4ebb-bbce-d4666657e34c"
        },
        {
            "document_uuid": "07226039-d402-4de0-a3b4-4b41359c497c",
            "extracted_data": [],
            "processing_time_ms": 1065,
            "template_uuid": "a134b46f-217d-4cd3-a953-927ad98525a8"
        }
    ],
    "upload_session_uuid": "8a67ce10-8670-11eb-a0e4-af71473a31ce"
}
```
In addition, the response includes a unique *upload_session_uuid*. This upload_session_uuid will be the same UUID as the initial upload_session_uuid provided in the OCR API. If you provide a upload_session_uuid, these documents will be added to that Session Folder. If not, the Areal.ai platform will create a new session and add these documents to that session.  


## Data Extraction API

**Data Extraction API: https://areal.ai/api/v1/ocr/**

Data extraction API is the same API as the Document Classification API. There is no need to make separate API calls to both classify and extract data from a document. Data extraction will automatically take place once a document is classified (when a template is assigned to a document) IF data extraction is provisioned for that specific template in your organization. 

In the case of data extraction, the API response will return a list of **extracted_data**. Each extracted_data will contain a **component** object which defines the details of the data extracted. Extracted Data field will also contain the confidence_rate, processed_value (extracted data) and page fields. While *component* objects are assigned to *template* objects and are standard across all documents with the same template, *extracted_data* object will change from document to document since different data points will be extracted from document.

```
{
    "results": [
        {
            "document_uuid": "d0d2e07a-1da4-4aec-97ff-5edeb2018ded",
            "extracted_data": [
                {
                    "component": {
                        "category": "text",
                        "name": "Wages",
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
                        "name": "Name",
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
                        "name": "State",
                        "uuid": "ffcfe2b0-e300-11e9-93dd-1f1efd155a62"
                    },
                    "confidence_rate": 0.97,
                    "page": 0,
                    "processed_value": "PA",
                    "uuid": "c8e69635-6b2e-4b0a-adca-d18707a2a96c"
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

## Get Documents

**Data Extraction API: https://areal.ai/api/v1/ocr/**

This is a GET API call. The following parameters must be provided:


## Sessions, Search and Get Sessions



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
