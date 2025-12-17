ANTI-SCAMMER API DOCUMENTATION
=============================

Version: v1
Base URL:
https://overclinical-kenia-loculicidally.ngrok-free.dev


OVERVIEW
--------
The AntiScammer API provides scammer ID lookup, message canonicalization,
and ML-based scam detection. All protected endpoints require an API key
sent via HTTP headers.


AUTHENTICATION
--------------
All protected endpoints require an API key.

Header:
X-API-Key: YOUR_API_KEY

Notes:
- API keys expire
- Expired, missing, or invalid keys return HTTP 401
- API keys must be sent as headers (not query parameters)
- For ngrok usage, include:
  ngrok-skip-browser-warning: true


SECURITY NOTES
--------------
- Do not cache lookup responses
- Do not expose API keys publicly
- Use HTTPS only
- Rotate API keys periodically


SUPPORT
-------
For access requests, issues, or partnerships, contact the AntiScammer
development team.
