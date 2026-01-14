ANTI-SCAMMER API DOCUMENTATION
=============================

Version
-------
v1.2

Base URL
--------
``https://overclinical-kenia-loculicidally.ngrok-free.dev``


Overview
--------
The AntiScammer API provides:

- Scammer ID lookup
- Message canonicalization
- ML-based scam detection

All protected endpoints require an API key sent via HTTP headers.


Authentication
--------------
All protected endpoints require an API key.

**Header**
::

   X-API-Key: YOUR_API_KEY

**Notes**

- API keys expire
- Expired, missing, or invalid keys return **HTTP 401**
- API keys must be sent as headers (not query parameters)
- When using ngrok, include the following header:

::

   ngrok-skip-browser-warning: true


Health Check
------------
**GET** ``/health``

**Authentication**
Not required

**Description**
Check if the API is online.

**Example Request**
::

   curl https://overclinical-kenia-loculicidally.ngrok-free.dev/health \
     -H "ngrok-skip-browser-warning: true"

**Example Response**
::

   {
     "ok": true
   }


Scammer Lookup (Single)
----------------------
**GET** ``/lookup/{user_id}``

**Authentication**
Required

**Description**
Check if a Discord user ID is flagged as a scammer.

**Path Parameters**

- ``user_id`` (string)  
  Discord user ID. Non-digit characters are stripped automatically.

**Query Parameters**

- ``include_reason`` (boolean, optional)  
  If true, includes the scam reason when flagged.

**Example Request**
::

   curl "https://overclinical-kenia-loculicidally.ngrok-free.dev/lookup/992618366844014592?include_reason=true" \
     -H "X-API-Key: YOUR_API_KEY" \
     -H "ngrok-skip-browser-warning: true"

**Example Response (Flagged)**
::

   {
     "user_id": "992618366844014592",
     "is_flagged": true,
     "reason": "Nitro phishing scam"
   }

**Example Response (Not Flagged)**
::

   {
     "user_id": "992618366844014592",
     "is_flagged": false
   }


Scammer Lookup (Batch)
---------------------
**POST** ``/lookup``

**Authentication**
Required

**Description**
Lookup multiple Discord user IDs in a single request.

**Request Body**
::

   {
     "user_ids": [
       "992618366844014592",
       "123456789012345678"
     ],
     "include_reason": true
   }

**Limits**

- 1 to 500 user IDs per request

**Example Request**
::

   curl -X POST "https://overclinical-kenia-loculicidally.ngrok-free.dev/lookup" \
     -H "Content-Type: application/json" \
     -H "X-API-Key: YOUR_API_KEY" \
     -H "ngrok-skip-browser-warning: true" \
     -d '{
       "user_ids": ["992618366844014592"],
       "include_reason": true
     }'

**Example Response**
::

   {
     "count": 1,
     "results": [
       {
         "user_id": "992618366844014592",
         "is_flagged": true,
         "reason": "Crypto impersonation scam"
       }
     ]
   }


Message Canonicalization
------------------------
**POST** ``/canonicalize``

**Authentication**
Required

**Description**
Normalize a message and detect obfuscation techniques such as:

- Vertical text
- Emoji padding
- Markdown abuse
- Excessive whitespace

**Request Body**
::

   {
     "message": "FnRnEnEnNnInTnRnO"
   }

**Example Response**
::

   {
     "raw": "FnRnEnEnNnInTnRnO",
     "clean": "F R E E N I T R O",
     "joined": "FREENITRO",
     "obfuscation": {
       "looks_vertical": true,
       "line_count": 9,
       "single_char_line_ratio": 1.0,
       "whitespace_ratio": 0.42
     }
   }


Scam Detection
--------------
**POST** ``/detect``

**Authentication**
Required

**Description**
Runs ML-based scam detection with optional conversation context.

**Request Body**
::

   {
     "message": "Free nitro click here",
     "context_messages": [
       {
         "created_at": "2025-12-16T21:00:00",
         "author": "user123",
         "content": "check this out"
       }
     ]
   }

**Example Response**
::

   {
     "probability": 97.3,
     "is_scam": true,
     "reason": "Known Discord Nitro phishing pattern with link bait."
   }

**Detection Notes**

- Probability ranges from 0 to 100
- Strong obfuscation may force ``is_scam = true``
- Hard bypass rules override model output

Ban request
---------------
Post /banrequest
**Authentication**
Required

Description: A way for API users to send me requests for users to be banned. Ideal for mass reports. REQUIRES PROOF

Example request:

::
curl -X POST "https://your-api.ngrok-free.dev/banrequest" \
  -H "X-API-Key: antiscammer-internal-KEY-456" \
  -F "user_id=546449742318534681" \
  -F "reason=Crypto wallet drain scam" \
  -F "notes=User DM’d multiple members with a fake support link" \
  -F "proof=@./evidence/screenshot1.png"

**Response:** 
{
  "ok": true,
  "case_id": "A9F3D8C21B7E"
}

Error Responses
---------------
**401 Unauthorized**
::

   {
     "detail": "Missing API key"
   }

::

   {
     "detail": "API key expired"
   }

**400 Bad Request**
::

   {
     "detail": "Invalid user_id"
   }


Security Notes
--------------
- Do not expose API keys publicly
- Use HTTPS only
- Rotate API keys periodically (Ask ram2 for new keys. We can provide any keys as you ask)
- Dont constantly flood the API (Heavy traffic is expected but dont create a inf loop)

Support
-------
For access requests, issues, or partnerships, contact the AntiScammer development team.
