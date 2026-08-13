
JWT (JSON Web Token) is one of the most commonly used authentication mechanisms in modern web applications, APIs, microservices, and mobile applications.

Understanding JWT properly requires understanding:

1. Authentication vs Authorization
2. Traditional Session Authentication
3. JWT Structure
4. JWT Authentication Flow
5. Access Tokens & Refresh Tokens
6. Security Risks
7. Industry Standard Architecture
8. Production Best Practices

---

# 1. Authentication vs Authorization

### Authentication

Verifies who you are.

Example:

- Username + Password
- Google Login
- GitHub Login

Result:

```
User = Shashank
```

---

### Authorization

Determines what you can access.

Example:

```
User = Shashank
Role = Admin
```

Can:

- Create users
- Delete users

Cannot:

- Access super admin resources

---

# 2. Traditional Session Authentication

Before JWT, most applications used sessions.

## Flow

### Login

```
Client
  |
  | username/password
  ↓
Server
```

Server validates credentials.

Creates session:

```
{
  "sessionId": "abc123",
  "userId": 1
}
```

Stores it in database or memory.

Returns:

```
Set-Cookie: sessionId=abc123
```

---

### Future Requests

```
Cookie: sessionId=abc123
```

Server:

```
Find session
→ Get user
→ Allow request
```

---

### Problems

In distributed systems:

```
Server A
Server B
Server C
```

User may hit different servers.

Need:

- Shared session storage
- Redis
- Sticky sessions

Scaling becomes harder.

---

# 3. What is JWT?

JWT is a self-contained token.

Instead of storing session data on server:

```
Store user data inside token
```

Server doesn't need session storage.

---

# JWT Format

A JWT contains 3 parts:

```
HEADER.PAYLOAD.SIGNATURE
```

Example:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

.

eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiJ9

.

