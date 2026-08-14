# A03:2025 Software Supply Chain Failures — Explained Simply

## What Is a "Software Supply Chain"?
When you build an application, you almost never write 100% of the code yourself. You use:
- **Other people's code** (called libraries or dependencies) to save time
- **Tools** to build, test, and deploy your app (like CI/CD pipelines)
- **Third-party services** your app connects to

All of these pieces — plus everyone and everything involved in creating, building, and delivering your software — make up your **"supply chain."** It's the same idea as a supply chain for a physical product (like a car made from parts built by many different suppliers), just applied to software.

## What Is a "Supply Chain Failure"?
It's when **something in this chain goes wrong or gets compromised** — not because of a bug in *your* code, but because of a problem in a piece someone else built, or in the tools/process used to build and deliver your software.

## Why This Is Dangerous
If you use a library that someone else wrote, and that library has a security flaw (or is secretly malicious), **your application inherits that problem too** — even though you never wrote that faulty code yourself. Attackers know this, so instead of attacking one company directly, they attack **one popular library or tool that thousands of companies use** — one successful attack compromises everyone downstream at once.

## Signs You're Vulnerable (Simplified)
- You don't know exactly **which versions** of external code/libraries you're using (including the "dependencies of your dependencies")
- You're using **outdated or unsupported** software (old OS, old libraries, old frameworks)
- You don't regularly **scan for known vulnerabilities** in the components you use
- You don't track/log **changes** made anywhere in your build process
- You haven't locked down **who can access** your code, build tools, and deployment systems
- **Anyone can push code straight to production alone**, with no second person reviewing it
- You use components from **untrusted/unofficial sources**
- You're slow to **patch/update** known vulnerabilities (e.g., only patching monthly instead of right away)
- Your **build/deployment pipeline (CI/CD)** is less protected than your actual live application

## How to Prevent It (Simplified)
- Keep a full, updated **inventory list of every component** your software uses (this list is called an **SBOM — Software Bill of Materials**) — including indirect/nested dependencies
- Remove anything you don't actually need (unused libraries, features, files)
- Use automated tools to **continuously check for known vulnerabilities** in your components, and subscribe to security alerts
- Only download components from **official, trusted sources**, preferably ones that are **digitally signed** (proof they weren't tampered with)
- Don't blindly auto-update everything — **choose versions deliberately**, and roll out updates gradually (not to every system at once) so one bad update doesn't break/compromise everything simultaneously
- Track and log **every change** made to your code repositories, build tools, and pipelines
- **Harden and protect** your code repository, developer computers, and build servers — require MFA (multi-factor authentication), limit who has access, don't allow secrets/passwords to be stored in code
- Make sure **no single person** can write code and push it live completely alone — require a review process

## Real Attack Examples (Simplified)

1. **SolarWinds (2019):** Attackers secretly compromised a trusted software vendor. When ~18,000 organizations installed a routine software update from that vendor, they were unknowingly installing malware too.

2. **Bybit theft (2025):** A crypto exchange lost $1.5 billion because attackers compromised wallet software — but the malicious code only activated under one very specific condition, making it hard to detect beforehand.

3. **Shai-Hulud npm worm (2025):** Attackers uploaded malicious versions of popular free code packages. Once installed, the malicious code **automatically stole data and spread itself to other packages** — no human attacker needed to do it manually. It infected over 500 packages before being stopped. This showed that **developers' own computers** are now direct targets.

4. **Log4Shell (2021):** A hugely popular logging library (used inside countless applications worldwide) had a critical flaw that let attackers **run any code they wanted** on affected servers — because so many apps used this one library, the impact was massive and global.
