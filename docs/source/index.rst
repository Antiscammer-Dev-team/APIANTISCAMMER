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
- Ban request submission with required proof

All protected endpoints require an API key sent via HTTP headers.


Authentication
--------------
All protected endpoints require an API key.

**Headers**
::

   X-API-Key: YOUR_API_KEY
   ngrok-skip-browser-warning: true

**Notes**

- API keys expire
- Expired, missing, or invalid keys return **HTTP 401**
- API keys must be sent as headers (not query parameters)
- When using ngrok, the ngrok header is required


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
     "decision": true,
     "uncertanty": false,
     "confidence": "high",
     "reason": "Known Discord Nitro phishing pattern with link bait."
   }

**Detection Notes**

- Probability ranges from 0 to 100
- Strong obfuscation may force ``is_scam = true``
- Hard bypass rules override model output


Ban Request
-----------
**POST** ``/banrequest``

**Authentication**
Required

**Description**
Submit a request for a user to be banned.
Proof is required.

**Form Fields**

- ``user_id`` (string, required)
- ``reason`` (string, required)
- ``notes`` (string, optional)
- ``proof`` (file, required)

**Example Request**
::

   curl -X POST "https://overclinical-kenia-loculicidally.ngrok-free.dev/banrequest" \
     -H "X-API-Key: YOUR_API_KEY" \
     -H "ngrok-skip-browser-warning: true" \
     -F "user_id=1372638432412827771" \
     -F "reason=Testing via API" \
     -F "notes=Testing" \
     -F "proof=@proof.png"

**Example Response**
::

   {
     "case_id": "BD42775FBBCB",
     "created_at": "2026-01-14T15:50:36.324074+00:00",
     "user_id": "1372638432412827771",
     "reason": "Testing via API",
     "notes": "Testing",
     "proof_file": "proof.png",
     "proof_original_name": "A_GATITO_AZUL_EMOTES_MEDIDAS_DISCORD-19.png",
     "reporter_meta": {
       "expires_at": "3072-12-31T23:59:59Z",
       "label": "Modara"
     },
     "status": "pending"
   }
Ban Request Lookup
------------------
**GET** ``/banrequest/{case_id}``

**Authentication**
Required

**Description**
Retrieve the current status and details of a previously submitted ban request.

This endpoint allows API users to check whether a case is still pending,
approved, or rejected. The endpoint is read-only.

**Path Parameters**

- ``case_id`` (string)  
  The case identifier returned when the ban request was created.

**Example Request**
::

   curl "https://overclinical-kenia-loculicidally.ngrok-free.dev/banrequest/BD42775FBBCB" \
     -H "X-API-Key: YOUR_API_KEY" \
     -H "ngrok-skip-browser-warning: true"

**Example Response (Pending)**
::

   {
     "case_id": "BD42775FBBCB",
     "status": "pending",
     "created_at": "2026-01-14T15:50:36.324074+00:00",
     "user_id": "1372638432412827771",
     "reason": "Testing via API",
     "notes": "Testing",
     "proof_file": "proof.png",
     "reporter_meta": {
       "expires_at": "3072-12-31T23:59:59Z",
       "label": "Modara"
     }
   }

**Example Response (Approved)**
::

   {
     "case_id": "BD42775FBBCB",
     "status": "approved",
     "created_at": "2026-01-14T15:50:36.324074+00:00",
     "user_id": "1372638432412827771",
     "reason": "Testing via API",
     "notes": "Testing",
     "proof_file": "proof.png",
     "review": {
       "reviewed_at": "2026-01-15T02:41:10Z",
       "reviewed_by": "Ram2",
       "decision": "Verified proof, ban executed"
     }
   }

**Example Response (Rejected)**
::

   {
     "case_id": "BD42775FBBCB",
     "status": "rejected",
     "created_at": "2026-01-14T15:50:36.324074+00:00",
     "user_id": "1372638432412827771",
     "reason": "Testing via API",
     "notes": "Testing",
     "proof_file": "proof.png",
     "review": {
       "reviewed_at": "2026-01-15T02:41:10Z",
       "reviewed_by": "Ram2",
       "decision": "Insufficient evidence"
     }
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
- Rotate API keys periodically
- Heavy traffic is expected, but do not flood the API or create infinite loops


Support
-------
For access requests, issues, or partnerships, contact the AntiScammer development team.
