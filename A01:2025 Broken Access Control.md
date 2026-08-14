# Summary: Broken Access Control — Description, Prevention, and Attack Scenarios

## What Is "Access Control," and What Happens When It Fails?
**Access control** is the security mechanism that decides **what a logged-in (or anonymous) user is allowed to do** within an application — which pages they can see, which data they can read, and which actions they can perform. It's essentially the system's way of enforcing the rule: *"You may only do what you're specifically permitted to do — nothing more."*

When access control is **broken** (fails to work correctly), the consequences are severe because an attacker can step outside their intended boundaries. This typically results in one of three outcomes:
- **Unauthorized information disclosure** — seeing data they shouldn't see (e.g., another user's private information)
- **Unauthorized modification or destruction of data** — changing or deleting things they shouldn't be able to touch
- **Performing business functions outside their permission level** — e.g., a regular user performing an admin-only action

## Common Ways Access Control Gets Broken (Explained One by One)

### 1. Violation of "Least Privilege" / "Deny by Default"
The correct security mindset is: **by default, nobody has access to anything**, and specific access is only granted when explicitly needed for a particular role or capability ("deny by default"). A violation happens when this principle is flipped — meaning something that should be restricted to a specific role or user ends up being **accessible to everyone** because no explicit restriction was ever put in place.

