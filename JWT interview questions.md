
---

# JWT Interview Questions (Most Asked → Advanced)

I've organized these from **fresher**, **mid-level**, to **senior/system design** questions.

---

# Basic JWT Questions

### 1. What is JWT?

**Answer:**

JWT (JSON Web Token) is a compact, URL-safe token format used for securely transmitting claims between parties.

Format:

```
Header.Payload.Signature
```

---

### 2. What does JWT stand for?

**Answer:**

JSON Web Token.

Defined by RFC 7519.

---

### 3. Why is JWT used?

**Answer:**

Used for:

- Authentication
- Authorization
- Information exchange

Main advantage:

```
Stateless authentication
```

Server does not need to store session data.

---

### 4. What are the three parts of JWT?

**Answer:**

```
Header
Payload
Signature
```

Example:

```
xxxxx.yyyyy.zzzzz
```

---

### 5. What is contained in the Header?

**Answer:**

Metadata.

Example:

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

### 6. What is contained in the Payload?

**Answer:**

Claims.

Example:

```
{
  "sub": "123",
  "role": "admin"
}
```

---

### 7. What is a Claim?

**Answer:**

A claim is information stored inside JWT.

Examples:

```
{
  "sub": "123",
  "email": "user@gmail.com"
}
```

---

### 8. Is JWT encrypted?

**Answer:**

No.

JWT is typically:

```
Encoded
NOT Encrypted
```

Anyone can decode payload.

Only integrity is protected through signature.

---

### 9. Can we store passwords in JWT?

**Answer:**

Never.

JWT payload is visible after decoding.

Do not store:

- Passwords
- API keys
- Secrets
- Credit card data

---

### 10. How is JWT different from Session Authentication?

**Answer:**

|Session|JWT|
|---|---|
|Stateful|Stateless|
|Server stores session|Token stores claims|
|Session ID in cookie|JWT token|
|Scaling harder|Scaling easier|

---

# Authentication Questions

### 11. How does JWT authentication work?

**Answer:**

1. User logs in
2. Server validates credentials
3. Server generates JWT
4. Client stores token
5. Client sends token with requests
6. Server verifies token

---

### 12. How is JWT sent to the server?

**Answer:**

Usually:

```
Authorization: Bearer TOKEN
```

---

### 13. What is Bearer Token?

**Answer:**

A token that grants access simply by possession.

Anyone holding it can use it.

---

### 14. How does the server verify JWT?

**Answer:**

Checks:

- Signature
- Expiration
- Issuer
- Audience

Example:

```
jwt.verify(token, secret)
```

---

### 15. What happens if JWT expires?

**Answer:**

Server returns:

```
401 Unauthorized
```

Client must refresh or re-login.

---

# Claims Questions

### 16. What is the `sub` claim?

**Answer:**

Subject.

Usually stores:

```
{
  "sub": "userId"
}
```

---

### 17. What is `exp`?

**Answer:**

Expiration time.

```
{
  "exp": 1730000000
}
```

---

### 18. What is `iat`?

**Answer:**

Issued At timestamp.

---

### 19. What is `iss`?

**Answer:**

Issuer.

Example:

```
{
  "iss": "auth-service"
}
```

---

### 20. What is `aud`?

**Answer:**

Audience.

Specifies intended recipient.

---

### 21. What is `nbf`?

**Answer:**

Not Before.

Token valid only after specified time.

---

# Security Questions

### 22. Why is JWT considered stateless?

**Answer:**

Server does not store session data.

Everything needed is inside token.

---

### 23. What is the biggest disadvantage of JWT?

**Answer:**

Revocation is difficult.

Once issued:

```
Token remains valid until expiration
```

unless additional mechanisms exist.

---

### 24. What happens if JWT is stolen?

**Answer:**

Attacker can impersonate user until token expires.

Protection:

- HTTPS
- Short expiration
- HttpOnly cookies

---

### 25. Why should access tokens have short expiry?

**Answer:**

Reduces damage if token is compromised.

Common:

```
5–15 minutes
```

---

### 26. Why should JWT be sent over HTTPS?

**Answer:**

Without HTTPS:

```
Token can be intercepted
```

using network sniffing.

---

### 27. What is XSS risk with JWT?

**Answer:**

