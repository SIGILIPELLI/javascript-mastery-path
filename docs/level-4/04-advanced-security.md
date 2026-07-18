# 04 · Advanced Security Practices

Security at production scale isn't a checklist you run once — it's a set of
defaults you bake into architecture. This module covers authentication with
JWTs and the OWASP Top 10 risks as they specifically show up in JavaScript
applications.

## Authentication vs. authorization

**Authentication** answers "who are you?" (login). **Authorization** answers
"what are you allowed to do?" (permissions). Conflating the two is a common
source of vulnerabilities — a valid, authenticated token doesn't automatically
mean the request should be allowed.

```javascript
// authentication: verifying identity
function authenticate(req, res, next) {
  const token = req.headers.authorization?.replace("Bearer ", "");
  if (!token) return res.status(401).json({ error: "missing token" });

  try {
    req.user = verifyToken(token); // throws if invalid/expired
    next();
  } catch {
    res.status(401).json({ error: "invalid or expired token" });
  }
}

// authorization: checking permission for THIS specific action
function requireRole(role) {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({ error: "forbidden" });
    }
    next();
  };
}

app.delete("/users/:id", authenticate, requireRole("admin"), deleteUserHandler);
```

## JWTs: signing and verifying

A JSON Web Token (JWT) is a signed, self-contained token — the server can
verify it hasn't been tampered with without a database lookup, at the cost of
being hard to revoke early (mitigations below).

```bash
npm install jsonwebtoken
```

```javascript
// auth/tokens.js
import jwt from "jsonwebtoken";

const ACCESS_SECRET = process.env.JWT_ACCESS_SECRET; // never hardcode this
const ACCESS_TOKEN_TTL = "15m";                        // short-lived on purpose
const REFRESH_TOKEN_TTL = "7d";

export function signAccessToken(user) {
  return jwt.sign(
    { sub: user.id, role: user.role }, // payload — keep it small, no sensitive data
    ACCESS_SECRET,
    { expiresIn: ACCESS_TOKEN_TTL, issuer: "acme-api" }
  );
}

export function verifyAccessToken(token) {
  return jwt.verify(token, ACCESS_SECRET, { issuer: "acme-api" });
  // throws JsonWebTokenError (bad signature) or TokenExpiredError (expired)
}
```

```javascript
// login route using the helpers above
app.post("/auth/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await usersRepository.findByEmail(email);

  if (!user || !(await bcrypt.compare(password, user.passwordHash))) {
    // deliberately identical message for "no such user" and "wrong password" —
    // don't leak which one it was
    return res.status(401).json({ error: "invalid email or password" });
  }

  const accessToken = signAccessToken(user);
  res.json({ accessToken });
});
```

Short-lived access tokens (minutes, not days) limit the damage window if one
leaks. Pair them with a longer-lived **refresh token**, stored server-side (or
in an `httpOnly` cookie) so it can be revoked, to re-issue access tokens
without forcing re-login.

```javascript
// middleware — drop-in for any protected route
function authenticate(req, res, next) {
  const token = req.headers.authorization?.replace("Bearer ", "");
  if (!token) return res.status(401).json({ error: "missing token" });

  try {
    req.user = verifyAccessToken(token);
    next();
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      return res.status(401).json({ error: "token expired", code: "TOKEN_EXPIRED" });
    }
    return res.status(401).json({ error: "invalid token" });
  }
}
```

## Password storage

Never store plaintext passwords, and never roll your own hashing. Use
`bcrypt` (or `argon2`), which is deliberately slow and salts automatically.

```bash
npm install bcrypt
```

```javascript
import bcrypt from "bcrypt";

const SALT_ROUNDS = 12; // higher = slower = more resistant to brute force

async function hashPassword(plaintext) {
  return bcrypt.hash(plaintext, SALT_ROUNDS);
}

async function verifyPassword(plaintext, hash) {
  return bcrypt.compare(plaintext, hash);
}
```

## The OWASP Top 10, in JavaScript terms

| OWASP risk | How it shows up in a JS app | Mitigation |
|---|---|---|
| Broken access control | Missing `requireRole`/ownership checks on routes | Authorize every mutating route, check resource ownership server-side |
| Cryptographic failures | Plaintext passwords, hardcoded JWT secrets | `bcrypt`/`argon2`, secrets from env vars/secret manager |
| Injection | String-concatenated SQL, `eval()` on user input | Parameterized queries/ORM, never `eval`/`new Function` on untrusted input |
| Insecure design | No rate limiting on login, no idempotency on payments | Threat-model auth/payment flows before writing code |
| Security misconfiguration | Default Express headers, verbose error stacks in prod | `helmet`, disable stack traces in production responses |
| Vulnerable/outdated components | Unpatched npm dependencies | `npm audit`, Dependabot/Renovate, pin versions |
| Identification/authentication failures | Long-lived tokens, no lockout on brute force | Short-lived JWTs, refresh token rotation, rate-limit `/login` |
| Software/data integrity failures | Installing packages from untrusted sources, no lockfile | `package-lock.json` committed, verify package provenance |
| Logging/monitoring failures | No audit trail for sensitive actions | Structured logs for auth events — see [Module 9](09-observability.md) |
| Server-side request forgery (SSRF) | Fetching a user-supplied URL server-side unchecked | Allow-list destinations, block internal IP ranges |

## Common injection example, fixed

```javascript
// VULNERABLE — string concatenation lets an attacker inject SQL
const query = `SELECT * FROM users WHERE email = '${email}'`;

// SAFE — parameterized query; the driver escapes the value
const result = await db.query("SELECT * FROM users WHERE email = $1", [email]);
```

## Security headers and rate limiting

```bash
npm install helmet express-rate-limit
```

```javascript
import helmet from "helmet";
import rateLimit from "express-rate-limit";

app.use(helmet()); // sets sensible security headers (CSP, X-Frame-Options, etc.)

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10,                    // 10 attempts per IP per window
  message: { error: "too many login attempts, try again later" },
});

app.post("/auth/login", loginLimiter, loginHandler);
```

## Exercise

A code review flags this route:

```javascript
app.get("/api/orders/:id", authenticate, async (req, res) => {
  const order = await ordersRepository.findById(req.params.id);
  res.json(order);
});
```

Identify the OWASP Top 10 category this violates (hint: any authenticated
user can fetch any order by guessing/incrementing IDs), and rewrite the
handler to fix it. Then add rate limiting to a `/auth/login` route using
`express-rate-limit` with a stricter window than the example above.
