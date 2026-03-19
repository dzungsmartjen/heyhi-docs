# SSO Integration Guide: Moodle → HeyHi (Custom JWT Flow)

## Overview

This document defines the SSO integration flow from Moodle to HeyHi using a custom token-based approach.

- HeyHi is the SSO issuer.
- Moodle is the client.
- Authentication uses a short-lived, one-time token.

## End-to-End Flow

```
[Moodle] --(API call)--> [HeyHi issues token]
[Moodle] --(redirect)--> [HeyHi SSO endpoint]

1. User logs into Moodle
2. User clicks "Go to HeyHi"
3. Moodle calls HeyHi API to obtain SSO token
4. Moodle redirects user to HeyHi with token
5. HeyHi validates token
6. HeyHi logs in user and creates session
7. User is redirected to target page
```

## Security Requirements

- Token must be **single-use (jti enforced)**.
- All traffic must use **HTTPS**.
- Token validation must be **server-side only**.

## Token Issuance API

### Endpoint

```
POST https://api.heyhiapi.com/api/v1/access
```

### Request Headers

| Header        | Type   | Required | Description         |
|---------------|--------|----------|---------------------|
| Content-Type  | string | Yes      | multipart/form-data |

### Request Body (form-data)

| Field     | Type   | Required | Description                            |
|-----------|--------|----------|----------------------------------------|
| username  | string | Yes      | Username (must be unique per user)     |
| user_role | int    | Yes      | Role ID (1=Tutor, 2=Student, 3=Parent) |

Allowed values:

- `username`: `student`
- `user_role`: `1`, `2`, `3`

Example request body:

```json
{
  "username": "student",
  "user_role": 1,
}
```

### Response

Content-Type : application/json; charset=UTF-8

```json
{
  "token": "YTozOntzOjg6InVzZXJuYW1lIjtzO.........."
}
```

Token characteristics:

- Token is an opaque token (not JWT).
- Token is generated and signed internally by HeyHi.
- Token is single-use only. but this no expired and can reuse

## Redirect Flow

### Endpoint

```
GET https://{branch-url}/profile?token={generated_token}
```

### Query Parameters

| Param | Type   | Required | Description |
|-------|--------|----------|-------------|
| token | string | Yes      | SSO token   |

## Token Validation Rules (HeyHi)

When HeyHi receives the SSO request at `/sso/callback`, it must:

1. Validate token existence.
2. Verify token integrity (signature/internal check).
3. Retrieve user info associated with token.
4. Create or update user session.
5. Authenticate user.

## Post-Authentication Redirect

After successful authentication:

```
GET /dashboard
```

