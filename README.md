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

## Organization Specific URLs

If your organization is provided with a specific Areal.ai url such as "acme.areal.ai", then you need to make all API calls to that specific URL such as:

```
response = requests.post('https://acme.areal.ai/api/v1/ocr/', headers=headers, json=data)
```

## Document Classification API

**Document Classification API: https://areal.ai/api/v1/ocr/**

Request Method: **POST**. 

The following parameters must be provided:

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

Other optional fields are **template_uuid**, **upload_session_uuid** and **upload_session_name**. Template UUID basically asks platform to skip Document Classification and use provided template to extract data from the document. This could be useful for speed optimization if you alredy know the type of the document you are uploading.  

Upload Sessions are used to group documents under a specific folder. Sessions are extremely useful to group all files of a mortgage application or a title document package. We will cover this more in the Sessions section below. 

```
post_data = {
   'source': "web",
   'name': file_name,
   'image': image_type + str(image_base64),
   'template_uuid': template_uuid,
   'upload_session_uuid':session_uuid
   'upload_session_name':session_name
}
```

Image type could be either 'data:application/pdf;base64,' for PDF files, 'data:image/png;base64,' or 'data:image/jpeg;base64,' for different image types.

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

Request Method: **POST**.

Data extraction API is the same API as the Document Classification API. There is no need to make separate API calls to both classify and extract data from a document. Data extraction will automatically take place once a document is classified (when a template is assigned to a document) IF data extraction is provisioned for that specific template in your organization. 

In the case of data extraction, the API response will return a list of **component_results**. 

Each Component Result contains category, name, UUID and extracted_data keys. etracted_data key field contains the details of the extracted data for that component. While *component* objects are assigned to *template* objects and are standard across all documents with the same template, *extracted_data* object will change from document to document since different data points will be extracted from document.

The response currently contains also a list of **extracted_data**. **The extracted_data list will be deprecated by March 31, 2023**. Each extracted_data contains a **component** object which defines the details of the data extracted. Extracted Data field contains the confidence_rate, processed_value (extracted data) and page fields.

