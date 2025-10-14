# JSON Web Tokens (JWTs)

_Your digital passport for the internet._

## What's a JWT? (The Passport Metaphor)

Imagine you're traveling internationally. Your passport contains:
- **Your photo and info** (who you are)
- **Visa stamps** (what you're allowed to do)
- **Issuing country seal** (who vouches for you)
- **Expiration date** (how long it's valid)

A JWT works the same way for web applications. When you log in, the auth server hands you a "digital passport" that proves who you are and what you're allowed to access — without needing to check back with the server every single time.

The magic? Just like border agents can verify your passport without calling your home country, any server can verify your JWT without asking the auth server "is this legit?"

## Anatomy of a JWT

A JWT looks like a long random string, but it's actually three parts separated by dots (`.`):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 1. Header (The Passport Type)
Think of this as "what kind of document is this?" — it tells you it's a JWT and how it's secured.
```json
{
  "alg": "HS256",    // The security seal type
  "typ": "JWT"       // Document type: "This is a JWT"
}
```

### 2. Payload (The Passport Pages)
This is where all your info lives — who you are, what you can do, when your passport expires.
```json
{
  "sub": "1234567890",       // Your unique ID
  "name": "John Doe",        // Your name
  "iat": 1516239022          // When this was issued
}
```

### 3. Signature (The Official Seal)
Like the hologram on your passport, this proves the document is authentic and hasn't been tampered with. Only the issuing authority (the auth server) has the special stamp to create this seal.

---

## Understanding Claims (The Passport Details)

Claims are just the pieces of information written in your JWT passport. Think of them like different fields on your actual passport — some are standard (like your name and photo), others are custom (like visa stamps for specific countries).

### Standard Claims (The Usual Passport Fields)
These are the internationally recognized fields that most JWTs include. They're optional, but using them makes your JWTs play nicely with other systems.

| Claim | Name | Passport Equivalent | Example |
|-------|------|---------------------|---------|
| `iss` | Issuer | Issuing country | `"https://auth.example.com"` |
| `sub` | Subject | Passport holder ID | `"user123"` or `"alex@example.com"` |
| `aud` | Audience | Valid for entry to... | `"https://api.example.com"` |
| `exp` | Expiration | Expiration date | `1735689600` (Jan 1, 2025) |
| `nbf` | Not Before | Valid from date | `1735603200` |
| `iat` | Issued At | Issue date | `1735603200` |
| `jti` | JWT ID | Passport number | `"abc123xyz"` |

### Public Claims (Registered Add-ons)
Think of these like standardized visa stamps that multiple countries recognize. You register them publicly so everyone knows what they mean and there's no confusion.
```json
{
  "email": "alex@example.com",
  "roles": ["admin", "editor"],
  "https://myapp.com/claims/department": "Engineering"
}
```

### Private Claims (Your Custom Stamps)
These are the special markings you and your team agree on. Like internal company badges or special clearances — they only matter to the systems that know about them.
```json
{
  "userId": "12345",
  "plan": "premium",
  "features": ["api-access", "advanced-analytics"]
}
```

---

## The Issuer (`iss`) — Who Stamped Your Passport?

The `iss` (issuer) claim is like looking at which country issued your passport. It tells you **who vouches for this person**.

### Why This Matters

Back to our passport analogy: you wouldn't accept a passport from a random person claiming they're a country — you only trust passports from real governments. Same with JWTs.

1. **Trust Verification**: You only accept "passports" from authorities you trust (like Google, Auth0, or your own auth server)
2. **Multi-Provider Support**: Your app might accept passports from multiple countries (Google login, Microsoft login, your own login)
3. **Security**: Prevents someone from using a valid JWT meant for a different service at your service — like trying to enter France with a US military ID

### Real-World Example

```json
{
  "iss": "https://accounts.google.com",
  "sub": "110169484474386276334",
  "aud": "your-app-client-id.apps.googleusercontent.com",
  "exp": 1735689600,
  "email": "alex@example.com",
  "email_verified": true
}
```

In this example:
- `iss`: Google issued this token
- `sub`: Google's unique ID for this user
- `aud`: This token is for YOUR app specifically
- Your app validates the issuer matches `accounts.google.com` before trusting it

### How Passport Verification Works (In JWT Land)

```
1. You log in to a website
2. The auth server hands you a JWT "passport" with iss="https://auth.yourcompany.com"
3. You show this passport to the API when requesting data
4. The API acts like a border agent and checks:
   ✓ Is this from a trusted issuing authority?
   ✓ Is the security seal (signature) legitimate?
   ✓ Has it expired?
5. If all checks pass → Access granted!
```

It's like showing your passport at the airport. The agent doesn't need to call your home country — they can verify it right there because they have a list of trusted countries and know how to check the security features.

---

## Common Patterns

### Authentication Token
```json
{
  "iss": "https://auth.myapp.com",
  "sub": "user-456",
  "aud": "https://api.myapp.com",
  "exp": 1735689600,
  "iat": 1735603200,
  "email": "alex@example.com",
  "roles": ["user", "admin"]
}
```

### Microservices Authorization
```json
{
  "iss": "https://auth.myapp.com",
  "sub": "service-payment-api",
  "aud": "https://user-service.internal",
  "exp": 1735603500,
  "scope": "read:users write:orders"
}
```

### Single Sign-On (SSO)
```json
{
  "iss": "https://okta.example.com",
  "sub": "00u1abc2def3ghi4jkl",
  "aud": "multiple-apps",
  "exp": 1735689600,
  "groups": ["Engineering", "Admin"],
  "email": "alex@example.com"
}
```

---

## Security Best Practices (Passport Safety 101)

**Always Check the Passport Thoroughly**
- Verify the security seal (signature)
- Check if it's expired (`exp`)
- Make sure it's from a trusted issuing country (`iss`)
- Confirm it's actually meant for your country/service (`aud`)

**Keep Your Stamp Machine Safe**
- Use strong signing keys (think: sophisticated holograms, not a rubber stamp)
- Change your security features regularly
- Never leave your passport-printing tools where anyone can access them

**Don't Write Secrets in Your Passport**
- Remember: anyone can open and read a passport (JWTs are just encoded, not encrypted)
- Never put passwords, credit card numbers, or sensitive personal data in there
- Use it to say "User #12345 is an admin" not "User #12345's password is hunter2"

**Issue Short-Lived Passports**
- Access tokens: 15 minutes to 1 hour (like a day pass)
- For longer access, use refresh tokens (like renewing your passport)

---

## Quick Reference

### Reading a JWT (JavaScript)
Want to peek inside a JWT? You can decode it (but remember, decoding ≠ verifying!):

```javascript
const [header, payload, signature] = token.split('.');
const decoded = JSON.parse(atob(payload));
console.log(decoded);
// ⚠️  This only READS it, doesn't verify it's legitimate!
// Like looking at a passport photo - you're not checking if it's real
```

### Useful Tools
- [jwt.io](https://jwt.io) - Decode and verify JWTs in your browser (great for debugging)
- [jwt-cli](https://github.com/mike-engel/jwt-cli) - Command-line JWT decoder

---

## Resources

- [RFC 7519 - JWT Specification](https://tools.ietf.org/html/rfc7519)
- [jwt.io Introduction](https://jwt.io/introduction)
- [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)

---

_Understanding the tokens that power modern authentication._
