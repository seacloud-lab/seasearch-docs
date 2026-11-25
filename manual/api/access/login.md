# Login

Validate credentials and return basic user information. This does not issue a token or start a session; subsequent API calls still use [Basic authentication](../authentication.md).

```http
POST /api/login
```

No Basic header is required for this credential check. Send a JSON object with `_id` and `password` (strings). No path/query parameters are used.

```sh title="Validate credentials without creating a session"
curl -X POST "$SS_URL/api/login" \
  -H 'Content-Type: application/json' \
  --data-binary '{"_id":"your-user","password":"your-password"}'
```

HTTP 200 success:

```json title="HTTP 200 · Valid credentials"
{"validated":true,"user":{"_id":"your-user","name":"Example user","role":"reader"}}
```

!!! warning "Check validated, not only HTTP status"
    Invalid credentials also return 200, but with `validated: false` and empty user strings. Check `validated`, not only status.

Malformed JSON returns 400. Use HTTPS to protect the body credentials.