```
[
    {
        "document_name": "W2_1_Borrower-W2.pdf",
        "document_uuid": "1df9bdbd-dddd-bbbb-bfe6-a85870ad998f",
        "component_results": [
            {
                "category": "text",
                "extracted_data": [
                    {
                        "approved_value": "1545-0008",
                        "confidence_rate": 0.9711745381355286,
                        "grouped_data": [],
                        "height_percent": 0.005555555555555556,
                        "page": 0,
                        "processed_value": "1545-0008",
                        "uuid": "c569e403-b7c2-4056-81ea-9fed428064a9",
                        "width_percent": 0.055992141453831044,
                        "x_percent": 0.5176817288801572,
                        "y_percent": 0.32083333333333336
                    }
                ],
                "name": "00 - OMD",
                "uuid": "6eab0e40-aaaa-bbbb-cccc-1f1efd155a62"
            },
            {
                "category": "text",
                "extracted_data": [
                    {
                        "approved_value": "2017",
                        "confidence_rate": 0.9711745381355286,
                        "grouped_data": [],
                        "height_percent": 0.022222222222222223,
                        "page": 0,
                        "processed_value": "2017",
                        "uuid": "3e182053-ca70-4409-81af-f6b02ae2753a",
                        "width_percent": 0.10805500982318271,
                        "x_percent": 0.4351669941060904,
                        "y_percent": 0.6673611111111111
                    }
                ],
                "name": "00 - Year",
                "uuid": "bb21fca0-aaaa-bbbb-cccc-e5e69c6c59af"
            },
            {
                "category": "text",
                "extracted_data": [
                    {
                        "approved_value": "123-45-6789",
                        "confidence_rate": 0.9711745381355286,
                        "grouped_data": [],
                        "height_percent": 0.006944444444444444,
                        "page": 0,
                        "processed_value": "123-45-6789",
                        "uuid": "e48b265a-4cde-4408-83d6-f7b227d8d4ed",
                        "width_percent": 0.08644400785854617,
                        "x_percent": 0.30353634577603145,
                        "y_percent": 0.3229166666666667
                    }
                ],
                "name": "00A - Employee SSN",
                "uuid": "76d9a130-aaaa-bbbb-cccc-1f1efd155a62"
            },
            {
                "category": "text",
                "extracted_data": [
                    {
                        "approved_value": "Jane A. Doe",
                        "confidence_rate": 0.9711745381355286,
                        "grouped_data": [],
                        "height_percent": 0.008333333333333333,
                        "page": 0,
                        "processed_value": "Jane A. Doe",
                        "uuid": "82533e77-e466-4d80-b779-e950fae19b9e",
                        "width_percent": 0.0825147347740668,
                        "x_percent": 0.21905697445972494,
                        "y_percent": 0.5076388888888889
                    }
                ],
                "name": "00B - Employee Name",
                "uuid": "413007d5-aaaa-bbbb-cccc-bc0e9ae59d65"
            }
        ],
        "extracted_data": [
            {
                "component": {
                    "category": "text",
                    "name": "00 - OMD",
                    "uuid": "6eab0e40-aaaa-bbbb-cccc-1f1efd155a62"
                },
                "confidence_rate": 0.97,
                "page": 0,
                "processed_value": "1545-0008",
                "uuid": "c569e403-b7c2-4056-81ea-9fed428064a9"
            },
            {
                "component": {
                    "category": "text",
                    "name": "00 - Year",
                    "uuid": "bb21fca0-aaaa-bbbb-cccc-e5e69c6c59af"
                },
                "confidence_rate": 0.97,
                "page": 0,
                "processed_value": "2017",
                "uuid": "3e182053-ca70-4409-81af-f6b02ae2753a"
            },
            {
                "component": {
                    "category": "text",
                    "name": "00A - Employee SSN",
                    "uuid": "76d9a130-aaaa-bbbb-cccc-1f1efd155a62"
                },
                "confidence_rate": 0.97,
                "page": 0,
                "processed_value": "123-45-6789",
                "uuid": "e48b265a-4cde-4408-83d6-f7b227d8d4ed"
            },
            {
                "component": {
                    "category": "text",
                    "name": "00B - Employee Name",
                    "uuid": "413007d5-aaaa-bbbb-cccc-bc0e9ae59d65"
                },
                "confidence_rate": 0.97,
                "page": 0,
                "processed_value": "Jane A. Doe",
                "uuid": "82533e77-e466-4d80-b779-e950fae19b9e"
            }
        ]
    }
]
```

## Annotation API (Beta)

Areal.ai can also detect areas to be annotationed in a document. Areal.ai currently finds areas to be signed, initialized or sealed in a document. The API call to get the annotations is exactly same as the API call to classify/extract a document. 

Annotation API currently supports the following entities:

- Borrower(s) (Multiple Entities)
- Seller(s) (Multiple Entities)
- Lender (Single Entity)
- Loan Officer (Single Entity)
- Notary (Single Entity)
- Closing Agent (Single Entity)
- Witness (Single Entity)
- Title Holder (Single Entity)
- Acknowledgment Party (Single Entity)
- All Other Signers (Multiple Entities)

In order to get access to our newest functionality to find annotations in a document, please contract your Areal.ai administrator. 


## Loan Info Meta (Beta)

Developers can provide additional meta data while making an API call to Areal. For example, if you would like to provide the Borrower's Name information, this information could be added to the Body section of the API call. In this case, this information should be mapped to the "borrower" key within the "loan_info" dictionary of the API Body. For example:

```
{
  "name":"Annotation_doc.pdf",
  "loan_info":{"borrower":"Michael Curtis"},
  "image":"data:application/pdf;base64,JVBERi0x...
}
```

While the "loan_info" section within the Body of an API call provides tons of flexibility, Areal currently supports the following keys in a "loan_info" dictionary:

- borrower
- coborrower_1
- coborrower_2
- coborrower_3
- seller
- coseller_1
- coseller_2
- coseller_3
- lender
- loan_officer
- notary
- closing_agent
- witness_1
- title_holder
- acknowledgment_party
- signer_1
- signer_2
- signer_N

Please note that "cosigner" is deprecated as "signer" field is now available.

Please contact your administrator for more information.