abcxyz123signature
```

Combined:

```
xxxxx.yyyyy.zzzzz
```

---

# 4. JWT Structure Deep Dive

---

## Header

Contains metadata.

Example:

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Meaning:

```
Algorithm = HMAC SHA256
Type = JWT
```

Base64 encoded.

---

## Payload

Contains claims.

Example:

```
{
  "userId": 1,
  "email": "user@gmail.com",
  "role": "admin"
}
```

Encoded:

```
Base64(payload)
```

Important:

### JWT Payload is NOT encrypted

Anyone can decode it.

Never store:

❌ Passwords

❌ Credit card numbers

❌ Secrets

---

## Signature

Created using:

```
Header
+
Payload
+
Secret Key
```

Example:

```
HMACSHA256(
 base64(header) + "." + base64(payload),
 secret
)
```

Output:

```
signature
```

---

# Final JWT

```
JWT =
base64(header)
+
"."
+
base64(payload)
+
"."
+
signature
```

Example:

```
xxxxx.yyyyy.zzzzz
```

---

# 5. JWT Authentication Flow

---

## Login Request

```
POST /login
```

Body:

```
{
  "email": "user@gmail.com",
  "password": "123456"
}
```

---

## Server Validation

```
Check email
Check password hash
```

If valid:

Generate JWT.

Payload:

```
{
  "sub": "123",
  "role": "admin"
}
```

---

## Return Token

```
{
  "accessToken": "jwt_here"
}
```

---

## Store Token

Frontend stores token.

Options:

### Local Storage

```
localStorage.setItem("token", token);
```

Easy but vulnerable to XSS.

---

### HttpOnly Cookie

Industry preferred.

```
Set-Cookie:
token=jwt;
HttpOnly;
Secure;
SameSite=Strict
```

JavaScript cannot access it.

---

# Authenticated Request

```
GET /profile
Authorization: Bearer JWT_TOKEN
```

---

# Server Verification

```
jwt.verify(token, SECRET)
```

Checks:

1. Signature
2. Expiration
3. Valid format

---

# If Valid

```
User authenticated
```

Request continues.

---

# 6. JWT Claims

Claims = information stored in payload.

---

## Registered Claims

### sub

Subject.

```
{
  "sub": "123"
}
```

User ID.

---

### exp

Expiration.

```
{
  "exp": 1750000000
}
```

Token expires automatically.

---

### iat

Issued At.

```
{
  "iat": 1700000000
}
```

---

### iss

Issuer.

```
{
  "iss": "auth-service"
}
```

---

### aud

Audience.

```
{
  "aud": "mobile-app"
}
```

---

### nbf

Not before.

```
{
  "nbf": 1700000000
}
```

Token valid only after this time.

---

# 7. Access Token and Refresh Token

This is where beginners usually stop.

Industry systems go much further.

---

## Problem

Suppose access token lasts:

```
24 hours
```

If stolen:

Attacker gets 24 hours access.

Bad.

---

## Solution

Short-Lived Access Token

Example:

```
15 minutes
```

---

## Refresh Token

Long-lived token.

Example:

```
30 days
```

Used only to generate new access tokens.

---

# Industry Flow

Login:

```
Access Token = 15 min
Refresh Token = 30 days
```

---

Client stores:

```
Access Token
Refresh Token
```

---

After 15 minutes:

```
Access Token expired
```

---

Client sends:

```
POST /refresh
```

```
{
  "refreshToken": "..."
}
```

---

Server validates refresh token.

Generates:

```
New Access Token
```

User stays logged in.

---

# 8. Refresh Token Rotation

Basic refresh tokens are not enough.

Industry uses rotation.

---

Old token:

```
RT1
```

Refresh request:

```
RT1
```

Server returns:

```
AT2
RT2
```

And invalidates:

```
RT1
```

Now only:

```
RT2
```

works.

---

If attacker steals RT1:

```
RT1 already invalid
```

Attack blocked.

---

# 9. Symmetric vs Asymmetric JWT

---

## HS256

Uses one secret.

```
Sign -> Secret
Verify -> Same Secret
```

---

### Problem

Every service needs secret.

```
User Service
Order Service
Payment Service
```

All know secret.

Risky.

---

## RS256

Uses:

```
Private Key
Public Key
```

Sign:

```
Private Key
```

Verify:

```
Public Key
```

---

Industry preference:

```
RS256
ES256
```

Especially microservices.

---

# 10. JWT Middleware

Every protected request passes through middleware.

Example:

```
const auth = (req, res, next) => {
  const token =
    req.headers.authorization?.split(" ")[1];

  try {
    const payload =
      jwt.verify(token, PUBLIC_KEY);

    req.user = payload;

    next();
  } catch {
    return res.status(401).json({
      message: "Unauthorized"
    });
  }
};
```

---

# 11. Role Based Access Control (RBAC)

JWT often carries role information.

Payload:

```
{
  "sub": "123",
  "role": "admin"
}
```

---

Middleware:

```
const adminOnly = (req,res,next)=>{
   if(req.user.role !== "admin")
      return res.status(403);

   next();
}
```

---

Levels:

```
User
Moderator
Admin
SuperAdmin
```

---

# 12. OAuth + JWT

Google Login:

```
User
↓
Google
↓
Backend
↓
JWT
```

Google verifies identity.

Your backend issues JWT.

JWT becomes internal auth token.

---

# 13. Common JWT Attacks

---

## 1. Token Theft

Stolen token.

Protection:

- HTTPS
- HttpOnly cookies
- Short expiration

---

## 2. XSS

Malicious JS reads local storage.

Bad:

```
localStorage.getItem("token")
```

Prefer:

```
HttpOnly Cookies
```

---

## 3. CSRF

Cookies automatically sent.

Protection:

```
SameSite=Strict
CSRF token
```

---

## 4. Weak Secret

Bad:

```
secret123
```

Good:

```
256-bit random secret
```

---

## 5. No Expiration

Bad:

```
{
}
```

No exp claim.

Always add:

```
{
  "exp": "..."
}
```

---

# 14. Industry Standard Production Architecture

```
                Login
                  |
                  v
         +----------------+
         | Auth Service   |
         +----------------+
                  |
                  v
       +-------------------+
       | Access Token      |
       | 15 minutes        |
       +-------------------+

       +-------------------+
       | Refresh Token     |
       | 30 days           |
       +-------------------+

                  |
                  v

+-----------+  +-----------+  +-----------+
| User API  |  | Order API |  | Payment   |
+-----------+  +-----------+  +-----------+

Verify using Public Key
(RS256)
```

---

# 15. What Big Companies Typically Do

### Netflix

- OAuth/OIDC
- JWT
- Short-lived access tokens
- Refresh token rotation

### Google

- OAuth 2.0
- OpenID Connect
- JWT ID Tokens

### Microsoft

- Azure AD
- JWT Access Tokens
- RS256

### AWS Cognito

- JWT
- JWK public keys
- OAuth2
# Recommended Industry Setup (2026)

For a production-grade web application:

```
Password → Argon2/Bcrypt

↓

Login

↓

Access Token (JWT)
15 minutes

↓

Refresh Token
30 days

↓

HttpOnly Cookie

↓

RS256

↓

Refresh Token Rotation

↓

Role/Permission Claims

↓

HTTPS Everywhere

↓

Rate Limiting

↓

Audit Logging
```

## JWT Checklist

✅ Access token expiry (10–15 min)

✅ Refresh token expiry (7–30 days)

✅ Refresh token rotation

✅ RS256/ES256

✅ HttpOnly cookies

✅ Secure cookies

✅ SameSite protection

✅ HTTPS only

✅ Role-based authorization

✅ Token revocation strategy

✅ Audit logs

✅ Rate limiting