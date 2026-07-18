# 09 · Security Basics

Backend code is exposed to untrusted input from the internet, which makes
it a target. This module covers two of the most common web vulnerabilities
— XSS and CSRF — plus input sanitization and using `npm audit` to catch
known-vulnerable dependencies.

## Cross-Site Scripting (XSS)

XSS happens when untrusted input is rendered as HTML/JavaScript in a page,
letting an attacker run their own script in another user's browser (steal
cookies, act on their behalf, etc.).

```javascript
// VULNERABLE: directly inserting user input as HTML
function renderComment(comment) {
  document.getElementById("comments").innerHTML += `<p>${comment}</p>`;
}

renderComment('<img src=x onerror="alert(document.cookie)">');
// The browser executes the injected script — classic stored/reflected XSS
```

```javascript
// SAFE: use textContent (or a templating engine that escapes by default)
function renderCommentSafely(comment) {
  const p = document.createElement("p");
  p.textContent = comment; // treated as plain text, never parsed as HTML
  document.getElementById("comments").appendChild(p);
}
```

```javascript
// If you must build HTML strings, escape special characters manually
function escapeHtml(str) {
  return str
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

console.log(escapeHtml('<script>alert("hi")</script>'));
// &lt;script&gt;alert(&quot;hi&quot;)&lt;/script&gt;
```

Server-rendered templating engines (EJS, Pug, Handlebars) escape
interpolated values by default — the danger is almost always an explicit
"raw"/"unescaped" output helper (e.g. EJS's `<%- value %>` instead of
`<%= value %>`) used on untrusted input.

## Cross-Site Request Forgery (CSRF)

CSRF tricks a logged-in user's browser into submitting a request to your
site that they didn't intend — e.g. an `<img>` tag or hidden auto-submitting
form on an attacker's page pointed at your `POST /transfer-funds` endpoint,
riding on the victim's existing session cookie.

```html
<!-- On an attacker's unrelated website: -->
<form action="https://bank.example.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker-account" />
  <input type="hidden" name="amount" value="1000" />
</form>
<script>document.forms[0].submit();</script>
<!-- If the victim is logged into bank.example.com, their session cookie
     rides along automatically and the transfer looks legitimate. -->
```

```javascript
// Mitigation: a CSRF token the attacker's page can't know or forge
import crypto from "node:crypto";

function generateCsrfToken(session) {
  const token = crypto.randomBytes(32).toString("hex");
  session.csrfToken = token; // stored server-side, tied to this user's session
  return token;
}

function verifyCsrfToken(session, submittedToken) {
  return Boolean(session.csrfToken) && session.csrfToken === submittedToken;
}

// Middleware applied to state-changing routes (POST/PUT/DELETE)
function csrfProtection(req, res, next) {
  if (["POST", "PUT", "DELETE"].includes(req.method)) {
    const submitted = req.body._csrf ?? req.headers["x-csrf-token"];
    if (!verifyCsrfToken(req.session, submitted)) {
      return res.status(403).json({ error: "invalid or missing CSRF token" });
    }
  }
  next();
}
```

Other CSRF mitigations that pair well with tokens: setting cookies with
`SameSite=Lax` or `SameSite=Strict`, and checking the `Origin`/`Referer`
header on state-changing requests.

## Input sanitization and validation

Never trust data from the client — validate its shape/type, and sanitize
anything that will be rendered, logged, or used to build a query.

```javascript
function validateSignup(body) {
  const errors = [];

  if (typeof body.email !== "string" || !/^\S+@\S+\.\S+$/.test(body.email)) {
    errors.push("a valid email is required");
  }
  if (typeof body.password !== "string" || body.password.length < 8) {
    errors.push("password must be at least 8 characters");
  }
  if (typeof body.username !== "string" || !/^[a-zA-Z0-9_]{3,20}$/.test(body.username)) {
    errors.push("username must be 3-20 alphanumeric characters or underscores");
  }

  return errors;
}

console.log(validateSignup({ email: "not-an-email", password: "short", username: "ok_name" }));
// [ 'a valid email is required', 'password must be at least 8 characters' ]
```

```javascript
// Never build SQL from string concatenation — this is a SQL injection hole:
// db.exec(`SELECT * FROM users WHERE email = '${email}'`); // NEVER do this

// Instead, always use parameterized queries (covered in Module 4):
const stmt = db.prepare("SELECT * FROM users WHERE email = ?");
const user = stmt.get(email); // the driver handles escaping safely
```

A popular library for validating/sanitizing structured request bodies is
`zod` or `express-validator` — they centralize validation rules instead of
scattering manual `if` checks across every route.

```bash
npm install zod
```

```javascript
import { z } from "zod";

const SignupSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  username: z.string().regex(/^[a-zA-Z0-9_]{3,20}$/),
});

const result = SignupSchema.safeParse({ email: "ada@example.com", password: "supersecret", username: "ada99" });
console.log(result.success); // true

const bad = SignupSchema.safeParse({ email: "nope", password: "short", username: "a" });
console.log(bad.success); // false
console.log(bad.error.issues.map((i) => i.message));
// [ 'Invalid email', 'String must contain at least 8 character(s)', 'Invalid' ]
```

## npm audit — catching vulnerable dependencies

Every dependency you install can itself contain known vulnerabilities.
`npm audit` checks your installed packages against a public vulnerability
database.

```bash
npm audit
```

```text
# found 3 vulnerabilities (1 low, 1 moderate, 1 high)
#   high            Prototype Pollution in lodash
#   Package         lodash
#   Dependency of   my-app
#   Path            my-app > some-lib > lodash
#   More info       https://github.com/advisories/GHSA-xxxx-xxxx-xxxx
#
# run `npm audit fix` to fix 2 of 3 vulnerabilities
```

```bash
npm audit fix          # automatically upgrades to non-breaking safe versions
npm audit fix --force  # also allows breaking/major version bumps — review changes after!
npm audit --json       # machine-readable output, useful in CI pipelines
```

Run `npm audit` (or wire it into CI) regularly, not just once at project
start — new vulnerabilities are disclosed against existing package versions
all the time.

## Security cheat sheet

| Threat | Defense |
|--------|---------|
| XSS | escape/sanitize output, use `textContent`, rely on templating auto-escaping |
| CSRF | CSRF tokens, `SameSite` cookies, verify `Origin`/`Referer` |
| SQL injection | parameterized queries / prepared statements, never string-concatenate SQL |
| Malformed/malicious input | validate shape and type at the boundary (schema libraries like `zod`) |
| Vulnerable dependencies | `npm audit`, keep packages updated, minimize dependency count |

## Exercise

Add input validation to the `POST /notes` route from
[Module 3](03-rest-api-express.md) using either hand-written checks or
`zod`: `text` must be a non-empty string under 280 characters. Then write an
`escapeHtml` sanitizer and use it before storing/echoing back the note text
in a hypothetical HTML view, and run `npm audit` on a scratch project to see
what real output looks like on your machine.