## Async API Call 

**Async API Call: https://areal.ai/api/v1/ocr/?async_proc=1**

Request Method: **POST**.

Areal.ai supports asynchronous API calls for document processing. In order to make an Async API call, the Post request should be made to the document processing endpoint with "async_proc" parameter set to 1. The following shows the required URL for the Async API call.

```
https://areal.ai/api/v1/ocr/?async_proc=1
```
 
Async API call response will be like the following:

```
{
    "in_progress": true,
    "results": [],
    "session_url": "https://areal.ai/documents/sessions/aaaa1111-bbbb-cccc-dddd-eeee22224444",
    "upload_session_uuid": "aaaa1111-bbbb-cccc-dddd-eeee22224444"
}
```

The "session_url" provided in this response can be presented within 3rd party software solutions to allow users to quickly locate the processed documents in Areal.ai Platform.

The "upload_session_uuid" can be used to programmatically pull the results of the document processing session. Please check the "Sessions, Search and Get Sessions" section below for more information.

## Webhooks

Areal.ai also supports Webhooks (GET requests only). In order to activate a Webhook, URL address of an active endpoint should be provided to the Areal.ai team. Please make sure that the provided URL address is not blocked for HTTPS calls coming from the Areal.ai servers. 

The Webhook payload will include different information for different cases. Currently, there are 2 different payloads.

**First Notification: Session is Created.**

A notification with JSON payload will be sent when a new Session/Loan is created, this is just for the reference and does not mean that document(s) are processed yet (which might be still getting processed.) A separate notification with 'document' details will be sent when all document(s) are processed within this Session.

Payload:
```
{
   "notification": {
        "session_uuid": "aaaa1111-bbbb-cccc-dddd-eeee22224444", 
        "session_name": "Loan #12345", 
        "created_by": "username", 
        "created_at": "2022-01-04 00:00:46", 
        "sent_at": "2022-01-04 00:00:47", 
        "description": "New session has been created"
    }
}
```

**Second Notification: Documents are Processed.**

This notification is sent when the documents in this Session are processed. The "documents" array briefly contains all new documents created as a result of the processing of the original document. 

Payload:
```
{
    "notification": {
        "session_uuid": "aaaa1111-bbbb-cccc-dddd-eeee22224444", 
        "session_name": "Loan #12345", 
        "created_by": "username", 
        "sent_at": "2021-11-01 00:01:40",
        "description": "New document has been processed",  
        "documents": [{
                "doc_id": "baaa0911-ebbb-eccc-dddd-aaaa33331111", 
                "file_name": "123345_Unclassified.pdf", 
                "loan_id": "loan_1233", 
                "template_uuid": "{a GUID of template}", 
                "elapsed_time": "{how much time this process spent in ms}", 
                "num_pages": "3", 
                "doc_url": "{a URL to download the doc, expires in 10 min}", 
                "created_at": "2022-01-04 00:02:05"
            }, {
                "doc_id": "caaa1211-abbb-dccc-dddd-123400996677", 
                "file_name": "34567_Title.pdf", 
                "loan_id": "loan_7890", 
                "template_uuid": "{a GUID of template}", 
                "elapsed_time": "{how much time this process spent in ms}", 
                "num_pages": "5", 
                "doc_url": "{a URL to download the doc, expires in 10 min}", 
                "created_at": "2022-01-04 00:02:26"
            }, 
            {...}
        ]
    }
}
```

The "session_uuid" can be used to programmatically pull the results of the document processing Session. Please check the "Sessions, Search and Get Sessions" section below for more information.

For additional quesitons, please contact the Areal.ai Team.


## Get Documents

**Document API: https://areal.ai/api/v1/document/**

Request Method: **GET**. 

The following filters are applicable:

**document_uuid**

Returns a specific document. A sample call: 
```
https://areal.ai/api/v1/document/2dbd28a1-4b64-4525-bb51-4eacf8363f03
```