If stored in Local Storage:

```
localStorage.getItem("token")
```

malicious scripts can steal it.

---

### 28. How can XSS risk be reduced?

**Answer:**

Use:

```
HttpOnly Cookies
```

JavaScript cannot access them.

---

### 29. What is CSRF?

**Answer:**

Cross-Site Request Forgery.

Victim's browser sends authenticated requests without user's intent.

---

### 30. How do you protect JWT cookies from CSRF?

**Answer:**

Use:

- SameSite
- CSRF token
- Double submit cookie pattern

---

# Access Token & Refresh Token

### 31. What is an Access Token?

**Answer:**

Short-lived JWT used to access APIs.

---

### 32. What is a Refresh Token?

**Answer:**

Long-lived token used to obtain new access tokens.

---

### 33. Why use Refresh Tokens?

**Answer:**

Provides:

- Better security
- Better user experience

User stays logged in.

---

### 34. What is Refresh Token Rotation?

**Answer:**

Every refresh request generates:

```
New Access Token
New Refresh Token
```

Old refresh token becomes invalid.

---

### 35. Why is Refresh Token Rotation important?

**Answer:**

Helps detect token theft.

Reduces replay attacks.

---

### 36. Where should Refresh Tokens be stored?

**Answer:**

Prefer:

```
HttpOnly Secure Cookies
```

---

# Cryptography Questions

### 37. What is HS256?

**Answer:**

HMAC SHA256.

Uses one shared secret.

```
Sign = Secret
Verify = Secret
```

---

### 38. What is RS256?

**Answer:**

RSA SHA256.

Uses:

```
Private Key → Sign
Public Key → Verify
```

---

### 39. Which is preferred in microservices?

**Answer:**

RS256.

Services only need public key.

---

### 40. Difference between HS256 and RS256?

|HS256|RS256|
|---|---|
|Shared secret|Public/private keys|
|Simpler|More secure for distributed systems|
|All services know secret|Only auth server knows private key|

---

### 41. Why is RS256 considered safer?

**Answer:**

Verification does not require private key.

Compromised service cannot mint tokens.

---

# Authorization Questions

### 42. Can JWT be used for authorization?

**Answer:**

Yes.

Example:

```
{
  "role": "admin"
}
```

---

### 43. What is RBAC?

**Answer:**

Role Based Access Control.

Roles:

```
Admin
Manager
User
```

---

### 44. What is ABAC?

**Answer:**

Attribute Based Access Control.

Decisions based on:

- Role
- Department
- Region
- Ownership

---

### 45. Should permissions be stored in JWT?

**Answer:**

Small permission sets:

```
Yes
```

Large dynamic permissions:

```
No
```

Fetch from database.

---

# Advanced Questions

### 46. How do you revoke JWT?

**Answer:**

Options:

- Blacklist
- Token versioning
- Short expiry
- Refresh token revocation

---

### 47. What is Token Blacklisting?

**Answer:**

Store revoked token IDs.

Reject matching tokens.

Usually stored in Redis.

---

### 48. Why is JWT revocation difficult?

**Answer:**

JWT is stateless.

Server normally doesn't track issued tokens.

---

### 49. What is `jti` claim?

**Answer:**

JWT ID.

Unique identifier.

```
{
  "jti": "uuid"
}
```

Used for revocation.

---

### 50. How do you logout in JWT?

**Answer:**

Common approaches:

- Delete access token
- Revoke refresh token
- Blacklist token

---

# OAuth and JWT

### 51. Is JWT same as OAuth?

**Answer:**

No.

JWT = Token format

OAuth2 = Authorization framework

---

### 52. Is JWT same as OAuth Access Token?

**Answer:**

Not always.

OAuth access token can be:

- JWT
- Opaque token

---

### 53. What is OpenID Connect?

**Answer:**

Identity layer built on OAuth2.

Used for:

- Google Login
- Microsoft Login

---

### 54. What is an ID Token?

**Answer:**

JWT containing user identity information.

Provided by OIDC.

---

### 55. Google Login uses which standard?

**Answer:**

OAuth2 + OpenID Connect + JWT.

---

# Senior-Level Questions

### 56. Why can JWT become large?

**Answer:**

Too many claims.

Example:

```
{
  "permissions": [...]
}
```

