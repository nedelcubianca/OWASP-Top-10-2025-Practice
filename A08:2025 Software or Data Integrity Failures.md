# A08:2025 Software or Data Integrity Failures — Explained Simply

## What Is This Category About?
This is about **trusting code or data without actually verifying it's legitimate and hasn't been tampered with.** The core idea: your application ends up **treating something as safe/trusted just because it "looks" like it came from the right place** — without actually *checking*.

## Key Examples From the Description

- Your app uses **plugins, libraries, or code from outside sources** (repositories, CDNs) without verifying they're genuinely trustworthy
- Your **CI/CD pipeline** (the automated system that builds and deploys your code) pulls code from somewhere **without checking a signature** or similar proof of authenticity — so malicious code could sneak in during the build/deploy process itself
- Apps with **auto-update features** download and install updates **without properly verifying** they're legitimate — an attacker could potentially push a fake "update" that gets installed on everyone's system
- **Insecure deserialization** — this needs its own explanation below

### What Is "Deserialization," and Why Is It Dangerous?
**Serialization** = converting a program's internal data (an object) into a format that can be **sent over a network or stored** (like converting it into text or bytes).
**Deserialization** = the reverse — taking that stored/transmitted data and **converting it back into a usable object** in the program.

**The danger:** if this serialized data is sent to the **client's browser and back** (like in Scenario #4 below), an attacker can potentially **see and modify** that data before it's sent back to the server. If the server blindly **deserializes** (reconstructs) whatever it receives without checking it hasn't been tampered with, an attacker can craft **malicious serialized data** that, when reconstructed by the server, **executes attacker-controlled code** — essentially handing the attacker control of the server.

## How to Prevent It (Simplified)

- Use **digital signatures** (cryptographic proof of authenticity) to confirm software/data really came from the expected source and wasn't altered
- Only pull libraries/dependencies from **trusted, official repositories** — for higher-risk environments, consider hosting your own internal, vetted copy of trusted packages
- Require **code review** for all changes, to catch malicious code before it gets in
- **Lock down your CI/CD pipeline** — control who can access it and ensure it can't be tampered with
- **Never trust serialized data coming from users/clients** without verifying it wasn't tampered with (via integrity checks or signatures)

## Example Attack Scenarios (Explained)

### Scenario #1: Untrusted Third-Party Support Widget Steals Cookies
A company sets up their support provider's tool at `support.myCompany.com`, which is actually just pointing (via DNS) to the support provider's own external servers (`SupportProvider.com`). **The problem:** because this subdomain still looks like part of `myCompany.com` to the browser, **all of `myCompany.com`'s cookies (including login/authentication cookies) get automatically sent to it** — even though it's really controlled by a separate outside company. Anyone with access to that support provider's systems can now **steal those cookies** and hijack user sessions, because the company essentially extended trust to infrastructure it doesn't fully control.

### Scenario #2: Unsigned Firmware Updates
Many devices (home routers, set-top boxes) install **firmware updates without verifying a digital signature** first. This means an attacker could potentially create a **fake malicious "update"** and get affected devices to install it as if it were legitimate. This is especially bad because there's often **no way to fix an already-compromised device** except waiting for a future patched version — and many old devices never get updated at all.

### Scenario #3: Downloading a Package From an Untrusted Website
A developer can't find the version of a code package they need through the normal, trusted package manager (like npm), so they download it instead from some **random website**. Since it's not from the official source and has **no signature to verify it**, there's no way to confirm it's legitimate — and it turns out to contain **malicious code**. **Lesson:** convenience shortcuts around official, trusted sources are a major risk.

### Scenario #4: Insecure Deserialization Leading to Remote Code Execution
A team wants their code to be "immutable" (data that doesn't change once created — a programming best practice), so instead of managing user state properly on the server, they **serialize the user's data and pass it back and forth between the frontend and backend with every request.**

An attacker notices a specific pattern (`"rO0"` in the data) that reveals **this is a serialized Java object** — a well-known signature. Using a specialized tool built for finding this exact weakness, the attacker crafts **malicious serialized data**, sends it to the server, and because the server deserializes it **without verifying it's safe/unmodified**, the attacker achieves **remote code execution** — meaning they can now run arbitrary commands on the server. This is a severe outcome, all because user-controllable data was trusted and reconstructed without any integrity check.