A sample response:
```
{
    "created_at": "2021-03-16T03:59:26.446073",
    "extracted_data": [
        {
            "approved_value": "50,000.00",
            "component": {
                "category": "text",
                "name": "Wages",
                "uuid": "9b1153e0-e300-11e9-93dd-1f1efd155a62"
            },
            "confidence_rate": "0.97",
            "from_page_number": 0,
            "height_percent": "0.0291",
            "name": "1 Wages",
            "processed_value": "50,000.00",
            "uuid": "87390cf8-8f2a-4b6c-ab9e-1fb616097df8",
            "width_percent": "0.0667",
            "x_percent": "0.3469",
            "y_percent": "0.7557"
        },
    ],
    "name": "postman_W2.png",
    "notes": null,
    "original_document_uuid": "207d61e6-268e-4ba7-94ea-5a8112ea15c1",
    "page_count": 1,
    "pages": [
        "34efc562-05ac-48a8-886a-7b3efbc4c876"
    ],
    "resource_uri": "/api/v1/document/2dbd28a1-4b64-4525-bb51-4eacf8363f03/",
    "status": "draft",
    "template": {
        "name": "W2",
        "uuid": "4e0550c2-ecbf-40d4-bdf0-2f9de24b525f"
    },
    "updated_at": "2021-03-16T03:59:26.446099",
    "upload_session": {
        "name": "postman",
        "uuid": "6d1cd890-85f9-11eb-9ba8-614f814eb3d6"
    },
    "user_extracted_data": null,
    "uuid": "2dbd28a1-4b64-4525-bb51-4eacf8363f03"
}

```

**upload_session Filter**

Returns all documents within that session. A sample request: 
```
https://areal.ai/api/v1/document/?upload_session=6d1cd890-85f9-11eb-9ba8-614f814eb3d6&order_by=-updated_at&limit=100&offset=0
```
This request orders the results based on the updated_date.

**Applicable Sorting and Filtering Parameters**

Default parameters are:

- status
- created_at
- updated_at


## Sessions, Search and Get Sessions

**Session API: https://areal.ai/api/v1/upload_session/**

Request Method: **GET**.

Sessions are used to group documents under a "folder". In a mortgage application use-case, a session will represent the loan file. While in a title underwriting use-case, a session will represent the title file. 

Sessions can also be used to identify a specific document processing request. For example, if you trigger a document processing request and specify a *upload_session_uuid* in that request, then you can find the processed documents by simply making a Get Document request and adding the *upload_session* filter.


3 common use-cases are:

* Getting specifics of a session

A sample request is:

```
https://areal.ai/api/v1/upload_session/45bd28a1-4b64-4525-bb51-4eacf8363f55
```

* Getting a list of sessions

This is a **GET** method. A sample request is:

```
https://areal.ai/api/v1/upload_session/?order_by=-updated_at&limit=20&offset=0&status=all
```

* Searching for a session

You can search for different phrases in session names and retrive sessions containing the words you are looking for. This is extremely helpful to search for mortgage or title files with certain keywords such as the address of the property. Please recall that, a session name including certain keywords (address, file number etc.) should be provided in document processing requests in order to be able to search for these keywords later.

A sample search request looks like:

```
http://areal.ai/api/v1/document/?limit=100&offset=0&search=SEARCH-PHRASE
```
The search results will return  sessions and documents whose names contain or match the search phrase.

**Applicable Sorting and Filtering Parameters**

Default parameters are:

- status
- created_at
- updated_at


## Giving Feedback for Extracted Data or Document

**Feedback API: http://areal.ai/api/v1/extracted_data/extracted-data-uuid/**

Request Method: **PATCH**.

This API is used to replace the wrong or the missing extracted data value with the correct one. This request should include the following *approved_value* key-value pair in the body.

```
{
   approved_value: "3.8750"   
}
```
**Document Feedback API: http://areal.ai/api/v1/document/document-uuid/**

Request Method: **PATCH**.

This API is used to approve the entire document. Approving a document means that the assigned classification to a document and all extracted data fields are correct. This API should be called once all data fields are corrected using the API call above to give the best feedback back to the platform.

```
{
   status: "active"
}
```
The status key-value pair above should be added to this request.


## Sample Response