Large payload increases network overhead.

---

### 57. What should never be stored in JWT?

**Answer:**

- Passwords
- Secrets
- API Keys
- Sensitive personal data

---

### 58. How do microservices validate JWT?

**Answer:**

Using public keys.

Often fetched through:

```
JWKS endpoint
```

---

### 59. What is JWKS?

**Answer:**

JSON Web Key Set.

Public key repository.

Used for token verification.

---

### 60. Describe a production JWT architecture.

**Answer:**

```
User Login
    ↓
Auth Service
    ↓
RS256 Access Token (15 min)
    ↓
Refresh Token Rotation
    ↓
HttpOnly Cookies
    ↓
Microservices verify via JWKS
    ↓
Role/Permission Authorization
```

---

# Frequently Asked Practical Interview Questions

### Q: Why JWT over Sessions?

**Expected Answer:**

Stateless, scalable, works well with APIs and microservices.

---

### Q: LocalStorage or Cookies?

**Expected Answer:**

For web apps:

```
HttpOnly Secure Cookies
```

are generally preferred.

---

### Q: Access token expiry?

**Expected Answer:**

```
5–15 minutes
```

---

### Q: Refresh token expiry?

**Expected Answer:**

```
7–30 days
```

depends on security requirements.

---

### Q: Which algorithm would you use?

**Expected Answer:**

```
RS256
```

for production distributed systems.

---

### Q: How do you implement logout?

**Expected Answer:**

Revoke refresh token, clear cookies, optionally blacklist access token.

---

### Q: What is the biggest JWT misconception?

**Expected Answer:**

> JWT is not encrypted. It is only signed by default.

This is one of the most commonly asked JWT interview points.



### dumb questions from me

Q:  what if user use rt1 and use /refresh and used to get at2 and rt2 in jwt refresh and acess token?

This is exactly where **refresh token rotation** becomes important.

## Scenario

User logs in:

```
Access Token = AT1 (15 min)
Refresh Token = RT1 (30 days)
```

Stored on client.

---

## Step 1: Access Token Expires

Client sends:

```
POST /refresh
RefreshToken = RT1
```

Server validates RT1.

Server returns:

```
AT2
RT2
```

and marks:

```
RT1 = REVOKED
```

Database:

|Refresh Token|Status|
|---|---|
|RT1|Revoked|
|RT2|Active|

---

## Step 2: Client Uses AT2

Normal requests:

```
Authorization: Bearer AT2
```

work fine until AT2 expires.

---

## Step 3: Client Refreshes Again

When AT2 expires:

```
POST /refresh
RefreshToken = RT2
```

Server returns:

```
AT3
RT3
```

and revokes RT2.

Database:

|Refresh Token|Status|
|---|---|
|RT1|Revoked|
|RT2|Revoked|
|RT3|Active|

---

# What if Someone Reuses RT1?

Suppose an attacker stole RT1.

After legitimate refresh:

```
RT1 → revoked
RT2 → active
```

Attacker later sends:

```
POST /refresh
RefreshToken = RT1
```

Server checks database:

```
RT1 status = revoked
```

Result:

```
401 Unauthorized
```

or

```
403 Forbidden
```

---

# Industry-Standard Reuse Detection

Many systems do more than simply reject RT1.

### Timeline

Legitimate user:

```
RT1 → RT2
```

RT1 revoked.

Later attacker uses RT1:

```
POST /refresh RT1
```

This indicates:

```
A previously-used refresh token was replayed.
```

Meaning:

- token leaked
- account may be compromised

---

## Security Response

Auth server can revoke the entire refresh-token family.

Example:

```
RT1 → RT2 → RT3 → RT4
```

If RT1 gets reused:

```
RT1 REUSE DETECTED
```

Server revokes:

```
RT2
RT3
RT4
```

forcing user to log in again.

This is called:

```
Refresh Token Family Invalidation
```

Used by many OAuth/OIDC providers.

---

# What if Two Refresh Requests Happen Simultaneously?

This is a common interview question.

### Example

Browser sends two refresh requests at the same time:

```
Request A → RT1
Request B → RT1
```

Without protection:

```
A returns RT2
B returns RT3
```

Now the client has inconsistent tokens.

---

## Solution

Use atomic database operations.

