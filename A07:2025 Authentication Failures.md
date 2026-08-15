# Authentication Failures — Explained Simply

## What Is This Category About?
This is about weaknesses in **login systems** — flaws that let an attacker **pretend to be a legitimate user** without actually having valid credentials, or that let them steal/reuse someone else's login session.

## Signs an App Is Vulnerable (Simplified)

- **Credential stuffing** — attackers use huge lists of **username/password combos stolen from other data breaches** and try them on your login page (since many people reuse passwords across sites)
- **Password spraying / hybrid attacks** — attackers guess **slightly modified versions** of leaked passwords (e.g., `Winter2025` → `Winter2026`), based on common human habits
- **No protection against brute force** — the app doesn't block repeated automated login attempts
- **Weak/default passwords allowed** — e.g., `admin`/`admin`, or `Password1`
- **Letting users register with already-breached passwords**
- **Weak account recovery** — like "security questions" (mentioned before: not truly secret, so unreliable)
- **Storing passwords insecurely** — plain text or weak hashing (connects to the Cryptographic Failures topic)
- **Missing or weak Multi-Factor Authentication (MFA)** — MFA means requiring a **second proof of identity** beyond just a password (like a code sent to your phone)
- **Exposing session IDs** insecurely (e.g., in the URL, where they can leak easily)
- **Reusing the same session ID after login** — should generate a fresh one
- **Not properly ending sessions** on logout or after inactivity
- **Not verifying that a token (like a JWT) is being used for its intended purpose**

## How to Prevent It (Simplified)

- **Enforce MFA** wherever possible — this is the single strongest defense against stolen/guessed passwords
- **Encourage password managers** — helps users create genuinely strong, unique passwords
- **Never ship default credentials** (especially for admin accounts)
- **Check new passwords** against lists of the most common weak passwords, and against known **breached password databases** (e.g., haveibeenpwned.com)
- **Don't force regular password changes** — modern guidance (NIST) says this actually backfires, since people respond by picking weaker or predictable passwords. Only force a reset if you suspect an actual breach
- **Use identical error messages** for login/registration failures (e.g., always say "Invalid username or password" — never reveal whether the username specifically exists), to prevent attackers from **figuring out (enumerating) which accounts are real**
- **Limit/delay failed login attempts**, and log + alert on suspicious patterns — but carefully, so you don't accidentally lock out legitimate users (denial of service)
- **Use a secure, proven session management system** — generate a **new random session ID after login**, never put it in the URL, store it safely, and properly invalidate it on logout/timeout
- **Prefer using a well-established, pre-built authentication system** rather than building your own from scratch — reduces risk significantly
- **Validate token scope/purpose** — e.g., for JWTs, check the `aud` (intended audience) and `iss` (issuer) fields to make sure the token is actually meant for this use

## Example Attack Scenarios (Explained)

**Scenario #1 — Credential Stuffing / Password Spraying:**
Attackers use leaked username/password lists from other breaches, and also try **slightly adjusted variations** (like `ILoveMyDog6` → `ILoveMyDog7`), since people often just increment passwords over time. If the app doesn't detect or block this automated guessing, it becomes a **"password oracle"** — attackers can just keep testing combinations until one works.

**Scenario #2 — Relying on Passwords Alone:**
Most successful login attacks happen simply because **passwords are the only factor** required. Older "best practices" like forcing frequent password changes actually **backfire** — they push users toward reusing passwords or picking weaker ones just to remember them. The fix: **stop forcing rotation**, and **require MFA** instead.

**Scenario #3 — Sessions Not Properly Ending:**
A user logs into an app on a **public computer**, but instead of clicking "logout," they just close the browser tab and leave. If the session isn't properly invalidated, the **next person to use that computer is still logged in as them**. A related issue: with **Single Sign-On (SSO)** — one login gives access to multiple connected apps (email, chat, documents) — logging out of *one* app might not log you out of the *others*, leaving the account still accessible if someone else uses the same browser afterward.

# Is SSO Related to OAuth?

Yes, they're related — but they're **not the same thing**. Let me clarify the relationship clearly.

## SSO (Single Sign-On) = The *Goal* / *User Experience*
**SSO is a concept/outcome**: it means **logging in once gives you access to multiple different applications**, without having to log in separately to each one.

Example: at a company, you log into your work account **once** in the morning, and that single login also gives you access to your email, chat app, HR system, and document storage — you never have to type your password again for each one.

**SSO describes *what* happens (one login → many apps)**, not *how* it's technically achieved.

## OAuth = One of the *Technical Mechanisms* Used to Achieve SSO
**OAuth is a specific protocol** (a defined set of rules for how systems talk to each other) that can be **used as one of the building blocks to implement SSO** — but OAuth itself wasn't originally designed purely for "login." Remember from earlier: OAuth's core purpose is **authorization** — granting one application limited access to your data/resources on another service, using tokens instead of passwords.

## How They Connect
When companies build SSO systems, they typically use **OAuth combined with another protocol called OpenID Connect (OIDC)**, which is built **on top of** OAuth specifically to handle the "**prove who this person actually is**" part (authentication), since OAuth alone was mainly built for authorization, not identity verification.

**Simplified breakdown:**
| Concept | What it does |
|---|---|
| **SSO** | The overall *experience/goal*: log in once, access many apps |
| **OAuth** | A protocol for granting limited access/permissions using tokens (authorization) |
| **OpenID Connect (OIDC)** | Built on top of OAuth, specifically adds "prove identity" (authentication) — this is what actually makes true SSO login possible |

So in practice: **modern SSO systems are very often built using OAuth + OpenID Connect together** — OIDC handles verifying *who you are*, and OAuth handles *what you're allowed to access* once you're identified. That's why you'll often see them mentioned together, and why it's easy to think of them as one thing.

## Connecting Back to Scenario #3 (SSO Logout Problem)
This also explains the **logout problem** mentioned in the last scenario: when you log into multiple connected apps through SSO, each app typically receives its **own separate session/token** (often tied to OAuth's token system). Logging out of *one* app might only clear *that app's* local session — it doesn't automatically tell the other connected apps "hey, this user logged out too." Properly fixing this requires a mechanism called **Single Logout (SLO)**, which explicitly propagates the logout event to *all* connected applications — but as the text noted, this isn't always implemented correctly, which is exactly why that vulnerability scenario exists.