```
[
  {
    "component_results": [
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "1545-0008",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.005555555555555556,
            "page": 0,
            "processed_value": "1545-0008",
            "uuid": "c569e403-b7c2-4056-81ea-9fed428064a9",
            "width_percent": 0.055992141453831044,
            "x_percent": 0.5176817288801572,
            "y_percent": 0.32083333333333336
          }
        ],
        "name": "00 - OMD",
        "uuid": "6eab0e40-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "2017",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.022222222222222223,
            "page": 0,
            "processed_value": "2017",
            "uuid": "3e182053-ca70-4409-81af-f6b02ae2753a",
            "width_percent": 0.10805500982318271,
            "x_percent": 0.4351669941060904,
            "y_percent": 0.6673611111111111
          }
        ],
        "name": "00 - Year",
        "uuid": "bb21fca0-f74d-11e9-8b38-e5e69c6c59af"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "123-45-6789",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.006944444444444444,
            "page": 0,
            "processed_value": "123-45-6789",
            "uuid": "e48b265a-4cde-4408-83d6-f7b227d8d4ed",
            "width_percent": 0.08644400785854617,
            "x_percent": 0.30353634577603145,
            "y_percent": 0.3229166666666667
          }
        ],
        "name": "00A - Employee SSN",
        "uuid": "76d9a130-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "Jane A. Doe",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.008333333333333333,
            "page": 0,
            "processed_value": "Jane A. Doe",
            "uuid": "82533e77-e466-4d80-b779-e950fae19b9e",
            "width_percent": 0.0825147347740668,
            "x_percent": 0.21905697445972494,
            "y_percent": 0.5076388888888889
          }
        ],
        "name": "00B - Employee Name",
        "uuid": "413007d5-57d3-4cf8-97ad-bc0e9ae59d65"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "123 Elm Street Anywhere Else, PA 17111",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.022222222222222223,
            "page": 0,
            "processed_value": "123 Elm Street Anywhere Else, PA 17111",
            "uuid": "3bdbd22f-55e6-42d5-82d9-2018199d8a9c",
            "width_percent": 0.17288801571709234,
            "x_percent": 0.1738703339882122,
            "y_percent": 0.5201388888888889
          }
        ],
        "name": "00C - Employee Address",
        "uuid": "861132d0-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0,
            "page": 0,
            "processed_value": "",
            "uuid": "d52ffc8d-ec51-44e1-9d0d-4da89a3ae10f",
            "width_percent": 0,
            "x_percent": 0,
            "y_percent": 0
          }
        ],
        "name": "00D - Employer TIN",
        "uuid": "7c16b980-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "The Big Company",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.010416666666666666,
            "page": 0,
            "processed_value": "The Big Company",
            "uuid": "5d8bdf55-c37a-4651-b935-43f1c8c71cdb",
            "width_percent": 0.1237721021611002,
            "x_percent": 0.20923379174852652,
            "y_percent": 0.38680555555555557
          }
        ],
        "name": "00E - Employer Name",
        "uuid": "30a5303d-5773-4ff0-8b0b-445bcbcee24b"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "12 Main Street Anywhere, NC 28111",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.022916666666666665,
            "page": 0,
            "processed_value": "12 Main Street Anywhere, NC 28111",
            "uuid": "1efbf7ef-f8ce-479a-8b4b-2377c7cadfc7",
            "width_percent": 0.14047151277013753,
            "x_percent": 0.19842829076620824,
            "y_percent": 0.3993055555555556
          }
        ],
        "name": "00F - Employer Address",
        "uuid": "94b81d30-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "50,000.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.008333333333333333,
            "page": 0,
            "processed_value": "50,000.00",
            "uuid": "b71da0a6-a09c-41e2-bd27-044f3f097969",
            "width_percent": 0.06777996070726916,
            "x_percent": 0.34577603143418467,
            "y_percent": 0.61875
          }
        ],
        "name": "01 - Wages",
        "uuid": "9b1153e0-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "6,835.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.006944444444444444,
            "page": 0,
            "processed_value": "6,835.00",
            "uuid": "ce048185-2421-4ee7-8943-0409ed4c588d",
            "width_percent": 0.05795677799607073,
            "x_percent": 0.8005893909626719,
            "y_percent": 0.3506944444444444
          }
        ],
        "name": "02 - Federal",
        "uuid": "a4e77c50-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "50,000.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.008333333333333333,
            "page": 0,
            "processed_value": "50,000.00",
            "uuid": "be6c2c6e-8c3a-4db0-8bda-df44a5c29727",
            "width_percent": 0.06679764243614932,
            "x_percent": 0.5972495088408645,
            "y_percent": 0.3798611111111111
          }
        ],
        "name": "03 - SS Wages",
        "uuid": "abcc2a20-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "3,100.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.006944444444444444,
            "page": 0,
            "processed_value": "3,100.00",
            "uuid": "01cd34dc-127f-43ce-8fa7-1b42b818ec26",
            "width_percent": 0.05893909626719057,
            "x_percent": 0.8005893909626719,
            "y_percent": 0.3798611111111111
          }
        ],
        "name": "04 - SS Tax",
        "uuid": "b0ca8bc0-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "50,000.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.009027777777777777,
            "page": 0,
            "processed_value": "50,000.00",
            "uuid": "12513257-b679-4d93-9d9f-09afd0f6ca31",
            "width_percent": 0.06581532416502947,
            "x_percent": 0.5972495088408645,
            "y_percent": 0.40694444444444444
          }
        ],
        "name": "05 - Medicare Wages",
        "uuid": "ca43fff0-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "725.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.007638888888888889,
            "page": 0,
            "processed_value": "725.00",
            "uuid": "39865b67-4761-472f-93c5-c078ee94d276",
            "width_percent": 0.047151277013752456,
            "x_percent": 0.806483300589391,
            "y_percent": 0.4083333333333333
          }
        ],
        "name": "06 - Medicare Tax",
        "uuid": "d0fcbbc0-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0,
            "page": 0,
            "processed_value": "",
            "uuid": "dd40e782-043d-483b-b58b-ba25ca866d69",
            "width_percent": 0,
            "x_percent": 0,
            "y_percent": 0
          }
        ],
        "name": "12a Box",
        "uuid": "cd08decf-e179-47da-b6bb-0bc45c8efa85"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "PA",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.009027777777777777,
            "page": 0,
            "processed_value": "PA",
            "uuid": "9e02a065-1c2c-4314-95ca-3f796703783e",
            "width_percent": 0.018664047151277015,
            "x_percent": 0.060903732809430254,
            "y_percent": 0.6194444444444445
          }
        ],
        "name": "15 State",
        "uuid": "ffcfe2b0-e300-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "50,000.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.008333333333333333,
            "page": 0,
            "processed_value": "50,000.00",
            "uuid": "0922bf86-2095-44b7-b20e-bc8f008ebcdc",
            "width_percent": 0.06777996070726916,
            "x_percent": 0.34577603143418467,
            "y_percent": 0.61875
          }
        ],
        "name": "16 State Wages",
        "uuid": "0c0075e0-e301-11e9-93dd-1f1efd155a62"
      },
      {
        "category": "text",
        "extracted_data": [
          {
            "approved_value": "1,535.00",
            "confidence_rate": 0.9711745381355286,
            "grouped_data": [],
            "height_percent": 0.006944444444444444,
            "page": 0,
            "processed_value": "1,535.00",
            "uuid": "40cc2b53-1aeb-48f1-a6f4-41e7338681d4",
            "width_percent": 0.05795677799607073,
            "x_percent": 0.48821218074656186,
            "y_percent": 0.6194444444444445
          }
        ],
        "name": "17 State Tax",
        "uuid": "16f024a0-e301-11e9-93dd-1f1efd155a62"
      }
    ],
    "data": {
      "document": {
        "created_at": "2023-02-15 18:30:12.904496",
        "download_urls": {
          "doc_url": "",
          "thumb_url": ""
        },
        "page_count": 1,
        "status": "draft",
        "updated_at": "2023-02-15 18:30:12.904517",
        "upload_session": "02209a41-49c3-42f8-a2d8-516be2c914ae"
      }
    },
    "document_name": "W2_1_Borrower-W2.pdf",
    "document_uuid": "1df9bdbd-b1b1-4cf2-bfe6-a85870ad998f",
    "extracted_data": [
      {
        "component": {
          "category": "text",
          "name": "00 - OMD",
          "uuid": "6eab0e40-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "1545-0008",
        "uuid": "c569e403-b7c2-4056-81ea-9fed428064a9"
      },
      {
        "component": {
          "category": "text",
          "name": "00 - Year",
          "uuid": "bb21fca0-f74d-11e9-8b38-e5e69c6c59af"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "2017",
        "uuid": "3e182053-ca70-4409-81af-f6b02ae2753a"
      },
      {
        "component": {
          "category": "text",
          "name": "00A - Employee SSN",
          "uuid": "76d9a130-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "123-45-6789",
        "uuid": "e48b265a-4cde-4408-83d6-f7b227d8d4ed"
      },
      {
        "component": {
          "category": "text",
          "name": "00B - Employee Name",
          "uuid": "413007d5-57d3-4cf8-97ad-bc0e9ae59d65"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "Jane A. Doe",
        "uuid": "82533e77-e466-4d80-b779-e950fae19b9e"
      },
      {
        "component": {
          "category": "text",
          "name": "00C - Employee Address",
          "uuid": "861132d0-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "123 Elm Street Anywhere Else, PA 17111",
        "uuid": "3bdbd22f-55e6-42d5-82d9-2018199d8a9c"
      },
      {
        "component": {
          "category": "text",
          "name": "00D - Employer TIN",
          "uuid": "7c16b980-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "",
        "uuid": "d52ffc8d-ec51-44e1-9d0d-4da89a3ae10f"
      },
      {
        "component": {
          "category": "text",
          "name": "00E - Employer Name",
          "uuid": "30a5303d-5773-4ff0-8b0b-445bcbcee24b"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "The Big Company",
        "uuid": "5d8bdf55-c37a-4651-b935-43f1c8c71cdb"
      },
      {
        "component": {
          "category": "text",
          "name": "00F - Employer Address",
          "uuid": "94b81d30-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "12 Main Street Anywhere, NC 28111",
        "uuid": "1efbf7ef-f8ce-479a-8b4b-2377c7cadfc7"
      },
      {
        "component": {
          "category": "text",
          "name": "01 - Wages",
          "uuid": "9b1153e0-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "50,000.00",
        "uuid": "b71da0a6-a09c-41e2-bd27-044f3f097969"
      },
      {
        "component": {
          "category": "text",
          "name": "02 - Federal",
          "uuid": "a4e77c50-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "6,835.00",
        "uuid": "ce048185-2421-4ee7-8943-0409ed4c588d"
      },
      {
        "component": {
          "category": "text",
          "name": "03 - SS Wages",
          "uuid": "abcc2a20-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "50,000.00",
        "uuid": "be6c2c6e-8c3a-4db0-8bda-df44a5c29727"
      },
      {
        "component": {
          "category": "text",
          "name": "04 - SS Tax",
          "uuid": "b0ca8bc0-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "3,100.00",
        "uuid": "01cd34dc-127f-43ce-8fa7-1b42b818ec26"
      },
      {
        "component": {
          "category": "text",
          "name": "05 - Medicare Wages",
          "uuid": "ca43fff0-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "50,000.00",
        "uuid": "12513257-b679-4d93-9d9f-09afd0f6ca31"
      },
      {
        "component": {
          "category": "text",
          "name": "06 - Medicare Tax",
          "uuid": "d0fcbbc0-e300-11e9-93dd-1f1efd155a62"
        },
        "confidence_rate": 0.97,
        "page": 0,
        "processed_value": "725.00",
        "uuid": "39865b67-4761-472f-93c5-c078ee94d276"
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
        "uuid": "dd40e782-043d-483b-b58b-ba25ca866d69"
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
        "uuid": "9e02a065-1c2c-4314-95ca-3f796703783e"
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
        "uuid": "0922bf86-2095-44b7-b20e-bc8f008ebcdc"
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
        "uuid": "40cc2b53-1aeb-48f1-a6f4-41e7338681d4"
      }
    ],
    "processing_time_ms": 2792,
    "template_name": "",
    "template_uuid": "4e0550c2-dddd-bbbb-aaaa-2f9de24b525f"
  }
]
```
