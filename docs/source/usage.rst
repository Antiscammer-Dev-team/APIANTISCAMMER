HEALTH CHECK
GET /health

Authentication: Not required

Description: Check if the API is online.

Example request: curl https://overclinical-kenia-loculicidally.ngrok-free.dev/health

-H "ngrok-skip-browser-warning: true"
Example response: {

"ok": true
}

SCAMMER LOOKUP (SINGLE)
GET /lookup/{user_id}

Authentication: Required

Description: Check if a Discord user ID is flagged as a scammer.

Path Parameters: - user_id (string)

Discord user ID. Non-digit characters are stripped automatically.
Query Parameters: - include_reason (boolean, optional)

If true, includes the scam reason when flagged.
Example request: curl "https://overclinical-kenia-loculicidally.ngrok-free.dev/lookup/992618366844014592?include_reason=true"

-H "X-API-Key: YOUR_API_KEY" -H "ngrok-skip-browser-warning: true"
Example response (flagged): {

"user_id": "992618366844014592", "is_flagged": true, "reason": "Nitro phishing scam"
}

Example response (not flagged): {

"user_id": "992618366844014592", "is_flagged": false
}

SCAMMER LOOKUP (BATCH)
POST /lookup

Authentication: Required

Description: Lookup multiple Discord user IDs in a single request.

Request Body: {

"user_ids": [
"992618366844014592", "123456789012345678"
], "include_reason": true

}

Limits: - 1 to 500 user IDs per request

Example request: curl -X POST "https://overclinical-kenia-loculicidally.ngrok-free.dev/lookup"

-H "Content-Type: application/json" -H "X-API-Key: YOUR_API_KEY" -H "ngrok-skip-browser-warning: true" -d '{

"user_ids": ["992618366844014592"], "include_reason": true
}'

Example response: {

"count": 1, "results": [

{
"user_id": "992618366844014592", "is_flagged": true, "reason": "Crypto impersonation scam"
}

]

}

MESSAGE CANONICALIZATION
POST /canonicalize

Authentication: Required

Description: Normalize a message and detect obfuscation techniques such as: - Vertical text - Emoji padding - Markdown abuse - Excessive whitespace

Request Body: {

"message": "FnRnEnEnNnInTnRnO"
}

Example response: {

"raw": "FnRnEnEnNnInTnRnO", "clean": "F R E E N I T R O", "joined": "FREENITRO", "obfuscation": {

"looks_vertical": true, "line_count": 9, "single_char_line_ratio": 1.0, "whitespace_ratio": 0.42
}

}

SCAM DETECTION
POST /detect

Authentication: Required

Description: Runs ML-based scam detection with optional conversation context.

Request Body: {

"message": "Free nitro click here", "context_messages": [

{
"created_at": "2025-12-16T21:00:00", "author": "user123", "content": "check this out"
}

]

}

Example response: {

"probability": 97.3, "is_scam": true, "reason": "Known Discord Nitro phishing pattern with link bait."
}

Detection Notes: - Probability ranges from 0 to 100 - Strong obfuscation may force is_scam = true - Hard bypass rules override model output

ERROR RESPONSES
401 Unauthorized: {

"detail": "Missing API key"
}

{
"detail": "API key expired"
}

400 Bad Request: {

"detail": "Invalid user_id"
}

SECURITY NOTES
Do not cache lookup responses
Do not expose API keys publicly
Use HTTPS only
Rotate API keys periodically
SUPPORT
For access requests, issues, or partnerships, contact the AntiScammer development team.