Pseudo code:

```
UPDATE refresh_tokens
SET used = true
WHERE token = RT1
AND used = false;
```

Only one request succeeds.

The second request sees:

```
RT1 already used
```

and is rejected.

---

# Typical Database Schema

```
refresh_tokens
---------------
id
user_id
token_hash
expires_at
used
revoked
parent_token_id
```

Example chain:

```
RT1
 ↓
RT2
 ↓
RT3
 ↓
RT4
```

Only the latest token is valid.

---

# Interview Answer (Short Version)

**Question: What happens after RT1 is used to get AT2 and RT2?**

**Answer:**

- RT1 is immediately invalidated/revoked.
- Server issues AT2 and RT2.
- Future refreshes must use RT2.
- Reusing RT1 should fail.
- Production systems often detect RT1 reuse and revoke the entire refresh-token chain to prevent replay attacks.
- Concurrent refresh requests should be handled atomically to avoid issuing multiple valid refresh tokens.

This is the refresh-token rotation pattern recommended by OAuth 2.0 Security Best Practices and used by providers such as [Auth0](https://auth0.com?utm_source=chatgpt.com), [Okta](https://www.okta.com?utm_source=chatgpt.com), and [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/?utm_source=chatgpt.com).

what if attacker sends refresh before at1 expired

This is one of the most important JWT security scenarios.

## Scenario

Initial login:

```
AT1 (15 min)
RT1 (30 days)
```

Attacker somehow steals:

```
RT1
```

before AT1 expires.

---

## Case 1: Attacker Uses RT1 First

### Attacker

```
POST /refresh
RT1
```

Server:

```
RT1 valid
```

Issues:

```
AT2
RT2
```

Revokes:

```
RT1
```

---

### Later Legitimate User

User still has:

```
AT1
RT1
```

AT1 may still work because it hasn't expired.

But when user tries:

```
POST /refresh
RT1
```

Server sees:

```
RT1 already used
```

Refresh fails.

---

## What Should Server Do?

A reused refresh token is a strong indicator of compromise.

Server detects:

```
RT1 was already exchanged
```

Possible conclusion:

```
Someone stole the token.
```

---

## Industry Response

Revoke entire token family.

Example:

```
RT1
 ↓
RT2
 ↓
RT3
```

If RT1 is reused:

```
RT1 replay detected
```

Server revokes:

```
RT2
RT3
```

Force login again.

---

# Important Point

### AT1 Is Still Valid

Even after RT1 is stolen:

```
AT1 remains valid
```

until expiration.

JWT access tokens are usually:

```
self-contained
stateless
```

The server generally cannot instantly revoke them.

That's why access tokens are kept short-lived.

Example:

```
5–15 minutes
```

Maximum attacker access window is small.

---

# Timeline Example

```
10:00 Login
AT1 expires at 10:15
RT1 expires in 30 days
```

---

Attacker steals RT1 at:

```
10:05
```

Uses it immediately:

```
10:06
RT1 → AT2 + RT2
```

---

Legitimate user:

```
still using AT1
```

until:

```
10:15
```

When user refreshes:

```
RT1 invalid
```

Compromise detected.

---

# How Large Companies Reduce This Risk

## 1. Very Short Access Tokens

```
5-15 minutes
```

---

## 2. Refresh Token Rotation

```
RT1 → RT2 → RT3
```

---

## 3. Reuse Detection

Detect old refresh token usage.

---

## 4. Device Binding

Store metadata:

```
device id
browser fingerprint
IP
location
```

If refresh request suddenly comes from another device:

```
suspicious
```

Require re-authentication.

---

## 5. Continuous Session Monitoring

Many identity providers monitor:

```
IP changes
country changes
device changes
impossible travel
```

---

# Senior Interview Answer

**Question:** What if an attacker uses RT1 before the legitimate user and before AT1 expires?

**Answer:**

The attacker can obtain a new access token and refresh token because RT1 is still valid. When the legitimate user later attempts to refresh using RT1, the server detects that RT1 has already been used. With refresh token rotation and reuse detection, the server should treat this as a token theft event, revoke the entire refresh-token family, and force re-authentication. The attacker may still use the newly issued access token until it expires, which is why access tokens should be short-lived (typically 5–15 minutes).