### 2. Bypassing Access Control Checks
This is when an attacker **tricks the system into skipping or ignoring its own permission checks**. This can be done by:
- **Modifying the URL** (e.g., changing a parameter in the web address, or trying to directly guess/access a page URL that isn't linked anywhere — called "force browsing")
- **Tampering with the application's internal state** (manipulating data the app uses behind the scenes to make decisions)
- **Editing the HTML page itself** in the browser before submitting it
- **Using specialized attack tools** that intercept and modify API requests before they reach the server

### 3. Insecure Direct Object References (IDOR)
This happens when an application lets users access a specific record (like an account, invoice, or profile) **simply by knowing or guessing its unique ID number** — without checking whether *that specific user* is actually allowed to view or edit *that specific record*. For example, if changing a number in a web address from `account=123` to `account=124` lets you view someone else's account, that's an IDOR flaw.

### 4. APIs Missing Access Controls on Data-Changing Requests
Web APIs typically support different types of requests: **GET** (read data), **POST** (create new data), **PUT** (update existing data), and **DELETE** (remove data). A common flaw is when an API properly restricts who can *read* data (GET) but **forgets to apply the same restrictions to the requests that create, modify, or delete data** (POST/PUT/DELETE) — meaning an attacker could potentially create, change, or destroy data even if they can't normally view it.

### 5. Elevation of Privilege
This is when someone manages to **gain more permissions than they should have** — either by acting as a logged-in user without actually being authenticated at all, or by a regular logged-in user somehow gaining **admin-level powers** they were never supposed to have.

### 6. Metadata Manipulation
Modern web applications often use small pieces of data to track who you are and what you're allowed to do — such as:
- **JWTs (JSON Web Tokens)** — a compact, encoded token that proves a user's identity and permissions
- **Cookies** — small pieces of data stored in the browser to maintain login sessions
- **Hidden form fields** — invisible data fields on a web page that carry information the user isn't meant to see or edit directly

If an attacker can **tamper with or replay (re-use) these tokens/cookies/fields**, they may be able to trick the server into thinking they have higher privileges than they actually do. This category also includes **abusing JWT invalidation** — meaning exploiting weaknesses in how the system is supposed to "cancel" or "expire" a token, so that an old/revoked token can still be used.

### 7. CORS Misconfiguration
**CORS (Cross-Origin Resource Sharing)** is a browser security feature that controls **which external websites are allowed to make requests to your API**. Normally, a browser blocks a random third-party website from silently calling *your* application's API on a user's behalf. A **CORS misconfiguration** happens when this protection is set up too loosely, **allowing untrusted or unauthorized outside websites to make API calls** that should have been blocked — potentially letting a malicious website interact with your API using a victim's active login session.

### 8. Force Browsing
This means an attacker **manually guesses or tries URLs** for pages they shouldn't be able to reach — for example, trying to access an internal admin dashboard URL directly (without ever being given a link to it), hoping the page exists and isn't properly protected, even though they're not logged in (or not privileged enough).

## How to Prevent Broken Access Control

**The single most important principle:** access control must be enforced in **trusted server-side code** (the backend, code running on servers the organization controls) — or in secure serverless functions — **never solely in code that runs in the user's browser (client-side)**, because anything running in the browser can be inspected, modified, or bypassed by the attacker.

Specific recommended defenses:

- **Deny by default** — Except for genuinely public content, assume nothing is accessible unless explicitly permitted.
- **Centralize access control logic** — Build the permission-checking mechanism **once**, and reuse it consistently everywhere in the application, rather than re-implementing it differently in each feature (which creates inconsistencies attackers can exploit). This also means minimizing how much CORS access is granted.
- **Enforce record ownership** — The system's underlying data model should inherently check "does this specific record belong to this specific user?" rather than blindly allowing any logged-in user to create, read, update, or delete *any* record in the database.
- **Enforce business-specific limits at the model level** — Any special business rules (e.g., "a user can only submit 5 requests per day") should be built into the core application logic (the "domain model"), not left to be optionally checked elsewhere.
- **Disable directory listing and hide sensitive files** — Web servers should not reveal a raw file listing of their folders, and files like `.git` (version control metadata) or backup files should never be left accessible within the publicly reachable parts of the website, since they can leak sensitive internal information.
- **Log and alert on access control failures** — When someone is denied access, that event should be recorded, and administrators should be alerted especially when there are repeated failures (which could indicate an attack in progress).
- **Apply rate limiting** — Restrict how many requests a user or IP address can make to APIs/controllers in a given time period, to reduce the damage automated attack tools can do (since attackers often rely on making huge numbers of rapid requests to find flaws).
- **Properly manage session and token lifetimes:**
  - Traditional server-tracked ("stateful") session IDs should be **invalidated (cancelled) on the server** the moment a user logs out.
  - **JWTs are "stateless"** (the server doesn't track them individually — the token itself carries all the proof), so they should be given a **short lifespan** to limit how long a stolen token remains useful to an attacker.
  - For situations needing longer-lived access, use **refresh tokens** and follow established **OAuth** standards, which provide proper mechanisms to revoke access when needed.
- **Use trusted, well-established frameworks/toolkits** that offer simple, ready-made ("declarative") ways to define access control rules, rather than building custom permission logic from scratch (which is more error-prone).
- **Test access control explicitly** — Developers and QA teams should write dedicated **unit tests and integration tests** that specifically verify permission boundaries are enforced correctly (not just assume the code works).

## Example Attack Scenarios (Explained)

### Scenario #1: Manipulating a Parameter to Access Someone Else's Account
The code below takes a value directly from the user's request (`acct`) and uses it in a database query to fetch account information, **without verifying that the logged-in user actually owns that account**:
```
pstmt.setString(1, request.getParameter("acct"));
ResultSet results = pstmt.executeQuery();
```
Because the application trusts whatever value is passed in the URL, an attacker can simply **edit the `acct` parameter in the browser's address bar** to point to a different account number:
```
https://example.com/app/accountInfo?acct=notmyacct
```
If the server doesn't double-check that the requesting user is actually authorized to view *that specific* account, the attacker gains access to someone else's private account data. **This is a textbook example of an Insecure Direct Object Reference (IDOR).**

### Scenario #2: Force Browsing to an Admin Page
Imagine a regular application page and an admin-only page:
```
https://example.com/app/getappInfo
https://example.com/app/admin_getappInfo
```
The admin page is *supposed* to require special admin-level permissions. But if the application never actually verifies this on the server side, an attacker can simply **type or guess the admin URL directly** into their browser. Two possible flaws are illustrated here:
- If **even an unauthenticated (not logged in) person** can load either page, that's a serious flaw.
- If a logged-in but **non-admin user** can load the admin page, that's also a flaw — the server should have checked their role/permissions before serving that page.

### Scenario #3: Client-Side-Only Protection Is Not Real Protection
In this scenario, the developers made a common mistake: they implemented access control **only in the front-end JavaScript code** running in the user's browser. This means that if a regular user tries to navigate to the admin URL through the normal website interface, JavaScript blocks them and prevents the page from loading.

**However, this "protection" is an illusion** — because the JavaScript only runs *inside the browser*, an attacker can simply **bypass the browser entirely** and talk directly to the server using a command-line tool like `curl`:
```
$ curl https://example.com/app/admin_getappInfo
```
Since `curl` doesn't run any of the website's JavaScript, none of the front-end restrictions apply — the request goes straight to the server. **If the server itself never re-checks permissions and just blindly returns the admin data**, the attacker successfully accesses restricted information despite the front-end "block." This scenario perfectly illustrates the earlier prevention principle: **access control enforced only in the browser (client-side) is not real security — it must be enforced on the trusted server-side**, since anything running in the user's browser can always be circumvented by an attacker who skips the browser altogether.

# Explanations: OAuth, Stateful vs. Stateless, and CORS Misconfiguration (Revisited)

## What Is OAuth?
**OAuth** (Open Authorization) is an **industry-standard protocol (a set of agreed-upon rules) that allows one application to access certain resources on your behalf, on another service, without ever seeing your actual password.**

The classic real-world example: Imagine you're on a random website, and it offers a button that says **"Log in with Google"** or **"Connect your Google Calendar."** Instead of typing your Google password directly into that random website (which would be dangerous — that site would then possess your real Google password), OAuth works like this:

1. The website redirects you to **Google's own login page** (a domain you trust).
2. You log in directly with Google, and Google asks: *"This website wants permission to see your calendar. Allow it?"*
3. If you approve, Google gives the website **a special token** (not your password) — a temporary "access pass" that only allows that specific limited action (e.g., "read calendar events") for a limited time.
4. The website then uses that token to access just your calendar — nothing else, and it never touches your actual Google password.

**Why this matters for security:** OAuth solves the problem of granting **limited, revocable, specific access** to third-party apps without ever handing over your actual login credentials. If that token is ever compromised or you want to cut off access, you (or Google) can **revoke just that token**, without needing to change your actual password.

This connects back to what was mentioned earlier about **JWTs and refresh tokens**: OAuth defines standard ways to issue these access tokens, give them **short lifespans** (so a stolen token becomes useless quickly), and use **refresh tokens** (a separate, longer-lived token used only to request a *new* short-lived access token when the old one expires) — all while providing an official mechanism to **revoke** access if needed.

## Stateful vs. Stateless — What's the Difference?

This distinction is about **where the "proof that you're logged in" is stored and tracked.**

### Stateful Sessions
"Stateful" means the **server keeps track of ("remembers") who is logged in**, by storing session information in its own memory or database. Here's how it works:

1. You log in → the server creates a **session record** (stored server-side) containing your identity and login status.
2. The server gives your browser a small random ID (a **session ID**, usually stored in a cookie) — this ID itself contains no meaningful information; it's just a reference/pointer.
3. On every future request, your browser sends that session ID back, and the **server looks up its own internal records** to check: "Does this session ID correspond to a valid, currently-logged-in user?"

**Key characteristic:** the server has to actively **maintain state** (a memory of who's logged in) for every user, and — critically — **when you log out, the server can immediately delete/invalidate that session record**, instantly making the session ID useless even if someone stole it. This is why the earlier text said *"stateful session identifiers should be invalidated on the server after logout"* — it's straightforward to do, because the server was tracking it the whole time anyway.

### Stateless Sessions (e.g., JWTs)
"Stateless" means the **server does NOT keep any memory of who's logged in.** Instead, **all the necessary information is packed directly into the token itself** (like a JWT), and the server simply **verifies the token's validity mathematically** (using a cryptographic signature) each time, without needing to look anything up in a database.

1. You log in → the server generates a **JWT** that contains your identity/permissions data directly inside it, cryptographically signed so it can't be tampered with.
2. Your browser stores this JWT and sends it along with every request.
3. The server just **checks the token's signature** to confirm it's authentic and unexpired — it doesn't need to store or look up anything, because the token is "self-contained proof."

**The key problem this creates:** Because the server isn't tracking sessions, **there's no simple built-in way to "cancel" a JWT before its natural expiration.** If you log out, the JWT itself is still technically valid and functional until it naturally expires — the server has no record to delete, because it never kept one. If someone stole your JWT, they could keep using it right up until it expires, **even after you've "logged out."**

**This is exactly why the earlier text recommended:**
- Making **stateless JWTs short-lived** — so even if one is stolen or someone forgets to properly log out, the window during which it can be misused is small.
- Using **refresh tokens with OAuth standards** for longer sessions — the short-lived JWT expires quickly, but a separate, more carefully controlled refresh token (which *can* be revoked, since it's typically tracked server-side) is used to request a new JWT when needed. This gives you the convenience of long login sessions while still preserving a way to cut off access if necessary.

**In short:**
| | Stateful | Stateless (JWT) |
|---|---|---|
| Where is login info stored? | On the server | Inside the token itself |
| Can you instantly revoke access? | Yes — just delete the server record | Not directly — must wait for expiration (unless combined with refresh tokens) |
| Server workload | Must track every active session | No tracking needed — just verifies token signature |

## CORS Misconfiguration — Explained Again, From the Ground Up

### The Basic Problem CORS Is Trying to Solve
By default, web browsers enforce something called the **"Same-Origin Policy."** This is a fundamental browser security rule that says: **a script running on Website A is not allowed to make requests to Website B's API and read the response, unless Website B explicitly says it's okay.**

**Why does this rule exist at all?** Because of how browsers handle **cookies and login sessions**. Here's the critical part: when you're logged into a website (say, your online banking site), your browser stores a **cookie** that proves you're logged in. That cookie gets **automatically attached** to *any* request your browser sends to that banking site's domain — **regardless of which website or tab triggered that request.**

This means: if you have your banking site open (and logged in) in one tab, and you visit a **malicious website** in another tab, that malicious website's JavaScript could try to silently send a request in the background to `https://yourbank.com/transferMoney` — and your browser would **automatically attach your valid login cookie** to that request, because as far as the browser is concerned, it's just a request going to yourbank.com. Without any protection, the malicious site could pretend to be you and perform actions on your bank account **without you ever knowingly doing anything on that malicious site**.

**This is called a "Cross-Site" attack** — and it's precisely what the Same-Origin Policy (enforced by browsers by default) prevents: by default, JavaScript from `malicious-site.com` is **blocked** from reading responses from `yourbank.com`, even though the cookie still gets sent. The malicious site can *fire* the request, but the browser won't let its JavaScript *read the sensitive response* back — which blocks most practical attacks of this style, though other protections exist too (like CSRF tokens) for cases where "firing and forgetting" a request is still enough to do damage.

### So What Is CORS, and What's a "Misconfiguration"?
**CORS (Cross-Origin Resource Sharing)** is the *official, legitimate way* for a website to say: *"Actually, I trust these specific other websites — please allow them to make requests to my API and read the responses."* This is necessary and normal in modern web development — for example, if your company has `app.example.com` (the frontend) talking to `api.example.com` (the backend API), those are technically different "origins," so the API needs to explicitly whitelist the frontend domain via CORS headers to allow that legitimate communication to work.

**A CORS misconfiguration happens when this trust is granted too broadly or carelessly** — for example:
- Setting the API to allow requests from **literally any origin** (`Access-Control-Allow-Origin: *`) when it shouldn't, especially combined with allowing credentials/cookies to be sent
- Using overly permissive wildcard patterns that accidentally include domains the developers didn't intend to trust
- Reflecting whatever origin the request came from back as "allowed" without actually validating it against a real whitelist

**When this happens, you've essentially undone the Same-Origin Policy protection on purpose (by accident).** Now, **any random malicious website** can make JavaScript calls directly to your API, **and this time the browser *will* let that malicious site's JavaScript read the actual response data** — because your server explicitly told the browser "this origin is trusted." If the user is logged in (and their cookie/token gets sent along, especially if credentials are allowed in the CORS policy), the malicious site can now **silently make authenticated API calls as that user, and actually read back their private data** — combining the "firing a request with valid cookies attached" problem with the added danger of being able to **read the sensitive response** too.

**In short: CORS misconfiguration turns a browser security feature that's supposed to protect users from malicious cross-site requests into an open door that explicitly invites malicious sites in — letting them impersonate logged-in users and steal their data through your own API.**
