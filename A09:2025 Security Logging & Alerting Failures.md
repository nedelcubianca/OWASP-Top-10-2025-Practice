# A09:2025 Security Logging and Alerting Failures — Explained Simply

## What Is This Category About?
This is about **not knowing when you're being attacked.** Even if every other defense fails, **good logging and monitoring is what lets you notice an attack is happening (or already happened) and respond** — without it, attackers can operate undetected for a very long time.

Two connected ideas here:
- **Logging** = recording what happens in your system (logins, errors, transactions) so there's a record to look back on
- **Monitoring/Alerting** = actively watching those logs in real-time and **notifying someone** when something suspicious happens

## Signs You're Vulnerable (Simplified)

- Important events (logins, **failed** logins, big transactions) **aren't logged**, or only logged inconsistently (e.g., logging successful logins but not failed ones)
- Errors/warnings produce **unclear or missing** log messages
- Logs **can be tampered with** — no protection against someone editing/deleting them
- Logs exist but **nobody actually watches them** for suspicious activity
- Logs are **only stored on the local machine**, with no backup — if that machine is compromised, the evidence is gone too
- No proper **alert thresholds** or response process — even if something suspicious is logged, no one is notified in time
- Security scans/penetration tests don't trigger any alerts (meaning real attacks wouldn't either)
- The app can't detect or escalate an attack **while it's happening**
- **Sensitive data** (like personal health info) accidentally ends up **inside the logs themselves** — creating a new leak risk
- Log data isn't properly handled, so **attackers can inject malicious content into the logs themselves**
- Errors are silently swallowed/mishandled, so the system doesn't even realize something went wrong — nothing gets logged because the app never "noticed" the problem
- **Too many false alarms** — real threats get buried in noise and missed (the team watching logs becomes overwhelmed)
- Even when a real alert fires, there's **no clear response plan (playbook)** for what to do about it

## How to Prevent It (Simplified)

- Log all **security-relevant events** — logins, failed logins, access control failures, input validation failures — with enough detail (user, context) to investigate later
- Log both **successes and failures** of any security control, not just failures
- Store logs in a format that **log management tools** can easily read/analyze
- **Encode log data properly**, so attackers can't inject malicious content through the logging system itself
- Make logs **tamper-resistant** — e.g., using "append-only" storage, where entries can be added but never edited or deleted
- On errors, **fail safely** ("fail closed") — cancel/rollback the action rather than continuing in an unknown state
The Core Idea
When something goes wrong during an operation (an error occurs), your system has to decide: "Do I stop and block the action, or do I let it continue anyway?"

There are two opposite approaches to handling this:

Fail open = when an error happens, the system lets the action continue/succeed anyway, treating the error as "not a big deal"
Fail closed = when an error happens, the system stops everything and blocks/cancels the action, treating the error as "something went wrong, so don't proceed"

"Fail closed" is the safe choice, because it means: when in doubt, deny access / cancel the action, rather than risk letting something dangerous slip through.
- Set up **real alerting** for suspicious behavior, with clear guidance for developers on what "suspicious" looks like
- Security teams should build proper **monitoring use-cases and response playbooks**, so the team knows exactly what to do when an alert fires
- Use **"honeytokens"** — fake data/accounts that a real user would never touch. Since legitimate use never triggers them, **any activity involving them is almost certainly an attacker** — giving you a very reliable, low-false-positive alert
- Consider **behavioral analysis / AI tools** to help reduce false positives
- Have an actual **incident response plan** ready in advance (e.g., following NIST 800-61), and train developers to recognize what an attack looks like so they can report it
- Consider existing tools that help with this: e.g., **ModSecurity** (web application firewall rules) or the **ELK stack** (a popular open-source system for collecting, searching, and visualizing logs)

## Example Attack Scenarios (Explained)

**Scenario #1 — A 7-Year Undetected Breach:**
A children's health website had **no logging or monitoring in place**. An attacker accessed and modified sensitive health records for **3.5 million children** — and because there were no logs to detect it, the breach may have been happening **since 2013, undetected for over 7 years**, until an outside party discovered and reported it.

**Scenario #2 — Breach Discovered by a Third Party:**
An airline's passenger data (over a decade's worth, including passports and credit card info) was breached — but at a **third-party cloud hosting provider**, not the airline itself. The airline only found out because the **third party eventually notified them** — showing that logging/monitoring gaps can exist not just in your own systems, but in vendors you rely on too.

**Scenario #3 — Costly Regulatory Fine:**
A European airline had **400,000+ customer payment records** stolen due to security flaws in their payment system. Because this qualified as a reportable data breach under **GDPR** (EU privacy law), the airline was fined **£20 million** — showing that logging/monitoring failures don't just cause technical damage, they can lead to **massive financial and legal consequences** too.

